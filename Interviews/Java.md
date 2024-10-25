## 1. **Java 메모리 관리와 Garbage Collection (GC)에 대해 설명하고, GC 튜닝 방법에 대해 이야기해보세요.**

#### 이 질문의 목적:

- Java 메모리 모델에 대한 깊이 있는 이해를 확인.
- Heap, Stack, Young/Old Generation, PermGen(Metaspace) 등의 구조적 이해도를 평가.
- Garbage Collection 프로세스가 어떻게 동작하는지(Stop-the-world, Minor/Major GC, CMS, G1 GC 등)에 대한 지식 확인.
- 애플리케이션의 성능 최적화를 위해 GC 튜닝 방법을 알고 있는지 평가.

#### 추가 질문 예시:

- **GC 동작을 모니터링하고 튜닝한 경험**에 대해 구체적으로 설명할 수 있나요?
- 어떤 **GC 알고리즘을 사용**했고, 어떻게 애플리케이션에 맞게 조정했나요?
- **OutOfMemoryError**가 발생할 때 이를 해결하기 위한 방법은 무엇인가요?

#### 답변 기대치:

- 메모리 구조에 대한 설명(Heap, Stack, Young/Old Gen 등).
- 기본적인 GC 알고리즘(Serial, Parallel, CMS, G1 GC)에 대한 설명과 각각의 장단점.
- GC 튜닝 시 사용하는 JVM 옵션들(`-Xms`, `-Xmx`, `-XX:+UseG1GC` 등) 및 성능 문제 해결 경험.


## 2. **멀티스레드 환경에서 발생할 수 있는 문제(예: 데드락, 레이스 컨디션)를 설명하고, 이를 방지하거나 해결하는 방법은 무엇인가요?**

#### 이 질문의 목적:

- 멀티스레드 프로그래밍의 복잡성에 대한 이해도를 평가.
- 동시성 문제(데드락, 레이스 컨디션, 리소스 경합 등)에 대한 경험과 해결 능력을 확인.
- **synchronized**, **ReentrantLock**, **volatile**, **ThreadPool** 등 자바 동시성 도구를 적절히 사용할 수 있는지 확인.
- 멀티스레드 프로그래밍에서의 성능 최적화 경험을 평가.

#### 추가 질문 예시:

- 동기화(synchronization)를 과도하게 사용했을 때 발생할 수 있는 성능 문제는 무엇인가요?
- Java에서 **ThreadPool**을 사용하여 멀티스레딩을 최적화한 경험이 있나요?
- **Lock-Free 알고리즘**을 구현해본 적이 있나요?

#### 답변 기대치:

- 데드락(Deadlock), 레이스 컨디션(Race Condition), 라이브락(Livelock) 등의 문제를 설명하고 이를 해결하는 방법(동기화, 락, Atomic 클래스, 비동기 처리 등).
- 자바에서 동시성 문제를 해결할 때 사용할 수 있는 다양한 방법들(Ex: `synchronized`, `ReentrantLock`, `volatile`, `java.util.concurrent` 패키지 사용).
- 실제 프로젝트에서 멀티스레드 문제를 해결한 사례 공유.


## 3. Java의 동시성(concurrency) 처리에서 `synchronized` 키워드와 `ReentrantLock`, 그리고 `java.util.concurrent.atomic` 패키지의 클래스들 간의 차이점을 설명해주세요. 각각이 적합한 사용 사례는 무엇이며, 이들을 사용할 때 발생할 수 있는 잠재적인 문제점이나 성능상의 고려 사항은 무엇인가요?

**추가 질문:** _`CompletableFuture`와 같은 비동기 프로그래밍 도구를 사용하여 동시성을 관리하는 방법을 설명해주세요. 또한, `ForkJoinPool`과 병렬 스트림(parallel streams)이 멀티코어 시스템에서 어떻게 성능을 향상시키는지 논의해주세요._

## 4. Java 9에서 도입된 모듈 시스템(Project Jigsaw)은 애플리케이션 개발 및 배포에 어떤 영향을 미치나요? 기존의 클래스패스(classpath) 메커니즘과 어떻게 다르며, 대규모의 기존 애플리케이션을 모듈 시스템으로 마이그레이션할 때의 모범 사례는 무엇인가요? 마이그레이션 과정에서 발생할 수 있는 도전 과제와 그에 대한 해결 방법을 상세히 논의해주세요.

**추가 질문:** _모듈 시스템에서의 접근 제어는 기존의 접근 제어자와 어떻게 다르며, `exports`와 `opens` 키워드를 사용하여 패키지의 가시성을 어떻게 제어할 수 있는지 설명해주세요. 또한, 리플렉션(reflection) API와 모듈 시스템의 상호 작용에 대해 논의해주세요._


