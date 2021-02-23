### HTTPS
- HTTP에 암호화나 인증 등의 구조를 더한 것을 HTTPS(HTTP Secure) 라고 한다.
- HTTP 통신을 하는 소켓 부분을 SSL(Secure Socket Layer) or TLS(Transport Layer Security) 프로토콜로 대체하고 있다.
- 구조
    - 애플리케이션(HTTP)
    - SSL
    - TCP
    - IP

### 암호화 방식
- 공개키 암호화 방식
- 공개키를 사용해서 암호화하고 비밀키를 사용해서 복호화한다.

### SSL 단점
- 통신속도가 떨어진다.
- CPU나 메모리 등의 리소스를 다량으로 소비함으로써 처리가 느려진다.
- 암호화와 복호화를 하는 것이기 때문에 접근이 많은 웹페이지에서 부담이 커진다.
- 증명서가 필요하기 때문에 CA에서 증명서를 구입해야한다.