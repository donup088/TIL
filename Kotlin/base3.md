### 컬렉션 다루기
- List, Set, Map 사용
```
val numbers = listOf(100, 200)
val emptyList = emptyList<Int>()

println(numbers[0])

for (number in numbers) {
    println(number)
}

for ((index, number) in numbers.withIndex()) {
    println("$index $number")
}

val setNumber= mutableSetOf(100,200)

val oldMap= mutableMapOf<Int,String>()
oldMap[1]="MONDAY"

mapOf(1 to "MONDAY", 2 to "TUESDAY")
```

- 코틀린에서는 컬렉션을 만들 때 불변/가변을 지정해야한다.
- Java와 Kotlin을 섞어 사용할 때는 컬렉션 사용에 주의해야한다. Java에서 Kotlin 컬렉션을 가져갈 때는 불변 컬렉션을 수정할 수도 있고, non-nullable 컬렉션에 null을 넣을 수도 있다. 그리고 Kotlin에서 Java 컬렉션을 가져갈 때는 플랫폼타입을 주의해야한다.

### 확장함수
- 확장함수는 원본 클래스의 private, protected 멤버 접근이 안된다.
- 멤버함수, 확장함수 중 멤버함수에 우선권이 있다.
- 확장함수는 현재 타입을 기준으로 호출된다.