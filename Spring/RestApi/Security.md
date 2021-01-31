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