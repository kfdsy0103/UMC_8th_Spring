## RestControllerAdvice
- 컨트롤러에서 발생하는 예외를 전역적으로 처리해주는 어노테이션
- @RestControllerAdvice = @ControllerAdvice + @ResponseBody

- @ControllerAdvice와의 차이점
    - @ControllerAdvice는 리턴 값을 통해 HTML 뷰를 랜더링한다.
    - @RestControllerAdvice는 리턴 값을 body에 넣어 클라이언트에게 바로 전달한다.
    - 이러한 차이는 @ResponseBody로 인해 발생하는 것이다.

- 따라서 RESTful API를 만든다면 JSON 형식으로 반환이 가능한 @RestControllerAdvice를 사용한다.

- @RestControllerAdvice 사용 예시
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // CustomException 처리
    @ExceptionHandler(CustomException.class) 
    public ResponseEntity<String> handleIllegalArgumentException(CustomException e) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
    }
}
```

- 어노테이션의 활용
    - 특정 패키지에만 적용하기
```java
@ControllerAdvice(basePackages = "com.example.controller")
```
- 특정 클래스에만 적용하기
```java
@RestControllerAdvice(assignableTypes = {UserController.class, OrderController.class})
```
- 우선 순위
```java
@Order(1)
@RestControllerAdvice
public class HighPriorityExceptionHandler {
// 예외를 우선적으로 처리한다.
}

@Order(2)
@RestControllerAdvice
public class LowPriorityExceptionHandler {
// 앞에서 처리되지 않거나 정의되지 않은 예외를 처리한다.
}
```

- @RestControllerAdvice의 장점
    - 모든 컨트롤러의 예외 처리를 하나의 클래스로 처리할 수 있다.
    - 일관성있는 에러 응답을 보낼 수 있다.
    - try-catch를 쓰지 않아 코드의 가독성이 향상되고, 수정이 용이해진다.

- 주의점
    - ControllerAdvice를 여러 개 등록하면 스프링은 임의의 순서로 에러를 처리할 수 있다.
    - @Order로 순서를 처리할 수 있으나, 일관된 예외처리를 위해 하나의 프로젝트에는 하나의 @ControllerAdvice를 권장한다.
    - 여러 개의 ControllerAdvice가 필요하다면 속성값을 통해 구체적으로 지정한다.
    - 직접 구현한 Exception 클래스는 한 공간에서 관리하는 것이 좋다.

- 하나의 ControllerAdvice가 권장되는 이유
    - 여러 개를 사용하면, basePackageClasses와 같은 설정을 통해 적용 위치를 탐색하는 연산이 필요하다.
    - 이 과정은 런타임에 수행되므로, 많아질수록 성능에 영향을 미칠 수 있다.

## lombok
- getter, setter와 같이 반복적으로 작성해야하는 코드를 자동 생성해주는 라이브러리

- 장점
    - 반복적인 코드 작성이 줄어들어 코드가 간결해지고 개발 시간이 단축된다.

- 단점
    - 컴파일 시점에 코드를 생성해주기 때문에 개발환경에서 추가적인 설정이 필요하다.
    - Lombok 라이브러리에 대한 종속성이 생긴다.

- @Getter, @Setter
```java
// getName, setName, getAge, setAge 자동생성
@Getter
@Setter
public class User {
    private String name;
    private int age;
}
```

- @ToString
```java
// 출력 형식을 생성해준다.
@ToString
public class User {
    private String name;
    private int age;
}

User(name=Alice, age=25, email=alice@example.com)
```

- @EqualsAndHashCode
```java
// 객체의 논리적 동등성 비교를 위한 equals(), hashCode() 생성
@EqualsAndHashCode
public class User {
    private String name;
    private int age;
}
```
- @Data
```java
// @Data는 다음 어노테이션들이 모두 합쳐진 것이다.
// @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor
@Data
public class User {
    private String name;
    private int age;
}
```

- @NoArgsConstructor
    - 파라미터가 없는 기본 생성자

- @AllArgsConstructor
    - 모든 필드를 받는 생성자
    - 주로 final 키워드를 필드에 붙이고 함께 사용한다.

- @RequiredArgsConstructor
    - final 필드, @NonNull 필드만 파라미터로 받는 생성자

- @Builder
```java
// 객체를 빌더 패턴으로 생성하도록 해준다.
@Builder
public class User {
    private String name;
    private int age;
}

// 사용 예시
User user = User.builder()
                .name("Alice")
                .age(25)
                .build();
```

- 생성된 빌더 코드
```java
public class User {
    private String name;
    private int age;

    // private 생성자
    private User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public static UserBuilder builder() {
        return new UserBuilder();
    }

    public static class UserBuilder {
        private String name;
        private int age;

        public UserBuilder name(String name) {
            this.name = name;
            return this;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public User build() {
            return new User(name, age);
        }
    }
}
```

- @Builder 주의사항
    - 필드에 대한 초기값을 지정하지 않으면 타입에 따라 0, null 등으로 초기화 된다.
    - 따라서 @Builder.Default를 명시하고 그 변수에 값을 초기화한다.

- @Builder.Default
```java
@Entity
public class Member {
    @Builder.Default
    @OneToMany(mappedBy = "member")
    private List<Review> reviewList = new ArrayList<>(); // 없으면 null... 초기값이 있음을 알린다.
}
```