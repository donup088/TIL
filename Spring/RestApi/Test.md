### api 테스트
```
mvc.perform(post("/api/events")
                .contentType(MediaType.APPLICATION_JSON)
                .accept(MediaTypes.HAL_JSON)
                .content(objectMapper.writeValueAsString(event)))
                .andDo(print())
                .andExpect(status().isCreated())
                .andExpect(jsonPath("id").exists());
```
- accept(MediaTypes.HAL_JSON) : MediaTypes.HAL_JSON 타입을 응답으로 원한다.
-  .content(objectMapper.writeValueAsString(event))) : event 객체를 json으로 objectMapper를 통해서 바꾼다.
- print() 사용으로 결과를 볼 수 있다.
- objectMapper를 사용

### Mokito 사용
- 객체의 메소드를 실행했을 때 리턴되는 값을 지정해줄 수있다. Test에서 주입받은 객체의 메소드를 사용할 때 NullPointer가 나지 않도록 한다.
    ```
     Mockito.when(eventRepository.save(event)).thenReturn(event);
    ```

### 테스트 description 애노테이션 생성
- 테스트를 잘 알아볼 수 있도록 애노테이션을 생성한다.
- Junit 5에서는 지원하는 기능이고 4에서는 지원하지 않기 때문에 애노테이션을 직접 만들어 사용하면 편하다.
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface TestDescription {
    String value();
}
```

### 웹 테스트 Tips
- @SpringbootTest 를 사용하여 mocking을 줄인다. @SpringbootTest를 사용하면 더 많은 Bean들을 등록하여 테스트가 수월해진다.

- json 값 테스트
    - jsonPath("id").value("1") 이렇게 확인할 수 있다.


### 파라미터를 사용하여 테스트코드 만들기
- 의존성 추가
```
    <dependency>
        <groupId>pl.pragmatists</groupId>
        <artifactId>JUnitParams</artifactId>
        <version>1.1.1</version>
        <scope>test</scope>
    </dependency>
```
- @RunWith(JUnitParamsRunner.class) 추가
```
    @ParameterizedTest
    @MethodSource("paramsForTestFree")
    public void testFree(int basePrice, int maxPrice, boolean isFree) {
        // given
        Event event = Event.builder()
                .basePrice(basePrice)
                .maxPrice(maxPrice)
                .build();

        // when
        event.update();

        // then
        assertThat(event.isFree()).isEqualTo(isFree);
    }

    private static Stream<Arguments> paramsForTestFree() { 
        return Stream.of(
                Arguments.of(0,0, true),
                Arguments.of(100, 0, false),
                Arguments.of(0, 100, false),
                Arguments.of(100, 200, false)
        );
    }
```
- 