## IoC 제어의 역전(Inversion of Control)
- 프로그램이 제어 흐름을 직접 제어하는 것이 아니라 외부에서 직접 관리하는 것을 말한다.

## DI 의존관계 주입(Dependecy Injection)
- 사용 클래스가 아닌 config를 담당하는 곳에서 객체를 주입해주는 것을 말한다. 따라서 사용하는 클래스에서는 어떤 객체가 주입되는지 알지 못한다.
- 애플리케이션 실행시점에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달하여 클라이언트와 서버의 실제 의존관계가 연결 되는 것을 의미한다.
- 이를 사용함으로써 클라이언트 코드를 변경하지 않고 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.

## IOC 컨테이너 & DI 컨테이너
- 객체 생성을 관리하면서 의존관계를 연결해주는 것을 IoC 컨테이너 DI 컨테이너라고 한다.

## 스프링 컨테이너
- ApplicationContext를 스프링 컨테이너라고 한다.
- @Configuration 이 붙은 설정 정보를 사용한다. 설정 정보에 @Bean 이 붙은 메소드를 모두 호출해서 스프링 컨테이너에 등록한다.
- 스프링 빈 찾아서 사용하기
    ```
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    MemberService memberService = ac.getBean("memberService", MemberService.class);
    ```
