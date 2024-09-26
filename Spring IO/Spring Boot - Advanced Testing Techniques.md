
[# Beyond Built-in: Advanced Testing Techniques for Spring Boot Applications by Michael Vitz](https://youtu.be/vn9P38o03TQ?si=giK06fABqUGRzRMW)

```xml
<dependency>  
  <groupId>org.springframework.boot</groupId>  
  <artifactId>spring-boot-starter-test</artifactId>  
  <scope>test</scope>  
</dependency>
```

> Dependencies
> 
> jupitter: JUnit5에서 테스트 및 Extension을 작성하기 위한 새로운 프로그래밍 모델과 확장 모델의 조합
> 
> assertj:  풍부한 assertions 세트와 유용한 오류 메시지를 제공하고 테스트 코드 가독성을 향상시키며,IDE내에서 매우 쉽게 사용할 수 있도록 설계된 Java 라이브러리
> 
> hamcrest:  JUnit 기반의 단위 테스트에서 사용할 수 있는 Assertion Framework
> 
> mockito: Mock 이라는 가짜 객체를 생성하여, 테스트를 가능하도록 해주는 테스트 프레임워크
> 
> awaitility: awaitility는 비동기 테스트에 효과적인 라이브러리
> 
> json-path: 라이브러리는 Java에서 JSON 데이터를 쉽게 탐색하고 조작할 수 있도록 도와주는 라이브러리
> 
> jsonassert: JSON 데이터의 단언(assertion)을 쉽게 수행할 수 있도록 도와주는 Java 라이브러리
> 
> xmlunit-core: XML 데이터를 비교하고 검증하는 데 사용되는 Java 라이브러리
> 

**JUnit 5 = _JUnit Platform_ + _JUnit Jupiter_ + _JUnit Vintage_**

The **JUnit Platform** serves as a foundation for [launching testing frameworks](https://junit.org/junit5/docs/current/user-guide/#launcher-api) on the JVM The Jupiter sub-project provides a `TestEngine` for running Jupiter based tests on the platform.

**JUnit Jupiter** is the combination of the [programming model](https://junit.org/junit5/docs/current/user-guide/#writing-tests) and [extension model](https://junit.org/junit5/docs/current/user-guide/#extensions) for writing tests and extensions in JUnit 5. The Jupiter sub-project provides a `TestEngine` for running Jupiter based tests on the platform

**JUnit Vintage** provides a `TestEngine` for running JUnit 3 and JUnit 4 based tests on the platform. It requires JUnit 4.12 or later to be present on the class path or module path.

