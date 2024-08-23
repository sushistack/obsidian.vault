# 테스트 방법

* 테스트 대상
    * spring-boot 3.3.2
    * netty-grpc 서버
    * r2dbc
        * pool size : 20 (core \* 5)
* 비교군
    * spring-boot 2.7.3
    * tomcat mvc 서버
    * hikari
        * pool size : 20 (fixed size)

* 각 서버는 2대씩 부하를 주면 2대가 나눠서 받음.
* client 정보
    * coroutine / WebClient 등 논블로킹 방식을 활용하여 호출
        * client 에서는 Thread 를 많이 사용하지 않음.

***

* connection pool size 의 경우 더 크게 설정할 수 있지만 알파 DB에 너무 과도한 부하를 주기 힘들어 각각 20 정도로 테스트
    * 대부분의 병목 현상이 DB에서 발생
    * http1.1 / grpc (http2) 통신의 특성만 비교하기 위해 같은 pool size 를 설정
* 쿼리 대상
    * getMemberIdByMemberNo (간단한 쿼리)
    * 파라미터 고정 (즉, 동일한 쿼리를 여러번 호출)

***

* 부하 방법
    * 환경 m1 맥북
    * [spring-boot grpc client](obsidian://open?vault=obsidian.vault&file=gRPC%2FSpring-boot%20gRPC%20client) 가이드에 따라 개발한 후 테스트
        * 10ms 간격으로 10/30/50/100 까지 늘리면서 1초간 100회 반복 부하를 발생시켰을 때 최종 경과 시간
    * 외부 제공 툴 설치하여 테스트 가능하지만 직접 만든 client 로 테스트
        * grpc 테스트 : [https://ghz.sh/docs/install](https://ghz.sh/docs/install)
        * http restful 테스트 : [https://github.com/rakyll/hey](https://github.com/rakyll/hey)
    * 테스트 전 서버에 2\~3회 웜업을 시켜준 후 테스트

## 용어 정리

* 아래 테스트에서
    * gRPC : http2 를 사용하는 통신
    * HTTP : http1.1 을 사용하는 통신

# 테스트 결과

## grpc 와 http1.1 , http1.1(pooling) 비교

|  | 10건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 30건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 50건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 100건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 총평 |
| --- | -------------- | -------------- | -------------- | --------------- | --- |
| gRPC | total count: 1000<br>total : 897 ms<br>slowest : 239 ms<br>fastest : 12 ms<br>average : 48.069 ms | total count: 3000<br>total : 2,099 ms<br>slowest : 576 ms<br>fastest : 16 ms<br>average : 153 ms | total count: 5000<br>total : 12,406 ms<br>slowest : 2993 ms<br>fastest : 17 ms<br>average : 330.3108 ms | total count: 10000<br>total : 13,590 ms<br>slowest : 4,645 ms<br>fastest : 14 ms<br>average : 883.579 ms | 저 부하 수준일 때 grpc 속도가 빠르지만 고 부하일때 동일하게 느림 |
| HTTP<br>(client : WebClient | total count: 1000<br>total : 1676 ms<br>slowest : 368 ms<br>fastest : 19 ms<br>average : 102.41 ms | total count: 3000<br>total : 3852 ms<br>slowest : 638 ms<br>fastest : 20 ms<br>average : 204.67 ms | total count: 5000<br>total : 5933 ms<br>slowest : 721 ms<br>fastest : 22 ms<br>average : 331.508 ms | total count: 10000<br>total : 12,057 ms<br>slowest : 1,410 ms<br>fastest : 37 ms<br>average : 613.78 ms |  |
| HTTP + Pooling (500)<br>(client : WebClient | total count: 1000<br>total : 1680 ms<br>slowest : 377 ms<br>fastest : 23 ms<br>average : 104.271 ms | total count: 3000<br>total : 4001 ms<br>slowest : 602 ms<br>fastest : 19 ms<br>average : 206.82 ms | total count: 5000<br>total : 6546 ms<br>slowest : 850 ms<br>fastest : 33 ms<br>average : 338.478 ms | total count: 10000<br>total : 11,103 ms<br>slowest : 1,488 ms<br>fastest : 43 ms<br>average : 576.78 ms | WebClient Pooling 사용으로 이점이 없음. |

* 발생 오류

```
- System error::executeMany; SQL [SELECT member.member_id FROM member WHERE member.member_no = ? LIMIT 1]; Got packets out of order
- System error::executeMany; SQL [SELECT member.member_id FROM member WHERE member.member_no = ? LIMIT 1]; Connection unexpectedly closed

- NOT_FOUND: Not found::param: value: "4101001000018016"
```

* DB Connection 관련 오류 발생
* NOT\_FOUND : 차라리 SQL 이나 오류가 발생하는게 더 낫지 않지?

# 문제점

* 알파 DB가 공용 DB여서 업무시간 사용 시 부하를 받아주질 못함 (테스트 환경 자체가 문제가 있음)
* gRPC 통신이 고 부하 상태에서 생각보다 너무 성능이 나오지 않음
* gRPC 통신 자체의 문제라기 보다 논블로킹 서버여서 DB연결에 사용된 r2dbc 이슈로 의심

# hikariCP 로 변경하여 다시 테스트

## r2dbc 를 hikariCP 로 변경하였을 때 성능 차이

* 비교군
    * spring-boot 2.7.3
    * tomcat mvc 서버
    * hikari
        * pool size : 20 (fixed size)
* 비교군
    * spring-boot 3.3.2
    * netty-grpc 서버
    * r2dbc
        * pool size : 13
* 테스트 대상
    * spring-boot 3.3.2
    * netty-grpc서버
    * hikari
        * pool size : 13
* client 정보
    * coroutine / WebClient 등 논블로킹 방식을 활용하여 호출
        * client 에서는 Thread 를 많이 사용하지 않음.
* 1초에 걸쳐서 10ms 간격으로 나눠서 호출
* 알파는 공용 DB여서 아침 일찍(오전 7시 \~ 오전 8시 사이) 테스트 진행



|  | 10건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 30건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 50건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 100건 / 10ms<br><span style="color:rgb(255, 255, 255);">1초동안</span> | 총평 |
| --- | -------------- | -------------- | -------------- | --------------- | --- |
| HTTP<br>(client : WebClient) | total count: **1000**<br>total : 2292 ms<br>slowest : 2056 ms<br>fastest : 14 ms<br>average : 527 ms | total count: **3000**<br>total : 3852 ms<br>slowest : 3088 ms<br>fastest : 200 ms<br>average : 1213.5 ms | total count: **5000**<br>total : 6282 ms<br>slowest : 5022 ms<br>fastest : 303 ms<br>average : 2320.68 ms | total count: **10000**<br>total : 11582 ms<br>slowest : 10155 ms<br>fastest : 544 ms<br>average : 4926 ms | db관련된 오류 발생 하지 않음 |
| gRPC + r2dbc<br>(server DB pool size : 20) | total count: 1000<br>total : 1237 ms<br>slowest : 247 ms<br>fastest : 4 ms<br>average : 31 ms | total count: 3000<br>total : 1955 ms<br>slowest : 1750 ms<br>fastest : 6 ms<br>average : 93 ms | total count: 5000<br>total : 3237 ms<br>slowest : 2141 ms<br>fastest : 27 ms<br>average : 661 ms | total count: 10000<br>total : 2350 ms<br>slowest : 2157 ms<br>fastest : 44 ms<br>average : 608 ms | 알파 DB 테스트 환경 이슈 회피 이후<br> 고부하에서도 준수한 성능  |
| gRPC + hikariCP<br>(server DB pool size : 13) | total count: 1000<br>total : 1264 ms<br>slowest : 254 ms<br>fastest : 5 ms<br>average : 33.844 ms | total count: 3000<br>total : 1259 ms<br>slowest : 405 ms<br>fastest : 6 ms<br>average : 88.52 ms | total count: 5000<br>total : 1474 ms<br>slowest : 458 ms<br>fastest : 6 ms<br>average : 194.0706 ms | total count: 10000<br>total : 2443 ms<br>slowest : 1577 ms<br>fastest : 84 ms<br>average : 894 ms | blocking 방식인 hikariCP 를 사용해도 <br>서버가 block 되지 않고 유사한 성능을 보여줌. |

### HTTP2.0 이어서 빠른게 아니라 네티 서버여서 단순히 대역폭이 큰게 아닐까?


* 테스트 대상
    * spring-boot 3.3.2
    * netty-grpc 서버
    * r2dbc
        * pool size : 13
* 비교군
    * spring-boot 3.3.2
    * netty webflux 서버
    * hikari
        * pool size : 13
* client 정보
    * coroutine / WebClient 등 논블로킹 방식을 활용하여 호출
        * client 에서는 Thread 를 많이 사용하지 않음.

<br>


|  | 10건 / 10ms<br>1초동안 | 30건 / 10ms<br>1초동안 | 50건 / 10ms<br>1초동안 | 100건 / 10ms<br>1초동안 | 총평 |
| --- | -------------- | -------------- | -------------- | --------------- | --- |
| HTTP 1.1<br>netty + hikariCP<br>(client : WebClient<br>connection pool : 500)<br>(server DB pool size : 13) | total count: **1000**<br>total : 1304 ms<br>slowest : 433 ms<br>fastest : 9 ms<br>average : 71 ms | total count: **3000**<br>total : 1759 ms<br>slowest : 1155 ms<br>fastest : 13 ms<br>average : 335 ms | total count: **5000**<br>total : 2497 ms<br>slowest : 1317 ms<br>fastest : 139 ms<br>average : 654 ms | total count: **10000**<br>total : 6638 ms<br>slowest : 5026 ms<br>fastest : 465 ms<br>average : 2352 ms | gRPC 에 비해 느림<br>그러나 큰 차이가 발생하지 않음 |
| gRPC (HTTP 2.0)<br>netty + hikariCP<br>(server DB pool size : 13) | total count: 1000<br>total : 1237 ms<br>slowest : 247 ms<br>fastest : 4 ms<br>average : 31 ms | total count: 3000<br>total : 1287 ms<br>slowest : 337 ms<br>fastest : 6 ms<br>average : 57 ms | total count: 5000<br>total : 1350 ms<br>slowest : 547 ms<br>fastest : 8 ms<br>average : 158 ms | total count: 10000<br>total : 2104 ms<br>slowest : 1364 ms<br>fastest : 6 ms<br>average : 724 ms | DB Connection 관리가 좀 더<br>안정적으로 느껴짐. |

<br>
<br>
# 최종 분석 및 결론

* netty / gRPC 를 사용하면 성능이 향상된다.
    * 속도가 빨라진다기 보다 자원을 효율적으로 사용하여 서버의 요청 수용 용량(대역폭?)이 늘어난다.
    * gRPC 가 가장 빠르긴 하다.
    * 다만 http1.1 -> http2 에 의한 성능 향상 보다 tomcat -> netty 에 의한 성능향상이 더 크다.

* netty 에서 불안하고 불편한 r2dbc 대신 익숙한 hikariCP + JPA 를 사용할 수 있다.
    * 즉, netty (non-blocking) + hikariCP (blocking) 조합으로도 준수한 성능을 보여 줌.
    * reactor-project 를 이용하는것도 가능하지만 kotlin + Coroutine 을 사용할 경우 허들이 좀 더 낮을 듯 함.
    * 단, transaction 이 복잡하거나 느린 쿼리가 있을 경우 netty 를 사용하더라도 큰 성능 향상이 발생하지 않을 수 있음.

# 그럼에도 gRPC?
* spring-boot gRPC 를 이용하면 간단하고 쉽게 준수한 성능을 제공하는 Server / Client 를 만들 수 있다.
    * .proto 파일 추가, properties 설정 -> Stub 자동 생성
    * stub 이용하여 API 호출 후 응답 Status 별 처리 로직
    * .proto 파일을 이용하여 간편하게 가이드를 전달할수도 있다.


* HTTP 1.1 과 HTTP 2.0 을 비교해 봤을 때 HTTP 2.0 가 **네트워크 리소스**를 덜 사용 함.
        * http2.0 의 경우 keep-alive 를 적극 활용하므로 port / file opens, tw 소캣 등이 덜 발생함
        * 압축된 헤더 처리로 네트워크 전송 byte 양 자체가 줄어들 수 있음.
    * 둘 다 keep-alive / 멀티플랙싱을 사용할 수 있음
        * 그러나 Head-of-Line Blocking 현상에 대해 http2 가 좀 더 수월 함.