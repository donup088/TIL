## api 통신을 하는 프로젝트에서 카카오톡 로그인 하기

### 카카오 로그인이 되는 과정
- 카카오 개발자 센터 로그인하고 해야할 것
    - 앱 생성하고 client key, secret key 발급받기
    - Redirect URI 설정
1. front에서 window.location.href = 'http://localhost:8080/oauth2/authorization/kakao'; 요청을 보낸다.
2. backend에서 소셜로그인이 성공하면 카카오 에 인증 코드를 요청한다.
3. 받은 인증 코드를 이용해 access_token을 요청한다.
4. 받은 access_token으로 resource에 접근한다.
5. 로직이 모두 통과하면 access_token을 만들어서 클라이언트로 전달한다.
6. 소셜 로그인이 모두 성공하면 'http://localhost:3000/token=accesstoken' 으로 리다이렉트 시킨다. 
7. front에서 token을 받아 저장하고 홈페이지로 이동시킨다.

### Vue 
- 카카오톡 로그인 버튼을 누르면 window.location.href = 'http://localhost:8080/oauth2/authorization/kakao'로 location을 이동시킨다.

### Spring boot
- implementation 'org.springframework.security:spring-security-oauth2-client' 의존성 추가
- CustomOAuth2Provider 생성
    ```
    public enum CustomOAuth2Provider {
        KAKAO {
            @Override
            public ClientRegistration.Builder getBuilder() {
                return getBuilder("kakao", ClientAuthenticationMethod.POST)
                        .scope("profile_nickname")  // 요청 할 권한
                        .authorizationUri("https://kauth.kakao.com/oauth/authorize") //authorization code 얻는 API
                        .tokenUri("https://kauth.kakao.com/oauth/token")  //access Token 얻는 API
                        .userInfoUri("https://kapi.kakao.com/v2/user/me") // 유저 정보 조회 API
                        .userNameAttributeName("id") // userInfo API Response에서 얻어올 ID 프로퍼티
                        .clientName("Kakao"); // spring 내에서 인식할 OAuth2 Provider Name
            }
        };

        private static final String DEFAULT_LOGIN_REDIRECT_URL = "{baseUrl}/login/oauth2/code/{registrationId}";

        protected final ClientRegistration.Builder getBuilder(String registrationId, ClientAuthenticationMethod method) {
            ClientRegistration.Builder builder = ClientRegistration.withRegistrationId(registrationId);
            builder.clientAuthenticationMethod(method);
            builder.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE);
            builder.redirectUri(CustomOAuth2Provider.DEFAULT_LOGIN_REDIRECT_URL);
            return builder;
        }

        public abstract ClientRegistration.Builder getBuilder();
    }
    ```
- OAuth2Config 클래스 생성
    ```
    @Configuration
    public class OAuth2Config {
        @Bean
        public ClientRegistrationRepository clientRegistrationRepository(@Value("${kakao.client.id}") String clientId, @Value("${kakao.client.secret}") String clientSecret) {
            final ClientRegistration clientRegistration = CustomOAuth2Provider.KAKAO
                    .getBuilder()
                    .clientId(clientId)
                    .clientSecret(clientSecret)
                    .build();

            return new InMemoryClientRegistrationRepository(Collections.singletonList(clientRegistration));
        }
    }
    ```
- OAuth2ClientService 생성
    ```
    @Service
    @RequiredArgsConstructor
    public class OAuth2ClientService implements OAuth2AuthorizedClientService {
        private final UserRepository userRepository;

        @Override
        public void saveAuthorizedClient(OAuth2AuthorizedClient oAuth2AuthorizedClient, Authentication authentication) {
            OAuth2User oauth2User = (OAuth2User) authentication.getPrincipal();
            Long id = Long.valueOf(oauth2User.getAttributes().get("id").toString());
            if (!userRepository.existsUserByKakaoId(id)) {
                String nickname = (String)
                        ((LinkedHashMap)
                                ((LinkedHashMap) oauth2User.getAttribute("kakao_account"))
                                        .get("profile"))
                                .get("nickname");

                userRepository.save(User.createKakao(id, nickname));
            }
        }

        @Override
        public <T extends OAuth2AuthorizedClient> T loadAuthorizedClient(String clientRegistrationId, String principalName) {
            throw new UnsupportedOperationException();
        }

        @Override
        public void removeAuthorizedClient(String clientRegistrationId, String principalName) {
            throw new UnsupportedOperationException();
        }
    }
    ```
