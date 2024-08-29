
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


### 한계

• 페치 조인 대상에는 별칭을 줄 수 없다. (=> where 절에 별칭에 대한 조건으로 별칭 대상 개 수가 달라진다. => JPA 그래프 탐색에 이슈가 있을 수 있다.)
• 하이버네이트는 가능, 가급적 사용X
• 둘 이상의 컬렉션은 페치 조인 할 수 없다.
• 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.
• 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
• 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)

• 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화
• 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함
• @OneToMany(fetch = FetchType.LAZY) //글로벌 로딩 전략
• 실무에서 글로벌 로딩 전략은 모두 지연 로딩
• 최적화가 필요한 곳은 페치 조인 적용