### 1. **Kotlin의 `coroutine`과 Java의 `Thread` 또는 `CompletableFuture`를 비교하고, `coroutine`이 비동기 처리에서 가지는 장점에 대해 설명해보세요. 실제로 프로젝트에서 어떻게 활용했나요?**

#### 이 질문의 목적:

- **Kotlin의 비동기 처리** 방식인 **coroutine**에 대한 깊이 있는 이해를 평가.
- Java의 기존 비동기 처리 방식과 비교하여, **코루틴의 경량성**, **구조적 동시성** 및 **성능 최적화**에 대한 이해도를 확인.
- 코루틴을 실무에서 어떻게 사용했는지에 대한 실제 경험을 통해 문제 해결 능력을 평가.

#### 추가 질문 예시:

- 코루틴의 **suspend** 함수와 **비동기 작업**이 어떻게 연결되나요?
- Java에서 **CompletableFuture**를 사용하는 방식과 코루틴을 사용하는 방식 간의 **성능 차이**는 어떤가요?
- 프로젝트에서 코루틴을 사용하여 **성능 개선**이나 **자원 절약**을 이루어낸 경험이 있나요?

#### 답변 기대치:

- 코루틴의 **비동기 프로그래밍** 패턴, 특히 **suspend** 함수와 **코루틴 빌더**(`launch`, `async` 등) 설명.
- **경량 스레드**로서의 코루틴이 **스레드 풀의 오버헤드**를 피하고 **비동기 작업을 더 효율적으로** 처리하는 방법.
- 구조적 동시성(Structured Concurrency)의 장점, 코루틴의 취소 및 예외 처리에 대한 설명.
- Java의 `CompletableFuture`와의 비교, 실무에서의 활용 경험을 통한 성능 개선 사례.
### 2. **Kotlin의 `null safety`와 Java의 `Optional`을 비교 설명하고, Kotlin의 `nullable` 처리 방식이 시스템 안정성에 어떻게 기여할 수 있는지 설명해보세요.**

#### 이 질문의 목적:

- Kotlin의 **null safety** 메커니즘을 얼마나 깊이 이해하고 있는지 평가.
- Java의 `Optional`과의 차이점을 설명하고, 두 언어의 **null 처리 방식**에 대한 비교를 통해 문제 해결 능력 및 코드 안정성에 대한 이해도를 확인.
- **실무에서의 null 처리 전략**을 통해 시스템 안정성과 오류 예방에 어떻게 기여했는지 평가.

#### 추가 질문 예시:

- Kotlin에서 `!!` 연산자 사용의 **문제점**과 이를 피하기 위한 **best practice**는 무엇인가요?
- Kotlin의 **엘비스 연산자**(`?:`)와 Java의 `Optional`의 **orElse** 메서드를 어떻게 비교할 수 있나요?
- 실제로 null 처리 관련 버그를 Kotlin의 **null safety**로 해결한 경험이 있나요?

#### 답변 기대치:

- Kotlin의 **nullable 타입**(`String?`)과 **non-null 타입**의 차이점 설명.
- `?.`, `?:`, `!!` 등의 연산자와 `let`, `apply` 등의 함수형 스타일을 활용한 null 처리 방법 설명.
- Java의 `Optional`과의 비교를 통해 Kotlin의 더 직관적이고 안전한 null 처리 방식 설명.
- 실무에서 null 관련 문제를 Kotlin의 **null safety**로 예방하거나 해결한 경험 공유.