## Thread 와 cpu core 수의 관계

### 1. **CPU 바운드 작업**:

• CPU 바운드 작업은 대부분의 시간을 계산 작업에 사용합니다. 이 경우, 스레드의 개수가 CPU 코어 수에 가까울 때 성능이 가장 좋습니다.
• 예를 들어, 4개의 CPU 코어가 있다면 동시에 실행되는 스레드의 수가 4개에 가깝게 유지되는 것이 유리합니다. 코어 수보다 많은 스레드를 실행하게 되면 문맥 전환(Context Switching)으로 인해 오히려 성능이 떨어질 수 있습니다.

### 2. **I/O 바운드 작업**:

• I/O 바운드 작업은 네트워크 통신, 파일 입출력 등 CPU보다 외부 자원 접근에 시간을 많이 사용합니다.
• 이 경우에는 스레드 수가 CPU 코어 수보다 더 많아도 괜찮습니다. 왜냐하면 한 스레드가 I/O 작업을 기다리는 동안 다른 스레드가 CPU를 사용할 수 있기 때문입니다.

결론적으로, **CPU 바운드 작업**에서는 CPU 코어 수와 비슷한 수의 스레드를 사용하는 것이 이상적이며, **I/O 바운드 작업**에서는 CPU 코어 수보다 더 많은 스레드를 사용할 수 있습니다.


### 3. 그러면 왜 tomcat 의 기본 thread 수는 200개임?

 트래픽이 많은 서버에서 maxThreads 값을 높게 설정하는 이유는 **모든 요청을 동시에 처리할 수 없더라도, 우선 요청을 수용하고 대기열에 넣어 처리**하기 위함.


- **동시 요청 수용 능력**
- **요청 처리 대기열 관리**
- **타임아웃 문제 방지**
- **트래픽 스파이크 대응**


## 4. Java 19의 가상 스레드(Virutal thread)

### **1. 가상 스레드의 특징:**

• **가벼운 스레드**: 가상 스레드는 전통적인 운영체제의 스레드와 달리, JVM 내부에서 가볍게 관리됩니다. 그래서 수천 개, 심지어 수백만 개의 가상 스레드를 만들 수 있습니다.

• **OS 스레드와 독립적**: 가상 스레드는 전통적인 OS 스레드에 직접적으로 매핑되지 않고, 필요한 시점에만 OS 스레드를 사용합니다. I/O 작업 등으로 블로킹이 발생하면 다른 가상 스레드로 전환하여 실행할 수 있습니다.

• **문맥 전환 비용 감소**: 가상 스레드는 JVM 레벨에서 문맥 전환을 관리하기 때문에, 전통적인 OS 스레드에 비해 문맥 전환의 오버헤드가 크게 줄어듭니다. 이는 가상 스레드가 경량화된 이유 중 하나입니다.


### 적용 시 이점과 고려사항

• **높은 동시성 처리**: 가상 스레드를 사용하면 수백 개 이상의 스레드를 동시에 실행하는 것이 부담이 되지 않으므로, 가상 스레드로 운영되는 Tomcat과 Spring 애플리케이션은 **수많은 동시 요청**을 쉽게 처리할 수 있습니다.

• **메모리 사용량 감소**: 가상 스레드는 각 스레드가 사용하는 스택 메모리가 더 작기 때문에, **기존 OS 스레드보다 메모리 사용량이 적습니다**.

• **CPU 자원 활용 최적화**: 가상 스레드 환경에서는 CPU 자원을 보다 효율적으로 사용할 수 있습니다. CPU 코어 수보다 훨씬 더 많은 수의 스레드가 존재하더라도, 필요에 따라 가상 스레드가 유연하게 스케줄링되기 때문입니다.

• **새로운 기술에 따른 안정성 고려**: 가상 스레드는 Java 19부터 도입된 새로운 기능이기 때문에, 기존의 안정적인 환경에서 전환할 때는 **테스트와 검증이 필수**입니다.


## 5. 하이퍼 스레드 (Hyper Thread)

 **CPU의 물리적 코어 하나를 두 개의 논리적 코어처럼 사용하는 기술**

### **1. 하이퍼 스레딩의 작동 원리:**

• 물리적 코어 하나가 두 개의 **논리적 프로세서**처럼 동작하도록 해서, 두 개의 스레드를 동시에 처리하는 것처럼 보이게 합니다.
• 예를 들어, **4개의 물리적 코어를 가진 CPU**가 하이퍼 스레딩을 지원하면, **8개의 논리적 코어**처럼 동작할 수 있습니다.
• 각 논리적 코어는 **독립적인 스레드**를 처리할 수 있으며, 물리적 코어의 유휴 자원을 활용하여 성능을 향상시킵니다.


