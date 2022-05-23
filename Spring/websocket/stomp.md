### Stomp란?
메시징 전송을 효율적으로 하기 위해 나온 프로토콜로 기본적으로 pub/sub 구조로 되어 메시지를 발송하고 메시지를 받아 처리하는 부분이 확실히 정해져 있기 때문에 개발하는 입장에서 명확하게 인지하고 개발할 수 있다. 또한 통신 메시지의 헤더에 값을 세팅할 수 있어 헤더 값을 기반으로 통신 시 인증처리를 구현하는 것도 가능하다.

- pub/sub 구조
pub/sub 이란 메시지를 공급하는 주체와 소비하는 주체를 분리하여 제공하는 방법이다.
채팅방 생성 -> pub/sub 구현을 구현 topic 하나 생성
채팅방 입장 -> topic 구독
채팅방에서 메시지를 주고 받기 -> 해당 topic 으로 메시지 발송(pub), 메시지 받기(sub)

### 의존성 추가
websocket 의존성 추가
```
implementation 'org.springframework.boot:spring-boot-starter-websocket'
```

### WebSocketConfig 설정
- Stomp를 사용하기 위해 @EnableWebSocketMessageBroker 선언
- 메시지를 발행하는 요청의 prefix 를 /pub으로 메시지를 구독하는 요청의 prefix 는 /sub으로 설정
- stomp websocket의 연결 endpoint는 /chat으로 설정
```
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/chat").setAllowedOrigins("http://localhost:3000").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/sub");
        registry.setApplicationDestinationPrefixes("/pub");
    }
}
```

### ChatController
- @MessageMapping을 통해 Websocket으로 들어오는 메시지 발행 처리
- 클라이언트에서 prefix를 붙여서 /pub/chat/message로 발행 요청을 하면 Controller가 해당 메시지를 받아 처리한다.
- 메시지가 발행되면 /sub/chat/room/{roomId} 로 메시지를 send 하는데 클라이언트에서는 해당 주소를 구독하고 있다가 메시지가 전달되면 화면에 출력하면 된다.
- /sub/chat/room/{roomId} 는 채팅룸을 구분하는 값으로 pub/sub에서 Topic의 역할을 한다고 볼 수 있다.

```
@RestController
@RequiredArgsConstructor
public class MessageController {
    private final SimpMessageSendingOperations messageTemplate;

    @MessageMapping("/chat/message")
    public void message(MessageRequest message) {
        if (message.getMessageType().equals(MessageRequest.MessageType.ENTER)) {
            message.setEnterMessage();
        }
        messageTemplate.convertAndSend("/sub/chat/room/" + message.getRoomId(), message);
    }
}
```

### 외부 메시지 브로커가 필요한 이유
위와 같은 환경에서 메시지 브로커, 메시지 큐는 스프링 부트 서버의 내부 메모리에 존재한다. 따라서 웹소켓 서버가 여러개일 때는 위와 같은 시스템으로 운영할 수 없다.

예를 들어 서버가 2대인 경우 사용자가 3명으로 모두 같은 채널을 구독하고 있다. 같은 채널을 구독 중이라면 발행 메시지는 사용자 3명 모두에게 전송되어야 한다. 
서버 1,2가 있고 사용자1,2는 서버1에 있는 채널에 구독중이고, 사용자 3은 서버2의 채널을 구독중이다. 이 때 발행자가 서버1로 메시지를 발행한다면 사용자1,2,3은 모두 같은 채널을 구독하고 있지만 사용자3은 메시지를 받지 못하게된다.

이와 같은 상황에서 외부 메시지 브로커에서 메시지 큐를 관리하면 된다.

발행자가 채널 test에 구독중인 사용자에게 메시지를 보내고 싶은 상황이다. 사용자 1,2는 서버1에 웹소켓을 연결중이고 사용자 3,4는 서버2에 웹소켓에 연결중이다. 사용자 1,2,3은 모두 같은 test 채널을 구독중이고 사용자 4는 다른 채널을 구독중이다. 발행자가 서버1에 메시지를 보냈을 때 서버1에서는 메시지 브로커의 exchange에 메시지를 전달한ㄷ나. exchange에 바인딩 되어 있는 사용자 1,2,3 큐에 메시지를 전달하게 되고 서버 2에 연결중인 사용자 3 에게도 메시지를 보낼 수 있다.

인메모리 기반 시스템은 메시지 유실 가능성이 있지만 외부 메시지 브로커를 사용하고 있다면 서버가 재실행시 외부 브로커에 저장중인 큐에 대기중인 메시지를 수신할 수 있다. 또한 인메모리 기반 시스템은 메시지 모니터링이 쉽지 않다. 여러가지 이슈로 인해 외부 브로커를 사용하는 것이 좋지만 인프라 비용이 증가하는 단점이 있다. 따라서 상황에 맞게 시스템 아키텍처를 구성해야한다.


- 참고
    - https://brunch.co.kr/@springboot/695,
    - https://daddyprogrammer.org/post/4691/spring-websocket-chatting-server-stomp-server/