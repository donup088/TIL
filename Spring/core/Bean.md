### 스프링 빈 
- 빈 이름은 메서드 이름을 사용한다.
- 빈 이름은 항상 다른 이름을 부여해야한다.
- 스프링은 빈을 생성하고 의존관계를 주입하는 단계가 나누어져있다.( 생성 -> 의존관게 연결 )

### 스프링 빈 조회
- 빈 이름으로 조회
    ```
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    MemberService memberService = ac.getBean("memberService", MemberService.class);
    ```
- 빈 타입으로 조회
    ```
     MemberService memberService = ac.getBean(MemberService.class);
    ```
- 타입으로 조회시 등록된 타입이 2개 이상이라면 빈 이름으로 지정해야한다.
- 부모 타입으로 조회하면 자식 타입도 함께 조회된다. 이 때 자식이 둘 이상 있으면 중복 오류가 발생하기 때문에 빈 이름을 지정해준다.
- 빈이 없다면 NoSuchBeanDefinitionException 예외가 발생한다.

### 스프링 빈의 라이프사이클
- 객체 생성 --> 의존관계 주입
- 스프링 빈의 이벤트 라이프사이클
    - 스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 ->소멸전 콜백 -> 스프링 종료
- InitializingBean은 afterPropertiesSet 지원한다.
- DisposableBean은 destroy를 지원한다.

### 빈 등록 초기화, 소멸 매서드
- 해당 매서드를 만들고 @Bean에 init과 destroyMethod를 설정한다.
```
 @Bean(initMethod = "init",destroyMethod = "close")
```
- 일반적인 경우에는 @PostConstruct, @PreDestroy를 사용하자.
- 외부 라이브러리를 초기화,종료가 필요하면 @Bean의 기능을 사용하자.

### 빈 스코프
- 싱글톤: 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다.
- 프로토타입: 프로토타입 빈의 생성과 의존관계 주입까지만 관여한다.
    - 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다. 클라이언트에 반환하고 더 이상 스프링 컨테이너가 관리하지 않아서 이후에 다시 요청이 들어오면 새로 생성한다.
    - @PreDestroy 같은 종료 메서드가 실행되지 않는다.
- 웹 관련 스코프
    - request: 웹 요청이 들어오고 나갈 때 까지 유지된다.
    - session: 웹 세션이 생성되고 종료될 때 까지 유지된다.
    - appplication: 웹의 서블릿 컨텍스와 같은 범위로 유지된다.
- @Scope("prototype") 이와 같이 사용한다.

### 싱글톤 안에서 프로토타입을 쓸 때 문제점
- 싱글톤 빈이 프로토타입 빈을 사용하면 싱글톤 빈이 생성 시점에만 의존관계를 주입 받기 때문에 프로토타입 빈이 새로 생성되기는 하지만 싱글톤 빈과 함께 계속 유지된다.
- 사용할 때마다 새로운 빈을 만들고 싶은 경우 ObjectProvider를 사용한다.
    ``` 
    @Autowired
    private ObjectProvider<PrototypeBean> provider;
    PrototypeBean prototypeBean= provider.getObject();
    prototypeBean.logic(); // 사용가능
    ```
    - ObjectProvider의 getObject()를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다.
