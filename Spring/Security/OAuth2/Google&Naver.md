## OAuth2 
---
#### 구글 로그인 연동

- 구글 서비스 등록
    - 구글 클라우드 플랫폼으로 가서 새 프로젝트 생성
    - 왼쪽 메뉴탭을 클릭하여 API 및 서비스 카테고리로 이동 후 사용자 인증 정보 만들기
    - OAuth 클라이언트 ID 만들기, 동의 화면 입력, 하단에 승인된 리디렉션 URI http://localhost:8080/login/oauth2/code/google로 설정 (스프링 부트 2 버전의
      시큐리티에서는 기본적으로 다음과 같이 설정함)
    - 생성된 클라이언트 목록에 가서 클라이언트 ID와 클라이언트 보안비밀을 프로젝트에서 설정
- application.yml 파일에 추가
  ```
  spring:
    security:
      oauth2:
        client:
          registration:
            google:
              client-id: 클라이언트 아이디
              client-secret: 클라이언트 보안비밀
              scope:
                - email
                - profile
  ```
- compile('org.springframework.boot:spring-boot-starter-oauth2-client') build.gradle에 추가
- CustomOAuth2UserService 구현
  ```
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest,OAuth2User> auth2UserService = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = auth2UserService.loadUser(userRequest);
        
        String registrationId = userRequest.getClientRegistration().getRegistrationId(); 
        String userNameAttributeName = userRequest.getClientRegistration()
                .getProviderDetails().getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.
                of(registrationId, userNameAttributeName, oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);
        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());
    }

    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(),attributes.getPicture()))
                .orElse(attributes.toEntity());

        return userRepository.save(user);
    }
  ```
    - registrationId 는 현재 로그인 진행 중인 서비스를 구분하는 코드 ex) 구글인지 네이버인지
    - userNameAttributeName 은 OAuth2 로그인 진행 시 키가 되는 필드 값 (구글의 경우 기본적으로 코드 지원 "sub")
    - OAuthAttributes OAuth2User의 attribute를 담을 클래스
    - SessionUser 세션에 사용자 정보를 저장하기 위한 Dto 클래스
    - 구글 사용자 정보가 업데이트 되었을 때 대비하여 update 구현
- OAuthAttributes 클래스
  ```
      @Getter
      public class OAuthAttributes {
      private Map<String, Object> attributes;
      private String nameAttributeKey;
      private String name;
      private String email;
      private String picture;
          
       ...
  
      public static OAuthAttributes of(String registrationId,
                                       String userNameAttributeName,
                                       Map<String, Object> attributes) {
          return ofGoogle(userNameAttributeName, attributes);
      }
  
      private static OAuthAttributes ofGoogle(String userNameAttributeName,
                                              Map<String, Object> attributes) {
          return OAuthAttributes.builder()
                  .name((String) attributes.get("name"))
                  .email((String) attributes.get("email"))
                  .picture((String) attributes.get("picture"))
                  .attributes(attributes)
                  .nameAttributeKey(userNameAttributeName)
                  .build();
      }
      ...
    }
  ```
- SessionUser 클래스
  ```
    @Getter
  public class SessionUser implements Serializable {
  private String name;
  private String email;
  private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
  }
  ```
  - Serializable(직렬화) 사용
  - 자바 직렬화란 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 
    바이트(byte) 형태로 데이터 변환하는 기술이다.
- SecurityConfig에 CustomOAuth2UserService 사용하도록 설정
  ```
     http.oauth2Login()
                  .userInfoEndpoint()
                  .userService(customOAuth2UserService)
  ```
---
#### 네이버 로그인 연동
- 네이버 API 등록
  - 네이버 오픈 API로 이동 후 네이버 서비스 등록
  - 서비스 URL http://localhost:8080/ 
  - Callback URL : http://localhost:8080/login/oauth2/code/naver
  - 네이버 서비스 등록 완료 후 클라이언트 아이디와 클라이언트 Secret을 프로젝트에 설정
- 프로젝트 설정
  application.yml 에 추가
  ```
  spring:
  security:
    oauth2:
      client:
        registration:
          naver:
            client-id: 클라이언트 아이디
            client-secret: 클라이언트 Secret
            scope:
              - name
              - email
              - profile_image
            redirect_uri: "{baseUrl}/{action}/oauth2/code/{registrationId}"
            authorization_grant_type: authorization_code
            client-name: Naver


        provider:
          naver:
            authorization_uri: https://nid.naver.com/oauth2.0/authorize
            token_uri: https://nid.naver.com/oauth2.0/token
            user-info-uri: https://openapi.naver.com/v1/nid/me
            user_name_attribute: response
  ```
- OAuthAttributes 클래스에 네이버인지 판단하는 코드 추가
  ```
   public static OAuthAttributes of(String registrationId,
                                     String userNameAttributeName,
                                     Map<String, Object> attributes) {
        if("naver".equals(registrationId)){
            return ofNaver("id",attributes);
        }
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
        Map<String,Object> response= (Map<String, Object>) attributes.get("response");
        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
  ```