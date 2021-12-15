## AOP 란?
- 부가 기능을 핵심 기능에서 분리하고 한 곳에서 관리할 수 있도록 도와준다.
- 해당 부가 기능을 어디에 적용할지 선택하는 기능이 있다.
- Aspect(관점)는 부가기능과 해당 부가기능을 어디에 적용할 것인지 정의한 것이다.
- 애플리케이션을 하나하나 바라보는 관점을 횡단 관심사 관점으로 달리보는 것이다.
- Aspect를 사용한 프로그래밍 방식을 관점 지향 프로그래밍 AOP 라고 한다.

### AOP 적용방식
- 위빙 : aspect와 실제 코드를 연결해 붙이는 것
- 컴파일 시점
    - .java 소스 코드를 컴파일러를 사용해서 .class를 만드는 시점에 부가 기능 로직을 추가할 수 있다.
    - 단점 : 특별한 컴파일러도 필요하고 복잡하다.
- 클래스 로딩 시점
    - 자바를 실행하면 .class 파일을 JVM 내부의 클래스 로더에 보관한다. 이때 중간에서 .class 파일을 조작한 다음 JVM에 올릴 수 있다.
    - 단점 : 특별한 옵션ㅇ르 통해 클래스 로더 조작기를 지정해야하는데 번거롭고 운영하기 어렵다.
- 런타임 시점
    - 스프링과 같은 컨테이너의 도움을 받고 프록시와 DI, 빈 포스트 프로세서와 같은 개념들을 모두 사용한다. 이렇게하면 최종적으로 프록시를 통해 스프링 빈에 부가 기능을 적용할 수 있다.
    - 프록시 방식을 사용하는 스프링 AOP는 스프링 컨테이너가 관리할 수 있는 스프링 빈에만 AOP를 적용할 수 있고 스프링 AOP의 조인 포인트는 메서드 실행으로 제한된다.

### 간단한 AOP 예시
```
@Slf4j
@Aspect
public class AspectV1 {

    @Around("execution(* hello.aop.order..*(..))")
    public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
        log.info("[log] {}", joinPoint.getSignature());
        return joinPoint.proceed();
    }
}
```
- @Around -> 포인트컷, doLog 매서드 -> 어드바이스 합쳐서 어드바이저가 된다.
- 이를 스프링 빈으로 등록만 하면 사용가능하다.

-포인트컷 분리
```
@Pointcut("execution(* hello.aop.order..*(..))")
private void allOrder(){}

@Around("allOrder()")
public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
    log.info("[log] {}", joinPoint.getSignature());
    return joinPoint.proceed();        
}
```

### 포인트컷 외부에서 관리하기
```
public class Pointcuts {
    @Pointcut("execution(* hello.aop.order..*(..))")
    public void allOrder() {}

    @Pointcut("execution(* *..*Service.*(..))")
    public void allService() {}

    @Pointcut("allOrder() && allService()")
    public void orderAndService() {}
}
```

```
@Around("hello.aop.order.aop.Pointcuts.allOrder()")
public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
    log.info("[log] {}", joinPoint.getSignature());
    return joinPoint.proceed();
}
```
- pointcut class를 만들고 사용하는 곳에서 패키지 이름을 사용하여 적용시킨다.

### 어드바이스 순서 바꾸기
```
@Slf4j
@Aspect
public class AspectV4Order {

    @Aspect
    @Order(2)
    public static class LogAspect {
        @Around("allOrder()")
        public Object doLog(ProceedingJoinPoint joinPoint) throws Throwable {
          ...
        }
    }

    @Aspect
    @Order(1)
    public static class TxAspect{
        @Around("allOrder() && allService()")
        public Object doTransaction(ProceedingJoinPoint joinPoint) throws Throwable {
           ...
    }
}

```
- @Order를 사용해서 순서를 바꿀 수있다.
- @Order는 @Aspect 단위로 적용되기 때문에 class에 적용시켜야한다.


### 어드바이스 종류
- @Around : 메서드 호출 전후에 수행
- @Before : 조인 포인트 실행 이전에 실행
- @After Returning : 조인 포인트가 정상 완료후 실행
- @After Throwing : 메서드가 에외를 던시는 경우 실행
- @After : 조인 포인트가 정상 또는 예외 관계없이 실행

- @Around를 사용하면 모든 것을 할 수 있지만 다른 어드바이스 종류를 사용하면 다른 개발자가 이 코드를 보고 고민해야 하는 범위가 줄어들게 되고 코드의 의도도 파악하기 쉽게 할 수 있다.

