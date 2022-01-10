## JVM(Java Virtual Machine) 이란??
- JVM의 역할
    - 자바 애플리케이션을 클래스 로더를 통해 읽어 들여 자바 API와 함께 실행하는 것
    - Java 와 OS 사이에서 중개자 역할을 수행하여 Java가 OS에 구애받지 않고 재사용하는 것을 가능하게 해줌
    - 메모리관리, Garbage Collection을 수행

- 자바 프로그램 실행과정
    1. JVM은 OS로부터 프로그램이 필요로 하는 메모리를 할당받는다. JVM은 이 메모리를 용도에 따라 여러 영역으로 나누어 관리한다.
    2. 자바 컴파일러가 자바 소스코드(.java)를 읽어들여 자바 바이트코드(.class)로 변환시킨다.
    3. Class Loader를 통해 class 파일들을 JVM으로 로딩한다.
    4. 로딩된 class 파일들은 Execution engine을 통해 해석된다.
    5. 해석된 바이트코드는 Runtime Data Areas 에 배치되어 수행이 이루어진다. 이러한 실행과정 속에서 JVM은 필요에 따라 Thread Synchronization과 GC같은 관리작업을 수행한다.

- JVM 구성
    - Garbage collector : Runtime Data Area 중 Heap 영역에 더 이상 사용하지 않고 자리만 차지하고 있는 객체들을 제거한다.
    - Class Loader : JVM내로 클래스를 로드하고 링크를 통해 배치하는 작업을 수행한다. .jar 파일 내 저장된 클래스들을 JVM위에 탑재하고 사용하지 않는 클래스들은 메모리에서 삭제한다.
    - Execution Engine(실행 엔진) : 클래스를 실행시키는 역할이다. 
    - Runtime Data Area
- Runtime Data Area(프로그램을 수행하기 위해 OS에서 할당받은 메모리 공간)
    - Method Area : java 어플리케이션이 실행되면 클래스 별로 클래스에서 필요한 패ㅐ키지 클래스, 런타임 상수, 인터페이스 상수, static 변수, final 변수, 클래스 멤버 변수 등 필드데이터, 생성자를 포함한 모든 메서드 정보가 있고 메모리에 항상 상주하고 있는 영역이다. 모든 스레드가 공유가능하다.
    - Heap Area : 메서드 안에서 사용되는 객체들을 위한 영역으로 new를 통해 생성된 객체, 배열, immutal 객체 등의 값이 저장된다. 해당 객체를 참조 하지 않으면 Garbage Collector에 의해 객체가 Heap 영역에서 제거된다.
    - Stack Area : Stack Area는 스레드마다 하나씩 존재하며 쓰레드가 시작될 때 할당된다. 메서드에서 직접 사용할 지역 변수, 파라미터, 리턴 값, 참조 변수 일 경우 주소 값들이 저장된다. 메서드 호출이 종료되면 해당 메서드는 Stack Area에서 제거된다.
    - PC register : 쓰레드가 생성될 때마다 생성되는 영역으로 현재 쓰레드가 실행되는 부분의 주소와 해당 명령을 저장하고 잇는 영역이다.
    - Native Method Stack : 자바 외 다른 언어로 작성된 네이티브 코드를 수행하기 위한 메모리 영역이다.

### Garbage Collector란 무엇인가..?
- Garbage Collector의 과정
    1. Garbage Collectorrk Stack의 모든 변수를 스캔하면서 어떤 객체를 참조하고 있는지 마킹한다.(Mark)
    2. Reachable Object가 참조하고 있는 객체도 마킹한다.(Mark)
    3. 마킹되지 않은 객체를 Heap에서 제거한다.(Sweep)
- Heap 구조
    - New Generation
        - Eden
        - Survival 0
        - Survival 1
    - Old Generation
- Heap과 Garbage Collector
    1. Eden 영역에 꽉차면 Garbage Collector가 작동하고 살아남은 객체들(Reachable Object)는 Survival 0으로 옮겨진다. Unreachable Object는 메모리에서 해제된다.
    2. 1번의 과정이 반복되어 Survival 0가 꽉차면 Survival 1로 똑같은 과정이 일어난다. (Age 증가)
    3. Survival 1에 객체가 생기면 Eden 영역이 꽉찼을 때 Survival 1로 바로 들어온다.
    4. 특정 Age를 넘어서면 Old Generation으로 가게된다.(Promotion)