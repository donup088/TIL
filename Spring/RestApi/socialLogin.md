## 카카오 로그인 & jwt 토큰
### 과정
- 클라이언트에서 카카오 로그인까지 완료.
- api 서버로 access_token을 전달
- 전달 받은 access_token 활용하여 카카오 인증 유저 정보를 토대로 DB에 저장
- 로그인시 jwt 토큰 생성
### 과정 구현
- 카카오 로그인 연동을 위해서 카카오 개발자 사이트에 들어간뒤 어플리케이션을 등록한다.
- 등록한 어플리케이션에서 login redirect주소 로그인시 동의여부 등 여러 설정을 추가할 수 있다.
- loginUrl 
    ```
    StringBuilder loginUrl = new StringBuilder()
                .append(env.getProperty("spring.social.kakao.url.login"))
                .append("?client_id=").append(kakaoClientId)
                .append("&response_type=code")
                .append("&redirect_uri=").append(baseUrl).append(kakaoRedirect);
    ```
    -  https://kauth.kakao.com/oauth/authorize?client_id=???&response_type=code&redirect_uri=http://localhost:8080/social/login/kakao 과 같은 url이 만들어진다.
    - 이렇게 생긴 url로 접근을 하면 access_token을 얻을 수 있다.

- 카카오 인증 완료후 리다이렉트처리
    - 카카오 로그인을 하게되면 code가 나온다. code를 param에 포함하여 https://kauth.kakao.com/oauth/token 으로
    요청하게 되면 accessToken을 얻을 수 있다.
    ```
     public KakaoAuth getKakaoTokenInfo(String code) {
        // Set header : Content-type: application/x-www-form-urlencoded
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        // Set parameter
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("grant_type", "authorization_code");
        params.add("client_id", kakaoClientId);
        params.add("redirect_uri", baseUrl + kakaoRedirect);
        params.add("code", code);
        // Set http entity
        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(params, headers);

        ResponseEntity<String> response = restTemplate.postForEntity(env.getProperty("spring.social.kakao.url.token"), request, String.class);
        System.out.println("response = " + response.getStatusCode());
        if (response.getStatusCode() == HttpStatus.OK) {
            KakaoAuth kakaoAuth = gson.fromJson(response.getBody(), KakaoAuth.class);
            System.out.println("kakaoAuth = " + kakaoAuth);
            return kakaoAuth;
        }
        return null;
    }
    ```

- login 하여 얻은 accessToken을 사용하여 user 정보를 만들어서 저장한다.
    - 이 떄 카카오로 부터 얻은 profile에는 고유 아이디가 있기 때문에 나중에 유저정보를 찾아오기
    위해서 profile 고유아이디와 어떤 provider인지 둘을 판단하여 유저정보를 가져온다. 이를 위해서 
    member 엔티티에 uid라는 것을 만들고 profile 고유아이디를 저장할 수 있도록 하였다.
    ```
    @PostMapping("/signup/{provider}")
    public ResponseEntity signupProvider(@PathVariable("provider") String provider,
                                         @RequestParam("accessToken") String accessToken,
                                         @RequestParam("name") String name) {
        System.out.println("signupProvider");
        KakaoProfile profile = kakaoService.getKakaoProfile(accessToken);

        memberRepository.save(Member.builder()
                .uid(profile.getId())
                .provider(provider)
                .nickname(name)
                .roles(List.of(MemberRole.USER))
                .build());
        return ResponseEntity.ok("성공하였습니다.");
    }
    ```
    - getKakaoProfile 매서드 
        - 헤더에 login하여 받은 accessToken을 담아서 https://kapi.kakao.com/v2/user/me url로 
        request를 보내서 유저정보를 알아오는 과정이다.
        ```
        public KakaoProfile getKakaoProfile(String accessToken) {
            // Set header : Content-type: application/x-www-form-urlencoded
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
            headers.set("Authorization", "Bearer " + accessToken);

            // Set http entity
            HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(null, headers);
            try {
                // Request profile
                ResponseEntity<String> response = restTemplate.postForEntity(env.getProperty("spring.social.kakao.url.profile"), request, String.class);
                if (response.getStatusCode() == HttpStatus.OK) {
                    KakaoProfile kakaoProfile = gson.fromJson(response.getBody(), KakaoProfile.class);
                    System.out.println("kakaoProfile = " + kakaoProfile);
                    return kakaoProfile;
                }
            } catch (Exception e) {
                throw new CCommunicationException();
            }
            throw new CCommunicationException();
        }
        ```
- 카카오 인증을 하고 사용자가 DB에 저장되었다면 로그인을 하고 jwt 토큰을 생성해준다.
    ```
    @PostMapping("/signin/{provider}")
    public ResponseEntity signinByProvider(@PathVariable String provider,
                                           @RequestParam String accessToken) {
        KakaoProfile profile = kakaoService.getKakaoProfile(accessToken);
        Member member = memberRepository.findByUidAndProvider(profile.getId(), provider).get();
        String jwt = tokenProvider.createToken(member.getEmail());
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.add(JwtFilter.AUTHORIZATION_HEADER, "Bearer " + jwt);
        return new ResponseEntity(new TokenDto(jwt), httpHeaders, HttpStatus.OK);
    }
    ```