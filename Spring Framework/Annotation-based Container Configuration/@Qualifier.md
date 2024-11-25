
```kotlin
class MovieRecommender {

	@Autowired
	@Qualifier("main")
	private lateinit var movieCatalog: MovieCatalog

	// ...
}
```

```kotlin
class MovieRecommender {

	private lateinit var movieCatalog: MovieCatalog

	private lateinit var customerPreferenceDao: CustomerPreferenceDao

	@Autowired
	fun prepare(@Qualifier("main") movieCatalog: MovieCatalog,
				customerPreferenceDao: CustomerPreferenceDao) {
		this.movieCatalog = movieCatalog
		this.customerPreferenceDao = customerPreferenceDao
	}

	// ...
}
```