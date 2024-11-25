
• 기본적으로 **이름** 기반으로 의존성을 주입합니다. 주입 대상 필드나 메서드의 이름과 동일한 이름을 가진 빈을 찾아 주입합니다.
• name 속성을 명시하면 해당 이름의 빈을 찾고, 없으면 예외를 발생시킵니다.
• type 속성을 명시하면 해당 타입의 빈을 찾습니다.
• 이름이 우선적으로 고려되므로, 이름 충돌을 방지하거나 특정 빈을 지정할 때 유용합니다.

```kotlin
class SimpleMovieLister {

	@Resource(name="myMovieFinder")
	private lateinit var movieFinder:MovieFinder
}
```

```kotlin
class SimpleMovieLister {

	@set:Resource
	private lateinit var movieFinder: MovieFinder

}
```

### 결론

• @Resource는 **이름 기반 주입**을 우선하며, Java 표준 애노테이션입니다.
• @Autowired는 스프링의 고유 애노테이션으로, **타입 기반 주입**이 기본이며 더 많은 커스터마이징 옵션을 제공합니다.
• @Inject는 Java 표준 CDI 애노테이션으로, 스프링과 유사한 타입 기반 주입을 제공합니다.