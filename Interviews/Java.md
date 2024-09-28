### 1. **Java 메모리 관리와 Garbage Collection (GC)에 대해 설명하고, GC 튜닝 방법에 대해 이야기해보세요.**

#### 이 질문의 목적:

- Java 메모리 모델에 대한 깊이 있는 이해를 확인.
- Heap, Stack, Young/Old Generation, PermGen(Metaspace) 등의 구조적 이해도를 평가.
- Garbage Collection 프로세스가 어떻게 동작하는지(Stop-the-world, Minor/Major GC, CMS, G1 GC 등)에 대한 지식 확인.
- 애플리케이션의 성능 최적화를 위해 GC 튜닝 방법을 알고 있는지 평가.

#### 추가 질문 예시:

- **GC 동작을 모니터링하고 튜닝한 경험**에 대해 구체적으로 설명할 수 있나요?
- 어떤 **GC 알고리즘을 사용**했고, 어떻게 애플리케이션에 맞게 조정했나요?
- **OutOfMemoryError**가 발생할 때 이를 해결하기 위한 방법은 무엇인가요?

#### 답변 기대치:

- 메모리 구조에 대한 설명(Heap, Stack, Young/Old Gen 등).
- 기본적인 GC 알고리즘(Serial, Parallel, CMS, G1 GC)에 대한 설명과 각각의 장단점.
- GC 튜닝 시 사용하는 JVM 옵션들(`-Xms`, `-Xmx`, `-XX:+UseG1GC` 등) 및 성능 문제 해결 경험.


### 2. **멀티스레드 환경에서 발생할 수 있는 문제(예: 데드락, 레이스 컨디션)를 설명하고, 이를 방지하거나 해결하는 방법은 무엇인가요?**

#### 이 질문의 목적:

- 멀티스레드 프로그래밍의 복잡성에 대한 이해도를 평가.
- 동시성 문제(데드락, 레이스 컨디션, 리소스 경합 등)에 대한 경험과 해결 능력을 확인.
- **synchronized**, **ReentrantLock**, **volatile**, **ThreadPool** 등 자바 동시성 도구를 적절히 사용할 수 있는지 확인.
- 멀티스레드 프로그래밍에서의 성능 최적화 경험을 평가.

#### 추가 질문 예시:

- 동기화(synchronization)를 과도하게 사용했을 때 발생할 수 있는 성능 문제는 무엇인가요?
- Java에서 **ThreadPool**을 사용하여 멀티스레딩을 최적화한 경험이 있나요?
- **Lock-Free 알고리즘**을 구현해본 적이 있나요?

#### 답변 기대치:

- 데드락(Deadlock), 레이스 컨디션(Race Condition), 라이브락(Livelock) 등의 문제를 설명하고 이를 해결하는 방법(동기화, 락, Atomic 클래스, 비동기 처리 등).
- 자바에서 동시성 문제를 해결할 때 사용할 수 있는 다양한 방법들(Ex: `synchronized`, `ReentrantLock`, `volatile`, `java.util.concurrent` 패키지 사용).
- 실제 프로젝트에서 멀티스레드 문제를 해결한 사례 공유.