## JPA를 왜 사용하는가?
1. 기본적인 CRUD SQL 반복 작성을 대신 해주어 생산성 향상
2. 상속, 연관관계, 객체그래프 탐색 등의 패러다임의 불일치 문제를 해결
3. JPA는 다양한 성능최적화 기능 제공
4. 관계형 데이터베이스는 데이터베이스 벤더마다 사용법이 다른 경우가 많지만 JPA는 추상화된 데이터접근 계층을 제공하여 애플리케이션이 특정 데이터베이스 기술에 종속되지 않도록 도와준다.

## JPA 영속성 컨텍스트 이점
1. 1차 캐시 : 1차캐시에 없다면 DB에서 조회하여 1차 캐시에 저장하고 결과를 반환한다.
2. 동일성 보장
3. 쓰기 지연 : 트랜잭션을 지원하는 쓰기 지연이 가능하며 트랜잭션 커밋전까지 SQL을 바로 보내지 않고 모아서 한번에 보낸다.
4. 변경 감지 : 1차 캐시에 들어온 데이터를 스냅샷으로 사용하고 커밋이 되는 시점에 엔티티와 스냅샷을 비교하여 달라진점이 있다면 update 쿼리가 수행된다.
5. 지연 로딩

## N+1 문제
N+1 문제는 하위엔티티들이 존재하는 경우 한 쿼리에서 모두 가져오는 것이 아니라 필요한 곳에서 각각 쿼리들이 발생하는 경우이다.

상위 엔티티에서 하위 엔티티를 가져올 때 JPA에서 해당 엔티티를 조회하기 위해 추가적인 쿼리들 발생

- 해결방법
    1. fetch join , join fetch 사용 
        - inner join
        - MultipleBagFetchException 
            - OneToOne , ManyToOne 과 같이 단일 관계 자식 테이블에는 fetch join을 연결해서 사용해도 된다. 하지만 2개 이상의 OneToMany 자식 테이블에 fetch join을 선언한다면 MultipleBagFetchException 이 발생한다.
            - 이를 해결하기 위해 자식 테이블 하나에만 Fetch Join을 걸고 나머지는 Lazy Loading을 사용하거나 모든 자식 테이블을 다 Lazy Loading으로 할 수 있다. 하지만 이렇게 한다면 성능 이슈가 없어지지 않는다. 이럴 경우 default_batch_fetch_size 옵션을 사용하여 in 쿼리로 한번에 가져오도록 한다. (100~1000개)
    2. @EntityGraph 사용
        - @EntityGraph의 attributePaths에 쿼리 수행시 바로 가져올 필드명을 지정하면 Eager 조회로 가져오게 된다.
        - outer join : 따라서 카테시안 곱이 발생하여 엔티티가 늘어날 수 있다. 이를 해결하기 위해서 일대다 필드타입을 set으로 설정하는 방법 (순서를 보장하기 위해 LinkedHashSet 사용), distinct를 사용하는 방법이 있다.
    3. @BatchSize 사용

## JPA 트랜잭션의 전파 및 격리
- Propagation.REQUIRED (@Transactional(propagation = Propagation.REQUIRED)) -> 기본값
    - 이미 만들어진 트랜잭션이 존재하면 해당 트랜잭션 관리 범위 안에 함께 들어간다. 이미 만들어진 트랜잭션이 존재하지 않으면 새로운 트랜잭션을 만든다.
- Propagation.REQUIRES_NEW (@Transactional(propagation = Propagation.REQUIRES_NEW))
    - 매번 새로운 트랜잭션을 만든다. 이미 만들어진 트랜잭션이 종료되지 않았다면 새로운 트랜잭션은 대기상태가 되어 이전 트랜잭션이 끝나는 것을 기다려야한다.
- Propagation.MANDATORY 
    - 이미 만들어진 트랜잭션 범위안으로 들어가야한다. 이미 만들어진 트랜잭션이 없다면 예외가 발생한다.
- Propagation.NESTED
    - 기본적인 동작은 Propagation.REQUIRED 와 동일하다. 차이점은 SAVEPOINT를 지정한 시점까지 부분 롤백이 가능하다. 데이터베이스가 SAVEPOINT 기능을 지원해야 사용 가능하다. ex) Oracle