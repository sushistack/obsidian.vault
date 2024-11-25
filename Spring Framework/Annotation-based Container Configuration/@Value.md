일반적으로 외부 설정들을 주입할 때 사용한다.

```kotlin
@Component
class MovieRecommender(@Value("\${catalog.name}") private val catalog: String)
```

