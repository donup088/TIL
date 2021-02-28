### LocalDateTime ISO 8601 형식의 문자열로 사용하기
- ObjectMapper를 Bean으로 등록하여 new ObjectMapper()로 할 경우 LocalDateTime이 객체로 변환되어 장황하게 나온다.
- LocalDateTime을 ISO 8601 형식의 문자열로 변환하기 위해서 ObjectMapper를 커스터마이징한다.
```
@Bean
public ObjectMapper objectMapper() {
    return Jackson2ObjectMapperBuilder
            .json()
            .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .modules(new JavaTimeModule())
            .build();
    }
```
- 사용하는 Dto 클래스에서 @JsonFormat을 사용한다
```
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ss", timezone = "Asia/Seoul")
```
- "endMeetingDate":"2021-02-28T15:51:18" 처럼 response를 얻을 수 있다.