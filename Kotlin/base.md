### 변수 선언
- var: 가변
- val: 불변
- 타입 명시
    - 타입 명시를 해주지 않아도 컴파일러가 추론해주미나 타입 명시를 할 경우에는 var a: Long = 10L 이렇게 할 수 있다.
- 초기화하지 않고 사용할 때는 타입을 명시해줘야한다.
- boxing / unboxing을 고려하지 않아도 되도록 kotlin이 알아서 처리해준다.
- 코틀린에서 null이 변수에 들어갈 수 있다면 타입?를 사용해야한다.
    ```
    var c: Long? = 1000L
    ```
- 코틀린에서 객체 인스턴스화를 할 때 new를 붙이지 않아야 한다.

### Safe Call, Elvis 연산자
- Safe Call
    ```
    val str: String? = null
    println(str?.length)
    ```
    - str?.length는 str이 null이면 null을 반환하고 null이 아닐 경우 length를 반환한다.
- Elvis 연산자
    ```
    val str: String? = null
    println(str?.length ?: 0)
    ```
    - str?.length ?: 0 은 str이 null이면 0이 반환된다.


### 코틀린에서 자바코드 사용
코틀린에서 자바코드를 사용한다면 코틀린에서 해당 메소드가 null이 가능한지 아닌지 확인할 수 없기 때문에 런타임 NPE가 날 수 있다. 이를 플랫폼 타입이라고 한다. 따라서 자바코드의 null 여부를 유의해서 사용해야한다.

### 기본 타입
코틀린의 변수는 초기값을 보고 타입을 추론하며, 기본 타입들 간의 변환은 명시적으로 이루어진다.
```
val c: Long = 2
    println(c.toString() + "123")
```

### 타입 캐스팅
- value is Type 에서 value가 Type이면 true 그렇지 않다면 false가 반환된다.
- value !is Type 은 위와 반대로 작동한다.

- value as Type은 Type으로 타입 캐스팅이 된다. value가 Type이 아니면 예외가 발생한다.
- value as? Type은 value가 Type이면 타입 캐스팅이 되고 null이면 null이 반환되고 value가 Type이 아니어도 null이 반환된다.

### Any, Unit, Nothing
- Any
    - Java의 Object 역할
    - 모든 Primitive Type의 최상의 타입도 Any 이다.
    - Any 자체로는 null을 포함할 수 없고 null을 포함하고 싶다면 Any?로 표현한다.
- Unit
    - Java의 void와 동일한 역할
- Nothing
    - Nothing은 함수가 정상적으로 끝나지 않았다는 사실을 표현하는 역할
    - 무조건 예외를 반환하는 함수 등등

### 비교연산자
- 코틀린에서는 Java와 다르게 객체를 비교할 때 비교 연산자를 사용하면 자동으로 compareTo를 호출해준다.
- 주소값이 같은 지 비교하기 위해 ===을 사용하고 java의 equals와 같은 연산을 하려면 == 을 사용하면 된다.

### switch when
```
fun getGradeWithSwitch(score: Int): String {
    return when (score / 10) {
        9 -> "A"
        8 -> "B"
        7 -> "C"
        else -> "D"
    }
}
```
```
fun getGradeWithSwitch(score: Int): String {
    return when (score) {
        in 90..99 -> "A"
        in 80..89 -> "B"
        in 70..79 -> "C"
        else -> "D"
    }
}
```
```
fun judgeNumber(number: Int) {
    when (number) {
        1, 0, -1 -> println("good")
        else -> println("bad")
    }
}
```
```
fun judgeNumber2(number: Int) {
    when {
        number == 0 -> println("0입니다.")
        number % 2 == 0 -> println("짝수입니다.")
        else -> println("홀수입니다.")
    }
}

```
- 이처럼 when 안의 조건을 다양하게 사용할 수 있다.