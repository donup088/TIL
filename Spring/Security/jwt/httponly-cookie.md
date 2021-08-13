## Spring boot Jwt httponly cookie 사용

### HttpOnly Cookie 왜 사용하나??
- HttpOnly Cookie를 사용하면 js에서 쿠키에 접근을 할 수 없다. 그래서 XSS 공격으로 쿠키 정보를 탈취 할 수 없다.
- localStorage 등 다른 스토리지에 비해 안전하게 보관할 수 있다.

### Refresh Token 과 HttpOnly Cookie 로 안전하기 토큰 관리하기
- Refresh Token으로 CSRF 공격을 방어
- httpOnly Cookie 사용으로 XSS 공격 방어

### Cookie 관련 설정
- Cookie는 일반적으로 같은 도메인 내에서만 공유가 가능하다. 같은 도메인이라고 하면 포트번호까지 같아야한다.
- localhost:3000 -> localhost:8080 으로 쿠키를 전송하기 위해 클라이언트에서 axios 설정에 axios.defaults.withCredentials = true; 을 추가해줘야한다.
    - 이 옵션을 추가해줄 경우 springboot 서버에서는 cors 설정을 추가해줘야한다.
        ```
          @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**")
                    .allowedOrigins("http://localhost:3000")
                    .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
                    .allowedHeaders("*")
                    .allowCredentials(true)
                    .maxAge(3600);
        }
        ```
        - allowCredentials 을 true 로 설정해줘야하고 이 때문에 allowedOrigins에 구체적인 도메인을 적어줘야한다.
- springboot 서버에서도 쿠키관련 설정을 해줘야한다.
    ```
    ResponseCookie cookie = ResponseCookie.from(REFRESH_TOKEN, refreshToken)
                .httpOnly(true)
                .path("/")
                .maxAge(14 * 24 * 60 * 60)
                .build();

    response.addHeader("Set-Cookie", cookie.toString());
    ```
    - https 끼리 요청,응답을 주고 받는 다면 secure 옵션을 true로 해줄 수 있다. 하지만 http에서 사용할 경우 secure 옵션을 false로 해야한다.
    - Cookie의 도메인을 설정하지 않을 경우 기존 도메인으로 설정된다.
    - SameSite 설정시 None으로 할 경우 자동으로 Secure 옵션으로 된다. default는 lax 로 설정되어있다.