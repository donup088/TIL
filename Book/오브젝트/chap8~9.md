## 의존성
- 어떤 객체가 작업을 정상적으로 수행하기 위해 다른 객체를 필요로 하는 경우 두 객체 사이에 의존성이 존재한다. 의존성은 방향을 가진다.
- 객체들이 서로 의존하고 있다는 것은 변경이 이루어질 때 의존하고 있는 요소도 함께 변경될 수 있다는 것을 의미한다.

### 의존성 전이
- PeriodCondition이 Screening에 의존할 경우 PeriodCondition은 Screening이 의존하고 있는 대상에 대해서도 자동적으로 의존하게 된다.
- 위의 내요을 바탕으로 의존성은 직접 의존성과 간접 의존성으로 나뉜다.

### 런타임 의존성과 컴파일타임 의존성
- 런타임 의존성
    - 객체 사이의 의존성
- 컴파일타임 의존성
    - 클래스 사이의 의존성
- 런타임 의존성과 컴파일 타임 의존성을 다르게 만들어야 유연하고 재사용 가능한 코드를 설계할 수 있다.
    ```
    Movie 클래스는 DiscountPolicy에 의존하고 있다. DiscountPolicy의 구현체들은 AmountDiscountPolicy와
    PercentDiscountPolicy가 있다. Movie에서 메서드를 DiscountPolicy로 실행시켜도 결국 런타임 의존성은
    AmountDiscountPolicy와 PercentDiscountPolicy에 해당하게 된다.
    ```
- 런타임 의존성과 컴파일타임 의존성 분리하기
    - 각각의 객체에 interface를 변수로 사용하고 객체가 만들어질 때 해당 interface를 구현하고 있는 구현체를 전달한다.
    - 생성자 방식과 setter 방식을 혼합해서 사용하면 더 유연한 코드를 작성할 수 있다.

### 유연한 설계를 위한 의존성 개선
- 의존성이 있다고해서 안좋은 것이 아니라 의존하는 정도에 따라 달라진다.
- 의존성을 느슨한 결합도를 가지도록 설계해야한다.
- 결합도의 정도는 한 요소가 다른 요소에 대해 정보를 많이 알고 있을 수록 강하게 결합된다.
- 추상화 의존하자.
    - 구체 클래스 의존성 
    - 추상 클래스 의존성        -> 아래로 내려갈수록 추상적이다.
    - 인터페이스 의존성
    - 의존하는 대항이 더 추상적일수록 결합도는 낮아진다.

### new 사용을 함부로하지 말자.
- new 명령어를 사용하는 것은 해당객체와 강하게 연결되는 것이다. 이를 생각해서 사용해야한다.
- 하지만 해당 객체를 자주 사용하는 경우 사용해도 되는 것처럼 항상 안좋은 것은 아니다.

### 개방-폐쇄 원칙 (OCP)
- 소프트웨어 개체(클래스,모듈,함수 등등)는 확장에 대해 열려있어야하고 수정에 대해서는 닫혀 있어야한다.
- 기존 코드를 수정하지 않고 애플리케이션 동작을 확장시킬 수 있는 설계라고 말한다.
- 컴파일타임 의존성을 고정시키고 런타임 의존성을 변경하라.
    - Movie 클래스는 추상 클래스인 DiscountPolicy에 의존한다. 런타임 의존성 관점에서 보면 Movie 인스턴스는  AmountDiscountPolicy,PercentDiscountPolicy 에 의존한다.
- OCP에 핵심은 추상화에 의존하는 것이다.
- 생성과 사용을 분리하라.
    - Movie의 클라이언트가 적절한 DiscountPolicy 인스턴스를 생성한 후 Movie에게 전달한다.
- 유연하고 재사용 가능한 설계를 원한다면 모든 의존성의 방향이 추상 클래스나 인터페이스와 같은 추상화를 따라야한다. 구체 클래스는 의존성에 시작점이어야한다. 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안된다. 모두 추상화에 의존해야한다.
- Movie와 DiscountPolicy는 같은 패키지에 위치하고 AmountDiscountPolicy,PercentDiscountPolicy는 또다른 패키지에 위치해야한다. 