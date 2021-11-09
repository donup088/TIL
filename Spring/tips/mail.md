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