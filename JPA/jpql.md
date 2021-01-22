### SELECT
- as는 생략가능하지만 별칭은 필수다.
- 테이블이름이 아니라 엔티티 이름을 사용한다.
    ```
    select m from Member as m where m.age > 10
    ```

 - 파라미터 바인딩
    ```
    select m from Member m where m.username=:usernamme
    query.setParameter("username",username);
    ```

- 결과 조회
    - query.getResultList() : 결과가 하나 이상일 때 리스트 반환, 결과가 없으면 빈 리스트 반환
    - query.getSingleResult(): 결과가 하나일 때 단일 객체 반환, 결과가 안나오거나 여러개이면 예외발생
- 프로젝션 
    - SELECT 절에 조회할 대상을 지정하는 것
    - 임베디드 타입 프로젝션
        ```
        em.createQuery("select o.address from Order o",Address.class).getResultList();
        ```
    - 여러 값을 조회할 때
        - new 연산자 사용 (패키지 이름 적어주고 생성자를 활용한다.)
            ```
            List<MemberDto> result = em.createQuery("select new jpql.MemberDto(m.username,m.age) from Member m",MemberDto.class).getResultList();
            ```
- 페이징 API
    ```
    em.createQuery("select m from Member m order by m.age desc",Member.class)
    .setFirstResult(0)
    .setMaxResults(10)
    .getResultList();
    ```
- 조인
    - 내부 조인(inner는 생략가능)
        ```
        select m from Member m inner join m.team t
        ```
    - 외부 조인
        ```
        select m from Member m left join m.team t
        ```
    - 세타 조인
    - ON 절(연관관계 없는 엔티티 외부 조인 가능)
        ```
        select m from Member m left join m.team t on t.name='A'
        ```
- 서브 쿼리 
    - from 절의 서브 쿼리가 불가능하다.
    - 조인으로 풀 수 있으면 조인으로 풀어서 하자.
- JPQL 기본 함수
    - CONCAT
    - SUBSTRING
    - TRIM 
    - SQL에서 지원하는 것 대부분 가능하다.
    
### 경로 표현식 특징
- 묵시적 내부 조인
    ```
    // inner join이 발생한다.
    select m.team from Member m
    ```
- 묵시적 내부 조인을 사용하면 쿼리 튜닝이 힘들어진다. -> join을 명시적으로 사용하자.
- 컬렉션 값 연관 경로
    - join을 통해서 별칭을 얻어와 사용한다.
    ```
    select m.username from Team t join t.members m
    ```
- 항상 명시적 조인을 사용하자.

### fetch join
- 연관된 엔티티나 컬렉션을 SQL 한번에 조회하는 기능
- join fetch 명령어 사용
```
select m from Member m join fetch m.team
```
- select m from Member m 을 하고 member.getTeam().getTeamName()을 하게 되면 Team을 프록시로 가져온다. 반복해서 팀 이름을 가져올 경우 N+1의 문제가 발생한다. 이때 join fetch를 사용해서 문제를 해결한다.
- 컬렉션 fetch join
    - 일대다 조인을 하면 데이터가 뻥튀기된다.
    - 중복된 결과가 나올 수 있으므로 중복된 결과가 나올 때 distinct로 제거한다.
    ```
    select distinct t from Team t join fetch t.members
    ```

- 한계
    - 대상에 별칭을 줄 수 없다.
    - 둘 이상의 컬렉션은 fetch join을 할 수 없다.
        - 데이터가 폭증할 수 있다.
    - 페이징 API를 사용할 수 없다.
        - 일대다일 경우 데이터 뻥튀기로 인해 페이징 처리가 불가능하다.
        - 다대일이나 일대일 경우에는 사용할 수 있다.
         - @BatchSize 사용, batch_fetch_size 설정으로 데이터를 한번에 여러개 가져올 수 있다.

### 벌크 연산
- 쿼리 한번으로 여러 테이블 row 변경
- 영속성 컨텍스트를 무시하고 DB에 직접 쿼리를 보냄
- 벌크 연산을 실행하고 영속성 컨텍스트를 초기화하자. 
```
int resultCount = em.createQuery("update Member m set m.age=20").executeUpdate()
```