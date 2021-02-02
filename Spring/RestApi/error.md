### @Valid 사용
- 의존성을 maven이나 gradle에 추가시켜줘야한다.
- 검증을 하고 Errors로 에러를 받을 수 있다.
- 검증 결과 나온 Errors를 통해 여러 기능을 사용할 수 있다.
    ```
    if (errors.hasErrors()) {
            return ResponseEntity.badRequest().build();
    }
    ```
- Errors 를 json 객체로 바꾸어 클라이언트에 전달하려고 할 때 body에 넣어서 바로 json 객체로 바꿀 수 없다. 따라서 따로 json객체로 변환시키는 클래스를 만들어줘야한다.
    - @JsonComponent : ObjectMapper에 등록을 쉽게 할 수 있다.
    -  JsonSerializer<?> 를 상속받아 serialize 매서드를 Override 한다.
    ```
    @JsonComponent
    public class ErrorsSerializer extends JsonSerializer<Errors> {
        @Override
        public void serialize(Errors errors, JsonGenerator gen, SerializerProvider serializerProvider) throws IOException {
            gen.writeStartArray();
            errors.getFieldErrors().forEach(e -> {
                try {
                    gen.writeStartObject();
                    gen.writeStringField("field", e.getField());
                    gen.writeStringField("objectName", e.getObjectName());
                    gen.writeStringField("code", e.getCode());
                    gen.writeStringField("defaultMessage", e.getDefaultMessage());
                    Object rejectedValue = e.getRejectedValue();
                    if (rejectedValue != null) {
                        gen.writeStringField("rejectedValue", rejectedValue.toString());
                    }
                    gen.writeEndObject();
                } catch (IOException ioException) {
                    ioException.printStackTrace();
                }
            });

            errors.getGlobalErrors().forEach(e -> {
                try {
                    gen.writeStartObject();
                    gen.writeStringField("objectName", e.getObjectName());
                    gen.writeStringField("code", e.getCode());
                    gen.writeStringField("defaultMessage", e.getDefaultMessage());
                    gen.writeEndObject();
                } catch (IOException ioException) {
                    ioException.printStackTrace();
                }
            });
            gen.writeEndArray();
        }
    }
    ```
    
### 애노테이션만으로 검증이 불가능 할 때
- 검증클래스를 만들어서 검증을 거친다.
- EventValidator 클래스 생성
- rejectValue()를 사용하면 필드에러에 들어가고 reject()을 사용하면 글로벌에러에 들어간다.
    ```
    @Component
    public class EventValidator {
        public void validate(EventDto eventDto, Errors errors) {
            if (eventDto.getBasePrice() > eventDto.getBasePrice() && eventDto.getMaxPrice() != 0) {
                errors.rejectValue("basePrice", "wrongValue", "BasePrice is Wrong");
                errors.rejectValue("maxPrice", "wrongValue", "MaxPrice is Wrong");
            }
            LocalDateTime endEventDateTime = eventDto.getEndEventDateTime();
            if (endEventDateTime.isBefore(eventDto.getBeginEventDateTime()) ||
                    endEventDateTime.isBefore(eventDto.getCloseEnrollmentDateTime()) ||
                    endEventDateTime.isBefore(eventDto.getBeginEnrollmentDateTime())) {
                errors.rejectValue("endEventDateTime", "wrongValue", "EndEventDateTime is Wrong");
            }
        }
    }
    ```
- 검증 클래스를 만들었다면 사용할 클래스에서 주입받아서 사용하면된다.

### 인덱스 페이지 만들기
- 에러가 났을 때 인덱스 api로 이동가능해야하기 때문에 링크를 만들어준다.
```
@GetMapping("/api")
public RepresentationModel index(){
    var index=new RepresentationModel<>();
    index.add(linkTo(EventController.class).withRel("events"));
    return index;
}
```
    - {"_links":{"events":{"href":"http://localhost:8080/api/events"}}} 링크가 생긴다.
- 에러 리소스 클래스 만들기 
    ```
    public class ErrorsResource extends RepresentationModel {
        private Errors errors;

        public ErrorsResource(Errors errors) {
            this.errors = errors;
            add(linkTo(methodOn(IndexController.class).index()).withRel("index"));
        }

        public Errors getErrors() {
            return errors;
        }
    }
    ```
    - "_links":{"index":{"href":"http://localhost:8080/api" 링크가 생긴다.