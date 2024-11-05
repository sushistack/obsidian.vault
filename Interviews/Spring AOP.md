
```java
// MyServiceImpl.java
package com.example.service;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class MyServiceImpl implements MyService {

    @Override
    public void a() {
        System.out.println("Executing method a()");
        b(); // 같은 클래스 내에서 직접 메서드 호출
    }

    @Override
    @Transactional
    public void b() {
        System.out.println("Executing method b() with @Transactional");
        // 트랜잭션이 필요한 로직
    }
}
```

동적 프록시 적용 후, 

```java
@SpringBootApplication
@EnableAspectJAutoProxy
public class Application implements CommandLineRunner {

    @Autowired
    private MyService myService;

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("Bean 클래스: " + myService.getClass().getName());
        myService.a(); // 메서드 a() 호출
        myService.b(); // 메서드 b() 직접 호출
    }
}
```

```java
Bean 클래스: com.sun.proxy.$Proxy2
Before method: void com.example.service.MyService.a()
Executing method a()
Executing method b() with @Transactional
After method: void com.example.service.MyService.a()

// 위에는 b에 대한 AOP advice 가 실행되지 않는다.


Before method: void com.example.service.MyService.b()
Executing method b() with @Transactional
After method: void com.example.service.MyService.b()
```


```sh
1 Before method: void com.example.service.MyService.b()
2 Executing method b() with @Transactional
3 After method: void com.example.service.MyService.b()
```

`@Transcational` 이라면 1, 3번에서 트랜잭션 시작 및 커밋/롤백을 진행

--- 

CGLIB 의 경우는 바이트 코드를 조작하여 