### **2. 하이퍼 스레딩의 장점:**

• **유휴 시간 활용**: 물리적 코어가 완전히 활용되지 않는 경우, 하이퍼 스레딩은 동일한 물리적 코어 내에서 다른 스레드를 처리함으로써 **유휴 자원**을 활용합니다.

• **동시 스레드 처리**: 하이퍼 스레딩을 사용하면 더 많은 스레드를 동시에 처리할 수 있으므로, **스레드 기반의 애플리케이션에서 성능 향상**을 기대할 수 있습니다. 이는 Tomcat, Spring과 같은 멀티스레드 기반 애플리케이션에서도 유리하게 작용할 수 있습니다.

### **3. 하이퍼 스레딩과 스레드 풀 설정:**

• **CPU 바운드 작업**: 하이퍼 스레딩이 있다고 해서 물리적 코어 수를 기준으로 설정하는 것과 완전히 다르게 잡을 필요는 없습니다. 하이퍼 스레딩은 물리적 코어의 성능을 **약 20~30% 정도 더 효율적으로** 사용할 수 있게 해주지만, **실제로는 물리적 코어가 많을 때처럼 성능이 올라가는 것은 아닙니다**.

• 예를 들어, **16코어 CPU에 하이퍼 스레딩이 적용되어 논리적 코어가 32개**라면, CPU 바운드 작업의 경우 **스레드 풀 크기를 16~24개** 정도로 설정하는 것이 일반적으로 적합합니다.

• **I/O 바운드 작업**: 하이퍼 스레딩은 I/O 바운드 작업에서 더 많은 이점을 줄 수 있습니다. 스레드가 I/O 대기 상태일 때 다른 스레드가 물리적 코어의 리소스를 더 잘 활용할 수 있기 때문입니다. 이 경우 논리적 코어 수를 기준으로 좀 더 여유롭게 스레드 풀을 설정할 수 있습니다.

• 예를 들어, **Tomcat의 maxThreads**를 설정할 때, 하이퍼 스레딩을 고려해 논리적 코어 수인 32개보다 많은 수의 스레드를 설정할 수 있습니다. 이는 각 논리적 코어가 블로킹되는 동안 다른 스레드가 물리적 코어의 자원을 활용할 수 있기 때문입니다.



# Java의 예외 생성 비용, 비용 절감 방법

자바로 프로젝트를 진행할 때, 보통 에러 처리의 일관성과 가독성, 로깅, 디버깅, 예외 처리 유연성을 위해서 `CustomException` 클래스를 정의하여 자주 사용한다.

그러나 여러 이점들이 있음에도, 자바에서는 Exception의 처리 비용이 매우 비싸다는 문제가 있다.

이번 글에서는 JVM이 Exception을 처리하는 순서와 생성 비용이 비싼 이유, 마지막으로 비용 절감 방법에 대해서 알아보도록 하겠다.

<br>

## JVM Exception 처리 순서

