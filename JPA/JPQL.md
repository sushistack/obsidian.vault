
![](Pasted%20image%2020240828125459.png)

![](Pasted%20image%2020240828125509.png)

## TypeQuery, Query

![](Pasted%20image%2020240828125621.png)


![](www.inflearn.com_course_lecture_courseSlug=ORM-JPA-Basic&unitId=21719&tab=curriculum.png)


![](Pasted%20image%2020240828125922.png)


Spring Data JPA -> 처리 해놔서 Optional or null 

![](Pasted%20image%2020240828130658.png)


## 프로젝션

SELECT 의 대상 컬럼 지정하기

### Entity 프로젝션

```sql
SELECT m FROM Member m
```

em.createQuery

날린 쿼리에 의한 Entity 들은 영속성에 포함 됨.

실제 나가는 쿼리와 명시적으로 작성한 쿼리가 일치하는 것이 좋다.

### 임베디드 프로젝션

```java
em.createQuery("SELECT o.address FROM Order o", Address.class)
```

어딘가에 속해 있는 값 타입(Address)의 경우는 소속(Order)을 표시해야한다.

### 스칼라

```java
em.createQuery("SELECT DISTINCT m.username, m.age FROM Member m");
```


## 여러 값 조회

### Query 타입

```java
List resultList = em.createQuery("SELECT DISTINCT m.username, m.age FROM Member m");
Object o = resultList.get(0);
```


### Object[] 타입

```java
List<Object[]> resultList = em.createQuery()
```

### New

```sql
SELECT new com.test.model.UserDTO(m.username, m.age) FROM
```

## 페이징 API


![](Pasted%20image%2020240828132153.png)

```java
em.createQuery()
	.setFirstResult(1)
	.getMaxResults(10)
	.getResultList();
```

## 조인


![](Pasted%20image%2020240828132553.png)

세타 조인 = 연관 관계 없을 때 사용

### ON 절

1. 조인 대상 필터링
2. 연관관계 없는 엔티티 외부 조인(하이버네이트 5.1부터)


```sql
SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'
```

## 서브쿼리


![](Pasted%20image%2020240828133231.png)

### 한계

![](Pasted%20image%2020240828133309.png)

![](Pasted%20image%2020240828133331.png)

