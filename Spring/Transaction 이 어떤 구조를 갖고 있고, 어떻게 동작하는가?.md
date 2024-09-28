
## 1. 스프링 트랜잭션의 주요 개념

### 1-1. **트랜잭션 (Transaction)**

- **정의**: 데이터베이스에서 하나의 작업 단위를 의미하며, 여러 작업이 논리적으로 하나의 작업으로 묶여야 할 때 사용합니다. 트랜잭션은 모두 성공하거나 모두 실패해야 하며, 중간 상태는 없습니다.
- **ACID 속성**:
    - **Atomicity(원자성)**: 트랜잭션 내의 작업들이 모두 성공하거나 모두 실패해야 합니다.
    - **Consistency(일관성)**: 트랜잭션이 끝난 후 데이터는 일관성 있는 상태를 유지해야 합니다.
    - **Isolation(격리성)**: 동시에 실행되는 트랜잭션들이 서로 영향을 미치지 않아야 합니다.
    - **Durability(지속성)**: 트랜잭션이 완료된 후 그 결과는 영구적으로 반영되어야 합니다.

### 1-2. **트랜잭션 전파 (Transaction Propagation)**

- 트랜잭션이 어떻게 전파되는지를 결정하는 방식으로, 하나의 트랜잭션이 시작된 후 여러 계층에서 해당 트랜잭션을 어떻게 이어나갈지 정의합니다. 스프링은 다양한 전파 옵션을 제공합니다.

| Propagation 타입 | 설명  |
| -------------- | --- |
|`REQUIRED`|현재 트랜잭션이 있으면 사용하고, 없으면 새로 생성 (기본값)|
|`REQUIRES_NEW`|항상 새로운 트랜잭션을 생성|
|`SUPPORTS`|트랜잭션이 있으면 사용하고, 없으면 트랜잭션 없이 실행|
|`MANDATORY`|반드시 트랜잭션 내에서 실행, 없으면 예외 발생|
|`NOT_SUPPORTED`|트랜잭션 없이 실행|
|`NEVER`|트랜잭션이 있으면 예외 발생|
|`NESTED`|중첩 트랜잭션을 생성 (Savepoint를 지원하는 트랜잭션)|

### 1-3. **트랜잭션 격리 수준 (Transaction Isolation Level)**

- 트랜잭션 간의 상호작용을 어떻게 제어할지를 결정하는 방식으로, 동시에 여러 트랜잭션이 실행될 때 데이터 일관성을 유지하기 위해 사용합니다.


| 격리 수준     | 설명               |
| --------- | ---------------- |
| `DEFAULT` | 기본 DB의 격리 수준을 따름 |
|`READ_UNCOMMITTED`|트랜잭션이 커밋되지 않은 데이터를 다른 트랜잭션에서 읽을 수 있음 (가장 낮은 격리 수준)|
|`READ_COMMITTED`|커밋된 데이터만 읽을 수 있음 (Oracle의 기본값)|
|`REPEATABLE_READ`|트랜잭션이 완료될 때까지 데이터를 고정하여 재읽기 가능|
|`SERIALIZABLE`|가장 높은 격리 수준, 완전히 직렬화된 데이터 처리|

---

## 2. 스프링 트랜잭션 관리 구현 구조

스프링 트랜잭션 관리의 핵심은 **트랜잭션 추상화**에 있습니다. 스프링은 트랜잭션의 세부 구현을 추상화하여, 데이터 접근 기술(JDBC, JPA, Hibernate 등)에 구애받지 않고 트랜잭션을 일관되게 관리할 수 있도록 합니다. 이때 주로 사용되는 주요 컴포넌트는 다음과 같습니다.

### 2-1. **`PlatformTransactionManager`**

- 스프링에서 트랜잭션을 관리하는 핵심 인터페이스로, 구체적인 트랜잭션 관리 로직은 `PlatformTransactionManager` 구현체들이 담당합니다.
    
