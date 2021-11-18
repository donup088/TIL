## spring boot 로 네이버 sms 문자 인증 구현하기

### 네이버 클라우드 요금
- 한달에 50개의 sms를 무료로 사용가능하다
- 50개 넘어가면 1개에 9원이다.
- sms 사용이 많으면 할인해주는 방식인 것 같다.

### 네이버 클라우드 설정
- 네이버 클라우드 플랫폼에 접속하여 Simple & Easy Notification Service 에 들어간다.
 - 프로젝트를 생성하고 발신번호 등록을 한다
- 마이페이지 -> 인증키 관리 -> 신규 API 인증키 생성을 통해 accesskey,secretkey를 받는다.
- 네이버 API 가이드와 블로그들이 많아서 쉽게 구현가능하다.

### 코드 작성
- application.yml 파일에 네이버 클라우드 key들을 설정한다.
    ```
    sms:
        serviceId: 프로젝트 서비스 아이디
        accessKey: 엑세스키
        secretKey: 비밀키
        from: 발신자 번호
    ```
- 네이버 오픈 API 가이드에 따라 작성시작
    - MessageDto class -> 메세지에 들어갈 내용과 수신자번호가 필요하다.
        ```
        @Getter
        @Builder
        @NoArgsConstructor
        @AllArgsConstructor
        public class MessageDto {
            private String to;
            private String content;

            public static List<MessageDto> build(String to, String content) {
                MessageDto messageDto = MessageDto.builder().to(to).content(content).build();
                return Collections.singletonList(messageDto);
            }
        }
        ```
    - SmsMessagerequest -> 네이버 api로 보낼 요청 데이터
        ```
        @Getter
        @Builder
        @NoArgsConstructor(access = AccessLevel.PRIVATE)
        @AllArgsConstructor(access = AccessLevel.PRIVATE)
        public class SmsMessageRequest {
            private String type;
            private String contentType;
            private String countryCode;
            private String from;
            private String content;
            private List<MessageDto> messages;

            public static SmsMessageRequest makeSmsRequest(String from, List<MessageDto> messages) {

                return SmsMessageRequest.builder()
                        .type("SMS")
                        .contentType("COMM")
                        .countryCode("82")
                        .from(String.join("", from.split("-")))
                        .content("내용")
                        .messages(messages)
                        .build();
            }
        }
        ```
    - SmsRequest -> 우리 웹서비스에서 인증번호 보낼 api 요청 데이터
        ```
        @Getter
        @Builder
        @NoArgsConstructor(access = AccessLevel.PRIVATE)
        @AllArgsConstructor(access = AccessLevel.PRIVATE)
        public class SmsRequest {
            @NotBlank(message = "수신자 번호는 공백일 수 없습니다.")
            private String receiverPhoneNumber;
        }
        ```
    - SmsResponse -> 네이버 API에서 주는 응답을 담을 객체
        ```
        @Getter
        @Builder
        @AllArgsConstructor
        @NoArgsConstructor(access = AccessLevel.PROTECTED)
        public class SmsResponse {
            private String requestId;
            private LocalDateTime requestTime;
            private String statusCode;
            private String statusName;
        }
        ```
    - SmsService
        - 네이버 API를 헤더세팅하고 RestTemplate을 사용하여 API 보내기
        ```
        @Service
        @Transactional
        @RequiredArgsConstructor
        public class SmsService {
            @Value("${sms.serviceId}")
            private String serviceId;
            @Value("${sms.accessKey}")
            private String accessKey;
            @Value("${sms.secretKey}")
            private String secretKey;
            @Value("${sms.from}")
            private String from;

            public SmsResponse sendSmsVerificationCode(String receiverPhoneNumber, Long userId) throws JsonProcessingException, URISyntaxException, NoSuchAlgorithmException, InvalidKeyException {
                Long time = System.currentTimeMillis();
             
                ...

                List<MessageDto> messageDtoList = MessageDto.build(receiverPhoneNumber, content);

                SmsMessageRequest smsMessageRequest = SmsMessageRequest.makeSmsRequest(from, messageDtoList);

                ObjectMapper objectMapper = new ObjectMapper();
                String jsonBody = objectMapper.writeValueAsString(smsMessageRequest);

                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                headers.set("x-ncp-apigw-timestamp", time.toString());
                headers.set("x-ncp-iam-access-key", this.accessKey);
                String sig = makeSignature(time);
                headers.set("x-ncp-apigw-signature-v2", sig);

                HttpEntity<String> body = new HttpEntity<>(jsonBody, headers);

                RestTemplate restTemplate = new RestTemplate();
                restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());

                return restTemplate.postForObject(new URI("https://sens.apigw.ntruss.com/sms/v2/services/" + this.serviceId + "/messages"), body, SmsResponse.class);
            }

            private String makeSignature(Long time) throws NoSuchAlgorithmException, InvalidKeyException {
                String space = " ";
                String newLine = "\n";
                String method = "POST";
                String url = "/sms/v2/services/" + this.serviceId + "/messages";
                String timestamp = time.toString();
                String accessKey = this.accessKey;
                String secretKey = this.secretKey;

                String message = new StringBuilder()
                        .append(method)
                        .append(space)
                        .append(url)
                        .append(newLine)
                        .append(timestamp)
                        .append(newLine)
                        .append(accessKey)
                        .toString();

                SecretKeySpec signingKey = new SecretKeySpec(secretKey.getBytes(StandardCharsets.UTF_8), "HmacSHA256");
                Mac mac = Mac.getInstance("HmacSHA256");
                mac.init(signingKey);

                byte[] rawHmac = mac.doFinal(message.getBytes(StandardCharsets.UTF_8));

                return Base64.encodeBase64String(rawHmac);
            }
        }
        ```