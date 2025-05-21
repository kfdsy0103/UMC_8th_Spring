## java의 Exception 종류들
- Exception은 예외 발생 시 메시지 처리를 도와주는 Throwable을 상속받는다.
    - getMessage(), printStackTrace()

- Exception은 크게 컴파일, 런타임 에러로 나뉘며, Exception을 상속하는 클래스들의 구조는 다음과 같다.
    - 컴파일 에러
        - IOException
        - FileNotFoundException
        - SQLException
        - ...
    - 런타임 에러 (RuntimeException)
        - RuntimeException을 상속하는 클래스들 ...

- 컴파일 에러와 런타임 에러의 차이
    - 컴파일 에러: 컴파일 단계에서 발생하는 오류로, 컴파일러가 감지할 수 있으며 try-catch로 처리
    - 런타임 에러: 컴파일은 문제 없으나, 프로그램 실행 중 발생하는 오류이다. 예외 처리는 선택적이다.

- 컴파일 에러와 런타임 에러의 예제
```java
// SQLException을 컴파일러가 감지하여 예외 처리 강제화
public static void main(String[] args) {
    try {
        Connection conn = DriverManager.getConnection("...");
    } catch (SQLException e) {
        e.printStackTrace();
    }
}

// 실제로 실행해야 예외가 발생한다. 
public static void main(String[] args) {
    String text = null;
    System.out.println(text.length()); // NPE 발생
}
```

- Checked / Unchecked Exception
    - 예외는 코드적 관점에서 예외 처리를 강제화하는지에 따라 Checked / Unchecked 예외로 분류할 수 있다.
    - Checked : RuntimeException과 하위 클래스
    - Unchecked : RuntimeException을 제외한 나머지 Exception을 상속받는 클래스

- Checked 예외
    - 명시적인 예외 처리를 강제화한다.
    - try-catch로 감싸거나 throw로 던져서 처리한다.

- Unchecked 예외
    - 예외 처리가 선택적이다.
    - try-catch로 처리하지 않아도 프로그램을 컴파일 및 실행할 수 있다.

- Check 예외의 문제점
    - try-catch로 코드가 복잡해진다.
    - throw로 던진다면 코드에 해당 예외를 import 하기 때문에 의존성이 생긴다.

## @Valid
- 빈 검증기(Bean Validator)를 사용해 검증을 지시하는 어노테이션

- 빈 검증기(Bean Validator)
    - 필드 값 검증 로직을 표준화한 기술로, 자바에서 제공하는 표준 기술이다.
    - 검증을 위한 어노테이션과 관련 인터페이스를 제공하며, 일반적으로 Hibernate Validator를 구현체로 사용한다.

- 사용법
    - Controller에서 사용자로부터 @RequestBody로 객체를 가져올 때, 필드 값을 검증하는 경우 사용한다.

- 예제
```java
// 각 필드에는 빈 검증기에서 제공하는 검증 어노테이션을 사용한다.
// message 속성으로 검증 실패 시 메시지를 커스텀할 수 있다.
public class User {
    
    @NotNull(message = "username can't be null")
    @Size(min = 3, max = 15, message = "Username must be between 3 and 15 characters")
    private String username;

    @NotNull(message = "password can't be null")
    @Size(min = 6, message = "Password must be at least 6 characters")
    private String password;
}

// 컨트롤러에서는 빈 검증을 실행하도록 @Valid를 사용한다.
@RestController
public class UserController {

    @PostMapping("/register")
    public String registerUser(@Valid @RequestBody User user) {
        return "success";
    }
}
```

- @Valid의 동작원리
    - DispatcherServlet -> Controller 과정에서 필요한 인자를 만들어주기 위해 ArgumentResolver가 사용된다.
    - 위 예제의 경우 @RequestBody 때문에 구현체로 RequestResponseBodyMethodProcessor가 사용될 것이다.
    - 리졸버 내부에서 @Valid로 시작하는 어노테이션을 확인하고, 검사를 진행한다.
    - 요약하자면, @Valid는 ArgumentResolver에서 사용된다.
    - 따라서 @Valid는 컨트롤러에서만 동작하고, 다른 계층에서는 검증이 되지 않는다.

- @Valid로 검증이 실패하는 경우
    - @Valid가 MethodArgumentNotValidException 예외를 발생시킨다.
    - 클라이언트에 그대로 전달되면 안되므로 @RestControllerAdvice에서 handleMethodArgumentNotValid메서드를 오버라이딩 하여 처리한다.

- 검증 어노테이션을 커스텀하는 방법
    - 커스텀 어노테이션 정의
    - 해당 어노테이션에 적용될 검증기 클래스 정의
    - 실제 사용

- 커스텀 예제
```java
// 1. 커스텀 어노테이션 정의
@Constraint(validatedBy = PasswordValidator.class)  // 검증기 클래스 지정
@Target({ ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidPassword {

    String message() default "최소 8자 이상, 대문자, 소문자, 숫자, 특수문자가 모두 포함되어야 합니다.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```
```java
// 2. 검증기 클래스 정의
public class PasswordValidator implements ConstraintValidator<ValidPassword, String> {
    
    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        if (password == null) {
            return false;
        }
        
        boolean hasUpperCase = false;
        boolean hasLowerCase = false;
        boolean hasDigit = false;
        boolean hasSpecialChar = false;

        // 확인 로직 ...
        
        return password.length() >= 8 && hasUpperCase && hasLowerCase && hasDigit && hasSpecialChar;
    }
}
```
```java
// 3. 실제 사용
@RestController
public class UserController {
    @PostMapping("/register")
    public String registerUser(@Valid @RequestBody User user) {
        return "success";
    }
}
```