### 로그인 기능

`POST`메서드로 로그인 요청을 받는다. 로그인이 성공하면, `/index.html`로 리디렉션되고, 실패하면 실패를 알리고 로그인 페이지에 머물러야 한다.

쿠키와 세션을 사용해 로그인 정보를 유지한다.
##### 무상태(stateless) 프로토콜인 HTTP가 어떻게 상태를 가지도록 할 수 있을까?

HTTP는 상태를 가지지 않는 프로토콜이다. 로그인 세션, 장바구니와 같이 상태를 유지할 필요가 있는 작업을 수행하고 싶을 때, 쿠키와 세션을 사용한다.

쿠키는 상태 정보를 기억하기 위해 사용하는 정보 조각이다. 서버가 `Set-Cookie`헤더를 통해 생성한 쿠키를 브라우저에 보내면, 브라우저는 이후 발생하는 요청의 `Cookie`헤더에 쿠키를 실어 전달한다.

쿠키는 사용자단에 저장되기 때문에 보안상 약점이 존재한다. 사용자의 컴퓨터에 저장된 쿠키 정보를 탈취하여 악용하는 공격이 가능하다. 이러한 쿠키의 단점을 보완하는 것이 세션이다. 세션은 쿠키와 같이 HTTP 상에서 상태를 유지하는 것을 목적으로 한다. 세션은 정보를 클라이언트가 아닌 서버에 저장한다. 그 후 각 클라이언트마다 고유한 세션ID를 발급해 이 ID를 쿠키 헤더로 전달한다. 서버는 세션 ID를 보고 상태 유지에 필요한 정보를 찾는다.

##### 쿠키/세션 관리

필요하다고 생각한 기능들
- 충분히 무작위적인 세션 ID 생성
- 한 사용자의 세션은 최대 한 개만 유지하기
- 모든 세션을 관리하는 저장소 `HttpSessions: Map<String, HttpSession>`; `SessionID`를 키로 사용한다.
- 세션 정보를 표현하는 `HttpSession`: 상태 정보를 저장한다.

