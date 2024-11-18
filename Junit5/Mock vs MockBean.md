
@Mock과 @MockBean은 둘 다 테스트에서 의존성을 모킹(mocking)하는 데 사용되지만, 사용하는 환경과 그 목적에 따라 차이가 있습니다. 주로 **Mockito**와 **Spring Boot**의 테스트 지원 기능에서 사용됩니다. 아래에서 두 가지의 차이점과 각각의 사용 사례를 자세히 설명하겠습니다.

  

**1. @Mock (Mockito)**

  

**개요**

• **라이브러리**: Mockito

• **목적**: 단위 테스트(Unit Testing)에서 객체의 의존성을 모킹하여 테스트 대상의 동작을 검증

**특징**

  

• **독립적 사용**: Spring 컨텍스트와 무관하게 독립적으로 사용

• **단순 모킹**: 단순히 인터페이스나 클래스를 모킹하여 원하는 동작을 정의

• **사용 예**:

  

```java
@RunWith(MockitoJUnitRunner.class)
public class MyServiceTest {

    @Mock
    private DependencyService dependencyService;

    @InjectMocks
    private MyService myService;

    @Test
    public void testServiceMethod() {
        when(dependencyService.someMethod()).thenReturn("Mocked Value");
        String result = myService.serviceMethod();
        assertEquals("Expected Result", result);
    }
}
```

  

  

  

**사용 시기**

  

• **순수 단위 테스트**: 스프링 컨텍스트 없이 순수하게 클래스의 동작을 검증할 때

• **빠른 테스트**: 스프링의 초기화 과정을 거치지 않아 빠르게 실행 가능

  

**2. @MockBean (Spring Boot)**

  

**개요**

• **라이브러리**: Spring Boot Test
• **목적**: 스프링 애플리케이션 컨텍스트에서 특정 빈(bean)을 모킹하여 통합 테스트나 슬라이스 테스트에서 사용

  

**특징**

  

• **Spring 컨텍스트 통합**: 모킹된 빈이 스프링의 애플리케이션 컨텍스트에 등록되어 실제 빈 대신 사용됨
• **자동 주입**: @Autowired 등으로 주입되는 모든 곳에서 모킹된 빈이 사용

• **사용 예**:

  

```kotlin
@SpringBootTest
public class MyServiceIntegrationTest {

    @MockBean
    private DependencyService dependencyService;

    @Autowired
    private MyService myService;

    @Test
    public void testServiceMethod() {
        when(dependencyService.someMethod()).thenReturn("Mocked Value");
        String result = myService.serviceMethod();
        assertEquals("Expected Result", result);
    }
}
```

  

**사용 시기**

• **통합 테스트**: 스프링 컨텍스트를 로드하면서 특정 빈만 모킹하여 실제 환경과 유사하게 테스트할 때

• **슬라이스 테스트**: 특정 계층(예: 웹, 데이터 접근 등)만 로드하면서 필요한 의존성만 모킹할 때

• **의존성 관리**: 전체 애플리케이션 컨텍스트에서 특정 빈의 동작을 제어하고자 할 때

  

**주요 차이점 요약**

**항목** @Mock **(Mockito)** @MockBean **(Spring Boot)**

**라이브러리** Mockito Spring Boot Test

**컨텍스트** 독립적, 스프링 컨텍스트와 무관 스프링 애플리케이션 컨텍스트에 통합

**사용 목적** 단위 테스트에서 클래스의 의존성 모킹 통합 테스트 또는 슬라이스 테스트에서 스프링 빈 모킹

**빈 주입** @InjectMocks 등을 사용하여 수동 주입 @Autowired 등을 통해 자동 주입

**테스트 속도** 빠름 스프링 컨텍스트 로딩에 따라 상대적으로 느릴 수 있음

**적용 범위** 단일 클래스 또는 메서드 스프링 컨텍스트 내의 전체 빈 또는 특정 계층에 걸쳐 적용 가능

  

**언제 어떤 것을 사용할까?**

  

• **단위 테스트**를 작성할 때는 @Mock을 사용하여 테스트 대상 클래스의 의존성을 모킹하는 것이 적합합니다. 이 경우 스프링 컨텍스트를 로드하지 않기 때문에 테스트가 빠르게 실행됩니다.

• **통합 테스트**나 **슬라이스 테스트**를 작성할 때는 @MockBean을 사용하여 스프링 애플리케이션 컨텍스트 내의 특정 빈을 모킹할 수 있습니다. 이는 실제 스프링 빈과의 상호작용을 테스트하면서도 특정 의존성만 모킹하여 제어할 수 있게 해줍니다.

  

**추가 팁**

  

• **혼용 사용**: 경우에 따라 @Mock과 @MockBean을 함께 사용할 수도 있지만, 일반적으로 하나의 테스트에서 둘 다 사용할 필요는 없습니다.

• **스프링의 다른 모킹 어노테이션**: @SpyBean 등 다른 스프링 모킹 어노테이션도 존재하므로 필요에 따라 활용할 수 있습니다.

• **테스트 설정 최적화**: @MockBean을 사용할 때는 필요한 빈만 모킹하여 테스트의 범위를 명확히 하고, 불필요한 모킹을 피하는 것이 좋습니다.

  

**결론**

  

@Mock과 @MockBean은 각각 다른 목적과 환경에서 사용되는 모킹 도구입니다. 단위 테스트에서는 빠르고 독립적인 테스트를 위해 @Mock을 사용하는 반면, 스프링 컨텍스트와 통합된 테스트에서는 @MockBean을 사용하여 스프링 빈을 효과적으로 모킹할 수 있습니다. 테스트의 목적과 범위에 따라 적절한 도구를 선택하여 사용하는 것이 중요합니다.