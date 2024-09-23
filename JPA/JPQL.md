
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

## 타입 표현

• 문자: ‘HELLO’, ‘She’’s’
• 숫자: 10L(Long), 10D(Double), 10F(Float)
• Boolean: TRUE, FALSE
• ENUM: jpabook.MemberType.Admin (패키지명 포함)
• 엔티티 타입: TYPE(m) = Member (상속 관계에서 사용)


![](www.inflearn.com_course_lecture_courseSlug=ORM-JPA-Basic&unitId=21719&tab=curriculum%20(1).png)

DTYPE 필요한 경우
DiscriminatorValue

```sql
select
	case when m.age <= 10 then '학생요금'
		when m.age >= 60 then '경로요금'
		else '일반요금'
	end
from Member m
```


COALESCE = 하나씩 조회해서 null이 아니면 반환
NULLIF = 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

```sql
select coalesce(m.username,'이름 없는 회원') from Member m
```

```sql
select NULLIF(m.username, '관리자') from Member m
```

### 함수

JPQL 표준함수

 • CONCAT

• SUBSTRING

• TRIM

• LOWER, UPPER

• LENGTH

• LOCATE = indexOf

• ABS, SQRT, MOD

• SIZE = 컬렉션 사이즈 반환, INDEX(JPA 용도)


### 커스텀

dialect 등록해야 함.

```sql
select function('group_concat', i.name) from Item i
```

### 경로 표현식

.(점)을 찍어 객체 그래프를 탐색하는 것

```sql
select m.username -> 상태 필드
from Member m
join m.team t -> 단일 값 연관 필드
join m.orders o -> 컬렉션 값 연관 필드
where t.name = '팀A'
```


• **상태** **필드**(state field): 단순히 값을 저장하기 위한 필드 (ex: m.username)
• **연관** **필드**(association field): 연관관계를 위한 필드
-  **단일** **값** **연관** **필드**: @ManyToOne, @OneToOne, 대상이 엔티티(ex: m.team)
-  **컬렉션** **값** **연관** **필드**: @OneToMany, @ManyToMany, 대상이 컬렉션(ex: m.orders)

```sql
SELECT m.team.name FROM Member m
```

내부조인 발생해서 조심해야 한다 
사용 하지마라

![](Pasted%20image%2020240828141225.png)

![](Pasted%20image%2020240828141237.png)

## Fetch Join (= 한방 쿼리)



```SQL
select m from Member m join fetch m.team
```

TO

```sql
SELECT M.*, T.* FROM MEMBER M

INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

명시적으로 한 번에 가져온다

EAGER와 비슷하지만, EAGER는 개발자의 의도 없다고 보는게 맞음...

```
회원1 조회 후, 팀(A, SQL)을 조회
회원2도 조회 후, 팀(A, 1차 캐시)을 조회한다면? -> 얘는 1차 캐시에서 조회 됨
```

100명 조회하면? -> 100번 쿼리가 날라간다. 

N + 1 문제는 fetch join으로 해결해야 한다.

```java
String jpql = "select m from Member m join fetch m.team";
List<Member> members = em.createQuery(jpql, Member.class)
.getResultList();

