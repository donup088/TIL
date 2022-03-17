## 브라우저에 www.google.com을 검색하면 생기는 일
1. 브라우저는 캐싱된 DNS 기록들을 통해 www.google.com에 대응되는 IP주소가 있는지 확인한다.
    - DNS(Domain Name System)는 URL들의 이름과 IP 주소를 저장하고 있는 데이터베이스이다. 인터넷에 있는 모든 URL들은 고유의 IP 주소가 지정되어 있다.
2. 요청한 URL이 캐시에 없으면 ISP의 DNS 서버가 www.google.com을 호스팅하고 있는 서버의  IP주소를 찾기 위해 DNS query를 날린다.
    - IP주소를 찾을 때 까지 DNS 서버에서 다른 DNS 서버를 오가면서 검색을 진행한다.
3. 브라우저가 서버와 TCP connection을 한다. 
    - TCP/IP three-way handshake
    1. 클라이언트가 SYN 패킷을 서버에 보내고 connection을 열어달라고 요청한다.
    2. 서버가 새로운 connection을 시작할 수 있는 포트가 있다면 SYN/ACK 패킷으로 응답을 한다.
    3. 클라이언트는 SYN/ACK 패킷을 서버로 부터 받으면 서버에게 ACK 패킷을 보낸다. 
    이 과정을 통해서 TCP connection이 이워진다.
4. 브라우저가 웹 서버에 HTTP 요청을 한다.
    - TCP로 연결이 되어 있기 때문에 데이터를 전송하면 된다.
    - GET 요청을 통해 서버에세 www.google.com 웹페이지를 요청한다. 이 요청을 보낼 때 여러가지 부가정보들도 함께 전달이 된다. ex) User-Agent 헤더, Accept 헤더, 쿠키정보 등
5. 서버가 요청을 처리하고 response를 생성한다.
6. 서버가 HTTP response를 보낸다.
    - 서버의 response에는 요청한 웹페이지, status code, 쿠키, Cache-Control 등 여러가지 정보가 포함되어 있다.
7. 브라우저가 HTML content를 보여준다.

## TCP/UDP
TCP는 신뢰성을 보장하는 프로토콜로 따로 신뢰성 보장이 없는 UDP에 속도가 느리다. 따라서 TCP는 신뢰성에 중요한 서비스에 사용되고 UDP는 스트리밍 같은 연속성이 더 중요한 서비스에 사용된다.

## 3-way handshake , 4-way handshake
- 3-way handshake : 클라이언트와 서버 연결
1. 클라이언트가 서버에게 접속을 요청하는 SYN 패킷을 보낸다.
2. 서버는 SYN 패킷을 받고 요청을 수락한다는 SYN/ACK 패킷을 보낸다.
3. 클라이언트는 서버로부터 SYN/ACK 를 받고 응답으로 ACK 패킷을 보낸다.

- 4-way handshake : 클라이언트와 서버 연결 해제
1. 클라이언트가 연결을 종료하겠다는 FIN 패킷을 보내고 FIN_WAIT 상태가 된다.
2. 서버는 클라이언트로부터 FIN을 받고 ACK 패킷을 보낸다 상태는 CLOSE_WAIT 상태가 된다.
3. 서버가 통신이 끝나고 연결 종료 준비 상태가 되면 클라이언트에게 FIN 패킷을 보내고 LAST_WAIT 상태가 된다.
4. 클라이언트는 ACK 패킷을 보내고 TIME_WAIT 상태가 된다.

서버는 클라이언트로부터 ACK 패킷을 받고 종료되고 클라이언트는 뒤늦게 도착하는 패킷을 대비하여 클라이언트는 ACK 패킷을 보낸 뒤 일정 시간 뒤에 종료된다. 

## HTTP vs HTTPS
HTTPS는 중간에 암호화 계층을 거쳐서 패킷을 암호화하기 때문에 패킷을 가로채거나 수정하는것을 방어할 수 있다.

## CORS (Cross Origin Resource Sharing)
CORS 문제는 보통 프론트엔드 개발시 로컬에서 API 서버에 요청을 보낼 때 흔하게 발생한다.
서로 다른 도메인간 자원을 공유하는 것을 뜻하며 대부분 브라우저에서는 이를 기본적으로 차단하며 서버측에서 헤더를 통해서 사용가능한 자원을 알려준다.

### REST API
HTTP URI를 통해 자원을 명시하고 HTTP Method를 통해 해당 자원에 대한 CRUD를 적용하는 것을 의미한다.
- REST API 특징
    1. 서버 클라이언트 구조
    2. 무상태
    3. 캐시 처리 가능
    4. 계층화
    5. 인터페이스의 일관성
스프링에서는 Self-descriptiveness 한 특징을 구현하기 위해 HATEOAS 라는 것을 사용할 수 있다. 이를 사용하면 동적인 API를 클라이언트에 제공하여 클라이언트가 API 변화에 일일이 대응하지 않아도 된다.

