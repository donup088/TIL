### 코틀린에서 클래스를 다루는 방법
- 클래스 만들기
```
public class JavaPerson {
    private final String name;
    private int age;
    public JavaPerson(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
- 위의 자바코드를 코틀린으로 바꾸게되면 아래처럼 작성할 수 있다.
```
class Person(val name: String, var age: Int)
```
- 코틀린에서는 getter,setter를 자동으로 만들어준다.
```
val person = Person("테스터", 123)
println(person.name)
person.age = 10
println(person.age)
```
- 위 코드처럼 getter, setter를 사용할 수 있다.
- init 블록사용
    ```
    class Person(val name: String, var age: Int){
        init {
            if(age<=0){
                throw IllegalArgumentException("나이가 0이하일 수 업습니다.")
            }
        }
    }
    ```
    - init은 생성자가 호출되는 시점에 호출된다.
- 생성자 추가하기
    ```
    class Person(val name: String, var age: Int) {
        init {
            if (age <= 0) {
                throw IllegalArgumentException("나이가 0이하일 수 업습니다.")
            }
        }

        constructor(name: String) : this(name, 1)
    }
    ```
    - 생성자를 추가하고 싶을 경우 consturctor로 생성자를 추가해야한다.
    - 하지만 해당 방법보단은 default parameter를 쓰는 것이 더 좋다.
- 커스텀 getter, setter 사용하기
    ```
    fun isAdultFun(): Boolean {
        return this.age >= 20
    }

     val isAdult: Boolean
        get() = this.age >= 20
    ```
    ```
    println(person.isAdultFun())
    println(person.isAdult)
    ```
    - 객체의 속성처럼 사용하고 싶을 때 커스텀 getter를 사용하면 좋다.


### 상속을 다루는 방법
```
abstract class Animal(protected val species: String, protected val legCount: Int) {
    abstract fun move()
}
```
```
class Cat(species: String) : Animal(species,4){
    override fun move() {
        println("고양이~!")
    }
}
```
- 상속받은 클래스는 상위 클래스의 생성자를 바로 호출한다.
- override를 필수적으로 사용해야한다.
- 추상 프로퍼티가 아니라면 상속받을 때 open을 꼭 사용해야한다.
```
abstract class Animal(protected val species: String, protected open val legCount: Int) {
    abstract fun move()
}
```
- 인터페이스 구현도 : 을 사용한다.
```
class Penguin(species: String) : Animal(species, 2), Swimable, Flyable {
    private val wingCount: Int = 2

    override fun move() {
        println("펭귄~")
    }

    override val legCount: Int
        get() = super.legCount + this.wingCount

    override fun act() {
        super<Swimable>.act()
        super<Flyable>.act()
    }
}
```
- 상위 클래스 설계할 때 생성자 또는 초기화 블록에 사용되는 프로퍼티에는 open을 피해야한다.

### 접근제어 
- public : 모든 곳에서 접근 가능
- protected : 선언된 클래스 또는 하위 클래스에서만 접근 가능
- internal: 같은 모듈에서만 접근 가능
- private : 선언된 클래스내에서만 접근 가능
- 생성자에 접근 지시어를 붙일 때는 constructor를 명시적으로 써주어야한다.

### object 키워드를 다루는 방법
- java의 static 변수와 함수를 만드려면 코틀린에서는 companion object를 사용해야한다.
```
class Person private constructor(
    var name: String,
    var age: Int
) {
    companion object {
        private const val MIN_AGE = 1
        fun newBaby(name: String): Person {
            return Person(name, MIN_AGE)
        }
    }
}
```
- 코틀린에서 싱글톤 클래스를 만들 때 object 키워드를 사용한다.
- 익명클래스를 만들 때 object : 타입을 사용한다.

### 중첩 클래스 다루기
```
class House(
    val address: String,
    val livingRoom: LivingRoom
) {
    class LivingRoom(
        private val area: Double
    )
}
```
- 바깥 클래스를 참조하고 싶다면 inner 키워드를 붙여야한다.
- inner class에서 바깥 클래스를 참조하려면 this@바깥클래스 를 사용해야한다.

### 다양한 클래스 다루기
- Data class : equeals, hashCode, toString 을 자동으로 만들어준다.
```
data class PersonDto(
    val name: String,
    val age: Int
)
```
- Enum Class : 자바의 Enum Class와 동일하지만 when과 함께 사용함으로써 큰 장점을 가지게 된다.
- Enum Class 보다 유연하지만 하위 클래스를 제한하는 Sealed Class 역시 when과 함께 주로 사용된다.