- OAuth2SuccessHandler 생성
    ```
    @Component
    @RequiredArgsConstructor
    public class OAuth2SuccessHandler implements AuthenticationSuccessHandler {
        private final TokenProvider tokenProvider;
        private final RefreshTokenRepository refreshTokenRepository;
        private final UserRepository userRepository;
        @Value("${oauth2.success.redirect.url}")
        private String url;

        @Override
        public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException {
            OAuth2User oauth2User = (OAuth2User) authentication.getPrincipal();
            String kakaoId = String.valueOf(oauth2User.getAttributes().get("id"));
            User user = userRepository.findByKakaoId(Long.valueOf(kakaoId)).orElseThrow(UserNotFoundException::new);

            Token token = tokenProvider.generateAccessToken(user);
            Optional<RefreshToken> optionalRefreshToken = refreshTokenRepository.findByUserId(user.getId());
            Token createdRefreshToken = tokenProvider.generateRefreshToken();

            if (optionalRefreshToken.isPresent()) {
                optionalRefreshToken.get().update(createdRefreshToken.getToken(), createdRefreshToken.getExpiredAt());
            } else {
                RefreshToken refreshToken = RefreshToken.create(createdRefreshToken.getToken(), createdRefreshToken.getExpiredAt(), user);
                refreshTokenRepository.save(refreshToken);
            }

            response.sendRedirect(url + token.getToken());
        }
    }
    ```
- security config에 OAuth2SuccessHandler 등록
    ...
     .and()
                .oauth2Login()
                .successHandler(oAuth2SuccessHandler);
    ```
### refresh 토큰 처리하기
- refresh 토큰 관리
    - 백엔드에서 유저가 로그인할 때 refresh token을 만들어주고 DB에 저장한다.
    - refresh token이 이미 DB에 존재한다면 update 해준다.
- 전체적인 흐름
1. 만료된 access_token을 가지고 api 요청을 보내면 401 error가 난다.
2. 401 error가 나면 vue interceptor에서 에러를 받아서 로그인한 유저의 userId와 함께 access_token 재발급 요청을 보낸다.
3. 받은 userId로 백엔드에서 user를 찾고 user에 해당하는 refreshToken의 만료기간을 확인하고 지나지 않았으면 새로운 access_token을 발급한다.
4. refreshToken이 만료되었다면 에러를 던지고 front에서 로그인 페이지로 리다이렉트한다.
5. 새로운 access_token으로 api를 다시 요청한다.
- vue interceptor 부분
    ```
    instance.interceptors.response.use(
            response => {
                return response;
            },
            async error => {
                const {
                    config,
                    response: { status },
                } = error;
                if (status === 401) {
                    const userRefreshReq = {
                        userId: store.getters['user/getUserId'],
                    };
                    try {
                        const { data } = await refreshToken(userRefreshReq);
                        localStorage.setItem('accessToken', data.accessToken);
                        error.response.config.headers.Authorization =
                            'Bearer ' + data.accessToken;
                    } catch (error) {
                        router.push('/login');
                    }
                    return axios(config);
                }

                return Promise.reject(error);
            },
        );
    ```
- spring userService 부분
    ```
    public UserResponse.Login refreshToken(UserRequest.RefreshToken request) {
        User user = userRepository.findById(request.getUserId()).orElseThrow(UserNotFoundException::new);
        RefreshToken refreshToken = refreshTokenRepository.findByUserId(request.getUserId()).orElseThrow(RefreshTokenNotFoundException::new);
        if (!refreshToken.verifyExpiration()) {
            throw new RefreshTokenExpiredException();
        }
        Token token = tokenProvider.generateRefreshToken();
        refreshToken.update(token.getToken(), token.getExpiredAt());

        return UserResponse.Login.build(user.getId(), user.getName(), tokenProvider.generateAccessToken(user));
    }
    ```
    

