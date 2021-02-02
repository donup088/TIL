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
- self링크와 profile 링크는 항상 추가해준다.
- Resource를 관리하는 클래스를 만들어준 뒤 RepresentationModel 상속 받고 생성자와 getter를 만들어준다.
- 객체를 리소스로 만들고 반환해야한다.
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

- 위와 같은 방법으로 EntityModel을 상속받아 클래스를 만들어 사용하는 방법이있다.
    ```
    public class EventResource extends EntityModel<Event> {
        public EventResource(Event event, Link... links){
            super(event,links);
            add(linkTo(EventController.class).slash(event.getId()).withSelfRel());
        }
    }
    ```
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
    - 링크를 배열로 만들어서 사용할 수도 있다.
    ```
     List<Link> links = Arrays.asList(
                selfLinkBuilder.withSelfRel(),
                selfLinkBuilder.withRel("query-events"),
                selfLinkBuilder.withRel("update-event")
        );
    EntityModel<Event> eventResource = EntityModel.of(newEvent, links);
    ```
    
- 페이징 링크 만들기
    - PagedResourcesAssembler를 받아서 사용한다.
    - toModel 매소드를 사용하여 페이징 링크를 만들게 된다.
    - 하나의 아이템마다 링크를 주기 위해서 toModel에  e -> new EventResource(e)를 추가한다.
    ```
    @GetMapping
    public ResponseEntity queryEvents(Pageable pageable, PagedResourcesAssembler<Event> assembler) {
        Page<Event> page = eventRepository.findAll(pageable);
        var entityModels = assembler.toModel(page, e -> new EventResource(e));
        entityModels.add(new Link("/docs/index.html#resources-events-list").withRel("profile"));
        return ResponseEntity.ok(entityModels);
    }
    ```


