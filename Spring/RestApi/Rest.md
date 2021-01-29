### REST
- 시스템 각각의 독립적인 진화를 보장하기 위한 방법
- REST 아키텍쳐 스타일을 따르는 API

### REST API
- Self-descriptive message
    - 서버가 변해서 메세지가 변해도 클라이언트는 메세지를 보고 해석이 가능해야한다.
    - 메세지 스스로 메세지에 대한 설명이 가능해야한다.
    - 구현 방법
        - 본문에 Profile 링크 헤더를 추가한다.
- HATEOAS
    - 하이퍼미디어(링크)를 통해 애플리케이션 상태 변화가 가능해야한다.
    - 링크 정보를 동적으로 바꿀 수 있다.
    - 구현 방법
        - HAL(Hypertext Application Language)을 사용하여 링크를 정의한다.

### Controller
```
@RequestMapping(value = "/api/events", produces = MediaTypes.HAL_JSON_VALUE)
```
- controller에서 MediaTypes.HAL_JSON_VALUE 타입만 보내게 할 수 있다.
- linkTo 사용
    ```
    URI createdUri = linkTo(EventController.class).slash("{id}").toUri();
    ```
    - spring hateoas 에서 제공하는 linkTo를 사용하여 uri를 만들 수 있다.
- return
    ```
     return ResponseEntity.created(createdUri).body(event);
    ```
    - create를 할 때는 created(uri)를 사용하여 헤더에 uri 정보를 담고 body에 반환하려는 객체를 담는다.