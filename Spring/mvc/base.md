### 멀티쓰레드
- 쓰레드
    - 애플리케이션 코드를 하나하나 순차적으로 실행하는 것
    - 동시 처리가 필요하면 쓰레드를 추가로 생성
    - 요청마다 쓰레드를 재생성하는 것을 비효율적이다. 그래서 쓰레드 풀을 만들어 놓고 사용한다.
    - 쓰레드 풀의 적정숫자를 성능테스트로 찾는 것이 좋다. 너무 적거나많은 숫자로 설정하면 문제가 발생한다.

### Http 요청 데이터
1. GET 쿼리 파라미터
    - 메시지 바디 없이 URL의 쿼리 파라미터에 데이터를 포함해서 전달
2. POST HTML Form
    - 메시지 바디에 쿼리 파라미터 형식으로 전달
3. HTTP message body
    - 데이터 형식은 주로 JSON사용

### Model View Controller
- 컨트롤러: HTTP 요청을 받아서 파라미터를검증하고 비즈니스 로직 실행, 뷰에 전달할 결과 데이터를 조회해서 모델에 담는다.
- 모델: 뷰에 출력할 데이터를 담아둔다.
- 뷰: 모델에 담겨있는 데이터를 사용하여 화면을 만든다.

### SpringMvc 구조
1. 핸들러 조회: 핸들러 매핑을 통해 URL에 매핑된 핸들러(컨트롤러)를 조회
2. 핸들러 어댑터 조회
3. 핸들러 어댑터 실행
4. 핸들러 어댑터가 실제 핸들러 실행
5. ModelAndView를 반환
6. viewResolver 호출
7. View 반환
8. 뷰 랜더링

### MutiValueMap
- 하나의 키에 여러 값을 받을 수 있다.
- HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용

### RequestParam
- @RequestParam(required=false) 설정가능 
- int age(required=false) 라고 하면 int에 null이 들어갈 수 없기 때문에 Integer로 바꿔줘야한다. int로 쓰기 위해서는 defaultValue 설정을 추가해준다.

### ModelAttribute
- @ModelAttribute
- 해당 객체 생성
- 해당 객체의 프로퍼티를 찾아 setter를 호출하여 파라미터값을 바인딩한다.

### RequestBody
- @RequestBody를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.

### HTTP 메시지 컨버터
1. ByteArrayHttpMessageConverter
    - byte[] 데이터 처리
2. StringHttpMessageConverter
    - String 문자로 데이터 처리
3. MappingJackson2HttpMessageConverter
    - application/json 데이터 처리
    - ex) ObjectMapper