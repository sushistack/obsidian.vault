
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

### ParameterizedTest

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class ParameterizedTestExample {

    @ParameterizedTest
    @CsvSource({
        "1, 1, 2",
        "2, 2, 4",
        "3, 3, 6",
        "4, 4, 8"
    })
    void testAddition(int a, int b, int expected) {
        assertEquals(expected, a + b);
    }
}
```

### @Nested

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class NestedTestExample {

    private Calculator calculator;

    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }

    @Nested
    class AdditionTests {
        @Test
        void testAddPositiveNumbers() {
            assertEquals(5, calculator.add(2, 3));
        }

        @Test
        void testAddNegativeNumbers() {
            assertEquals(-5, calculator.add(-2, -3));
        }
    }

    @Nested
    class SubtractionTests {
        @Test
        void testSubtractPositiveNumbers() {
            assertEquals(1, calculator.subtract(3, 2));
        }

        @Test
        void testSubtractNegativeNumbers() {
            assertEquals(-1, calculator.subtract(-2, -3));
        }
    }
}

class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }
}
```

다양한 단위(메소드 등)의 구조화?

### @EnabledOnOs, @EnabledOnOs

custom

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@EnabledOnOs(MAC)
@interface TestOnMac {
}
```

```java
@EnabledOnOs(architectures = "aarch64")
```

```java
@EnabledOnJre(JAVA_8)
@EnabledForJreRange(min = JAVA_9, max = JAVA_11)
@DisabledOnJre(JAVA_9)
@EnabledInNativeImage
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
@EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")
```

```java
@Test 
@EnabledIf("customCondition") 
void enabled() { 
	// ...
}

@Test 
@DisabledIf("customCondition") 
void disabled() { 
// ... 
} 
boolean customCondition() { return true; }
```


```java
package example; 
import org.junit.jupiter.api.Test; 
import org.junit.jupiter.api.condition.EnabledIf; 
class ExternalCustomConditionDemo { 

@Test 
@EnabledIf("example.ExternalCondition#customCondition") 
void enabled() { 
		// ... 
	} 
} 

class ExternalCondition { 
	static boolean customCondition() { return true; } 
}
```


```java
import org.junit.jupiter.api.Tag; 
import org.junit.jupiter.api.Test; 

@Tag("fast") 
@Tag("model") 
class TaggingDemo { 
	@Test 
	@Tag("taxes") 
	void testingTaxCalculation() { }

}
```


```java
import org.junit.jupiter.api.ClassOrderer; 
import org.junit.jupiter.api.Nested; 
import org.junit.jupiter.api.Order; 
import org.junit.jupiter.api.Test; 
import org.junit.jupiter.api.TestClassOrder; @TestClassOrder(ClassOrderer.OrderAnnotation.class) 
class OrderedNestedTestClassesDemo { 
@Nested @Order(1) 
class PrimaryTests { @Test void test1() { } } 

@Nested @Order(2) 
class SecondaryTests { @Test void test2() { } } }
```


```java
@ExtendWith(RandomParametersExtension.class)
class MyRandomParametersTest {

	@Test
	void injectsInteger(@Random int i, @Random int j) {
		assertNotEquals(i, j);
	}

  
	@Test
	void injectsDouble(@Random double d) {
		assertEquals(0.0, d, 1.0);
	}
}
```

### Interface Default 메소드 테스트

```java
@TestInstance(Lifecycle.PER_CLASS)
interface TestLifecycleLogger {

static final Logger logger = Logger.getLogger(TestLifecycleLogger.class.getName());

@BeforeAll
default void beforeAllTests() {
	logger.info("Before all tests");
}

@AfterAll
default void afterAllTests() {
	logger.info("After all tests");
}

@BeforeEach
default void beforeEachTest(TestInfo testInfo) {
	logger.info(() -> String.format("About to execute [%s]",
testInfo.getDisplayName()));
}

@AfterEach
default void afterEachTest(TestInfo testInfo) {
	logger.info(() -> String.format("Finished executing [%s]",
testInfo.getDisplayName()));
}

}
```

```java
interface TestInterfaceDynamicTestsDemo {
@TestFactory

default Stream<DynamicTest> dynamicTestsForPalindromes() {
return Stream.of("racecar", "radar", "mom", "dad")
.map(text -> dynamicTest(text, () -> assertTrue(isPalindrome(text))));
}

}
```


```java
@RepeatedTest(10) 
void repeatedTest() { 
// ... 
}
```


```java
@ParameterizedTest
@NullSource
@EmptySource
@ValueSource(strings = { "racecar", "radar", "able was I ere I saw elba" })
@EnumSource(ChronoUnit.class)
void palindromes(String candidate) {
assertTrue(StringUtils.isPalindrome(candidate));
}
```

```java
@ParameterizedTest
@MethodSource("stringProvider")
void testWithExplicitLocalMethodSource(String argument) {
	assertNotNull(argument);
}

static Stream<String> stringProvider() {
	return Stream.of("apple", "banana");
}
```

```java
@ParameterizedTest
@MethodSource("stringIntAndListProvider")
void testWithMultiArgMethodSource(String str, int num, List<String> list) {
	assertEquals(5, str.length());
	assertTrue(num >=1 && num <=2);
	assertEquals(2, list.size());
}


static Stream<Arguments> stringIntAndListProvider() {
return Stream.of(
arguments("apple", 1, Arrays.asList("a", "b")),
arguments("lemon", 2, Arrays.asList("x", "y"))
);

}
```

```java
package example;
import java.util.stream.Stream;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

class ExternalMethodSourceDemo {

@ParameterizedTest
@MethodSource("example.StringsProviders#tinyStrings")
void testWithExternalMethodSource(String tinyString) {
// test with tiny string
}
}

class StringsProviders {
static Stream<String> tinyStrings() {
return Stream.of(".", "oo", "OOO");
}
}
```

### FieldSource

### CsvSource


### ArgumentsSource


## Extensions

https://github.com/innoq/junit5-logging-extension

https://github.com/haasted/TestLogCollectors

