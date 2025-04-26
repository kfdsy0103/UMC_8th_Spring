## Domain
- 애플리케이션에서 해결하고자 하는 문제의 영역


- 예시)
    - 쇼핑몰 애플리케이션 : 상품, 주문, 결제, 배송 등
    - 은행 애플리케이션 : 에금/출금, 대출, 계좌 등


- 도메인 모델
    - 도메인을 이해하기 위한 형태로 구조화한 것
    - 도메인 자체를 이해하는 것이 목적이다.


- 도메인 모델의 분류
    - 엔티티 : 고유한 식별자를 가지는 객체, 식별자만으로 동일성 판단
    - 값 객체 : 고유한 식별자가 없는 객체, 모든 속성 값으로 동일성 판단
    - 서비스 : 엔티티와 값 객체에 속하는 않는 비즈니스 로직을 수행하는 객체, 도메인 모델 간 상호작용을 도움
    - 리포지토리 : 도메인 객체를 저장하고 조회하는 역할을 수행하는 객체
  

- 주문 시스템의 도메인 구성 예시
    - Order(엔티티) : 주문 정보 관리
    - Item(엔티티) : 상품 정보
    - Address(값 객체) : Order에 속하는 속성 값
    - OrderService(서비스) : 주문의 생성 및 취소
    - OrderRepository(리포지토리) : 주문 저장 및 조회


- 하나의 모델을 각각 엔티티와 값 객체로 표현한 예제
```java
// 엔티티로 표현한 User
class User {
    
    private final String userId; // 고유 식별자로 사용
    private String name;
    private int age;

    // 생성자
    public User(String userId, String name, int age) {
        this.userId = userId;
        this.name = name;
        this.age = age;
    }
	
    // 동일성은 식별자(userId)를 기준으로 판단
    @Override
    public boolean equals(Object o) {
        // ...
        return userId.equals(user.userId);
    }
}
```
```java
// 값 객체로 표현한 User
final class User {
    
    private final String name;
    private final int age;

    // 생성자
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // 동일성 비교는 모든 속성 값을 기준으로 한다.
    @Override
    public boolean equals(Object o) {
        // ...
        return (age == user.age && name.equals(user.name));
    }
}
```

- 엔티티와 값 객체를 선택하는 기준
    - 고유한 식별자가 필요하다. -> 엔티티 (User, Order)
    - 상태가 바뀌어도 동일한 객체로 간주한다. -> 엔티티 (User의 개명)
    - 객체의 상태가 변하지 않는다.(final) -> 값 객체 (Address)
    - 독립적인 관리, 상태 변경, 추적이 필요하다면 엔티티


## 양방향 매핑
- 두 개체가 양방향 연관관계를 갖도록 설정하는 것


- 양방향 연관관계
    - 두 개체가 서로를 참조하고 있는 관계


- 관계형 데이터베이스의 양방향 매핑
    - 한 테이블이 다른 테이블에 대한 기본 키를 외래 키로 갖는다.
    - JOIN을 통해 서로 다른 테이블 간의 탐색이 가능해진다.


- 객체 지향 프로그래밍의 양방향 매핑
    - 서로 다른 두 객체가 각각을 인스턴스로 갖는다.
    - 즉 2번의 객체 참조로 양방향을 설정할 수 있다.


- 패러다임의 불일치
    - 관계형 데이터베이스에서는 하나의 테이블이 외래키를 관리한다.
    - 객체 지향 패러다임에서는 두 번의 참조로 양방향 관계를 구현한다.
    - 따라서 데이터의 무결성, 관계의 명확한 관리를 위해 한 엔티티만 외래키를 관리하도록 한다.


- '외래키를 관리한다'의 의미
    - DB 상에서 FK를 갖는다.
    - 실제 DB에 대한 저장, 수정, 삭제 권한을 갖는다.


- JPA 양방향 매핑 예시
```java
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "team_id")  // 외래 키 정의
    private Team team;
    
    // 생성자, Getter ...
    
    public void setTeam(Team team) {
        this.team = team;
    }
}
```
```java
@Entity
public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    // 생성자, Getter ...

    // 연관관계 편의 메서드는 자주 사용하는 엔티티에 위치하도록 한다.
    public void addMember(Member member) {
        members.add(member);
        member.setTeam(this);
    }
}
```


## N + 1 문제
- ORM 기술에서 엔티티 조회 시, 연관관계도 함께 조회하여 추가적인 쿼리가 나가는 문제


- 즉시로딩에서 N + 1 예제
```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    private long id;
    private String name;

    @OneToMany(fetch = FetchType.EAGER)
    private List<Member> members = new ArrayList<>();
}

```
```java
em.createQuery("select t from Team t", Team.class).getResultList(); // 패치 전략 무시
```


- 지연 로딩에서 N + 1 예제
```java
// @OneToMany(fetch = FetchType.Lazy)로 변경
List<Team> teams = teamRepository.findAll();

teams.stream().forEach(team -> {
        team.getMembers().size(); // 실제 사용 시점에 SELECT 쿼리가 나간다.
});
```


- 발생 원인
    - 즉시 로딩에서의 원인 : JPQL은 글로벌 패치 전략을 무시한 채 JPQL 그대로 조회하기 때문이다.
    - 지연 로딩에서의 원인 : 실제 엔티티가 사용되는 시점까지 조회를 미루기 때문이다.


- 해결 방법
    - 패치 조인 : @Query("select t from Team t join fetch t.members") - 1번의 쿼리로 연관된 객체도 로드
    - 엔티티 그래프 : @EntityGraph(attributePaths = {"members"}) - members 필드 Eager 조회
    - 배치 사이즈 : @BatchSize(size = 10) - IN 절 사용


- 각 연관관계의 default 속성
    - @ManyToOne : EAGER
    - @OneToOne : EAGER
    - @ManyToMany : LAZY
    - @OneToMany : LAZY