### PreAuthorizationToken 구현
- 인증되기전 Token 객체를 의미한다.
- UsernamePasswordAuthenticationToken을 상속받는다.
```
public class PreAuthorizationToken extends UsernamePasswordAuthenticationToken {
    public PreAuthorizationToken(String username, String password) {
        super(username, password);
    }

    public PreAuthorizationToken(LoginDto loginDto) {
        this(loginDto.getUserId(), loginDto.getPassword());
    }

    public String getUsername() {
        return (String) super.getPrincipal();
    }

    public String getPassword() {
        return (String) super.getCredentials();
    }
}

```
### PostAuthorizationToken 구현
- 인증되고 나서의 Token을 의미한다.
- UsernamePasswordAuthenticationToken을 상속받는다.
- AccountContext로부터 PostAuthorizationToken을 얻는 메소드를 포함한다.
    ```
    public static PostAuthorizationToken getTokenFromAccountContext(AccountContext accountContext) {
        return new PostAuthorizationToken(accountContext, accountContext.getPassword(), accountContext.getAuthorities());
    }
    ```

### 시큐리티의 User를 상속받는 AccountContext 생성
- 이 클래스는 User를 상속받고 Account 객체를 받아서 User를 상속받는 AccountContext를 반환해줄 수 있도록한다.
    ```
    public static AccountContext fromAccountModel(Account account) {
        return new AccountContext(account, account.getUsername(), account.getPassword(), parseAuthorities(account.getUserRole()));
    }

    private static List<SimpleGrantedAuthority> parseAuthorities(UserRole userRole) {
        return Arrays.asList(userRole).stream()
                .map(r -> new SimpleGrantedAuthority(r.getRoleName())).collect(Collectors.toList());
    }
    ```

### UserDetailsService를 implements해서 AccountContextService 클래스 구현
- 로그인한 유저를 조회해오는 부분을 구현한다(loadUserByUsername).


### AuthenticationProvider를 implements 해서 FormLoginAuthenticationProvider 구현
- authenticate 매소드와 supports 매소드를 override 한다.
- authenticate 메소드 안에서는 로그인한 유저정보가 맞는 지 확인하는 로직을 구현한다. 유저정보가 인증이 되었다면 PostAuthorizationToken을 만들어서 반환해준다.
    ```
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        PreAuthorizationToken token = (PreAuthorizationToken) authentication;
        String username = token.getUsername();
        String password = token.getPassword();

        Account account = accountRepository.findByUserId(username).
                orElseThrow(() -> new UsernameNotFoundException("정보에 맞는 계정이 없습니다."));
        if (isCorrectPassword(password, account)) {
            return PostAuthorizationToken.getTokenFromAccountContext(AccountContext.fromAccountModel(account));
        }
        throw new NoSuchElementException("인증 정보가 정확하지 않습니다.");
    }
    ```

### JwtFactory 구현
- 인증이 완료되었다면 jwt토큰을 만들어주기 위해 클래스를 구현한다.
- Jwt 토큰 Claim부분에 데이터들을 넣어줄 수 있다.
- Key를 암호화해서 등록한다.
```
    private static String signingKey = "jwttest";

    public String generateToken(AccountContext accountContext) {
        String token = null;
        try {
            token = JWT.create()
                    .withIssuer("Dong")
                    .withClaim("USER_ROLE", accountContext.getAccount().getUserRole().getRoleName())
                    .withClaim("USERNAME",accountContext.getAccount().getUsername())
                    .sign(generateAlgorithm());
        } catch (Exception e) {
            log.error(e.getMessage());
        }
        return token;
    }
    private Algorithm generateAlgorithm() {
        return Algorithm.HMAC256(signingKey);
    }
```

### AuthenticationSuccessHandler implements 해서 FormLoginAuthenticationSuccessHandler 구현
- 로그인 인증이 성공했을 때 토큰을 생성하고 요청바디에 토큰정보를 포함시켜 반환해준다.
- 이 클래스와 비슷하게 실패했을 때도 구현할 수 있다.
    ```
    private final JwtFactory factory;
    private final ObjectMapper objectMapper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        PostAuthorizationToken token = (PostAuthorizationToken) authentication;
        AccountContext context = (AccountContext) token.getPrincipal();
        String tokenString = factory.generateToken(context);
        processResponse(response, writeDto(tokenString));
    }

    private TokenDto writeDto(String token) {
        return new TokenDto(token);
    }

    private void processResponse(HttpServletResponse res, TokenDto tokenDto) throws IOException {
        res.setContentType(MediaType.APPLICATION_JSON_VALUE);
        res.setStatus(HttpStatus.OK.value());
        res.getWriter().write(objectMapper.writeValueAsString(tokenDto));
    }
    ```
### AbstractAuthenticationProcessingFilter를 상속받아서 FormLoginFilter를 구현한다.
- 지금까지 만든 클래스들을 필터에 적용시킨다.
- Component로 만들지 않고 생성자를 사용한다.
- attemptAuthentication(인증시도), successfulAuthentication(성공시) ,unsuccessfulAuthentication(실패시) 를 오버라이드한다.
- attemptAuthentication에서 AuthenticationManager로 인증전 토큰을 받아서 인증을 시도한다.
    ```
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException, IOException, ServletException {
        LoginDto loginDto = new ObjectMapper().readValue(request.getReader(), LoginDto.class);
        PreAuthorizationToken token = new PreAuthorizationToken(loginDto);
        log.info("token: " + token);
        return super.getAuthenticationManager().authenticate(token);
    }
    ```

### SecurtyConfig에 필터, 핸들러, 프로바이더 등을 설정한다.
```
    private final FormLoginAuthenticationSuccessHandler formLoginAuthenticationSuccessHandler;
    private final FormLoginAuthenticationProvider provider;

    @Bean
    @Override
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }

    protected FormLoginFilter formLoginFilter() throws Exception {
        FormLoginFilter filter = new FormLoginFilter("/formlogin", formLoginAuthenticationSuccessHandler);
        filter.setAuthenticationManager(authenticationManager());

        return filter;
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(this.provider);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        http
                .csrf().disable();
        http
                .headers().frameOptions().disable();
        http
                .addFilterBefore(formLoginFilter(), UsernamePasswordAuthenticationFilter.class);
    }
```
