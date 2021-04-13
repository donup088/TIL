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