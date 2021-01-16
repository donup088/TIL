## 스프링 빈 
- 빈 이름은 메서드 이름을 사용한다.
- 빈 이름은 항상 다른 이름을 부여해야한다.
- 스프링은 빈을 생성하고 의존관계를 주입하는 단계가 나누어져있다.( 생성 -> 의존관게 연결 )

## 스프링 빈 조회
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