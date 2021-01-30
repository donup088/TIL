### HATEOAS
- 애플리케이션 상태에 따라 링크정보가 동적으로 바뀌어야한다.
- 링크를 만드는 기능 (linkTo 사용, new Link() 사용)
    - HREF
    - REL
        - self
        - profile
        - update-event
        - query-event
- 리소스를 만드는 기능



### 링크 만들기
- Resource를 관리하는 클래스를 만들어준 뒤 RepresentationModel 상속 받고 생성자와 getter를 만들어준다.
```
public class EventResource extends RepresentationModel {
    //@JsonUnwrapped
    private Event event;

    public EventResource(Event event) {
        this.event = event;
    }

    public Event getEvent() {
        return event;
    }
}
```
- 이렇게 코드를 작성하면 api가 event에 감싸지게 된다. 그렇지 않길 원하면 @JsonUnwrapped 을 사용한다.
- EventResource를 사용하는 곳에서 add를 사용하여 링크를 만든다.
    ```
    EventResource eventResource = new EventResource(event);
    eventResource.add(linkTo(EventController.class).withRel("query-events"));
    eventResource.add(selfLinkBuilder.withSelfRel());
    eventResource.add(selfLinkBuilder.withRel("update-event"));
    ```
- @JsonUnwrapped 대신 EntityModel 을 사용해도 api Event에 감싸여서 나오지 않는다.
    ```
    EntityModel eventResource = EntityModel.of(newEvent);
    eventResource.add(linkTo(EventController.class).withRel("query-events"));
    ```