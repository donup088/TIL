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