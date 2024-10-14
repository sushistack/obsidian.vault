```java
import static org.mockito.Mockito.mockStatic;

// When  
try (MockedStatic<LocalDateTime> mocked = mockStatic(LocalDateTime.class)) {  
    mocked.when(LocalDateTime::now).thenReturn(now);  
  
    // Then  
    Assertions.assertThat(filter.apply(collection)).isEqualTo(expected);  
}
```
