### 리플렉션 사용
- 리플렉션을 사용하면 클래스와 메서드의 메타정보를 사용해서 애플리케이션을 동적으로 유연하게 만들 수 있다.
- 하지만 리플렉션은 런타임에 동작하기 때문에 컴파일 시점에 오류를 잡을 수 없다.
- 프레임워크 개발이나 매우 일반적인 공통 처리가 필요할 때만 부분적으로 주의해서 사용해야한다.

### JDK 동적 프록시
- 인터페이스 기반으로 프록시를 동적으로 만들기 때문에 인터페이스가 필수다.
```
public interface AInterface {
    String call();
}

```
```
@Slf4j
public class AImpl implements AInterface{
    @Override
    public String call() {
        log.info("A 호출");
        return "a";
    }
}
```
```
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        Object result = method.invoke(target, args);

        long endTime = System.currentTimeMillis();

        long resultTime = endTime - startTime;

        log.info("TimeProxy 종료 resultTime={}", resultTime);

        return result;
    }
}
```
```
@Test
void dynamicA() {
    AInterface target = new AImpl();
    TimeInvocationHandler handler = new TimeInvocationHandler(target);

    AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(), new Class[]{AInterface.class}, handler);

    proxy.call();
    log.info("targetClass={}",target.getClass());
    log.info("proxyClass={}",proxy.getClass());
}
```

- 한계 : 인터페이스가 없으면 사용할 수 없다.

### CGLIB
- CGLIB 는 바이트코드를 조작해서 동적으로 클래스를 생성하는 기술을 제공하는 라이브러리다.
- 인터페이스가 없어도 구체 클래스만 가지고 동적 프록시를 사용할 수 있다.
- 스프링을 사용한다면 별도의 외부 라이브러리를 추가하지 않아도 사용할 수 있다.