- **주요 구현체**:
    
    - `DataSourceTransactionManager`: JDBC 기반 트랜잭션 관리
    - `JpaTransactionManager`: JPA 기반 트랜잭션 관리
    - `HibernateTransactionManager`: Hibernate 기반 트랜잭션 관리

### 2-2. **`TransactionDefinition`**

- 트랜잭션 속성(전파, 격리 수준, 타임아웃, 읽기 전용 여부 등)을 정의하는 인터페이스입니다. 트랜잭션이 어떻게 처리될지를 세밀하게 설정할 수 있습니다.

### 2-3. **`TransactionStatus`**

- 트랜잭션의 현재 상태를 나타내는 객체로, 트랜잭션을 커밋할지 롤백할지를 결정하는 데 사용됩니다.

### 2-4. **`TransactionTemplate`**

- 프로그램 방식으로 트랜잭션을 관리할 때 사용하는 템플릿 클래스입니다. 트랜잭션 경계를 설정하고 명시적으로 트랜잭션을 처리할 때 유용합니다.

---

## 3. 스프링 선언적 트랜잭션 관리 동작 방식

스프링에서 트랜잭션 관리는 주로 선언적으로 이루어지며, 이를 위해 `@Transactional` 어노테이션을 사용합니다. 선언적 트랜잭션은 프록시 기반으로 동작하며, 다음과 같은 방식으로 구현됩니다.

### 3-1. **프록시 패턴**

- 스프링은 `@Transactional`이 적용된 메서드에 대해 **프록시 객체**를 생성합니다. 이 프록시 객체가 원본 객체의 메서드를 호출할 때, 트랜잭션을 시작하고 메서드 실행이 끝나면 트랜잭션을 커밋하거나 롤백합니다.

### 3-2. **동작 과정**

1. 클라이언트가 `@Transactional`이 선언된 메서드를 호출합니다.
2. 스프링은 프록시 객체를 통해 트랜잭션을 시작합니다.
3. 메서드가 실행됩니다.
4. 메서드 실행 중 예외가 발생하면, 트랜잭션을 롤백합니다.
5. 예외가 발생하지 않으면, 트랜잭션을 커밋합니다.

### 3-3. **프록시 객체의 역할**

- 트랜잭션 관리와 관련된 로직은 실제 메서드 호출 전후에 처리됩니다. 프록시 객체는 **AOP(Aspect-Oriented Programming)**를 기반으로 트랜잭션 전후 처리를 자동으로 수행합니다.

```java
@Service  
public class MyService {  
  
    @Transactional  
    public void someMethod() {  
        // 이 메서드는 트랜잭션 안에서 실행됨  
    }  
}
```

### 3-4. **예외 처리와 롤백**

- 스프링은 런타임 예외(`RuntimeException`)가 발생할 경우 자동으로 트랜잭션을 롤백합니다. 하지만 체크 예외(`CheckedException`)는 기본적으로 트랜잭션을 커밋합니다.
- 체크 예외 발생 시에도 롤백하려면 `rollbackFor` 속성을 명시해야 합니다.

```java
@Transactional(rollbackFor = Exception.class)  
public void someMethod() throws Exception {  
    // 체크 예외 발생 시에도 롤백  
}
```

---

## 4. 트랜잭션 관리 방법

### 4-1. **선언적 트랜잭션 관리**

- 주로 `@Transactional` 어노테이션을 사용하여 트랜잭션을 관리합니다. 메서드나 클래스에 붙여서 트랜잭션을 자동으로 처리하도록 할 수 있습니다.


```java
@Transactional  
public void process() {  
    // 트랜잭션 내에서 수행할 작업  
}
```

### 4-2. **프로그래밍 방식 트랜잭션 관리**

- `TransactionTemplate` 또는 `PlatformTransactionManager`를 사용하여 직접 트랜잭션을 제어합니다. 선언적 방식보다 유연하지만 코드가 복잡해질 수 있습니다.

```java
@Autowired  
private PlatformTransactionManager transactionManager;  
  
public void process() {  
    TransactionTemplate template = new TransactionTemplate(transactionManager);  
    template.execute(status -> {  
        // 트랜잭션 내에서 수행할 작업  
        return null;  
    });  
}
```