### Pointcut execution 문법
```
 @Test
void exactMatch() {
    pointcut.setExpression("execution(public String hello.aop.member.MemberServiceImpl.hello(String))");
    Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
}
```
- 매칭 조건
    - 접근제어자 : public
    - 반환 타입 : String
    - 선언 타입 : hello.aop.member.MemberServiceImpl
    - 매서드 이름 : hello
    - 파라미터 : (String)
    - 예외 : 생략

- 패키지에서 '*', '**' 의 차이  
    - '*' : 정확하게 해당 위치의 패키지
    - '**' : 해당 위치의 패키지와 그 하위 패키지도 포함
```
@Test
void packageExactFalse(){
    pointcut.setExpression("execution(* hello.aop.*.*(..))");
    Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isFalse();
}

@Test
void packageMatchSubPackage1(){
    pointcut.setExpression("execution(* hello.aop.member..*.*(..))");
    Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
}
```

- 타입 매칭 
    - 부모타입을 선언해도 자식 타입을 매칭된다.
    ```
    @Test
    void typeMatchSuperType(){
        pointcut.setExpression("execution(* hello.aop.member.MemberService.*(..))");
        Assertions.assertThat(pointcut.matches(helloMethod, MemberServiceImpl.class)).isTrue();
    }
    ```
    - 부모 타입에 선언된 메서드만 매칭이 된다. 
    - 부모 타입을 표현식에 선언한 경우 부모 타입에서 선언한 메서드가 자식 타입에 있어야 매칭에 성공한다.

### Pointcut within , args 문법
- within
    - execution 에서 타입부분만 쓰는 것이다.
    - 표현식에 부모 타입을 지정하면 안되고 타입이 정확하게 맞아야한다.
- args
    - 파라미터 타입을 보고 적용한다.
    - execution과의 차이점
        - execution은 정확하게 파라미터 타입이 일치해야한다.
        - args는 부모타입으로도 매칭시킨다. ex) String - Object 매칭됨.


### @taget, @within
- @taget : 인스턴스의 모든 메서드를 조인 포인트로 사용한다.
- @within : 해당 타입 내에 있는 매서드만 조인 포인트로 적용한다.
- @taget은 부모 클래스의 매서드까지 어드바이스를 다 적용하고 @within은 자신의 클래스에 정의된 매서드에만 어드바이스를 적용한다.

- args, @args, @target과 같은 포인트컷 지시자들은 execution으로 먼저 적용대상을 줄이고 사용해야한다. 해당 지시자들은 프록시가 있어야 실행 시점에 판단을 할 수 있기 때문에 프록시가 없으면 판단일 불가능하다.


### 매개변수 전달
- 매개변수를 전달받는 방법
    - 주의
        - this : 프록시 객체를 전달받는다
        - target : 실제 대상 객체를 전달받는다.
    - 애노테이션을 전달받아서 annotaion.value()로 해당 애노테이션의 값을 출력할 수도 있다.
    ```
    @Slf4j
    @Aspect
    static class ParameterAspect {
        @Pointcut("execution(* hello.aop.member..*.*(..))")
        private void allMember() {
        }

        @Around("allMember()")
        public Object logArgs1(ProceedingJoinPoint joinPoint) throws Throwable {
            Object arg1 = joinPoint.getArgs()[0];
            log.info("[logArgs1]{}, args={}", joinPoint.getSignature(), arg1);
            return joinPoint.proceed();
        }

        @Around("allMember() && args(arg,..)")
        public Object logArgs2(ProceedingJoinPoint joinPoint, Object arg) throws Throwable {
            log.info("[logArg2]{}, args={}", joinPoint.getSignature(), arg);
            return joinPoint.proceed();
        }

        @Before("allMember() && args(arg,..)")
        public void logArgs3(String arg) {
            log.info("[logArgs3], args={}", arg);
        }

        @Before("allMember() && this(obj)")
        public void thisArgs(JoinPoint joinPoint, MemberService obj) {
            log.info("[this]{}, obj={}", joinPoint.getSignature(), obj.getClass());
        }

        @Before("allMember() && target(obj)")
        public void targetArgs(JoinPoint joinPoint, MemberService obj) {
            log.info("[target]{}, obj={}", joinPoint.getSignature(), obj.getClass());
        }

        @Before("allMember() && @annotation(annotation)")
        public void targetArgs(JoinPoint joinPoint, MethodAop annotation) {
            log.info("[target]{}, annotation={}", joinPoint.getSignature(), annotation.value());
        }
    }
    ```