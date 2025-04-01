## 1. 홈 화면
### API Endpoint
```
GET /api/home
```

### Query String
- 필요없음

### Path Variable
- 필요없음

### Request Header
```
Content-Type : application/json
Authorization: Bearer <token>
```

### Request Body
```json
{
  "regionName" : "상암동"
}
```

### Response Body
```json
{
  "success": "true",
  "data": {
    "point": "999999",
    "missionCount": "7",
    "missions": [
      {
        "id": "1",
        "restaurantName" : "짜장전설",
        "content" : "12000원 이상 식사하기",
        "point" : "500",
        "deadline" : "7"
      },
      {
        "id": "2",
        "restaurantName" : "요거트월드",
        "content" : "10000원 이상 식사하기",
        "point" : "500",
        "deadline" : "10"
      }
    ]
  }
}
```


## 2. 리뷰 작성
### API Endpoint
```
POST /api/restaurants/{restaurantId}/reviews
```

### Query String
- 필요없음

### Path Variable
```
restaurantId : 레스토랑 식별자 값
```

### Request Header
```
Content-Type : application/json
Authorization: Bearer <token>
```

### Request Body
```json
{
    "content" : "맛있어요!",
    "score" : "5",
    "images" : ["image1.png", "image2.png"]
}
```

### Response Body
```json
{
  "success" : "true",
  "data" : {
    "reviewId" : "123"
  }
}
```


## 3. 미션 목록 조회(진행중, 진행 완료)
### API Endpoint
```
GET /api/mypage/missions
```

### Query String
- 필요없음

### Path Variable
- 필요없음

### Request Header
```
Content-Type : application/json
Authorization: Bearer <token>
```

### Request Body
```json
{
  "regionName" : "상암동",
  "status" : "inprogress" or "completed"
}
```

### Response Body
```json
{
  "success": "true",
  "data": {
    "missions": [
      {
        "id" : "3",
        "restaurantId" : "3",
        "restaurantName" : "짜장전설",
        "content" : "12000원 이상 식사하기",
        "point" : "500",
        "deadline" : "7",
        "imageUrl" : "짜장전설.jpg"
      },
      {
        "id" : "4",
        "restaurantId" : "4",
        "restaurantName" : "요거트월드",
        "content" : "10000원 이상 식사하기",
        "point" : "500",
        "deadline" : "10",
        "imageUrl" : "요거트월드.jpg"
      }
    ]
  }
}
```


## 4. 미션 성공 누르기
### API Endpoint
```
POST /api/missions/{missionId}/complete_request
```

### Query String
- 필요없음

### Path Variable
```
missionId : 성공 요청을 하려는 미션의 식별자 값
```

### Request Header
```
Authorization: Bearer <token>
```

### Request Body
- 필요없음

### Response Body

```json
{
  "success": "true",
  "data": {
    "missionId": "123"
  }
}
```


## 5. 회원 가입 하기
### API Endpoint
```
POST /api/users/signup
```

### Query String
- 필요없음

### Path Variable
- 필요없음

### Request Header
```
Content-Type : application/json
```

### Request Body
```json
{
  "terms" : [
    {"termId" : "1", "isAgree" : "true"},
    {"termId" : "2", "isAgree" : "true"},
    {"termId" : "3", "isAgree" : "False"},
    {"termId" : "4", "isAgree" : "False"}
  ],
  "email" : "aaa@gmail.com",
  "name" : "홍길동",
  "gender" : "MALE",
  "birth" : "2002-10-30",
  "region" : "인천시 미추홀구 용현로",
  "addressDetail" : "00-00 아리스타 000호",
  "preferredCategories" : ["1", "4", "7"]
}
```

### Response Body
```json
{
  "success" : "true"
}
```

<hr>

## 시니어 미션 1. Soft Delete와 사용되는 Http 메서드
- 데이터의 실제 삭제가 아닌, 논리적 삭제를 의미한다.

- 구현 방법 : 컬럼으로 deleted(Boolean, ENUM), deleted_at(datetime)을 추가하여 삭제 여부와 삭제 이력을 함께 관리하는 방식으로 사용한다.

- 사용하는 경우
  - 데이터 복구나 삭제기록 관리가 필요한 경우
  - 결제 정보와 같이 민감한 정보를 기록하는 경우
  - 로그이력을 관리하는게 중요한 경우
  
