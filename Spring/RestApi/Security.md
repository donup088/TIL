### 스프링 시큐리티
- 웹 시큐리티(Filter 기반)
- 메소드 시큐리티
- Security Interceptor를 사용
    - Method와 Filter 가 있다.

- Security Interceptor가 AuthenticationManager를 사용하여 인증을 하고 인증정보를 SecurityContextHolder에 담아놓는다.

- UserDetailsService 구현
    - loadUserByUsername 구현
        - security의 User를 반환할 수 있도록 한다.
        ```
        Account account = accountRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException(username));
        
        return new User(account.getEmail(), account.getPassword(), authorities(account.getRoles()));
        ```
    - SimpleGrantedAuthority 사용
        ```
        private Collection<? extends GrantedAuthority> authorities(Set<AccountRole> roles) {
            return roles.stream()
                    .map(r -> new SimpleGrantedAuthority("ROLE_" + r.name())).collect(Collectors.toSet());
        }
        ```

### Auth 인증서버 설정
- Grant Type: 토큰을 받아오는 방법
- 구글이나 페이스북에서 만든 것이 아닌 직접 만든 클라이언트에서 사용하는 Grant Type으로 password를 사용할 수 있다. password로 Grant 방식을 사용하면 주어야하는 정보는 grant_type,username,password,client_id,client_secret이 있다.
- 적용을 하게 되면 /oauth/token 에 post 방식으로 토큰을 요청할 수 있다.
- @EnableAuthorizationServer 추가한다.
- AuthorizationServerConfigurerAdapter을 상속받아서 메서드를 오버라이드한다.
    - inMemory 대신에 jdbc를 사용하는 것이 좋다.
    ```
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.passwordEncoder(passwordEncoder);
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("myApp")
                .authorizedGrantTypes("password", "refresh_token")
                .scopes("read", "write")
                .secret(passwordEncoder.encode("pass"))
                .accessTokenValiditySeconds(10 * 60)
                .refreshTokenValiditySeconds(6 * 10 * 60);
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager)
                .userDetailsService(accountService)
                .tokenStore(tokenStore);
    }
    ```

### 현재 로그인한 유저 가져오기
- AccountAdapter 클래스를 만들어서 현재 유저 정보도 사용할 수 있도록 만든다.
    ```
    public class AccountAdapter extends User {
        private Account account;

        public AccountAdapter(Account account) {
            super(account.getEmail(), account.getPassword(), authorities(account.getRoles()));
            this.account=account;
        }

        private static Collection<? extends GrantedAuthority> authorities(Set<AccountRole> roles) {
            return roles.stream()
                    .map(r -> new SimpleGrantedAuthority("ROLE_" + r.name()))
                    .collect(Collectors.toSet());
        }

        public Account getAccount() {
            return account;
        }
    }
    ```

- 스프링 시큐리티에서 유저를 조회하여 가져오는 것은 UserDetails를 리턴하는 loadUserByUsername 을 통해서 가져온다. loadUserByUsername 메소드에서 AccountAdapter를 return 하도록 만들어줘서 Account로 바로 조회할 수 있도록 만들어준다.

- 커스텀 애노테이션 @CurrentUser 를 만들어서 사용에 편리성을 준다.
    - expression에 유저에 anonymousUser일 경우에는 문자열로 조회가 되기 때문에 anonymousUser 경우에는 null로 반환할 수 있도록 한다.
    - expession은 조회해오는 데이터를 getter로 expression에 명시된 이름의 객체를 바로 가져올 수 있도록 해준다.
    ```
    @Target(ElementType.PARAMETER)
    @Retention(RetentionPolicy.RUNTIME)
    @AuthenticationPrincipal(expression = "#this == 'anonymousUser' ? null : account")
    public @interface CurrentUser {
    }
    ```
