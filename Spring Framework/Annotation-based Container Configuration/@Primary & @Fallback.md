
```kotlin
@Configuration
class MovieConfiguration {

	@Bean
	@Primary
	fun firstMovieCatalog(): MovieCatalog { ... }

	@Bean
	fun secondMovieCatalog(): MovieCatalog { ... }

	// ...
}
```

```kotlin
@Configuration
class MovieConfiguration {

	@Bean
	fun firstMovieCatalog(): MovieCatalog { ... }

	@Bean
	@Fallback
	fun secondMovieCatalog(): MovieCatalog { ... }

	// ...
}
```