---

## 5. 트랜잭션 설정 커스터마이징

### 5-1. **전파 속성 변경**

- `@Transactional`의 `propagation` 속성을 사용하여 전파 전략을 설정할 수 있습니다.

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)  
public void newTransactionMethod() {  
    // 새로운 트랜잭션을 생성하여 실행  
}
```

### 5-2. **격리 수준 설정**

- 트랜잭션의 격리 수준은 여러 트랜잭션 간에 데이터 처리의 독립성을 설정하는 것으로, `@Transactional`의 `isolation` 속성을 통해 설정할 수 있습니다.

```java
@Transactional(isolation = Isolation.SERIALIZABLE)  
public void serializableTransactionMethod() {  
    // 가장 높은 격리 수준에서 트랜잭션 처리  
}
```
### 5-3. **트랜잭션 타임아웃 설정**

- 트랜잭션이 일정 시간 내에 완료되지 않으면 자동으로 롤백되도록 타임아웃을 설정할 수 있습니다. `timeout` 속성을 사용하여 초 단위로 지정합니다.

```java
@Transactional(timeout = 5)  // 5초 내에 트랜잭션을 완료해야 함  
public void timeoutTransactionMethod() {  
    // 5초 이상 걸리면 롤백  
}
```
### 5-4. **읽기 전용 트랜잭션**

- 데이터 수정이 필요 없는 트랜잭션의 경우, 읽기 전용 트랜잭션으로 설정하여 성능을 최적화할 수 있습니다. 데이터베이스에 따라 읽기 전용 트랜잭션은 캐시 최적화 등의 효과를 가져옵니다.

```java
@Transactional(readOnly = true)  
public List<MyEntity> getEntities() {  
    // 읽기 전용 트랜잭션으로 실행  
}
```

### 5-5. **체크 예외에 대한 롤백 설정**

- 기본적으로 체크 예외(`CheckedException`)는 트랜잭션을 롤백하지 않지만, 필요하다면 `rollbackFor` 또는 `noRollbackFor` 속성을 사용하여 예외 처리 전략을 세밀하게 조정할 수 있습니다.

```java
@Transactional(rollbackFor = IOException.class)  // 체크 예외에도 롤백  
public void checkedExceptionMethod() throws IOException {  
    // 예외 발생 시 롤백  
}
```

스프링에서 기본적으로 **체크 예외(Checked Exception)**에 대해 트랜잭션을 롤백하지 않는 이유는 체크 예외와 언체크 예외의 설계 철학과 관련이 있습니다.

체크 예외는 개발자가 예외 상황을 인지하고 이를 처리할 것을 기대합니다. 복구 가능성이 있는 체크 예외에서 무조건 트랜잭션을 롤백하는 것은 불필요한 자원 낭비가 될 수 있기 때문에, 스프링은 기본적으로 체크 예외에 대해서는 롤백하지 않고 커밋을 허용합니다.

## 6. 트랜잭션 관리의 주의점

### 6-1. **프록시와 자기 호출 문제**

- 스프링의 트랜잭션 관리가 프록시 기반으로 동작하기 때문에, 같은 클래스 내에서 `@Transactional`이 적용된 메서드가 다른 메서드를 호출하는 경우 트랜잭션이 적용되지 않는 문제가 발생할 수 있습니다. 이러한 문제를 해결하기 위해, 자기 호출(self-invocation) 대신 외부에서 서비스 메서드를 호출하는 방식으로 설계해야 합니다.
- 스프링의 AOP는 **프록시 객체**를 통해 트랜잭션을 관리하기 때문에, 원본 객체가 직접 자신의 메서드를 호출할 때는 프록시가 개입하지 않습니다. 즉, **자기 호출**의 경우 프록시가 메서드 호출을 가로채지 않기 때문에, 트랜잭션 관리 기능이 동작하지 않게 됩니다.

```java
// 트랜잭션이 적용되지 않음  
public void methodA() {  
    methodB();  // 내부 호출로 인해 트랜잭션 미적용  
}  
  
