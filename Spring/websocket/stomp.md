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