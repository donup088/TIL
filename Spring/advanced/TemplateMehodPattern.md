## 템플릿 메서드 패턴
- 템플릿이라는 틀에 변하지 않는 부분을 몰아둔다. 그리고 일부 변하는 부분을 별도로 호출하여 해결한다.
- 부모 클래스에 변하지 않는 템플릿 코드를 둔다. 그리고 자식 클래스에 변하는 부분을 두고 상속과 오버라이딩을 사용하여 처리한다.
```
@Slf4j
public abstract class AbstractTemplate {
    public void execute(){
        long startTime = System.currentTimeMillis();
        //log.info("비즈니스 로직1 실행");
        call();
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }

    protected abstract void call();
}
```
```
@Slf4j
public class SubClassLogic1 extends AbstractTemplate{
    @Override
    protected void call() {
        log.info("비즈니스 로직1 실행");
    }
}
```
```
 @Test
    void templateMethodV1() {
        AbstractTemplate template1 = new SubClassLogic1();
        template1.execute();
        AbstractTemplate template2 = new SubClassLogic2();
        template2.execute();
    }
```
- 다형성을 사용하여 변하는 부분과 변하지 않는 부분을 분리하는 방법이다.

### 템플릿 메서드 패턴 단점 보완
- 템플릿 메서드 패턴은 하나의 템플릿을 상속 받는 클래스를 계속 만들어야하는 단점이 있다. 익명 내부 클래스를 사용하면 이런 단점을 보완할 수 있다.
- 익명내부클래스를 사용하면 객체 인스턴스를 생성하면서 동시에 클래스를 상속 받은 자식 클래스를 정의할 수 있다.
```
@Test
void templateMethodV2() {
    AbstractTemplate template1 = new AbstractTemplate() {
        @Override
        protected void call() {
            log.info("비즈니스 로직1 실행");
        }
    };
    template1.execute();
    AbstractTemplate template2 = new AbstractTemplate() {
        @Override
        protected void call() {
            log.info("비즈니스 로직2 실행");
        }
    };
    template2.execute();
}
```
- 템플릿 메서드 패턴은 상속을 사용한다. 따라서 상속에서 오는 단점들을 그대로 안고가는 것이다. 특히 부모 클래스와 컴파일 시점에 강하게 결합되는 문제가 있다. 자식 클래스 입장에는 부모 클래스의 기능을 전혀 사용하지 않지만 코드를 모두 알고 있는 것이다. -> 부모 클래스의 변화가 생기면 자식 클래스는 모두 영향을 받는다.

## 템플릿 메서드의 단점을 보완할 전략패턴
- 변하지 않는 부분을 Context 라는 곳에 두고 변하는 부분을 인터페이스를 만들고 해당 인터페이스를 구현하도록 문제를 해결한다. 상속이 아닌 위임으로 문제를 해결한다.
```
public interface Strategy {
    void call();
}
```
```
@Slf4j
public class StrategyLogic1 implements Strategy{
    @Override
    public void call() {
        log.info("비즈니스 로직1 실행");
    }
}
```
```
@Slf4j
public class ContextV1 {
    private Strategy strategy;

    public ContextV1(Strategy strategy) {
        this.strategy = strategy;
    }

    public void execute(){
        long startTime = System.currentTimeMillis();
        strategy.call();
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
}
```
- context 내부에 strategy 필드를 가지고 있다. 필드에 변하는 부분만 Strategy 구현체를 주입하면 된다. Context는 Strategy 인터페이스에만 의존한다. Strategy의 구현체를 변경하거나 새로 만들어도 Context 코드에는 영향을 주지 않는다.
- 스프링에서 의존관계 주입에서 사용하는 방식이 전략 패턴이다.
- 위와 같은 방법으로 하게 되면 필드주입을 받기 때문에 한번 주입받고나면 변경이 어렵다는 단점이 있다. 이를 해결하는 방법으로는 파라미터로 직접 주입받는 방법이 있다.
```
@Slf4j
public class ContextV2 {
    public void execute(Strategy strategy) {
        long startTime = System.currentTimeMillis();
        strategy.call();
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
    }
}
```
- 이렇게 방법을 바꾸면 실행할 때마다 전략을 유연하게 변경할 수 있다.

### 템플릿 콜백 패턴
- 변하지 않는 템플릿이 있고 변하는 부분은 파라미터로 넘겨온 Strategy 의 코드를 실행해서 처리한다. 이 때 다른 코드의 인수로 넘겨주는 실행가능한 코드를 콜백이라고한다.
- call(코드를 호출) + back(호출한 코드를 뒤에서 실행) 
- java에서의 콜백
    - 자바 언어에서 실행 가능한 코드를 넘기려면 객체가 필요하다. 자바8부터는 람다를 사용할 수 있다.(자바 8이전은 주로 익명내부클래스사용)