- Soft Delete 구현을 위한 HTTP 메서드
  - DELETE 
    - 리소스의 삭제를 표현하는 데 사용되는 DELETE 메서드를 사용한다.
    - RESTful API 원칙의 일관성을 지킬 수 있다.
  - PATCH
    - 일부 컬럼 값(deleted)만 변경되는 것이기에 PATCH로도 구현이 가능할 것 같다.
  - PUT
    - PUT도 마찬가지로 리소스를 업데이트 할 수 있어 소프트 딜리트 구현이 가능해 보인다.

- 그렇다면 어떤 HTTP 메서드를 써야할까?
  - DELETE가 적절해 보인다. Soft Delete인지, Hard Delete인지는 오로지 백엔드 단에서의 관심영역이다. 
  - 따라서 사용하는 쪽에서는 구현에 신경쓰지 않고, DELETE 메서드만 보고 '엔티티의 삭제'라는 의미가 바로 전달되도록 하는 것이 중요하다고 판단했다.


## 시니어 미션 2. 컨트롤 URI와 사용 예시
- 컨트롤 URI란?
  - RESTful API 설계에서 CRUD 외에, 특정 작업을 명시적으로 수행할 필요가 있는 경우 사용하는 패턴을 의미한다.
  - HTTP 메서드와 자원만으로 설계가 어려운 경우, 동사를 포함하여 엔드포인트를 설계할 수 있다.
  
- 사용 예시
  - 상황 : 유저에 관한 api로 로그인, 로그아웃, 비활성화를 구현
    - 설계 원칙을 따른 POST /users 로는 표현이 어렵기에 동사를 추가함
    - POST /users/login
    - POST /users/logout
    - POST /users/deactivate : 유저 상태 비활성화


## 시니어 미션 3. 주어진 자료 요약
- REST
  - REST(Representational State Transfer)는 하이퍼미디어 기반 분산 시스템을 구축하는 방식이다.
  - 특징 : 특정 프로토콜에 종속되지는 않으나, 대부분 HTTP를 사용하여 리소스를 중심으로 API를 설계합니다.
  - 주요 원칙 
    - 모든 리소스는 고유한 URI로 식별
    - JSON으로 클라이언트와 데이터 교환
    - 표준 HTTP 메서드를 활용하여 균일한 인터페이스
    - 각 요청은 독립적이며, 클라이언트의 상태를 유지하지 않음

- 리소스 중심의 API 디자인
  - 비즈니스 엔티티가 중심이어야 하고, 명사 기반으로 작성
  - 계층적인 URI 구조를 설계해야함
  - 리소스의 적절한 비정규화로 과도한 API 호출을 방지해야함
  - 데이터베이스 테이블을 그대로 노출하지 않고 추상화하여 제공

- HTTP 메서드 측면에서의 API 설계
  - HTTP 메서드의 의미
    - GET : 리소스의 표현, 응답 메시지의 본문은 요청한 리소스의 세부 정보를 포함.
    - POST : 지정된 URI에 새 리소스를 만드는 메서드. 요청 메시지의 본문에 새 리소스의 세부 정보를 제공.
    - PUT : 지정된 URI에 리소스를 만들거나 대체합니다. 요청 메시지의 본문에 만들거나 업데이트할 리소스를 지정.
    - PATCH : 리소스의 부분 업데이트 수행. 요청 본문에 변경 내용을 지정.
    - DELETE : 리소스를 제거.
  
  - POST/PUT/PATCH
    - POST : 새로운 리소스를 생성, 식별자를 서버가 할당.
    - PUT : 리소스 전체를 업데이트/없으면 생성. 클라이언트가 식별자를 알고 있음.
    - PATCH : 리소스의 일부 내용만 수정

  - 멱등성 측면에서의 http 메서드
    - GET/PUT/DELETE: 같은 요청을 여러 번 보내도 동일한 결과이다. (멱등 O)
    - POST: 여러 번 호출하면 계속 새로운 리소스가 생성된다. (멱등 X)
    - PATCH : 특정 컬럼을 1씩 증가하도록 API가 설계되었다면 이 경우는 비멱등이다. (멱등일수도 있고, 아닐 수도 있다) 