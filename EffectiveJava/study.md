## String vs StringBuffer/StringBuilder, 
- String은 불변의 속성을 갖는다.
    ```
    String str = "hi";
    str=str+ " hello";
    ```
    - String 클래스는 "hi hello"라는 값을 가지고 있는 새로운 메모리영역을 가리키기 변경되고 처음 선언했던 "hi" 값이 할당되어 있던 메모리 영역은 Garbage로 남아있다가 Garbage Collection에 의해 사라진다.
    - String이 불변이기 때문에 문자열이 수정되면 새로운 String 인스턴스가 생성된다.
    - 불변성을 갖기 때문에 변하지 않는 문자열을 자주 읽어들이는 경우 String을 사용해주는 것이 좋다. 하지만 추가, 수정, 삭제 등 연산이 자주 발생하는 상황에 String 클래스를 사용하면 힙 메모리에 많은 Garbage가 생성되어 힙메모리 부족으로 성능에 치명적인 영향을 끼치게 된다.
- StringBuffer/StringBuilder는 가변성을 갖는다.
    - append(), delete() 등의 API를 이용하여 동일 객체내에서 문자열을 변경하는 것이 가능하다.
    - 따라서 문자열에 추가, 수정, 삭제 등 연산이 빈번하다면 StringBuffer/StringBuilder를 사용하는 것이 좋다.

## StringBuffer vs StringBuilder
- StringBuffer는 동기화 키워드를 지원하여 멀티쓰레드 환경에서 안전하다.(thread-safe)
- StringBuilder는 동기화를 지원하지 않기 떄문에 멀티쓰레드 환경에서 사용하는 것은 적합하지 않다. 하지만 단일쓰레드의 성능은 StringBuilder가 더 좋다.


