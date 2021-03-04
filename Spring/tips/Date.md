### spring boot에서 LocalDateTime 다루기
- LocalDateTime 형식 정하기
    - @JsonFormat 사용
    ```
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ss", timezone = "Asia/Seoul")
    ```
- 요청 데이터로 Year와Month만 받고 싶을 때
    - YearMonth 클래스 사용
    - 2021-3 이렇게 요청 데이터를 받을 수 있다.
    ```
    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    @Builder
    public class YearMonthRequest {
        @DateTimeFormat(pattern = "yyyy-MM")
        @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM", timezone = "Asia/Seoul")
        private YearMonth yearMonth;
    }
    ```
    