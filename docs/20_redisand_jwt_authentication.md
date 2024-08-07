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
##  