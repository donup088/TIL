## 컴포넌트 스캔
- @ComponentScan을 사용하면 @Component 애노테이션이 붙은 클래스를 스캔해서 스프링 빈으로 등록한다.
- 스프링 빈의 기본 이름은 클래스명을 사용한다.
- @Component 애노테이션이 붙은 클래스에서 의존관계 주입을 하기 위해서 @Autowired를 사용한다.
    - 생성자를 만들고 @Autowired를 사용(생성자 주입)

## 컴포넌트 스캔 옵션
- basePackages 를 사용하면 탐색할 패키지의 시작 위치를 지정할 수 있다.
- basePackageClasses 설정한 클래스 하위 클래스를 모두 스캔한다.
- 지정하지 않는다면 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.

## 빈의 중복 등록과 충돌
- 자동 빈 등록끼리 중복 등록을 할 경우 ConflictingBeanDefinitaionException 이 발생한다.
- 수동으로 만든 빈과 자동으로 만들어진 빈이 충돌할 경우 수동으로 만든 빈이 우선권을 갖는다.
    - 스프링 부트에서는 오류가 발생한다.

