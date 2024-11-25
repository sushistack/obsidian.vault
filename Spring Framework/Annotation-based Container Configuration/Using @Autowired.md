기본적으로, 자동 주입은 주어진 주입 지점에 대해 일치하는 후보 빈이 없을 경우 실패합니다. 배열, 컬렉션 또는 맵이 선언된 경우에는 적어도 하나의 일치하는 요소가 필요합니다.

기본 동작은 애노테이션이 적용된 메서드와 필드를 필수 의존성을 나타내는 것으로 간주합니다. 그러나 다음 예제에서 보여주는 것처럼, `@Autowired`의 `required` 속성을 `false`로 설정하여 프레임워크가 만족되지 않는 주입 지점을 건너뛸 수 있도록 이 동작을 변경할 수 있습니다:

```kotlin
class SimpleMovieLister {

	@Autowired(required = false)
	var movieFinder: MovieFinder? = null

	// ...
}
```

```
필수로 설정되지 않은(non-required) 메서드는 해당 의존성(또는 다중 인수의 경우 하나의 의존성이라도)이 없으면 전혀 호출되지 않습니다. 필수로 설정되지 않은 필드는 이러한 경우 전혀 값이 주입되지 않으며, 기본값이 그대로 유지됩니다.

즉, `required` 속성을 `false`로 설정하면 해당 속성이 자동 주입(auto-wiring) 관점에서 선택 사항임을 나타내며, 자동 주입이 불가능한 경우 해당 속성이 무시됩니다. 이를 통해 속성에 기본값을 설정하고, 필요에 따라 의존성 주입을 통해 선택적으로 덮어쓸 수 있습니다.
```
주입된 생성자 및 팩토리 메서드의 인수는 특수한 경우에 해당합니다. 이는 스프링의 생성자 해석 알고리즘이 다중 생성자를 처리할 가능성이 있기 때문에, `@Autowired`의 `required` 속성이 약간 다른 의미를 가지기 때문입니다. 생성자 및 팩토리 메서드의 인수는 기본적으로 필수로 간주됩니다. 그러나 단일 생성자 시나리오에서는 몇 가지 특별한 규칙이 적용됩니다. 예를 들어, 다중 요소 주입 지점(배열, 컬렉션, 맵 등)이 일치하는 빈이 없을 경우 빈 인스턴스로 해결됩니다. 

이를 통해 모든 의존성을 고유한 다중 인수 생성자에서 선언할 수 있는 일반적인 구현 패턴을 지원합니다. 예를 들어, `@Autowired` 애노테이션 없이 단일 `public` 생성자로 선언할 수 있습니다.

```
하나의 빈 클래스에 대해 `@Autowired`와 `required` 속성이 `true`로 설정된 생성자는 하나만 선언할 수 있습니다. 이는 스프링 빈으로 사용할 때 자동 주입할 생성자를 나타냅니다. 따라서, `required` 속성이 기본값인 `true`로 설정된 경우, `@Autowired`로 주석이 달린 생성자는 하나만 허용됩니다. 만약 여러 생성자가 `@Autowired`를 선언했다면, 자동 주입 후보로 고려되기 위해서는 모두 `required=false`를 선언해야 합니다(XML의 `autowire=constructor`와 유사).

스프링 컨테이너에서 일치하는 빈으로 충족할 수 있는 의존성이 가장 많은 생성자가 선택됩니다. 자동 주입 후보 생성자 중 어느 것도 충족할 수 없는 경우, 기본(primary) 또는 기본값(default) 생성자가 사용됩니다(있을 경우). 이와 유사하게, 클래스에 여러 생성자가 선언되어 있지만 그중 어떤 것도 `@Autowired`로 주석 처리되지 않은 경우에도 기본(primary) 또는 기본값(default) 생성자가 사용됩니다(있을 경우). 

클래스에 단일 생성자만 선언된 경우, 주석이 없어도 항상 해당 생성자가 사용됩니다. 참고로, 주석이 달린 생성자는 `public`일 필요가 없습니다.
```

또한, 특정 의존성이 필수가 아님을 나타내기 위해 Java 8의 `java.util.Optional`을 사용할 수도 있습니다. 다음 예제는 이를 보여줍니다:

```java
public class SimpleMovieLister {

	@Autowired
	public void setMovieFinder(Optional<MovieFinder> movieFinder) {
		...
	}
}
```

또한, `@Nullable` 애노테이션(어떤 패키지의 것이든 사용 가능 — 예: JSR-305의 `javax.annotation.Nullable`)을 사용할 수 있으며, 또는 Kotlin의 내장된 null 안전성 지원을 활용할 수도 있습니다:

```kotlin
class SimpleMovieLister {

	@Autowired
	var movieFinder: MovieFinder? = null

	// ...
}
```

또한, `@Autowired`를 사용하여 다음과 같은 잘 알려진 해석 가능한 의존성 인터페이스를 주입할 수도 있습니다: `BeanFactory`, `ApplicationContext`, `Environment`, `ResourceLoader`, `ApplicationEventPublisher`, `MessageSource`. 이들 인터페이스와 `ConfigurableApplicationContext`나 `ResourcePatternResolver` 같은 확장 인터페이스는 별도의 설정 없이 자동으로 해석됩니다. 다음 예제는 `ApplicationContext` 객체를 자동 주입하는 방법을 보여줍니다:

```kotlin
class MovieRecommender {

	@Autowired
	lateinit var context: ApplicationContext

	// ...
}
```

`@Autowired`, `@Inject`, `@Value`, `@Resource` 애노테이션은 스프링의 `BeanPostProcessor` 구현에 의해 처리됩니다. 이는 이러한 애노테이션을 자체 `BeanPostProcessor`나 `BeanFactoryPostProcessor` 타입(있는 경우) 내에서 사용할 수 없음을 의미합니다. 이러한 타입은 XML이나 스프링의 `@Bean` 메서드를 사용하여 명시적으로 "연결"되어야 합니다.