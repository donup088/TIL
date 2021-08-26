## Spring boot 에서 flyway 사용하기 

### flyway를 사용하는 이유
- 개발을 진행할 때 보통 개발환경,운영환경 등 여러 환경으로 개발을 하게 된다.하지만 개발환경에서 작업한 것들을 운영환경에 반영하는 것을 깜빡할 수 있다.
  이를 방지 하기 위해 flyway를 사용할 수 있다.
- 사람이 DB 관리하는 데 있어서 실수를 하지 않도록 도와준다.

### flyway 사용 방법
- plugin 설정
    ```
    plugins {
        ...
        id "org.flywaydb.flyway" version "7.0.3"
        ...
    }
    ```
- build.gradle 파일에 설정 추가
    ```
    flyway {
        baselineVersion = 0
        encoding = 'UTF-8'
        validateOnMigrate = true
        baselineOnMigrate = true 
        locations = ["filesystem:${file('src/main/resources/db/migration').absolutePath}"]
    }
    ```
- sql 파일 생성
    - V1,V2 이렇게 시작을 해도되고, 날짜로 시작을 해도 된다.
    - V1__create_user.sql, V2__create_team.sql
    - V3__alter_user.sql
    - 파일 순서를 변경하지 않고 삭제도 하지 않는다.
    - 테이블을 만들고 변경할 시점엔 alter문을 사용한다. default 를 사용해서 기본값을 미리 넣어둘 수 있다.
        ```
        ALTER table user add column test_column varchar(255);
        ```