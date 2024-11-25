
`CommonAnnotationBeanPostProcessor`는 `@Resource` 애노테이션뿐만 아니라 JSR-250 라이프사이클 애노테이션(`jakarta.annotation.PostConstruct`, `jakarta.annotation.PreDestroy`)도 인식합니다. 스프링 2.5에서 도입된 이러한 애노테이션 지원은 초기화 콜백 및 소멸 콜백에서 설명된 라이프사이클 콜백 메커니즘에 대한 대안을 제공합니다. 

`CommonAnnotationBeanPostProcessor`가 스프링 `ApplicationContext`에 등록되어 있는 경우, 이러한 애노테이션을 포함하는 메서드는 해당 스프링 라이프사이클 인터페이스 메서드나 명시적으로 선언된 콜백 메서드와 동일한 라이프사이클 시점에서 호출됩니다. 다음 예제에서는 캐시가 초기화 시점에 미리 채워지고, 소멸 시점에 비워지는 것을 보여줍니다:

```kotlin
class CachingMovieLister {

	@PostConstruct
	fun populateMovieCache() {
		// populates the movie cache upon initialization...
	}

	@PreDestroy
	fun clearMovieCache() {
		// clears the movie cache upon destruction...
	}
}
```

**@PostConstruct**

  

• **역할**: 빈 초기화 작업을 수행합니다.
• **호출 시점**: 빈이 스프링 컨테이너에 의해 생성되고, 모든 의존성 주입이 완료된 직후에 호출됩니다.
• **주요 용도**:
• 리소스를 초기화하거나 설정을 적용.
• 의존성 주입 이후 필요한 추가 작업 수행.
• 예를 들어, 데이터베이스 연결을 설정하거나 캐시를 채우는 작업.


**@PreDestroy**

• **역할**: 빈 소멸 전 정리 작업을 수행합니다.
• **호출 시점**: 스프링 컨테이너가 애플리케이션 종료 과정에서 빈을 제거하기 직전에 호출됩니다.
• **주요 용도**:
	• 사용 중인 리소스 해제 (예: 파일 닫기, 데이터베이스 연결 종료).
	• 스레드 중지 또는 로그 기록과 같은 정리 작업.
	• 예를 들어, 캐시 데이터를 비우거나 임시 파일을 삭제하는 작업.