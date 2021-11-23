## Spring boot mail 사용하기 및 java interface를 사용하여 다형성 잘 사용해보기
- 의존성 추가
```
implementation 'org.springframework.boot:spring-boot-starter-mail'
```



### 다형성 적용해보기
- mail을 여러가지 상태에따라 다르게 보내야하는 상황, 메일 제목, 내용만 다르고 다른 코드들은 중복이 될 것 같아서 중복을 없애보았다.
- mail condition class -> interface를 만들어서 해당메서드를 오버라이딩하도록 설계하였다.
    ```
    public interface MailCondition {
        SimpleMailMessage makeSimpleMessage(String toEmail, String fromEmail);
    }
    ```

- mail condition implements class
    ```
    public class ApproveCondition implements MailCondition {
        @Override
        public SimpleMailMessage makeSimpleMessage(String toEmail, String fromEmail) {
            SimpleMailMessage message = new SimpleMailMessage();
            message.setTo(toEmail);
            message.setFrom(fromEmail);
            message.setSubject("모집 확정 알림");
            message.setText("지원하신 게시물에 참여 확정 되었습니다.");
            return message;
        }
    }
    ```
    ```
    public class RejectCondition implements MailCondition {
        @Override
        public SimpleMailMessage makeSimpleMessage(String toEmail, String fromEmail) {
            SimpleMailMessage message = new SimpleMailMessage();
            message.setTo(toEmail);
            message.setFrom(fromEmail);
            message.setSubject("모집 반려 알림");
            message.setText("지원하신 게시물에서 반려 처리 되었습니다.");
            return message;
        }
    }

    ```
- interface를 변수로 가지고 있는 class를 만들어 객체 사용시점에 사용하고 싶은 객체를 넣어 해당 객체를 사용하도록 한다.
    ```
    @Getter
    @Builder
    @NoArgsConstructor(access = AccessLevel.PRIVATE)
    @AllArgsConstructor(access = AccessLevel.PRIVATE)
    public class PostStatusMail {
        private MailCondition mailCondition;
        private String address;

        public static PostStatusMail create(PostMailCondition postMailCondition, String address) {
            PostStatusMailBuilder postStatusMailBuilder = PostStatusMail.builder().address(address);
            switch (postMailCondition) {
                case APPROVE:
                    return postStatusMailBuilder.mailCondition(new ApproveCondition()).build();
                case REJECT:
                    return postStatusMailBuilder.mailCondition(new RejectCondition()).build();
                case APPROVECANCEL:
                    return postStatusMailBuilder.mailCondition(new ApproveCancelCondition()).build();
            }
            throw new IllegalArgumentException("해당 메일 조건이 없습니다.");
        }
    }
    ```

- service layer
    ```
    @Service
    @RequiredArgsConstructor
    public class PostStatusMailService {
        private final JavaMailSender mailSender;
        private static final String FROM_ADDRESS = "YOUR_EMAIL_ADDRESS";

        public void mailSend(PostStatusMail postStatusMail) {
            MailCondition mailCondition = postStatusMail.getMailCondition();
            SimpleMailMessage message = mailCondition.makeSimpleMessage(postStatusMail.getAddress(), FROM_ADDRESS);

            mailSender.send(message);
        }
    }
    ```

### spring boot 메일 사용
- 위에 있는 의존성을 추가해주고 yml 파일에 메일 추가 설정을 해준다.
```
spring:
    mail:
        host: smtp.gmail.com
        port: 587
        username: { YOUR_GMAIL_ADDRESS }
        password: { YOUR_GMAIL_PASSWORD }
        properties:
        mail:
            smtp:
            auth: true
            starttls:
                enable: true
```
- 구글계정에서 보안 수준이 낮은 앱 허용 설정을 허용으로 바꿔준다. 
    - 이렇게 안해주면 구글에서 로그인을 차단하여 메일보내는 과정에서 에러가 발생한다.
```
- JavaMailSender는 spring boot에서 자동으로 의존성 주입을 해준다.
@Service
@RequiredArgsConstructor
public class MailService {
    private final JavaMailSender mailSender;
    private static final String FROM_ADDRESS = "YOUR_EMAIL_ADDRESS";

    public void mailSend(MailDto mailDto) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(mailDto.getAddress());
        message.setFrom(MailService.FROM_ADDRESS);
        message.setSubject(mailDto.getTitle());
        message.setText(mailDto.getMessage());

        mailSender.send(message);
    }
}
```

## 메일 타임리프 사용해서 템플릿으로 전송 및 비동기 전송
- 타임리프 의존성 추가
```
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
```
- 메일 구성이 복잡해질 경우 타임리프를 사용하여 보내는 것이 더 효율적이고 더 편하다.

### SimpleMessage로 보내던 메일 수정하기(위 코드 수정)
- SimpleMessage -> MimeMessage 
- mail.html 로 타임리프를 사용해 수정
```
public class RejectPostCondition implements MailPostCondition {
    @Override
    public MimeMessage makeMimeMessage(MimeMessage mimeMailMessage, TemplateEngine templateEngine, String toEmail) throws MessagingException {
        mimeMailMessage.addRecipients(MimeMessage.RecipientType.TO, toEmail);
        mimeMailMessage.setSubject("메일 제목");
        mimeMailMessage.setText(setContext(templateEngine), "utf-8", "html");
        return mimeMailMessage;
    }

    private String setContext(TemplateEngine templateEngine) {
        Context context = new Context();

        return templateEngine.process("mail", context);
    }
}
```

- 타임리프로 값 전달
- 위에서 사용한 setContext에서 setVariable 를 사용하여 타임리프로 값을 전달할 수 있다.
- 전달받은 데이터를 타임리프에서 th 태그를 사용해 화면으로 보여준다.
```
private String setContext(String code) {
    Context context = new Context();
    context.setVariable("code", code);
    return templateEngine.process("emailVerification", context);
}
```

### 메일 전송 비동기로 수정하기
- 메일 전송하는데 걸리는 시간은 대략 3~4초 정도 걸린다. 다른 api안에서 메일을 전송해야할 때 응답을 3~4초동안 기다리는 것은 매우 느리다. 따라서 다른 api안에서 사용하되 api는 먼저 응답하고 비동기적으로 메일 전송을 따로하도록 설정할 수 있다.
- 비동기적으로 메일을 전송하려면 쓰레드를 사용해야한다. spring boot에서는 쓰레드 생성을 간편하게 할 수 있다.
- AsyncConfig 클래스 생성
```
@Configuration
@EnableAsync
public class AsyncConfig extends AsyncConfigurerSupport {
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(5000);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
    - MaxPoolSize : 동시에 동작하는 최대 쓰레드 수
    - QueueCapacity : MaxPoolSize를 초과하는 요이 쓰레드 생성 요청시 해당 내용을 Queue에 저장하게 되고 사용할 수 있는 여유자리가 생기면 하나씩 꺼내서 동작한다.
    - ThreadNamePrefix : 생성하는 쓰레드의 접두사
```
- async 설정을 한 후 비동기적으로 실행할 메서드에 @Async 을 붙혀준다.
```
@Async
public void mailSend(String toMail) {
    logic...
}
```
- 다른 api에 메일 전송을 섞어쓸 경우 응답이 3초를 넘어갔는데 이렇게 비동기적으로 메일 전송을 하면 원래 응답 속도대로 나온다.
- @Async 어노테이션은 private 메서드에서는 비동기로 작동하기 않고 public에서만 비동기로 작동한다.