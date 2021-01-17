## 싱글톤 패턴
- 클래스의 인스턴스가 1개만 생성되는 것을 보장하는 디자인 패턴
```
public class SingletonService {
    private static final SingletonService instance = new SingletonService();

    public static SingletonService getInstance(){
        return instance;
    }

    private SingletonService(){}


```
- static 영역에 객체 instance를 하나 생성한다.
- 객체 인스턴스가 필요하면 getInstance() 메소드를 통해서만 조회할 수 있다.
- 기본 생성자를 private으로 막아서 외부에서 new 로 객체 생성하는 것을 막는다.

## 싱글톤 컨테이너
- 스프링 컨테이너는 싱글톤 패턴을 적용하지 않아도 싱글톤으로 객체를 관리한다.
- 스프링 컨테이너는 요청이 올 때마다 이미 만들어진 객체를 재사용할 수 있다.
- 객체를 계속 생성하지 않아도 되기 때문에 메모리를 아낄 수 있다.

## 싱글톤 방식의 주의점
- 싱글톤 객체는 상태를 유지하게 설계하면 안된다. ( 무상태로 설계한다. )
- 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
- 필드 대신에 공유되지 않는 지역변수, 파라미터 등을 사용해야한다.
- 싱글톤 객체에 공유되는 필드는 만들지 않도록 하자.

- 문제가 되는 코드
```
    private int price; // 상태를 유지하는 필드

    public void order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        this.price=price;
    }

    public int getPrice(){
        return price;
    }
```
- 사용자들이 동시에 order를 했을 경우 price를 공유하는 상황이기 때문에 price가 올바르게 저장되지 않는다.

- 수정코드
```
public class StatefulService {
    public int order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        return price;
    }
}

```
### 스프링에서 싱글톤이 보장되는 방식
- @Configuration의 사용
    - 바이트코드를 조작하는 CGLIB의 기술을 사용해서 싱글톤이 보장되도록 해준다.
    - @Bean이 붙은 매소드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고 없다면 스프링 빈으로 등록하고 반환한다.
- @Configuration의 사용하지 않는다면 객체를 스프링  컨테이너가 관리하지 않는다.
