
#  gRPC 서버 구현

## dependency

- spring 공식은 아님

```kotlin
implementation("net.devh:grpc-server-spring-boot-starter:3.1.0.RELEASE")

 
implementation(platform("io.grpc:grpc-bom:1.57.2"))

implementation("io.grpc:grpc-kotlin-stub:1.4.1")
implementation("io.grpc:grpc-protobuf")
implementation("io.grpc:grpc-stub")
```



## 빌드

- gradle 기준

```kotlin 
plugins {
    id("com.google.protobuf") version "0.9.4"
}

 
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.19.4"
    }
    plugins {
        id("grpc") {
            artifact = "io.grpc:protoc-gen-grpc-java:1.44.1"
        }
        id("grpckt") { //kotlin을 위한 작업 -> coroutine stub 생성 해줌
            artifact = "io.grpc:protoc-gen-grpc-kotlin:1.4.1:jdk8@jar"
        }
    }
    generateProtoTasks {
        all().forEach { task ->
            task.plugins {
                id("grpc") {}
                id("grpckt") {}
            }
        }
    }
}

 
sourceSets {
    main {
        proto {
            srcDir("src/main/proto")
        }
        java {
            srcDirs("build/generated/source/proto/main/java", "build/generated/source/proto/main/grpc")
        }
        kotlin {
            srcDirs("build/generated/source/proto/main/kotlin")
        }
    }
}

 
tasks.withType<ProcessResources> {
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
}


```

## properties

```yml
grpc.server:
    port: 9093
    security.enabled: false //내부 통신이므로 ssl 사용x
```

 
## .proto 파일 생성

```
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "google/protobuf/wrappers.proto";

option java_package = "com.hid.grpc.member";
option java_multiple_files = true;
option java_outer_classname = "HidMemberProto";

service HidMemberService {
    rpc getMemberIdByMemberNo (HidMemberNo) returns (HidMemberId);
    rpc getMemberNoByMemberId (HidMemberId) returns (HidMemberNo);
    rpc getMemberAllByMemberNo (HidMemberNo) returns (HidMemberAll);
    rpc getMemberAllByMemberId (HidMemberId) returns (HidMemberAll);
}

message HidMemberNo {
    string value = 1;
}

message HidMemberId {
    string value = 1;
}

message HidMemberAll {
    string memberNo = 1;
    string memberId = 2;
    string customerNo = 3;
    string validCode = 4;
    string sidValidCode = 5;
    string snsCode = 6;
    google.protobuf.Timestamp passwordModificationYmdt = 8;
    google.protobuf.Timestamp passwordCampaignCheckYmdt = 9;
    string nickname = 10;
    string nicknameUniqueYn = 11;
    google.protobuf.Timestamp nicknameModificationYmdt = 12;
    string mobileNo = 13;
    string email = 14;
    int32 loginFailCount = 15;
    string loginCampaignCode = 16;
    string loginCountryCode = 17;
    string eventCode = 18;
    string ipTraceCode = 19;
    google.protobuf.Timestamp registrationYmdt = 20;
    google.protobuf.Timestamp modificationYmdt = 21;

}
```

## Procedure 구현

- HidMemberServiceGrpcKt.HidMemberServiceCoroutineImplBase 는 자동 생성 됨.
- 코틀린이 아닐 경우 HidMemberServiceGrpc.HidMemberServiceImplBase 를 구현

```kotlin 
@GrpcService
class HidMemberServiceProcedure(
    private val memberService: MemberService,
) : HidMemberServiceGrpcKt.HidMemberServiceCoroutineImplBase() {
    private val log = logger()

    override suspend fun getMemberIdByMemberNo(request: HidMemberNo): HidMemberId {
        val memberId = memberService.getMemberIdByMemberNo(request.value)
            ?: throw GrpcHidMemberException(GrpcHidMemberErrorCode.NOT_FOUND, "param: $request")
        return HidMemberId.newBuilder()
            .setValue(memberId)
            .build()
    }

    override suspend fun getMemberNoByMemberId(request: HidMemberId): HidMemberNo {
        val memberNo = memberService.getMemberNoByMemberId(request.value)
            ?: throw GrpcHidMemberException(GrpcHidMemberErrorCode.NOT_FOUND, "param: $request")
        return HidMemberNo.newBuilder()
            .setValue(memberNo)
            .build()
    }

    override suspend fun getMemberAllByMemberNo(request: HidMemberNo): HidMemberAll {
        return memberService.getMemberByMemberNo(request.value)?.let { ProtoConvertUtil.toProto(it) }
            ?: throw GrpcHidMemberException(GrpcHidMemberErrorCode.NOT_FOUND, "param: $request")
    }

    override suspend fun getMemberAllByMemberId(request: HidMemberId): HidMemberAll {
        return memberService.getMemberByMemberId(request.value)?.let { ProtoConvertUtil.toProto(it) }
            ?: throw GrpcHidMemberException(GrpcHidMemberErrorCode.NOT_FOUND, "param: $request")
    }

}
```


