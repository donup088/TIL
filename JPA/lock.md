## JPA에서 비관적락과 낙관적락 사용하기

### 낙관적락
낙관적락은 트랜잭션 충돌이 거의 발생하지 않는다는 낙관적인 가정에서 동작한다. 트랜잭션이 commit 되는 시점에 충돌 여부를 알 수 있다는 특징이 있다.

- LockModeType.None
    1. 커밋시점에 업데이트 쿼리를 보낸다. 이 때 where 절에 version 정보를 함께 보낸다.
    2. version 정보로 검색된다면 데이터가 수정되지 않았다는 것을 의미하고 version을 올린다.
    3. version 정보로 검색되지 않는다면 데이터가 이미 수정되었다는 것으로 ObjectOptimisticLockingFailureException이 발생한다.
update 쿼리를 포함하고 있지 않다면 조회쿼리만 발생한다.

- LockModeType.Optimistic
    1. 커밋 시점에 version 조회 쿼리를 보낸다. 
    2. version 정보로 검색된다면 데이터가 수정되지 않았다는 것을 의미하고 version을 올린다.
    3. version 정보로 검색되지 않는다면 데이터가 이미 수정되었다는 것으로 ObjectOptimisticLockingFailureException이 발생한다.
LockModeType.Optimistic 을 사용하면 update 쿼리를 포함하고 있지 않고 조회만 해도 version을 select하는 쿼리가 발생하여 dirty read, non-repeatable read를 방지할 수 있다.

- LockModeType.Force_Increment
    - 논리적 단위로 묶인 엔티티의 버전을 함께 관리할 수 있다.

### 비관적락
비관적락을 적용하면 락을 획득할 때까지 트랜잭션이 대기한다. 무한정 대기를 하면 안되기 때문에 타임아웃시간을 줄 수 있다.

- LockModeType.PESSIMISTIC_WRITE
    - 일반적으로 사용하는 비관적 락으로 select for update 쿼리를 이용해 현재 조회하고 있는 row에 lock을 건다. 따라서 DB에 데이터를 조회하는 시점에 트랜잭션 충돌 여부를 확인할 수 있다.
    - 락을 얻을 때까지 트랜잭션이 대기하기 때문에 격리성은 올리가지만 동시성이 매우 떨어진다. 따라서 매우 중요한 상황이 아니면 비관적락 사용을 고민해볼 필요가 있다.

### JPA의 Lock 사용
JPA에서 Lock을 사용하기 위해서 @Transactional 을 service layer에 사용하고 repository에 lock을 걸면 사용할 수 있다. 조회 기능에 lock을 걸더라도 @Transactional을 사용해야한다.

JPQL이나 Querydsl에서 lock을 사용하기 위해선 .setLockMode를 통해서 lock을 걸 수 있다.