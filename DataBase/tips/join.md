## Join 정리하기

### join 이란 ??
- 2개 이상의 테이블을 연결해서 데이터를 검색하는 방법
- join을 하기 위해선 적어도 하나의 컬럼을 공유하고 있어야한다. (주로 pk or fk 사용)

### join의 종류
- 예시 데이터
    ```
    A 테이블
    ID  NAME
    1   A
    2   B
    3   C
    B 테이블
    ID NICKNAME
    1   NICK1
    2   NICK2
    4   NICK4
    5   NICK5
    ```
- INNER JOIN 
    - 내부조인으로 불리고 교집합을 의미한다. 공통된 부분만 가져온다.
    - A,B 테이블 조인 결과
    ```
    ID  NAME    NICKNAME
    1   A       NICK1
    2   B       NICK2
    ```
- LEFT JOIN
    - 조인 기준 왼쪽에 있는 것이 있다면 다른 값이 NULL이라도 가져온다.
    - A,B 테이블 조인결과
    ```
    ID  NAME    NICKNAME
    1   A       NICK1
    2   B       NICK2
    3   C       NULL
    ```
- RIGHT JOIN
    - 조인 기준 오른쪽에 있는 것이 있다면 다른 값이 NULL이라도 가져온다.
    - A,B 테이블 조인결과
    ```
    ID  NAME    NICKNAME
    1   A       NICK1
    2   B       NICK2
    4   NULL    NICK4
    5   NULL    NICK5
    ```
- CROSS JOIN
    - 모든 경우의 수가 나온다. A테이블 row 개수 X B테이블 row 개수 만큼의 row를 가진 테이블이 출력된다.
    - 비용이 매우크고 CROSS JOIN을 사용하는 경우는 거의 없기 때문에 쿼리를 수정하는 것이 좋다.
    - A,B 테이블 조인결과
    ```
    ID  NAME    NICKNAME
    1   A       NICK1
    2   B       NICK2
    3   C       NULL
    1   A       NICK1
    2   B       NICK2
    3   C       NULL
    1   A       NICK1
    2   B       NICK2
    3   C       NULL
    ```