## ExceptionAdvice

- https://yidongnan.github.io/grpc-spring-boot-starter/en/server/exception-handling.html

## interceptor

- GrpcGlobalServerInterceptor 및 procedure 단위, client stub 단위 interceptor 지원
    - access log 등 GrpcGlobalServerInterceptor 를 활용하여 남겨야 함.

## server customizer

```kotlin
@Bean
fun serverConfigurers(): GrpcServerConfigurer {
    return GrpcServerConfigurer { server ->
        if (server is NettyServerBuilder) {
            server.flowControlWindow(65535); // 64KB (기본값: 65,535)
        }
        server.executor(Executors.newFixedThreadPool(AVAILABLE_PROCESSORS)) // 요청 / 응답만 하는 Thread
    }
}
```

## 주의사항

- spring-grpc 는 netty 기반의 non-blocking 서버
- ThreadLocal 사용에 주의해야 함
    - Context 나 scoped bean 을 지원하고 있음
        - https://yidongnan.github.io/grpc-spring-boot-starter/en/server/contextual-data.html
    - 혹은 CoroutineContext 나 webflux의 Context 를 사용해도 됨.


# 이슈 사항

## DB Connection ?

- netty 서버이므로 r2dbc 사용을 고려해봐야 함.

## ACL?

- Interceptor 를 통해 처리 가능


## 서버 쏠림 현상

- http2 는 특성상 keep-alive 가 켜져있고 웬만하면 connection 을 끊지 않음
- 배포 시 마지막 배포 서버가 재시작 된 이후 마지막 서버와 통신하는 client 는 없을 것으로 예상 됨

### 해결 방법

- 서버에서 주기적으로 connection 을 끊는다.
    - 10초 혹은 30초 마다 재연결하도록 설정
    - 재 연결되면서 자연스럽게 부하가 분산 처리 됨.
- Client 에서 Discovery 사용 : 
    - Client 는 모든 서버에 동시에 1개씩 연결해서 연결이 유지해 주고 있는것을 확인 함.
    - Client-Side Round-robin
        - 연결된 모든 서버에 round-robin 으로 하나씩 호출 함.
    - 즉, Spring grpc 라이브러리를 활용할 경우 서버에서 특별히 해줘야 할 것이 없음.
- Domain (DNS) 기반으로 제공
    - DNS SRV 레코드를 이용하여 하나의 도메인으로 여러 IP/ PORT 를 동시에 전달
    - Client 는 모든 서버에 동시에 1개씩 연결해서 연결이 유지
    - Client-Side Round-robin
        - 연결된 모든 서버에 round-robin 으로 하나씩 호출 함.
    - 다만 서버 down 시 반영이 느려질 수 있어 안정성이 떨어짐.
- NginX, Apache, Haproxy 등에서 gRPC 를 위한 proxy 설정
    - 주로 외부에 오픈된 gRPC 서버에서 필요할 것으로 보임.
    - proxy 에서 http2 연결 옵션을 활성화 필요
    - client <--h2--> proxy 연결 관리와는 독립적으로 proxy <--h2--> backend 연결 풀링 설정을 해야 함.
        - client 요청은 proxy 에서 연결 유지
        - proxy 에서 backend 로의 요청은 풀링 및 loadbalancing 되어 요청 됨
    - 각각 grpc 를 위한 옵션을 제공하기도 하고, http2 제어 옵션만 추가되기도 함.


# 참조 정보
 
- client <-h2-> Proxy <-http1.1-> backend 인 경우가 많은데 취약점이 있다고 함.
    - https://m.boannews.com/html/detail.html?idx=99638