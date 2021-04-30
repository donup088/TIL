### cascade remove 설정으로 연관된 데이터 삭제하는 경우
- Review와 File이 중간테이블 ReviewFile로 연관된 상황
    - cacade 옵션을 사용하여 Review가 삭제되었을 때 ReviewFile과 file을 삭제시키면 삭제 대상들을 전부 조회하는 쿼리가 1번 발생하고 삭제 대상들은 1건씩 삭제되어진다. cascade 옵션으로 연관되어진 엔티티들도 1건씩 삭제가 된다.
    이 부분을 In 쿼리로 하면 한번에 삭제가 가능하고 제 대상들을 전부 조회하는 select 쿼리도 적게나간다.

    ```
    //FileRepository
    // @Modifying을 사용하는 경우 clearAutomatically=ture로 설정하여 영속성 컨텍스트를 초기화해야하지만
    // service layer에서 변경감지 사용과 벌크성 삭제 쿼리를 하고 findBy로 다시 조회하는 것이 없기 때문에
    // 설정추가 X
    @Modifying
    @Query("delete from File f where f.id in :ids")
    void deleteAllByIds(@Param("ids") List<Long> fileIdList);
    //RouteReviewFileRepository
    @Modifying
    @Query("delete from RouteReviewFile rf where rf.file.id in :ids")
    void deleteAllByFileIds(@Param("ids") List<Long> fileIdList);

    // service layer
    routeReviewFileRepository.deleteAllByFileIds(fileIdList);
    fileRepository.deleteAllByIds(fileIdList);
    routeReviewRepository.deleteById(id);
    ```

### ToOne 관계는 모두 fetch join 사용
- ToOne 관계에서는 fetch join을 사용하고 페이징처리까지 가능하다.


### 일대다 컬렉션 fetch join
- 일대다인 경우가 한가지라면 distinct 를 사용해서 fetch join을 사용할 수 있다. 하지만 DB입장에서는 중복된 데이터가 그대로 적용되어 있으므로 페이징처리는 사용하면 안된다.


### 일대다에 일대다 이렇게 연결되어 있는 경우 fetch join말고 default_batch_fetch_size 를 설정하자.
- 컬렉션 fetch join을 여러번 사용하는 경우 데이터가 부정합하게 조회될 수 있기 때문에 여러번 사용하는 경우는 사용하지 않는 것이 좋다.
- 연관되어 있는 엔티티를 조회할 때 한번에 100개씩 in쿼리를 사용하여 가져와 쿼리를 줄일 수 있다.
- ToOne 관계까지는 fetch join을 사용하고 ToMany 관계 이후로 부터는 lazy 로딩으로 조회한다.
    ```
    spring:
        jpa:
            hibernate:
                default_batch_fetch_size: 100
    ```

### OSIV(Open Session In View)
- spring.jpa.open-in-view : true 기본값
- OSIV 전략은 트랜잭션 시작처럼 최초 데이터베이스 커넥션 시작 시점부터 API 응답이 끝날 때까지 영속성 컨텍스트와 데이터베이스 커넥션을 유지한다. 그래서 View Template이나 API 컨트롤러에서 지연로딩이 가능하다.
- 지연로딩은 영속성 컨텍스트가 있어야 가능하고 영속성 컨텍스트는 기본적으로 데이터베이스 커넥션을 유지한다.
- 이 전략의 단점은 너무 오랜시간동안 데이터베이스 커넥션 리소스를 사용하기 때문에 실시간 트래픽이 중요한 애플리케이션에서는 커넥션이 모자를 수 있다.
- ex) 컨트롤러에서 외부 API를 호출하면 외부 API 대기시간만큼 커넥션 리소스를 반환하지 못한다.
- OSIV를 false로 하는 경우
    - 트랜잭션이 종료할 때 영속성 컨텍스트를 닫고 데이터베이스 커넥션도 반환한다. 따라서 데이터베이스 커넥션을 낭비하지 않는다.
    - sevice layer와 repository layer 범위가 트랜잭션 범위이고 영속성 컨텍스트 생존 범위이다. 
    - 모든 지연로딩은 트랜잭션 안에서 처리해야한다. View template에서의 지연로딩도 동작하지 않는다.
- OSIV 전략의 선택
    - 고객 서비스의 실시간 API 같은 곳에서는 OSIV를 끄고 ADMIN 처럼 커넥션을 많이 사용하지 않는 곳에서는 OSIV를 키고 사용하면 좋다.
