# SG API 이용가이드

slogup에서는 Express의 Wrapper Module인 SG-Core Framework를 보유하고 있다. 해당 프레임워크는 앞으로 `코어`라고 명명한다.

코어에서 제공하는 핵심 API 기능은 다음과 같다.

- 계정관련 APIs
- 약관 관련 APIs
- 문자, 이메일 전송관련 APIs
- 노티피케이션 관련 APIs
- 멀티미디어 관련 APIs
- 신고/문의 관련 APIs
- 공지사항 관련 APIs
- 게시판 관련 APIs

본 문서는 이중에서 기본적인 API 형태와 이용방법에 대해서 설명한다.


## API 형태

코어 API의 형태는 RestFul서비스의 형태와 같으나 개발편의를 위한 예외사항이 몇 개가 있다. 기본적으로 4개의 Method를 사용하며 각 메소드는 `GET`, `POST`, `PUT`, `DELETE`가 있다. `GET`의 경우 복수와 단수 2가지로 나뉘며 보통 리소스명 끝이 복수형으로 끝나거나 단수로 끝나게 된다.

예로 http://mydomain.com/api/accounts/users 라는 리소스가 있다면 CRUD의 형태는 다음과 같다.
- GET http://mydomain.com/api/accounts/users : 유저 복수 반환
- GET http://mydomain.com/api/accounts/users/id : 해당 id의 유저 반환
- POST http://mydomain.com/api/accounts/users : 유저 생성 (회원가입)
- PUT http://mydomain.com/api/accounts/users/id : 해당 id의 유저 수정 (정보수정)
- DELETE http://mydomain.com/api/accounts/users/id : 해당 id의 유저 제거 (회원탈퇴)

또한 해당 리소스는 `도메인명`/`api`/`리소스그룹명`/`리소스객체명`의 형태로 이루어져 있다. 
(http://mydomain.com 도메인명을 해당 서비스의 도메인명으로 바꾸면 된다.)

일반적으로는 다음과 같은 형태를 유지하지만 예외적인 경우가 있다.

예로 http://mydomain.com/api/accounts/session 이라는 리소스가 있다면 CRUD의 형태는 다음과같다.
- GET http://mydomain.com/api/accounts/session : 세션 정보 반환 (유저 정보 반환)
- POST http://mydomain.com/api/accounts/session : 세션 정보 생성 (로그인)
- PUT http://mydomain.com/api/accounts/session : 세션 정보 갱신 후 반환 (유저 정보 갱신 후 반환)
- DELETE http://mydomain.com/api/accounts/session : 세션 정보 제거 (로그아웃)

세션리소스는 단수로 끝나게 되고, 세션의 여러정보반환 이라는 API는 제공하지 않게 된다. 또한 특이한 점은 session post가 로그인이라는 점이다.

해당 부분을 쉽게 생각하려면 다음과 같이 이해하면 된다. 세션이라는 리소스명을 갖고 있는 해당 API는 세션그룹에 무언가의 액션을 한다고 이해하면 쉽다.
`GET`은 세션그룹 (세션모임)에서 현재 로그인된 유저의 정보를 가져오는 기능이며, `POST`의 경우 세션그룹에 현재 로그인한 정보를 담아 두어 정보를 유지시키는 기능이라고 보면된다.
`PUT`의 경우 세션정보를 DB에서 find 해와서 redis와 같은 세션스토어에 갱신 후 반환한다. 마지막으로 `DELETE`의 경우 세션그룹에서 해당 세션을 지우는 기능이다.

따라서 리소스명이 어떠한 그룹이라고 생각하고 그 그룹에서 특정 값을 생성하거나, 읽어오거나, 갱신하거나 제거하는 CRUD액션을 API리소스와 Method의 형태라고 보면 된다.

## API 테스터 페이지

코어 API는 기본적으로 API 테스트 페이지를 제공한다. url의 형태는 다음과 같다.

http://mydomain.com/api/tester 로 접근하면 테스트 페이지에 들어갈 수 있다. 슬로그업에서는 모든 응답형태를 통일하기 위해 `resform`이라는 방식을 도입했다.
이는 특정 API응답의 관한 JSON형태를 통일 시키기 위함인데, 클라이언트 개발시 특정 object의 property를 이용할 때 에러율을 줄여줄 수 있다.


## API 이용 방법

API이용 방법은 일반적인 호출 방식과 비슷하다. 먼저 계정에 관한 API부터 살펴보도록 하자.
계정 API는 리소스그룹명이 accounts로 시작하게 된다.

### 회원가입
회원가입은 email, phone, social, phoneId, normalId, phoneEmail 총 6개의 형태가 있다. email의 경우 이메일 회원가입을 의미하며, phone의 경우 핸드폰번호로 가입하는 것을 의미한다. social의 경우 각 provider를 이용하여 소셜 가입을 하는 것을 의미하며, normalId의 경우 기본 아이디, 비밀번호 가입을 의미한다. phoneId의 경우 핸드폰가입이나 아이디가 따로 존재하는 것을 의미한다. 마지막으로 phoneEmail의 경우 휴대폰가입이면서 이메일 아이디를 이용해 가입하는 것을 의미한다.

회원가입시 필요한 리소스 목록은 다음과 같다.
- http://mydomain.com/api/accounts/sender-phone
- http://mydomain.com/api/accounts/sender-email
- http://mydomain.com/api/accounts/auth-phone
- http://mydomain.com/api/accounts/auth-email
- http://mydomain.com/api/accounts/users

#### 이메일가입
이메일 가입은 말그대로 이메일이 아이디가 되어 가입하는 형태를 의미하는데, 이때 두가지 옵션이 있다. 하나는 이메일 바로 인증이며, 또 하나는 이메일로 특정 해쉬값을 전송하여 재인증 받는 방식이다.

1. POST http://mydomain.com/api/accounts/users
  - type : [email]
  - uid : email id값
  - secret : 비밀번호

#### 핸드폰가입
핸드폰가입은 번호 인증을 먼저 받아야 한다. 이후 user post를 이용해서 인증번호와 함께 가입이 가능하다.

1. POST http://mydomain.com/api/accounts/sender-phone
  - phoneNum  : 인증받을 전화번호, 반드시 +와 국가코드를 붙이고 하이픈(-)은 뺀형태여야 한다. 예) +821012341234
  - type  : [phoneSignup]

2. POST http://mydomain.com/api/accounts/users
  - type : [phone]
  - uid : 인증받은 전화번호
  - secret : 1번에서 받은 인증번호



