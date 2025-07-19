#### 구조

- Router: `Function<HttpRequest, Service>`
- Service: `Function<HttpRequest, HttpResponse>`

`Router`는 `HttpRequest -> Service` 함수입니다.
`Service`는 `HttpRequest -> HttpResponse`함수입니다.

- `Router`
	- `ServerRouter`
	- `UserRouter`
- `Service`
	- `StaticFileService`
	- `UserCreateService`

직관적인 구조를 만들고자 개선 작업을 하였습니다.

#### 기능 요구사항

- `POST`로 회원가입
- 회원가입 성공시, `/index.html`로 리디렉션
	- 302 Found 응답
- `POST`외의 메서드로 회원가입을 시도하면 실패해야 한다.
	- 405 Method Not Allowed 응답

#### 추가로 해야하는 것들

- 입력값 무결성 검사
- 입력값 검사 실패시, 경우에 맞는 응답 반환
- 중복 ID 처리
 
