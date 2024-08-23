
# gRPC Client 구현
## dependency

```kotlin
implementation("net.devh:grpc-client-spring-boot-starter:3.1.0.RELEASE")

  
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


 
## .proto 파일 배포

- server 에서만든 .proto 파일을 공유 받아야 함.

 
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
## properties

```yml
grpc.client:
  hidMemberService:
    address: static://127.0.0.1:9091
    enable-keep-alive: true
    negotiation-type: plaintext
    keep-alive-time: 10m
    keep-alive-timeout: 3s #default : 20s
    keep-alive-without-calls: true
    shutdown-grace-period: 1s
    user-agent: test-grpc #client 가 직접 user-agent 에 string 을 추가해서 소속을 알릴 수 있음
    default-load-balancing-policy: round_robin
```


## stub 사용

```kotlin
 
@GrpcClient("hidMemberService")
private lateinit var hidMemberServiceStub: HidMemberServiceGrpc.HidMemberServiceBlockingStub
    
fun getMemberIdByMemberNo(): String {
    try {
        val memberId = hidMemberServiceStub
            .withDeadlineAfter(1, TimeUnit.SECONDS)
            .getMemberIdByMemberNo(
                HidMemberNo.newBuilder()
                    .setValue("4101001000018016")
                    .build()
            )
        return memberId.value
    } catch (e: Exception) {
        throw e;
    }
}
```

- 논블로킹 stub 도 지원
    - HidMemberServiceGrpc.HidMemberServiceFutureStub
    - HidMemberServiceGrpcKt.HidMemberServiceCoroutineStub
- withDeadlineAfter 
    - 지정된 시간 까지만 대기하겠다는 표시 사용하지 않을 경우 무제한 대기할 수 있음.

- 오류 처리
    - StatusException 혹은 StatusRuntimeException 을 catch 하여 처리해 줘야 함.

# 이슈 사항

## gRPC 서버 IP 는 discovery 를 통해 공유 받을 수 있나?
- consul discovery 기준으로 가능한 것을 확인
```yml
grpc.client.{ProcedureName}.address: discovery:///msa-hid-member-grpc
```

##  gRPC 네트워크 이슈 대응?
- 일단 조회 요청(GET) 요청에서만 사용하려고 하는데 retry 로 대응
- grpc라이브러리가 자체적으로 connection 관련 오류는 재연결 하는것 같음.


## 서버 쏠림 현상 해결?
- 한번 연결한 http2 grpc 통신은 별도의 옵션을 설정하지 않으면 계속 연결된 상태를 유지
    - keep-alive 연결에 의한 쏠림 현상 발생 가능성
- Pooling 을 통해 모든 서버에 동시에 연결 필요?
    - 직접 구현할 필요가 없음
    - @GrpcClient 어노테이션에 의해 자동 생성되는 Stub 의 경우 Discovery로 얻은 모든 서버에 동시에 연결
    - grpc 서버 재시작 완료되면 거의 곧바로 재연결이 이루어져서 로드밸런싱 처리 됨.
    -  default-load-balancing-policy 에 의해 분산하여 요청 발생
- consul discovery 를 사용할 경우 서버 down, up 발생 시 거의 곧바로 client 에서 update server list 로그가 발생하면서 서버 목록이 갱신 됨.
- Proxy 를 통해 http2 연결 할 때
    - proxy 에서 알아서 서버 쏠림 현상을 해결해야 함.