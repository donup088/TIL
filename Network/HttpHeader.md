### HTTP 메시지 헤더 구조
- 메시지 헤더: 클라이언트와 서버 처리에 필요한 주요정보
- 개행 문자
- 메시지 바디: 사용자와 리소스를 필요로 하는 정보가 있다.

### 일반 헤더 필드
- request와response 메시지 양쪽에서 모두 사용
- Cache-Control 헤더: 디렉티브로 불리는 명령을 사용하여 캐싱 동작을 지정
    - public 디렉티브
        - Cache-control: public --> 다른 유저에게도 돌려줄 수 있는 캐시를 해도 좋다.
    - private 디렉티브
        - Cache-control: private --> 특정 유저만을 대상으로 한다.
    - no-chache 디렉티브
        - Cache-control: no-cache --> 캐시로부터 오래된 리소스가 반환되는 것을 막기 위함, 오리진 서버는 캐시 서버가 이후의 요청에서 리소스의 유효성을 재확인하지 않고는 해당 response를 사용하지 못하도록 한다.
        - Cache-control: no-cache=Location : 지정된 헤더 필드만 캐시할 수 없게된다.
    - no-store 디렉티브
        - Cache-Control: no-store --> 요청or응답에 기밀 정보가 있기 때문에 캐시는 요청or응답의 일부분을 로컬 스토리지에 보관해서는 안되도록 지정
    - s-maxage 디렉티브
        - Cache-Control: s-maxage=604800(단위:초)
        - 캐시가 유효성을 확인하지 않고 리소스를 캐시에 보존해두는 최대시간을 의미
    - min-fresh 디렉티브
        - Cache-Control: min-fresh=60(단위:초)
        - 캐시된 리소스가 적어도 지정된 시간은 최신 상태의 것을 반환하도록 캐시 서버에 요구
    - max-stale 디렉티브
        - Cache-Control: max-stale=3600(단위:초)
        - 유효 기한이 지난 후로부터 지정 시간 내라면 받아드림
    - only-if-cached 디렉티브
        - Cache-Control: only-if-cached
        - 캐시서버에 리소스가 있는 경우에만 반환
    - must-revalidate 디렉티브
        - Cache-Control: must-revalidate
        - response의 캐시가 현재도 유효한지 오리진 서버에 조회를 요구
    - proxy-revalidate 디렉티브
        - Cache-Control: proxy-revalidate
        - 모든 캐시서버에 대해서 response 반환시 유효성 재확인을 하도록 요구
    - no-transform 디렉티브
        - Cache-Control: no-transform
        - 요청이나 응답 모두 캐시가 엔티티 바디의 미디어 타입을 변경하지 않도록함

        