// 트랜잭션이 적용됨  
@Autowired  
private MyService myService;  
  
public void methodA() {  
    myService.methodB();  // 외부 호출로 인해 트랜잭션 적용  
}
```

### 6-2. **비공개(private) 메서드에 트랜잭션 적용 불가**

- 스프링의 트랜잭션은 프록시 객체를 통해 적용되므로, `private` 메서드에는 `@Transactional`을 적용할 수 없습니다. 프록시 객체는 원본 객체의 `public` 메서드만 가로챌 수 있기 때문에, 트랜잭션이 필요한 메서드는 반드시 `public`이어야 합니다.

### 6-3. **비동기 메서드와 트랜잭션**

- `@Async`를 사용하여 비동기로 실행되는 메서드는 트랜잭션 컨텍스트에서 벗어나게 됩니다. 비동기 메서드에 트랜잭션을 적용하려면 별도로 트랜잭션을 관리해야 하며, 이는 스프링의 비동기 작업 관리와 트랜잭션 관리 간의 주의가 필요합니다.

### 6-4. **롤백 조건 명확히 하기**

- 기본적으로 런타임 예외에 대해서만 롤백이 이루어지기 때문에, 개발자는 트랜잭션 롤백이 필요한 체크 예외나 특정 예외에 대해 명시적으로 롤백 설정을 추가해야 합니다.


## 7. 스프링 트랜잭션 관리의 장점

1. **데이터베이스 연동 기술에 독립적**: JDBC, JPA, Hibernate 등 다양한 기술에서 트랜잭션 관리를 일관되게 적용할 수 있습니다.
2. **선언적 트랜잭션 관리**: `@Transactional` 어노테이션을 사용해 코드의 명시적인 트랜잭션 처리 코드를 줄이고, 비즈니스 로직에 집중할 수 있습니다.
3. **유연한 전파 및 격리 설정**: 다양한 전파 수준과 격리 수준을 설정할 수 있어 복잡한 트랜잭션 관리 요구를 처리할 수 있습니다.
4. **AOP 기반 프록시 처리**: 핵심 비즈니스 로직에 트랜잭션 관리 로직을 분리하여 코드 중복을 방지하고 유지보수를 용이하게 합니다.

## 8. 스프링 트랜잭션의 실제 사용 예시

```java
@Service  
public class OrderService {  
  
    @Autowired  
    private OrderRepository orderRepository;  
  
    @Autowired  
    private PaymentService paymentService;  
  
    @Transactional  
    public void placeOrder(Order order, Payment payment) {  
        // 주문 저장 (트랜잭션 시작)  
        orderRepository.save(order);  
  
        // 결제 처리  
        paymentService.processPayment(payment);  
  
        // 트랜잭션 커밋  
    }  
}  
  
@Service  
public class PaymentService {  
  
    @Autowired  
    private PaymentRepository paymentRepository;  
  
    @Transactional(propagation = Propagation.REQUIRES_NEW)  
    public void processPayment(Payment payment) {  
        // 결제 저장 (새로운 트랜잭션)  
        paymentRepository.save(payment);  
  
        // 트랜잭션 커밋  
    }  
}
```

이 예제에서는 `OrderService`가 `PaymentService`를 호출하는 동안 동일한 트랜잭션 내에서 작업이 진행되며, 결제 부분은 새로운 트랜잭션에서 처리됩니다. 이를 통해 트랜잭션 전파 및 관리의 유연성을 보여줍니다.

---

### 결론

스프링 트랜잭션 관리는 트랜잭션의 복잡성을 추상화하고 선언적으로 간편하게 처리할 수 있도록 돕는 강력한 도구입니다. 이를 통해 비즈니스 로직과 데이터 일관성을 효과적으로 유지하며, 다양한 데이터 접근 기술에서 일관된 트랜잭션 처리 방식을 제공합니다.