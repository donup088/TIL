### 입력하려는 값 이외의 다른 값이 들어왔을 때 
- 에러 발생 : properties에 추가
```
spring.jackson.deserialization.fail-on-unknown-properties=true
```
- 무시

### ResourceServer 설정
- 요청에 따라 토큰을 확인하고 권한을 검사한다.
- @EnableResourceServer 사용한다.
- 두 가지 메서드를 override해서 사용한다.
```
    @Override
    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
        resources.resourceId("event");
    }

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
                .anonymous()
                .and()
                .authorizeRequests()
                .mvcMatchers(HttpMethod.GET, "/api/**")
                .anonymous()
                .anyRequest()
                .authenticated()
                .and()
                .exceptionHandling()
                .accessDeniedHandler(new OAuth2AccessDeniedHandler());
    }
```

- 문자열을 외부로 빼내기
    - 의존성 추가하기
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    ```
    - AppProperties 클래스 만들기
    ```
    @Component
    @ConfigurationProperties(prefix = "my-app")
    @Getter
    @Setter
    public class AppProperties {
        @NotEmpty
        private String adminUsername;

        @NotEmpty
        private String adminPassword;

        @NotEmpty
        private String userUsername;

        @NotEmpty
        private String userPassword;

        @NotEmpty
        private String clientId;

        @NotEmpty
        private String clientSecret;
    }
    ```
    - properties에서 설정이 가능해진다.
    ```
    my-app.admin-username=admin@example.com
    my-app.admin-password=admin
    my-app.user-username=user@example.com
    my-app.user-password=user
    my-app.client-id=myApp
    my-app.client-secret=pass
    ```