[이 글](https://www.geeksforgeeks.org/exceptions-in-java/)을 참고해보면, Exception이 발생하면 다음과 같이 JVM에서 Exception을 수행한다.

![](https://media.geeksforgeeks.org/wp-content/uploads/20230714113633/Exceptions-in-Java2-768.png)


1. **예외 발생**: 예외가 발생하면 JVM은 예외 객체를 생성하고, 예외를 발생시킨 메서드의 호출 스택을 추적한다.
2. **예외 객체 전파**: JVM은 해당 예외를 발생시킨 메서드에서 예외 처리 코드를 찾는다. 예외 처리 코드가 없는 경우에는 예외 객체를 호출하여 스택의 상위 메서드로 전파시킨다.
3. **예외 처리**: 예외 객체가 상위 메서드로 전파되게 된다면, 해당 예외를 처리할 수 있는 catch 블록을 찾고, 없다면 다시 상위로 전파된다.
4. **예외 처리 실패**: 예외 객체가 최상위 메서드까지 전파되어도, 예외를 처리할 수 있는 catch 블록이 없는 경우 JVM은 예외를 처리하지 못한 것으로 판단하여 해당 예외를 처리할 수 있는 `DefaultExceptionHandler`를 사용해 예외를 처리한다.
5. **DefaultExceptionHandler 실행**: DefaultExceptionHandler는 예외 객체에 대한 정보를 출력하고 해당 예외를 처리하거나 스냅샷 정보를 수집하여 디버깅을 위한 정보로 제공한다.

예외가 발생한 메서드에서 바로 처리가 된다면 가장 좋지만, 바로 처리되지 못한다면 **JVM은 해당 예외를 처리할 수 있는 메서드를 찾을 때 까지 계속해서 상위 메서드로 거슬러 올라가면서 메모리의 호출 스택(call stack)을 탐색하게 된다.**


<br>

### Exception이 비용이 생기는 원인

위에서 언급한 것 처럼 호출 스택을 탐색하는 과정도 비용이지만, `fillInStackTrace()` 메서드가 호출 스택을 순회하며 클래스명, 메서드명, 코드 줄 번호 등의 정보를 모아 stacktrace로 만드는 과정 또한 비용 증가의 원인이라고 볼 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcvfD8q%2Fbtr3dbdvQIi%2FAdzJFLLWGbiCLyv2zpJ6qK%2Fimg.png)

`fillInStackTrace()`는 Throwable 클래스에 정의된 구현 메서드로 생성자에서 호출되도록 되어있다.

모든 Exception은 Throwable을 상속받도록 되어있기 때문에 해당 메서드를 가지고 있다.

일반적으로 stacktrace 생성하는 시간은 몇 밀리초에서 몇 초까지 다양하다. 하지만 이는 예외가 발생한 환경과 stacktrace의 depth, stackframe의 메서드 호출 수, JVM 버전 및 설정 등에 따라 다르기 때문에 시간을 특정하기는 어렵다.

그러나 확실한 것은 stacktrace가 깊을수록 오랜 시간이 걸린다는 것이다.

<br>

## 비용 절감 방법

두 가지 방법을 소개해보겠다.

### 1. fillInStackTrace() 재정의 하기

보통 NPE나 OOM와 같이 자바에서 기본적으로 제공하는 예외를 제외한, CustomException은 에러의 추족보다는 유효하지 않은 값일 때 하위 비즈니스 로직을 수행하지 못하도록 하기 위한 용도로 주로 사용된다. 따라서 보통 StackTrace가 필요가 없다.

```java
@Override 
public synchronized Throwable fillInStackTrace() { 
	return this; 
}
```

때문에 단순히 try-catch로 이후 flow를 제어하거나 Spring 환경에서 @ControllerAdvice로 예외 처리하는 경우에는 불필요한 성능 저하를 막기 위해 trace를 저장하지 않도록 오버라이딩 하여 처리할 수 있다.

```java
public class DuplicateLoginException extends RuntimeException {
	public DuplicateLoginException(String message) { 
		super(message); 
	} 
	
	@Override 
	public synchronized Throwable fillInStackTrace() {
		return this; 
    } 
}
```

### Before

```java
kancho.realestate.comparingprices.exception.DuplicateLoginException: 이미 로그인한 상태입니다. at kancho.realestate.comparingprices.controller.UserController.validateDuplicateLogin(UserController.java:67) at kancho.realestate.comparingprices.controller.UserController.login(UserController.java:44) at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) at java.base/java.lang.reflect.Method.invoke(Method.java:566) at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205) at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:150) at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:117) at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895) at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808) at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1067) at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:963) at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006) at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909) at javax.servlet.http.HttpServlet.service(HttpServlet.java:681) at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883) at org.springframework.test.web.servlet.TestDispatcherServlet.service(TestDispatcherServlet.java:72) at javax.servlet.http.HttpServlet.service(HttpServlet.java:764) at org.springframework.mock.web.MockFilterChain$ServletFilterProxy.doFilter(MockFilterChain.java:167) at org.springframework.mock.web.MockFilterChain.doFilter(MockFilterChain.java:134) at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100) at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) at org.springframework.mock.web.MockFilterChain.doFilter(MockFilterChain.java:134) at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93) at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:119) at org.springframework.mock.web.MockFilterChain.doFilter(MockFilterChain.java:134) at ... 생략
```

### After
```java
kancho.realestate.comparingprices.exception.DuplicateLoginException: 이미 로그인한 상태
```

<br>

### 예외 캐싱
static final로 선언하여 예외를 미리 캐싱해서 사용하는 것이다.

일종의 상수 값 형태로 예외를 캐싱해 두고 쓰는 것이 매번 같은 종류의 예외로 new로 생성하는 방법보다 효율적이다.

```java
public class CustomException extends RuntimeException {
	public static final CustomException INVALID_NICKNAME = new CustomException(ResponseType.INVALID_NICKNAME);     
	public static final CustomException INVALID_PARAMETER = new CustomException(ResponseType.INVALID_PARAMETER);     
	public static final CustomException INVALID_TOKEN = new CustomException(ResponseType.INVALID_TOKEN);     //생략 
}
```

위와 같이 Exception 클래스에 예외 상황에 대한 적당한 응답 메시지나 코드를 담도록 한 뒤, 아래처럼 예외 발생 상황에서 new 키워드 없이 throw를 수행하도록 한다.

```java
if (StringUtils.isBlank(parameter)) {
	throw WebtoonCoreException.INVALID_PARAMETER; 
}
```