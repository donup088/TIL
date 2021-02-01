### api 테스트
```
mvc.perform(post("/api/events")
                .contentType(MediaType.APPLICATION_JSON)
                .accept(MediaTypes.HAL_JSON)
                .content(objectMapper.writeValueAsString(event)))
                .andDo(print())
                .andExpect(status().isCreated())
                .andExpect(jsonPath("id").exists());
```
- accept(MediaTypes.HAL_JSON) : MediaTypes.HAL_JSON 타입을 응답으로 원한다.
-  .content(objectMapper.writeValueAsString(event))) : event 객체를 json으로 objectMapper를 통해서 바꾼다.
- print() 사용으로 결과를 볼 수 있다.
- objectMapper를 사용

### Mokito 사용
- 객체의 메소드를 실행했을 때 리턴되는 값을 지정해줄 수있다. Test에서 주입받은 객체의 메소드를 사용할 때 NullPointer가 나지 않도록 한다.
    ```
     Mockito.when(eventRepository.save(event)).thenReturn(event);
    ```

### 테스트 description 애노테이션 생성
- 테스트를 잘 알아볼 수 있도록 애노테이션을 생성한다.
- Junit 5에서는 지원하는 기능이고 4에서는 지원하지 않기 때문에 애노테이션을 직접 만들어 사용하면 편하다.
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface TestDescription {
    String value();
}
```

### 웹 테스트 Tips
- @SpringbootTest 를 사용하여 mocking을 줄인다. @SpringbootTest를 사용하면 더 많은 Bean들을 등록하여 테스트가 수월해진다.

- json 값 테스트
    - jsonPath("id").value("1") 이렇게 확인할 수 있다.


### 파라미터를 사용하여 테스트코드 만들기
- 의존성 추가
```
    <dependency>
        <groupId>pl.pragmatists</groupId>
        <artifactId>JUnitParams</artifactId>
        <version>1.1.1</version>
        <scope>test</scope>
    </dependency>
```
- @RunWith(JUnitParamsRunner.class) 추가
```
    @ParameterizedTest
    @MethodSource("paramsForTestFree")
    public void testFree(int basePrice, int maxPrice, boolean isFree) {
        // given
        Event event = Event.builder()
                .basePrice(basePrice)
                .maxPrice(maxPrice)
                .build();

        // when
        event.update();

        // then
        assertThat(event.isFree()).isEqualTo(isFree);
    }

    private static Stream<Arguments> paramsForTestFree() { 
        return Stream.of(
                Arguments.of(0,0, true),
                Arguments.of(100, 0, false),
                Arguments.of(0, 100, false),
                Arguments.of(100, 200, false)
        );
    }
```

- 테스트용 properties 만들기
    - test 폴더 밑에 resouces 폴더를 만들고 application-test.properties 를 만든다.
    - 테스트에서 @ActiveProfiles("test") 사용하여 해당 properties를 사용한다.
    - 기존 properties의 내용을 가져라면서 덮어 쓰게 되기 때문에 중복을 줄일 수 있다.

- 테스트 코드의 중복되는 애노테이션 줄이기
    - 다음과 같이 코드를 작성하고 테스트코드에서 상속받아서 사용하면 된다.
    - @Ignore 테스트 코드 대상에서 무시할 수 있다.
    ```
    @RunWith(SpringRunner.class)
    @SpringBootTest
    @AutoConfigureMockMvc
    @AutoConfigureRestDocs
    @Import(RestDocsConfiguration.class)
    @ActiveProfiles("test")
    @Ignore
    public class BaseControllerTest {
        @Autowired
        protected MockMvc mvc;

        @Autowired
        protected ObjectMapper objectMapper;

        @Autowired
        protected ModelMapper modelMapper;
    }
    ```

- 토큰을 받는 테스트 작성하기
    - httpBasic을 사용해서 사용할 클라이언트 아이디와 클라리언트 시크릿을 전달한다. 이 때 시큐리티 테스트 의존성을 추가해줘야 한다.
    ```
        String clientId = "myApp";
        String clientSecret = "pass";

        mvc.perform(post("/oauth/token")
                .with(httpBasic(clientId, clientSecret))
                .param("username", username)
                .param("password", password)
                .param("grant_type", "password"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("access_token").exists());
    ```

- Post 방식의 테스트에서 토큰을 사용하도록 수정하기
    - 시큐리티를 적용하면 토큰이 적용되지 않은 테스트들은 모두 깨지게 된다.
    - mvc 테스트에 .header(HttpHeaders.AUTHORIZATION, getBearToken()) 부분을 추가해서 토큰이 사용되도록한다.
        ```
        private String getBearToken() throws Exception {
            return "Bearer  " + getAccessToken();
        }

        private String getAccessToken() throws Exception {
            String username = "test2@example.com";
            String password = "pass2";
            Account account = Account.builder()
                    .email(username)
                    .password(password)
                    .roles(Set.of(AccountRole.ADMIN, AccountRole.USER))
                    .build();
            accountService.saveAccount(account);

            String clientId = "myApp";
            String clientSecret = "pass";

            ResultActions perform = mvc.perform(post("/oauth/token")
                    .with(httpBasic(clientId, clientSecret))
                    .param("username", username)
                    .param("password", password)
                    .param("grant_type", "password"));

            String responseBody = perform.andReturn().getResponse().getContentAsString();
            Jackson2JsonParser parser = new Jackson2JsonParser();
            return parser.parseMap(responseBody).get("access_token").toString();
        }
        ```