## Spring Scheduler
- Spring boot에서 설정한 시간마다 특정 동작을 하는 것을 구현할 수 있다.
    - @EnableScheduling 사용으로 스케줄링 활성화
    ```
    @EnableScheduling 
    @SpringBootApplication 
    public class SampleApplication { 
        public static void main(String[] args) {
             SpringApplication.run(SampleApplication.class, args);
        } 
    }
    ```
    - 주기적으로 수행해야할 메서드에  @Scheduled 를 사용한다.
    ```
    @Scheduled(cron = "0 0 5 * * *", zone = "Asia/Seoul")
    public void sample() {
        ... 설정한 시간마다 해야할 것을 구현
    }
    ```
    - @Scheduled 옵션
        - fixedDelay : 이전 작업이 종료된 후 설정 시간만큼 기다린 후 시작한다.(밀리초)
        - fixedRate : 이전 작업이 종료되지 않아도 설정된 시간마다 시작한다.
        - initailDelay : 작업 시작 시 설정된 시간만큼 기다린 후 시작한다.
        - cron : 원하는 시간대를 설정하여 작업을 진행한다.
            ```
            @Scheduled(cron=" * * * * * *")
            *순서대로 초,분,시간,일,월,요일을 나타낸다.
            ```
        - zone : 시간대 설정 미설정시 로컬 시간대 적용

- 간단한 배치 작업을 스케줄러를 통해서 구현할 수 있다.