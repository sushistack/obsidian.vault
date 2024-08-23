# 개요

* 얼핏 생각하기에 논 블로킹인 netty에서는 블로킹 io API 를 사용하는 hikariCP 를 사용하면 제대로된 netty event loop thread 가 블럭이 되어서 정상적인 요청을 받을 수 없는 상태가 됨.
* 그래서 보통 netty 서버에서 DB 접속 시 논블로킹 API(netty) 를 사용하는 r2dbc를 사용
* r2dbc 사용 시 JPA 를 사용하지 못하는 등의 불편함이 존재 함.
* r2dbc 를 그대로 활용할 수 있지만 익숙하지 않고 문제 발생시 원인 파악이 어려운 점이 있어 성능은 덜나올 수 있더라도 좀 더 안정적인 hikariCP 서버 운영을 할 수 있는 방법 공유

# Netty

![](Pasted%20image%2020240823112904.png)
* io 요청을 받으면 channel 을 만들고 event loop 에서 non-blocking 형식으로 io 를 처리해 준다.

# Netty 서버에서 r2dbc

![](Pasted%20image%2020240823112920.png)

* HTTP io 요청과 DB 접속용 io 요청 이벤트가 모두 non-blocking 으로 처리되고 있어서 각각 독립적으로 동작할 수 있음

# Netty 서버에서 hikariCP

![](Pasted%20image%2020240823112940.png)

* hikariCP 는 Thread Pool 을 따로 관리하지 않고 connection Pool 만 관리한다.
* 이때 각 Connector 에서 사용되는 io API 는 blocking 된다.
* 해당 Channel 이 Event Loop 를 점유한 상태로 Thread 가 Blocking 되어버리게 된다.
* 메인 EventLoop 가 모두 blocking 되어버리면 서버 Netty 는 더이상 동작할 수 없게 된다.

# Netty 서버에서 hikariCP 를 사용하려면...

* Coroutine 이나 Reactor 에는 별도 Thread Pool 에 위임하여 blocking 로직을 처리할 수 있게 하는 방법들이 존재 함.
* DB Blocking io 가 발생하는 Thread 를 HTTP 요청을 처리하는 Netty main Thread 와 분리하여 Netty main Thread 가 블럭되지 않도록 처리

## reactor Project

* publishOn

```kotlin
fun getMemberIdByMemberNo(memberNo: String): Mono<String> {
    return Mono.just(memberNo) //main Thread
        .publishOn(Schedulers.boundedElastic())
        .mapNotNull { //여기서 부터 boundedElastic Thread 에서 처리
            memberEntityRepository.findMemberIdByMemberNo(memberNo)
                ?.getMemberId()
        }
}
```

## coroutine

* withContext

```kotlin
val fasterDispatcher = CoroutineDispatcherProvider.fasterQueryForDB() //적절한 크기의 Thread Pool 을 갖는 Dispatcher 생성
suspend fun getMemberIdByMemberNo(memberNo: String): String? = withContext(fasterDispatcher) {
    memberEntityRepository.findMemberIdByMemberNo(memberNo)
        ?.getMemberId()
}
```

## 그림으로 보자면..

![](Pasted%20image%2020240823112953.png)

* hikariCP blocking 로직과 HTTP 요청 처리 Netty Thread 가 분리되어 서로에게 영향을 주지 않음

## 성능 체크

* [spring-boot grpc server 성능 테스트](obsidian://open?vault=obsidian.vault&file=gRPC%2FSpring-boot%20gRPC%20server%20%EC%84%B1%EB%8A%A5%20%ED%85%8C%EC%8A%A4%ED%8A%B8)
    * r2dbc 와 hikariCP 로 변경했을 경우 성능이 거의 같은것을 볼 수 있음
* 제일 앞단에 Http 요청 handshake 만 병렬로 처리해 줘도 충분한 성능 향상이 발생.
    * DB Connection 처리 부분은 동일한 병목 현상
* r2dbc 는 논블로킹 요청이어서 이론적으로 Connection 크기를 아주 크게 만들 수 있지만 한계가 있음
    1. connection size 를 무한정 늘릴 수 없음
        * 논블로킹 특성 상 connection size 를 Thread 개수보다 무한정 늘릴 수 있음
        * 다만, MySQL의 경우 단일 DB 서버가 연결을 받을 수 있는 최대 개수가 제한 하고있어 여러 서버에서 하나의 DB에 접속할 경우를 고려해야 함.
        * cpu core
            * core 성능과 개수에 따라 I/O를 처리하는 최대 성능이 제한 됨.
                * [https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)
            * core 개수가 늘어나면 connection 크기를 늘릴 수 있겠지만 hikariCP 를 사용해도 가능한 부분
        * 네트워크 대역폭
            * 모든 connection 요청을 동시에 처리할 수 있을 정도의 네트워크 대역폭이 필요하므로 고려사항이 될 수 있음
        * 메모리
            * 연결이 많아질수록 더 많은 메모리를 사용하게 됨.

## 주의사항

* HikariCP 는 Connection Pool 만 관리해 주므로 Thread Pool 을 별도로 잘 관리해 줘야 함.
* DB connection size 를 불필요하게 늘릴 필요가 없으므로 과도하게 많은 Thread Pool 이 필요하지는 않음.
* 적절한 크기의 DB Connection size 와 그 크기와 같거나 조금 더 적은 Thread Pool 크기가 필요
    * 복잡한 Transaction 이 많을 경우 Connection 할당 경쟁이 발생하면서 Dead Lock 에 빠질 수 있음.
* slow query 가 있다면 별도의 Thread Pool/Connection Pool로 처리하도록 해야 함.
    * gRPC 서버의 경우 slow 쿼리를 별도 서버로 분리하는게 좀 더 안전한 관리가 되지 않을지..
* spring-boot 낮은 버전 사용 시 Transaction 관리에 ThreadLocal 이 관여하고있어서 문제가 될 수 있음
    * Spring 5.2 버전 이상에서 Coroutine 사용시 문제는 해결 됨
        * CoroutineContext 를 사용하도록 변경 됨
    * webflux 에 Reactive 용 Transaction 관리가 별도로 있음.

# 결론

* http handshake 오버헤드가 커서 이 부분만 병렬 처리가능하게 해주면(netty를 사용해주면) 서버 성능이 많이 증가 함.
* r2dbc 도 충분히 사용할만한 라이브러리 이지만 기능이 제한적임
* netty 의 성능과 JPA 의 기능을 모두 활용할 수 있는 방법
* 다만 slow 쿼리가 발생하는 환경에서는 많은 Thread 가 필요해 질 것으로 보이며, 이럴 경우 netty 여도 충분한 성능이 나오지 않을 수 있음.