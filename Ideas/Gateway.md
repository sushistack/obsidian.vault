
## [토스ㅣSLASH 23 - 토스는 Gateway 이렇게 씁니다](https://youtu.be/Zs3jVelp0L8?si=PCSLXAR078Em08uV)

Gateway: Zuul (Neflix)
MSA의 중계자 역할

### 장단점

인증 등과 같은 공통 처리

### Gateway


![](Pasted%20image%2020240925162358.png)


1. 요청
2. 핸들러에 의한 라우팅
3. 필터링
4. 설정된 업스트림 서버로 요청을 프록시

#### Route

![](Pasted%20image%2020240925162722.png)

##### Predicate

요청을 구분할 때 사용

##### Filter

요청에 대한 전처리 혹은 응답에 대한 후처리

#### BFF (Backend For FrontEnd)

![](Pasted%20image%2020240925162917.png)

위 처럼 게이트웨이는 하나의 모놀리틱 시스템이 됨, 이를 해결하기 위해 BFF 패턴을 사용.


![](Pasted%20image%2020240925163051.png)

#### Ingress / Egress

![](Pasted%20image%2020240925163114.png)

![](Pasted%20image%2020240925163225.png)

![](Pasted%20image%2020240925163319.png)

![](Pasted%20image%2020240925163456.png)

요청을 지우거나/ 올바른 값으로 변경

![](Pasted%20image%2020240925163531.png)

TraceID를 사용 및 MDC 컨텍스트에도 저장함.

MDC 컨텍스트란?

MDC(Mapped Diagnostic Context)는 로깅 프레임워크에서 사용되는 기능으로, 로그 메시지에 추가적인 정보를 포함시킬 수 있게 해줍니다. MDC는 주로 로그 메시지의 컨텍스트를 제공하기 위해 사용됩니다. 예를 들어, 사용자 ID, 세션 ID, 트랜잭션 ID 등과 같은 정보를 로그 메시지에 포함시켜 로그를 더 쉽게 분석하고 추적할 수 있게 합니다.

MDC는 주로 SLF4J(Simple Logging Facade for Java)와 Logback, Log4j와 같은 로깅 프레임워크에서 사용됩니다.


![](Pasted%20image%2020240925163752.png)

![](Pasted%20image%2020240925163804.png)

### Toss Passport (powered by Neflix)

![](Pasted%20image%2020240925163915.png)

![](Pasted%20image%2020240925164008.png)

![](Pasted%20image%2020240925164053.png)

![](Pasted%20image%2020240925164214.png)

![](Pasted%20image%2020240925164332.png)


### 서비스 안정화

![](Pasted%20image%2020240925164434.png)


![](Pasted%20image%2020240925164501.png)

![](Pasted%20image%2020240925164617.png)

### 게이트웨이 구조

![](Pasted%20image%2020240925164737.png)

![](Pasted%20image%2020240925164854.png)

![](Pasted%20image%2020240925164906.png)

![](Pasted%20image%2020240925165007.png)

Github actions 로 검증


![](Pasted%20image%2020240925165024.png)

![](Pasted%20image%2020240925165136.png)

## 모니터링

![](Pasted%20image%2020240925165247.png)

![](Pasted%20image%2020240925165301.png)

![](Pasted%20image%2020240925165319.png)

### Metric

![](Pasted%20image%2020240925165347.png)

![](Pasted%20image%2020240925165401.png)

![](Pasted%20image%2020240925165452.png)

