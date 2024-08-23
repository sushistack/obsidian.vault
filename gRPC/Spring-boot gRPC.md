

### 정의
- 구글에서 만든 Remote Procedure Call 프로토콜
- http2 통신에 protobuf 사용하는 통신
- code generate -> 
    - protoc : 프로토콜 버퍼 컴파일러 / 플러그인으로 지원 됨
    - 각 언어별 컴파일 및 코드 생성

### gRPC 구조
- protobuf 
    - 데이터 직렬화 방식
    - 특징
        - 언어 중립
        - 플랫폼 확장성
    -  key-value 형태로 직력화 하지 않고, index - value 형태로 직렬화 하여 metadata 최소화 함
    - .proto 파일로 정의서를 공유해야될 필요가 있음

- 컴파일러 플러그인 지원
    - .proto 파일을 기반으로 다양한 build 도구들로 다양한 언어로 Code 를 자동 생성해 준다.

- http2 사용
    - keepalive 적극 사용
    - 강력한 멀티 플렉싱 기능
        - http1.1  : 
            - socket(port, fd) connection - 동시에 1개 요청만 처리 가능
            - 파이프라이닝이 정의되었지만 반드시 순서를 지켜서 처리되는 이슈가 남아 있음
        - http2 : 
            - binary framing layer
                - binary header 처리
                - stream 처리
            - 단일 socket(port, fd)  connection 에서 동시에 양방향 통신 가능
                - stream_id 로 요청구분
            - 각 stream 은 독립적으로 동작 -> 양방향 동시 통신 가능

![](Pasted%20image%2020240823110712.png)

![](Pasted%20image%2020240823110723.png)


  * HOL Blocking
      * http1.1 은 반드시 요청 순서대로 응답을 처리해야 함.
          * 나중에 한 요청은 응답이 먼저 도착한다고 먼저 처리되지 않음.
      * http1.1 keep-alive
        * HTTP Head-of-Line / TCP Head-of-Line 모두 발생
    * http1.1 : pipelinning (현재 거의 사용하지 않음.)
        * HTTP Head-of-Line / TCP Head-of-Line 모두 발생
    * http2 : stream
        * TCP Head-of-Line 만 발생

# Spring-Boot gRPC

- Spring 정식 지원 버전이 아님
    - https://github.com/grpc-ecosystem/grpc-spring
    - 그러나 개인이 관리하지는 않고 제대로 된 오픈소스로 관리되는 듯 함.


# 차이점은?

- protobuf, protoc 는 편리함을 주는 부분
- 성능 차이는 HTTP 1.1  / HTTP 2.0
    - 패킷 byte size 감소
    - 단일 연결 / keep-alive / 멀티 플렉싱 에 의한 차이



