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