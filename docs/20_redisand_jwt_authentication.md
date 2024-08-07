# 20. REDIS and jwt authentication

## 81 이번 장에서 구현하는 기능
지금 현재의 애플리케이션은 다음과 같은 기능이 있다`
• 태스크를추가한다
• 태스크를 열람한다
• 신규 사용자를 등록한다
이 장에서는 새롭게 다음 기능들을 추가한다
• 등록 완료된 사용자 정보를 사용해 액세스 토큰을 발행하는 로그인 기능
• 로그인한 사용자만 API 사용을 허가하는 기능
• 액세스 토큰에 포함된 식별 정보를 사용하는 기능
• 관리자 권한의 사용자만 접속할 수 있는 기능
이 기능들을 개발하면 다음과 같은 기술을 학습할 수 있다
레디스Remote Dictionary Server, Redis를 사용안 캐시
• JSON 웹 토큰JSON Web Token, JWT을 사용한 액세스 토큰 처리
• go:embed를 사용한 파일 추가
• 미들웨어 패턴을 사용한 HTTP 헤더 정보의 투과적 전달 방법
• 테스트의 사전 데이터 작성을 효율화하기 위한 fixture 함수 활용

## 82 redis 준비
`make up`
`make down`
`go get`
### redis 관리 키값 저장소
- store/kvs.go
- config/config.go
- testutil/kvs.go
testutil/db.go 파일에 작성한 OpenDBForTest 함수처럼 테스트 실행 환경마다 다른 레
디스 접속 정보를 사용하도록 OpenRedi유orTest 함수를 구현한다
- store/kvs_test.go
## 83 JWT 서명 준비
`openssl version`
`openssl genrsa 4096 > secret.pem`
`openssl rsa -pubout < secret.pen > public.pem`
- auth/cert
## 84 JWT를 사욤한 액세스 토큰 작성
JWTjsoNwebtokeE Base64URL로 인코딩된 JSON을 사용해 양자간 정보를 교환하는
수단이다 이 사양은 RFC7519에 정의돼 있다

실제로 사용할 때는 토큰 자체가 변조되는 것을 방지하기 위해 JWT를 서명해서 교환
하게 된다 이를 위해 JWT는 서명 및 암호화와 관련된 사양을 사용하며* 이를 통칭해
서 JOSEjsON Object Signing and Encryption이 라고 부른다

go：embed < pk 값을 바이너리에 포함한다.>
- auth/jwt.go
- auth/jwt_test.go
  - generatetoken
- fixture 함수 구현
  - fixture/user.go
- auth/jwt_test.go -> testjwt_genjwt

http요청에서 jwt받기
- auth/jwt.go -> gettoken
  - auth/jwt_test.go -> TestJWTer_GetJWT
  - auth/jwt_test.go -> TestJWTer_GetJWT_NG

JWT정보를 context.Context타입값에 넣기
- JWT를 생성 및 받아올 수 있게 됐다 하지만 애플리케이션 코드에서 매번 JWT를 생성
해 사용하는 것은 비효율적이다 따라서 context.Context 타입값에 JWT에서 가져온
사용자 ID와 권한 정보를 설정한다

- auth/jwt.go ->FilContext


## 사용자 로그인 엔드포인트 구현
- POST/register 엔드포인트를 사용해 등록 완료한 사용자를 대상으로 한다
- POST /login로 사용자명과 패스워드를 포함한 요청을 받는다
- 인증성공한 사용자에게는 액세스 토큰을 발행한다
► 액세스 토큰의 유효 기간은 30분
► 액세스 토큰은 변조 방지를 위해 서명된 것으로 로그인 정보가 포함된다
► 액세스 토큰에는 다음 정보가 포함된다
> 사용자명
> 권한
• 애플리케이션은 액세스 토큰으로부터 사용자 ID를 검색할 수 있다

### handler 패키지 구현
- handler/login.go
  - handler/login_test.go
- go:generate -> make generate
### service 패키지 구현
테스트를 고려해서 *store.Repository 타입과 *auth.JWTer을 직접 참조하지 않고
코드 20.29
처럼 UserGetter 인터페이스나 TokenGenerator 인터페이스를 정의한다
- service/login.go
- handler -> service호출 구조
- entity/user.go

### store 패키지 구현
영구화된 *entity.User 타입값을 가져오는 구현이다
. github.com/
jmoiron/sqlx 패키지에 정의된 *sqlx.DB.GetContext 메서드를 사용하면 쿼리 실행
결과가 설정된 구조체를 쉽게 얻을 수 있다.
- store/user.go - getUser

## 86 미들웨어 패턴을 사용한 인증 기능
- handler/middleware.go
  - context.Context에 저장

## 87 요청에 포함된 은증과 권한 정보를 사용한 엔드포인트 보호
로그인 후에 HTTP 요청에 인증 및 권한 정보가 부여되도록 구현했다 /tasks 엔드포인트를 수정해서 인증 및 권한 정보를 사용하게 만든다.

• GET /tasks와 POST /tasks 엔드포인트는 인증을 완료한 사용자만 이용할 수 있다
• POST /tasks 엔드포인트로 새로운 태스크를 저장할 때 태스크와 함께 사용자 정보도 저장한다
• GET /tasks 엔드포인트를 실행하면 자신이 등록한 태스크만 열람할 수 있다
- _tools/mysql/schema.sqt > migration 수행
- entitv/task.qo > user id 추가
- mux.go > 미들웨어추가
- service/add_task.go > 사용자 아이디 추가
- store/task.go > 사용자 ID를 이용해 태스크를 검색


### 테스트 작성
prepareTasks 함수는 prepareUser 함
수를 사용해 태스크를 등록한다 prepareTasks 함수에서는 사전 데이터로 세 개의 태
스크를 등록하지만 그중 하나는 사용자 ID가 달라서 테스트에서는 두 개의 태스크만
반환된다

