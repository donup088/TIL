### 스프링 빈의 생성
- @Bean이나 컴포넌트 스캔으로 스프링 빈을 등록하면 스프링을 대상 객체를 생성하고 스프링 컨테이너 내부의 빈 저장소에 등록한다. 그리고 스프링 컨테이너를 통해 스프링 빈을 조회해서 사용한다.

### 빈 후처리기
- 스프링이 빈 저장소에 등록할 목적으로 생성한 객체를 등록하기 직전에 조작하고 싶다면 빈 후처리기를 사용하면 된다.
- 객체를 조작할 수 있고 완전 다른 객체로도 바꿀 수 있다.
- 빈 후처리기를 사용하려면 BeanPostProcessor 인터페이스를 구현하고 스프링 빈으로 등록하면 된다.
```
@Configuration
static class BeanPostProcessorConfig {
    @Bean(name = "beanA")
    public A a() {
        return new A();
    }

    @Bean
    public AtoBPostProcessor helloPostProcessor() {
        return new AtoBPostProcessor();
    }
}
```
```
@Test
void basicConfig() {
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(BeanPostProcessorConfig.class);
    B b = applicationContext.getBean("beanA", B.class);
    b.helloB();

    Assertions.assertThrows(NoSuchBeanDefinitionException.class, () -> applicationContext.getBean("beanB", A.class));
}

```

### 스프링이 제공하는 빈 후처리기
- spring boot aop 의존성 추가
```
implementation 'org.springframwork.boot:spring-boot-starter-aop'
```
- 자동 프록시 생성기의 작동 과정
    1. 생성 : 스프링이 스프링 빈 대상이 되는 객체를 생성
    2. 전달 : 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달
    3. 모든 Advisor 빈 조회 : 자동 프록시 생성기 - 빈 후처리기는 스프링 컨테이너 모든 Advisor를 조회
    4. 프록시 적용 대상 체크 : Advisor에 포함되어 있는 포인트컷을 사용해서 해당 객체가 프록시 적용 대상인지 판단
    5. 프록시 생성 : 프록시 적용 대상이면 프록시를 생성하고 반환해서 프록시를 스프링 빈으로 등록
    6. 빈 등록 : 반환된 객체를 스프링 빈으로 등록


### 포인트컷이 사용되는 곳
- 프록시 적용 여부 판단 - 생성 단계
    - 자동 프록시 생성기는 포인트컷을 사용해서 해당 빈이 프록시에 생성할 필요가 있는지 체크한다.
    - 클래스+매서드 조건을 모두 비교하고 포인트컷 조건에 하나하나 매칭해본다. 조건에 맞는 것이 하나라도 있으면 프록시를 생성한다.
- 어드바이스 적용 여부 판단 - 사용 단계
    - 프록시가 호출되었을 때 부가 기능인 어드바이스를 적용할 지 포인트컷을 보고 판단한다.

### 프록시 사용 주의사항
- 프록시를 모든 곳에 사용하는 것은 낭비이다. 꼭 필요한 곳에만 최소한의 프록시를 적용해야한다. 자동 프록시 생성기는 모든 스프링 빈에 프록시를 적용하는 것이 아니라 포인트 컷으로 필터링해서 어드바이스가 사용될 가능성이 있는 곳에만 프록시를 생성한다.


### @Aspect 사용
- 자동 프록시 생성기는 @Aspect를 찾아서 이것을 어드바이저로 만들어준다.

- 자동프록시 생성기는 @Aspect를 보고 어드바이저로 변환해서 저장한다. 그리고 어드바이저 기반으로 프록시를 생성한다.

- 과정
    1. 실행 : 스프링 애플리케이션 로딩 시점에 자동 프록시 생성기 호출
    2. 모든 @Aspect 빈 조회 : 자동 프록시 생성기는 스프링 컨테이너에서 @Aspect 애노테이션 정보를 기반으로 어드바이저를 생성한다.
    3. 어드바이저 생성 : @Aspect 어드바이저 빌더를 통해 애노테이션 정보를 기반으로 어드바이저를 생성한다.
    4. @Aspect 기반 어드바이저 저장