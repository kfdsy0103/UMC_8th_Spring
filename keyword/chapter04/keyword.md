## DI
- 외부에서 객체의 의존성을 주입해주는 설계 패턴
- DI는 IoC(제어의 역전) 원칙을 적용한 기법으로 소프트웨어 구조의 유연성과 확장의 용이성을 확보
    - 개발자가 아닌 스프링 컨테이너가 객체의 생성과 주입, 라이프사이클을 관리


- DI의 목적
    - 객체간의 결합도를 줄이기 위함
    - 객체를 외부에서 주입하기에 테스트에 용이하다.
    - 코드 수정 없이 사용할 객체를 교체할 수 있어 유지보수성이 증가한다.


- DI의 방식
    - 생성자 주입 : 생성자를 통해 의존성을 주입하는 방식
    - 세터 주입 : setter를 통해 의존성을 주입하는 방식
    - 필드 주입 : 직접 필드에 의존성을 주입하는 방식


- 생성자 주입
```java
@Service
public class Service {
    private final Repository repository;    // Repository를 스프링이 생성자를 통해 주입
    
    @Autowired
    public Service(Repository repository) {
        this.repository = repository;
    }
}
```

- 세터 주입
```java
@Service
public class Service {
    private Repository repository;  // 1. 빈 생성 -> 2. setter로 주입
    
    @Autowired
    public void setRepository(Repository repository) {
        this.repository = repository;
    }
}
```

- 필드 주입
```java
@Service
public class Service {
    @Autowired
    private Repository repository;  // 1. 빈 생성 -> 2. 리플렉션 기술로 주입
}
```

- 생성자 주입이 권장되는 이유
    - final 키워드로 객체의 불변성을 확보할 수 있다.
    - 다른 방식들은 순수 자바 코드로 테스트가 어렵다.
    - Lombok 과의 결합(@RequiredArgsConstructor)으로 코드가 간결해진다.
    - 순환 참조 에러를 컴파일 시점에 방지할 수 있다.


- 순환 참조 예시
```java
// A가 B를 의존
@Component
public class ClassA {
  private final ClassB classB;

  @Autowired
  public ClassA(ClassB classB) {
    this.classB = classB;
  }
}

// B도 A를 의존
@Component
public class ClassB {
  private final ClassA classA;

  @Autowired
  public ClassB(ClassA classA) {
    this.classA = classA;
  }
}
```

## IoC
- 제어의 역전, 개발자가 아닌 프레임워크가 프로그램의 제어권을 관리하는 개념

- 스프링은 ApplicationContext라는 컨테이너와 함께 DI로 IoC를 구현한다.
    - 객체 생성 및 관리
    - DI를 통한 의존성 관리
    - 프레임워크에 객체의 라이프사이클 위임

## 프레임워크와 API의 차이
- 프레임워크 : 개발 구조와 규칙을 제공하는 소프트웨어 환경
- API : 특정 기능을 수행하는 프로그램 간 상호작용 인터페이스


- 차이점
    - 프레임워크는 개발자가 정해진 구조와 규칙에 따라 코드를 작성하도록 하고, 프레임워크가 제어권을 가진다.
    - 반면, API는 개발자가 필요에 따라 원하는 방식으로 자유롭게 사용한다.

## AOP
- 관점 지향 프로그래밍, 어떠한 로직을 핵심 기능과 부가 기능으로 나누고 각각을 모듈화하는 것


- 장점
    - 로깅, 트랜잭션 처리와 같이 중복되는 코드를 제거할 수 있다.
    - 비즈니스 로직에 집중할 수 있다.


- AOP 적용 방법
    - 컴파일 시점: AspectJ 컴파일러가 .java 컴파일 시 부가 기능을 추가 해 .class로 컴파일
    - 클래스 로딩 시점 : JVM 내 클래스 로더에 .class를 올리는 시점에 바이트코드를 조작
    - 런타임 시점 : 프록시를 통해 부가 기능 적용


- 스프링 AOP가 프록시 방식으로 동작하는 이유
    - 다른 방식은 별도의 컴파일러가 필요하고, 클래스로더 조작은 사용과정이 어렵다.
    - 프록시가 없다면 실제 Target 코드 내부에 부가기능 호출 로직이 포함된다. 이는 유지보수성을 크게 떨어뜨린다.


- 스프링 AOP 적용 예시 (@Transactional)
```java
// method() 호출 시 UserService 프록시가 요청을 가로채서 try-catch, commit, rollback ... 트랜잭션 실행
@Service
public class UserService {
  @Autowired
  private UserRepository userRepository;

  @Transactional
  public void method(String userName) {
    User user = new User();
    user.setName(userName);
    userRepository.save(user);  // 사용자 저장
  }
}
```

## 서블릿
- 요청에 대한 동적인 응답을 생성하는 자바 기반의 웹 기술
- 서블릿은 톰캣과 같은 서블릿 컨테이너에서 생성 및 관리된다.


- 동작 과정
    - 클라이언트가 URL을 입력하면 HTTP Request가 서블릿 컨테이너에 전송됨
    - 서블릿 컨테이너는 HttpServletRequest, HttpServletResponse 객체를 생성
    - URL이 어느 서블릿에 매핑되는지 확인한 뒤, 해당 서블릿의 service() 메서드 호출
    - 동적인 응답을 생성하여 HttpServletResponse에 담아 HTTP Response를 보냄
    - HTTPServletRequest, HTTPServletResponse 소멸


- 서블릿 컨테이너
    - 서블릿을 관리해주는 컨테이너 역할
    - 웹 서버와 소켓으로 통신한다.
    - 대표적으로 아파치 톰캣을 사용한다.


- 스프링의 DispatcherServlet
    - 스프링 MVC의 중심이 되는 서블릿
    - 프론트 컨트롤러 패턴
        - 애플리케이션으로 오는 모든 요청을 앞단에서 핸들링하고 공통작업을 처리해준다.


- DispatcherServlet의 핵심 메서드 요약
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {

	ModelAndView mv = null;
	Exception dispatchException = null;

	try {
		// 1. 핸들러 조회
		mappedHandler = getHandler(processedRequest);
		if (mappedHandler == null) {
			noHandlerFound(processedRequest, response);
			return;
		}

		// 2. 핸들러 어댑터 조회
		HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

		// 3. 핸들러 어댑터를 통해 핸들러 실행, ModelAndView 반환
		mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
	}

	// 4. 내부적으로 뷰 리졸버를 통해 랜더링
	processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
```