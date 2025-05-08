## 지연로딩과 즉시로딩의 차이
- 지연로딩 : 엔티티 조회 시, 연관된 엔티티는 실제 사용하는 시점에 조회하는 방식
- 즉시로딩 : 엔티티 조회 시, 연관된 엔티티를 함께 조회하는 방식


- 즉시로딩 예시
```java
public class Member {
    @ManyToOne(fetch = FetchType.EAGER)
    private Team team;
}

Member member = em.find(Member.class, 1L); // team도 join으로 함께 조회한다.
```
- 즉시로딩 장점 : 데이터를 한 번에 로딩하기 때문에 성능이 향상된다.
- 즉시로딩 단점 : 메모리 낭비, 필요하지 않은 데이터도 가져온다.


- 지연로딩 예시
```java
public class Member {
    @ManyToOne(fetch = FetchType.Lazy)
    private Team team;
}

Member member = em.find(Member.class, 1L);
team1.getTeam().getName(); // 실제 target이 사용되는 시점에 조회 쿼리가 나간다.
```
- 지연로딩 장점 : 필요한 데이터만 로딩하기 때문에 메모리 사용량이 줄어든다.
- 지연로딩 단점 : 사용하는 연관 객체가 많다면 성능 저하가 발생할 수 있다.


## Fetch Join
- JPQL에서 연관된 엔티티나 컬렉션을 SQL 한번에 함께 조회하는 기능


- 사용 예시
```java
// members가 LAZY여도 로딩하여 성능이 향상된다.
@Query("SELECT DISTINCT t FROM Team t JOIN FETCH t.members WHERE t.id = :id")
Team findTeamByMemberId(@Param("id") Long id);
```


- 일반 조인과 패치조인의 차이
    - 일반 조인은 연관 엔티티에 조인을 걸어도 SELECT 되지 않는다.
    - 패치 조인은 조인이 걸린 연관 엔티티도 함께 SELECT 하여 객체 그래프를 로드한다.
```java
@Query("SELECT distinct t FROM Team t JOIN t.members") // 팀 관련 컬럼만 가져온다.

@Query("SELECT distinct t FROM Team t JOIN FETCH t.members") // 팀과 멤버 컬럼을 가져와 영속화한다.
```



- 단점
    - 조인이다 보니, 1:N에서 중복이 발생하고 페이징 API 사용이 불가능하다.
    - 두 개 이상의 컬렉션을 패치 조인 X
    - 쿼리문을 직접 작성해야 한다.


## @EntityGraph
- 연관된 엔티티를 즉시로딩하는 어노테이션
- 패치 조인의 어노테이션 버전이라고 생각하면 된다.


- 사용 예시
```java
// 공통 메서드 + @EntityGraph
@Override
@EntityGraph(attributePaths = {"members"})
List<Team> findAll();

// 쿼리 메서드 + @EntityGraph
@EntityGraph(attributePaths = {"members"})
@Query("SELECT t FROM Team t WHERE t.name = :name")
List<Team> findByName(@Param("name") String name);
```


- 패치 조인과의 차이점
    - 패치조인은 inner join, @EntityGraph는 left outer join으로 동작한다.
    - attributePaths에 필드를 여러 개 지정할 수 있다.


## JPQL
- JPA에서 제공하는 객체 지향 쿼리 언어
- 테이블이 아닌 엔티티 객체를 대상으로 쿼리를 작성할 수 있다.


- 특징
    - 엔티티의 이름과 필드는 대소문자를 구분한다.
    - 엔티티의 별칭은 필수적으로 명시해야한다.(as 생략)


- JPQL 예시
```java
// Member Entity 대상 select
@Query("SELECT m FROM Member m")

// 객체의 필드에 접근하듯이 사용
@Query("SELECT m FROM Member m WHERE m.name = :name")

// 이름 기준 파라미터 바인딩
String jpql = "select m from Member m where m.name = :name";
TypedQuery<Member> query = em.createQuery(jpql, Book.class);
query.setParameter("name", param);

// DTO 조회 가능. 단 패키지명을 모두 명시해야 한다.
String jpql = "select new com.example.MemberDto(m.name, m.age) from Member m";
TypedQuery<MemberDto> query = em.createQuery(jpql, MemberDto.class);
```


- JPQL vs SQL
    - 조회 대상 : 엔티티 / 테이블
    - 필드 : 엔티티의 필드 / 테이블의 컬럼
    - 실행 과정 : JPA가 JPQL -> SQL 변환 / SQL 그대로 실행


## QueryDSL
- SQL/JPQL 쿼리를 안전하게 생성 및 관리해주는 프레임워크이다.


- 기존 JPQL의 문제점
    - 쿼리를 문자열로 입력하여 관리하는데 어려움이 있다.
    - 실제로 쿼리를 실행하기 전까지 오류를 확인하기 어렵다.
    - 동적 쿼리 작성이 어렵다.


- QueryDSL의 장점
    - 문자가 아닌 코드로 쿼리를 작성하여 컴파일 시점에 문법 오류를 확인한다.
    - 복잡한 쿼리와 동적 쿼리 작성이 편리해진다.


- JPQL을 QueryDSL로 리팩토링한 예시
```java
String jpql = "select * from Member m join Point p on p.member_id = m.id"
List<Member> result = em.createQuery(jpql, Member.class).getResultList();
```
```java
return jpaQueryFactory
        .from(member)
        .join(member.point, point)
        .fetch();
```


- 동적 쿼리 예시
```java
private final QMember member;

public List<Member> findMembers(String name, Integer minAge) {
        BooleanBuilder builder = new BooleanBuilder();

        // BooleanExpression으로 동적 조건 추가
        builder.and(nameEq(name));
        builder.and(ageGoe(minAge));

        return queryFactory
            .selectFrom(member)
            .where(builder)
            .fetch();
}

// 도시 조건
private BooleanExpression cityEq(String city) {
  return city != null ? member.city.eq(city) : null;
}

// 최소 나이 조건
private BooleanExpression ageGoe(Integer minAge) {
  return minAge != null ? member.age.goe(minAge) : null;
}
```


## N+1 문제 해결 방법
- BatchSize 사용
    - WHERE 절이 동일한 여러 개의 SELECT 쿼리를 하나의 IN 쿼리로 만들어준다.
    - size로 분할되는 만큼 SELECT 쿼리를 따로 날린다.


- BatchSize 예시
```java
@Entity
public class Parent {
    @BatchSize(size = 100)
    @OneToMany(mappedBy = "parent")
    private List<Child> children = new ArrayList<>();
}
```
```mysql-sql
// 적용 전
SELECT * FROM child WHERE child.parent_id = 1
SELECT * FROM child WHERE child.parent_id = 2
...
SELECT * FROM child WHERE child.parent_id = 100

// 적용 후
SELECT * FROM child WHERE child.parent IN (1, 2, ... ,100)
```


- Fetch Join 사용
    - 단일 SQL 쿼리로 연관된 엔티티를 함께 조회
    - 페이징 X, 모든 데이터를 메모리에 올려 JPA가 따로 페이징을 한다.
    - 둘 이상의 컬렉션을 한꺼번에 패치 조인 X


- Fetch Join 예시
```java
@Query("SELECT t FROM Team t JOIN FETCH t.members")
List<Team> findAllWithMembers();
```


- @EntityGraph 사용
    - fetch join과 유사한 기능을 한다.
    - 어노테이션으로 적용할 수 있어 쿼리를 직접 작성하지 않아도 된다.


- @EntityGraph 예시
```java
@EntityGraph(attributePaths = "members")
Page<Team> findByName(String name, Pageable pageable);
```