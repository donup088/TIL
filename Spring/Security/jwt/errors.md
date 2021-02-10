### 토큰 인증시 마주치는 Exception
- AuthenticationException
1. UsernameNotFoundException : 계정이 없는 경우
2. BadCredentialsException : 비밀번호 불일치
3. AccountExpiredException: 계정 만료
4. CredentialExpiredException : 비밀번호 만료
5. DisabledException : 계정 비활성화
6. LockedException : 계정잠김

### Provider의 authenticate 메소드 결과 던져지는 에러처리
- provider 예시코드
    ```
    @Component
    @RequiredArgsConstructor
    @Slf4j
    public class LoginAuthenticationProvider implements AuthenticationProvider {
        private final PasswordEncoder passwordEncoder;
        private final MemberRepository memberRepository;

        @Override
        public Authentication authenticate(Authentication authentication) throws AuthenticationException {
            String username = (String) authentication.getPrincipal();
            String password = (String) authentication.getCredentials();

            Member member = memberRepository.findByEmail(username)
                    .orElseThrow(() -> new UsernameNotFoundException("입력하신 이메일의 해당하는 계정이 없습니다."));

            if (isCorrectPassword(password, member)) {
                return PostAuthorizationToken.getTokenFromAccountContext(MemberAdapter.fromMemberModel(member));
            }

            throw new BadCredentialsException("비밀번호가 잘못되었습니다.");
        }

        @Override
        public boolean supports(Class<?> aClass) {
            return UsernamePasswordAuthenticationToken.class.isAssignableFrom(aClass);
        }

        private boolean isCorrectPassword(String password, Member member) {
            return passwordEncoder.matches(password, member.getPassword());
        }
    }
    ```

- provider를 작성하고 securityConfig에 등록하면 작성한 provider의 authenticate메소드를 사용하여 로그인 처리를 한다.

- 이 클래스에서 던져지는 오류들은 AuthenticationEntryPoint를 implements하여 구현한 클래스에서 잡을 수 있다.
    ```
    @Component
    @RequiredArgsConstructor
    @Slf4j
    public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {

        private final ObjectMapper objectMapper;

        @Override
        public void commence(HttpServletRequest request,
                            HttpServletResponse response,
                            AuthenticationException authException) throws IOException {
            // 유효한 자격증명을 제공하지 않고 접근하려 할때 401
            log.info("commence authException.class: " + authException.getClass());
            log.info(authException.getMessage());
            if (authException instanceof UsernameNotFoundException) {
                getResponse(response, "UsernameNotFoundException", "해당 이메일의 계정이 없습니다.");
            }
            if (authException instanceof InsufficientAuthenticationException) {
                getResponse(response, "InsufficientAuthenticationException", "토큰이 잘못되었습니다.");
            }
            if (authException instanceof BadCredentialsException) {
                getResponse(response, "BadCredentialsException", "비밀번호가 잘못되었습니다.");
            }
        }

        private void getResponse(HttpServletResponse response, String error, String message) throws IOException {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.setContentType("application/json;charset=utf-8");
            ApiError apiError = new ApiError(HttpStatus.UNAUTHORIZED, error, message);

            response.getWriter().write(objectMapper.writeValueAsString(apiError));
        }
    }
    ```
- commence 메소드에서 AuthenticationException에 들어오는 오류의 종류에 따라 분리하여 json형태로 반환할 수 있다.

- 권한과 관련되어 오류를 잡을 수 있는 방법은 AccessDeniedHandler를 implements한 클래스를 구현하여 작성한다.
    ```
    @Component
    @RequiredArgsConstructor
    public class JwtAccessDeniedHandler implements AccessDeniedHandler {

        private final ObjectMapper objectMapper;
        @Override
        public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException {
            //필요한 권한이 없이 접근하려 할때 403
            response.setStatus(HttpServletResponse.SC_FORBIDDEN);
            response.setContentType("application/json;charset=utf-8");
            ApiError apiError=new ApiError(HttpStatus.UNAUTHORIZED,"Wrong GrantType","권한이 없습니다.");

            response.getWriter().write(objectMapper.writeValueAsString(apiError));
        }
    }
    ```
