- Restful API란? 
HTTP URI로 자원을 명시하고, HTTP Method(GET, POST, PUT, DELETE)를 통해 해당 자원에 대한 CRUD 작업을 구분하는 'REST' 원칙을 철저히 준수한 API를 의미한다.

- Restful API 설계 규칙
  - URI에는 행위가 아닌 자원을 포함해야한다.
    - 자원을 복수형태로 표현하며, 행위는 HTTP Method를 통해 구분한다.
    
  - 슬래시 구분자는 계층 관계를 표현하는데 사용한다.
    > http://hello.com/schools/classes
    
  - URI 마지막 문자로 슬래시를 포함하지 않는다.
    - REST API는 분명한 URI를 만들어야 하기 때문에 혼동을 주지 않도록 URI 경로의 마지막에는 슬래시(/)를 사용하지 않는다.
    > http://hello.com/schools/classes/ (X)

  - 불가피하게 긴 URI 경로를 사용하게 된다면 하이픈(-)을 사용해 가독성을 높인다.
    > http://hello.com/schools/best-classes
  
  - 밑줄(_)은 URI에 사용하지 않는다.
    - 밑줄은 식별이 힘들고, 문자가 가려지기도 하므로 가독성을 위해 사용하지 않는다.

  - URI 경로는 소문자로 작성한다.
    - 대소문자 혼용으로 인한 혼란을 방지하고, 일관된 리소스 접근을 보장하기 위해 소문자로 통일하는 것이 좋다.

  - 파일확장자는 URI에 포함하지 않는다.
    - Accept header를 사용한다.
    > http://hello.com/members/123/photo.jpg (X)
     
    > GET <br>
      http://hello.com/members/123/photo <br>
      Accept: image/jpg (O)
   
  - 리소스 간 연관 관계 또한 URI에 표현한다.
    > /users/{userid}/devices : 유저가 기기를 소유하는 경우를 표현
  
- Restful API의 목적
  - 근본적인 목적은 성능 향상이 아닌, 일관적인 컨벤션을 통한 API의 이해도와 호환성을 높이는 것이다.
  - 특정 작업을 명시적으로 나타내기 위해 의도적으로 REST를 지키지 않는 경우도 존재한다. -> 컨트롤 URI
  ```
  POST : /users/login
  POST : /users/logout
  ```

- Restful API의 예시
  - 회원 목록 조회	GET	    /members
  - 특정 회원 조회	GET	    /members/{id}
  - 회원 생성	        POST    /members
  - 회원 정보 수정	PUT	    /members/{id}
  - 회원 삭제	        DELETE  /members/{id}