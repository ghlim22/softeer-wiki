#### 컨트롤러
```java
@RestController // JSON을 반환하는 컨트롤러
public class HelloController {
	@GetMapping("/hello") // "/hello" GET 요청에 대응하는 API
	public String hello() {
		return "hello";
	}

	@GetMapping("/hello/dto")
	public HelloResponseDto helloDto(@RequestParams("name") String name,
									// 외부에서 API로 넘긴 파라미터를 가져온다.
									 @RequestParams("amount") int amount) {
		return new HelloResponseDto(name, amount);

	}
}

@Getter // 선언된 모든 필드에 대한 Getter 생성
@RequiredArgsConstructor // 선언된 모든 final 필드를 인자로 받는 생성자 생성
public class HelloResponseDto {
	private final String name;
	private final int amount;
}

@RunWith(SpringRunner.class) // 테스트 진행 시 Junit 내장 실행자 외 추가 실행자
@WebMvcTest(controllers = HelloController.class) // Spring MVC 관련 테스트
public class HelloControllerTest {
	@AutoWired // Spring bean 주입
	private MockMvc test;

	@Test
	public void hello_반환() throws Exception {
		mvc.perform(get("/hello"))
		   .andExpect(status().isOk())	
		   .andExpect(content().string(hello));
	}

	@Test
	public void helloDto_반환() throws Exception {
		String hola = "hola";
		int amount = 42;

		mvc.perform(get("hello")
					.param("hello", hola)
					.param("amount", String.valueOf(amount)))
					.isExpect(status().isOk())
					.isExpect(jsonPath("$.name", is(name)))
					.isExpect(jsonPath("$.amount, is(amount)));
	}
	// param: 요청 파라미터 설정, 단 문자열만 넣을 수 있다.
	// jsonPath(): JSON 응답값을 필드별로 검증할 수 있는 메서드

}
```

### JPA
Java의 표쥰 Object Relational Mapping
기존 sql mapper 대비 장점
- 객체 지향 패러다임을 사용하여 DB에 접근할 수 있다.
- SQL 비종속적
**Spring Data JPA**
- JPA <- Hibernate <- Spring data JPA
**Spring Data**
- 여러 spring data 인터페이스들은 기본적인 CRUD 인터페이스를 공유한다.
-  구현체 교체 용이
- 저장소 교체 용이

#### Domain
소프트웨어가 해결할 요구사항 혹은 문제 영역

## 구성 요소 정리

#### 엔티디, Entity

릴레이션과 매핑될 클래스

`@Id`: 엔티티의 primary key. 
`@GeneratedValue(strategy=GenerationType.IDENTITY)` : PK의 생성 규칙 표현. GenerationType.IDENTITY 옵션을 사용해야 auto_increment(데이터가 삽입될 때마다 1 씩 증가된 값을 삽입)가 된다. (스프링 2.0)

엔티티에서 setter를 사용하면 데이터가 언제 어떤 목적으로 변화되는지 파악하기 어렵다. 엔티티 클래스에서는 절대 setter를 사용하지 않고, 상태 변경이 필요한 경우 그 목적과 의도를 명확히 드러내는 메서드를 추가해 사용한다.