for (Member member : members) {
//페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩X
System.out.println("username = " + member.getUsername() + ", " +
"teamName = " + member.getTeam().name());
}
```


```sql
select t
from Team t join fetch t.members
where t.name = ‘팀A'
```


조인에 의한 중복 row가 발생할 수 있다.

![](Pasted%20image%2020240829094508.png)

![](Pasted%20image%2020240829094651.png)


### 페치 조인과 일반 조인의 차이

=> 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않음

• JPQL은 결과를 반환할 때 연관관계 고려X
• 단지 SELECT 절에 지정한 엔티티만 조회할 뿐
• 여기서는 팀 엔티티만 조회하고, 회원 엔티티는 조회X
• 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)
• 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념


### 한계와 특징

#### 한계

• 페치 조인 대상에는 별칭을 줄 수 없다. (=> where 절에 별칭에 대한 조건으로 별칭 대상 개 수가 달라진다. => JPA 그래프 탐색에 이슈가 있을 수 있다, JPA 입장에서 데이터 매칭이 안될 수도 있다.)
• 하이버네이트는 가능, 가급적 사용X
• 둘 이상의 컬렉션(members)은 페치 조인 할 수 없다. 
• 컬렉션(members)을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다. (쿼리도 다 긁어옴....)
• 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능 (collection에 @BatchSize(100) 을 사용하면? IN 쿼리로 하나씩 날리던것을 한번에 날린다.)
• 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)
• `COUNT` 쿼리에서 페치 조인(fetch join)을 사용할 수 없습니다. `COUNT` 쿼리는 결과로 단일 값(숫자)을 반환하기 때문에 엔티티 객체를 조회하는 것이 아니라 단순히 행의 개수를 세는 것이기 때문입니다.

#### 특징

• 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화
• 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함
• @OneToMany(fetch = FetchType.LAZY) //글로벌 로딩 전략
• 실무에서 글로벌 로딩 전략은 모두 지연 로딩
• 최적화가 필요한 곳은 페치 조인 적용



### 정리

• 모든 것을 페치 조인으로 해결할 수 는 없음
• 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적
	- 객체 그래프를 유지한다는 것은?
• 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른
결과를 내야 하면, 페치 조인 보다는 일반 조인을 사용하고 필요
한 데이터들만 조회해서 DTO로 반환하는 것이 효과적


### QnA

일반 조인 대상에도 별칭을 이용하여 필터하는 것은

위험하다고 이해하면 될까요?

일반 조인의 경우에는 이런 문제가 발생하지 않습니다. 페치 조인은 객체 그래프를 유지하면서 객체들을 함께 조회하기 때문에 문제가 됩니다.


테스트를 해보면 fetch join JPQL에 on 조건을 주면 무조건 에러가 발생합니다 (with-clause not allowed on fetched association)


!!! **애플리케이션에서 fetch join의 결과는 연관된 모든 엔티티가 있을것이라 가정하고 사용해야 합니다.**

그렇지 않으면? => 객체의 상태와 DB의 상태 일관성이 깨지는 것이지요.

**결론: fetch join의 대상은 on, where 등에서 필터링 조건으로 사용하면 안된다.**

\

예를 들어서 DB에 데이터가 다음과 같이 있습니다.

team1 - memberA

team1 - memberB

team1 - memberC

그런데 조인 대상의 필터링을 제공해서 조회결과가 memberA, memberB만 조회하게 되면 JPA 애플리케이션은 다음과 같은 결과로 조회됩니다.

team1에서 회원 데이터를 찾으면 memberA, memberB만 반환되는 것이지요.

이렇게 되면 JPA 입장에서 DB와 데이터 일관성이 깨지고, 최악의 경우에 memberC가 DB에서 삭제될 수도 있습니다.

왜냐하면 JPA의 엔티티 객체 그래프는 DB와 데이터 일관성을 유지해야 하기 때문입니다! 잘 생각해보면 우리가 엔티티의 값을 변경하면 DB에 반영이 되어버리지요.

---

**1. fetch join과 일반 조인의 차이를 설명해주세요. 예제를 가지고, 실제 실행되는 JPQL, SQL, 그리고 애플리케이션에서 객체가 가지고 있는 데이터를 기반으로 비교해서 설명해주세요.**

일반 join과 fetch join의 가장 큰 차이점은 엔티티를 조회 하는 시점입니다.
일반 join은 sql의 join과 동일 하게 동작 합니다.
fetch join은 jpql을 이용하여 조회 할 때, 연관된 엔티티를 한번에 같이 조회하는 기능입니다.
별칭을 줄수 없기 때문에 select, where, 서브쿼리에는 사용이 불가 합니다.
둘 이상의 컬렉션을 fetch join 할 수 없습니다.


## 다형성 쿼리

### Type

• 조회 대상을 특정 자식으로 한정
• 예) Item 중에 Book, Movie를 조회해라

• **[JPQL]**
```sql
select i from Item i
where type(i) IN (Book, Movie)
```


• **[SQL]**

```sql
select i from i
where i.DTYPE in (‘B’, ‘M’)
```

### TREAT

• 자바의 타입 캐스팅과 유사
• 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용
• FROM, WHERE, SELECT(하이버네이트 지원) 사용



• 예) 부모인 Item과 자식 Book이 있다.

• **[JPQL]**

```sql
select i from Item i
where treat(i as Book).author = ‘kim’
```


• **[SQL]**

```sql
select i.* from Item i
where i.DTYPE = ‘B’ and i.author = ‘kim’
```


## 엔티티 직접 사용

```sql
select count(m) from Member m
```

to

```sql
select count(m.id) as cnt from Member m
```

## Named 쿼리

• 미리 정의해서 이름을 부여해두고 사용하는 JPQL
• 정적 쿼리 (만 가능하다)
• 어노테이션, XML에 정의
• **애플리케이션 로딩 시점에 초기화 후 재사용**
• **애플리케이션** **로딩** **시점에** **쿼리를** **검증**


```java
@Entity

@NamedQuery(
name = "Member.findByUsername",
query="select m from Member m where m.username = :username")
public class Member {
...
}
```

@Query()  이게 Named 쿼리임

## 벌크 연산

변경된 데이터가 100건이라면 100번의 UPDATE SQL 실행

• 쿼리 한 번으로 여러 테이블 로우 변경(엔티티)
• **executeUpdate()****의** **결과는** **영향받은** **엔티티** **수** **반환**
• **UPDATE, DELETE** **지원**
• **INSERT(insert into .. select,** **하이버네이트** **지원****)**


벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리
• 벌크 연산을 먼저 실행
• **벌크** **연산** **수행** **후** **영속성** **컨텍스트** **초기화**


```
String qlString = "update Product p " +

"set p.price = p.price * 1.1 " +

"where p.stockAmount < :stockAmount";

int resultCount = em.createQuery(qlString)

.setParameter("stockAmount", 10)

.executeUpdate();
```

@Modifying => ??