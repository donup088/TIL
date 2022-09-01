### jpa의 기본생성자 사용
```
@Entity
class Book(
    val name: String,
    @Enumerated(EnumType.STRING)
    val type: BookType,
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,
) 
```
- 해당 코드를 보면 jpa에서 기본생성자를 사용할 수 없기 때문에 에러가 발생한다. 이를 해결하기 위해 build.gradle 파일에 플러그인을 추가해준다.
```
plugins {
    ...
    id 'org.jetbrains.kotlin.plugin.jpa' version '1.6.21'
    ...
}
```
### 코틀린 클래스에 대해서 리플렉션을 가능하도록 의존성 추가
```
implementation 'org.jetbrains.kotlin:kotlin-reflect:1.6.21'
```

### 함수와 클래스에 모두 open을 넣는 것을 해결하기
```
plugins {
    ...
    id 'org.jetbrains.kotlin.plugin.spring' version '1.6.21'
    ...
}
```

### 코틀린의 dto 클래스에서 데이터를 못받아오는 경우 의존성 추가
```
implementation 'com.fasterxml.jackson.module:jackson-module-kotlin:2.13.3'
```