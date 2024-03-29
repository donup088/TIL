## 트랜잭션

### 트랜잭션 ACID
원자성: 트랜잭션 내에서 실행한 작업들은 하나의 작업인 것처럼 모두 성공하거나 실패해야한다.
일관성: 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야한다. 예를 들어 데이터베이스에서 정한 무결성 제약 조건을 항상 만족해야한다.
격리성: 동시에 실행되는 트랙잭션들이 서로에게 영향을 미치지 않도록 격리한다.
지속성: 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다.

### DB 연결 구조와 DB 세션
- 클라이언트는 데이터베이스 서버에 연결을 요청하고 커넥션을 맺는다. 이때 데이터베이스 서버는 내부에 세션이라는 것을 만든다. 앞으로 해당 커넥션을 통한 모든 요청은 이 세션을 통해서 실행하게 된다.
- 커넥션 풀이 10개의 커넥션을 생성하면 세션도 10개 만들어진다.

### DB 락
세션1이 트랜잭션을 시작하고 데이터를 수정하는 동안 아직 커밋을 수행하지 않았는데 세션2에서 동시에 같은 데이터를 수정하게 되면 여러가지 문제가 생긴다. 이런 문제를 방지하려면 세션이 트랜잭션을 시작하고 데이터를 수정하는 동안 커밋이나 롤백 전까지는 다른 세션에서 해당 데이터를 수정할 수 없게 막아야한다.

예시 상황
1. 세션1 트랜잭션 시작 
2. 해당 row의 락을 먼저 획득, 락이 남아있으므로 세션1은 락을 획득한다.
3. 세션1은 락을 획득했으므로 해당 row에 update sql을 수행한다.
4. 세션2 트랜잭션 시작
5. 세션2는 해당 row의 락이 없으므로 락이 돌아올 때까지 대기한다.
6. 이때 무한정 대기하는 것은 아니다. 락 대기 시간을 넘어가면 락 타임아웃 오류가 발생한다.
7. 세션1이 커밋을 수행하면 트랜잭션이 종료되었으므로 락도 반납한다.

일반적인 조회에서는 락을 사용하지 않는다. 하지만 데이터를 조회할 때도 락을 획득하고 싶을 때가 있다. 이럴 때는 select for update 구문을 사용하면 된다.

### 트랜잭션 적용
트랜잭션을 시작하려면 커넥션이 필요하다. 서비스 계층에서 커넥션을 만들고 트랜잭션 커밋 이후에 커넥션을 종료해야한다. 애플리케이션에서 DB 트랜잭션을 사용하려면 트랜잭션을 사용하는 동안 같은 커넥션을 유지 해야한다. 그래야 같은 DB 세션을 사용할 수 있다.

### 트랜잭션 추상화, 동기화
스프링 트랜잭션 추상화의 핵심은 PlatformTransactionManager 인터페이스이다. 해당 인터페이스를 구현하고 있는 JDBC 트랜잭션, JPA 트랜잭션 등 여러 구현체들이 있다.

스프링이 제공하는 트랜잭션 매니저는 트랜잭션 추상화, 리소스 동기화의 역할을 한다.
- 리소스 동기화
    - 스프링은 트랜잭션 동기화 매니저를 제공한다.
    - 트랜잭션 동기화 매니저는 쓰레드 로컬을 사용하여 동기화해주기 때문에 멀티쓰레드 환경에서도 안전하다.
    - 동작방식
        1. 트랜잭션을 시작하려면 커넥션이 필요하다. 트랜잭션 매니저는 데이터소스를 통해서 커넥션을 만들고 트랜잭션을 시작한다.
        2. 트랜잭션 매니저는 트랜잭션이 시작된 커넥션을 트랜잭션 동기화 매니저에 보관한다.
        3. 리포지토리는 트랜잭션 동기화 매니저에 보관된 커넥션을 꺼내서 사용한다. 따라서 파라미터로 커넥션을 전달하지 않아도 된다.
        4. 트랜잭션이 종료되면 트랜잭션 매니저는 트랜잭션 동기화 매니저에 보관된 커넥션을 통해 트랜잭션을 종료하고 커넥션도 닫는다.

### 트랜잭션 전파
1. REQUIRED 
    - 기본 옵션으로 부모 트랜잭션이 존재한다면 부모 트랜잭션에 합류하고 그렇지 않다면 새로운 트랜잭션을 만든다. 중간에 부모/자식에서 롤백이 발생한다면 부모와자식 트랜잭션 모두 롤백한다.
    ```
    @Service
    @Slf4j
    @RequiredArgsConstructor
    public class OuterService {
        private final InnerService innerService;
        private final TestObjectRepository repository;

        @Transactional
        public void txTest() {
            log.info("currentTransactionName : {}",
                    TransactionSynchronizationManager.getCurrentTransactionName());
            try {
                repository.save(new TestObject("부모"));
                innerService.logic();
            } catch (RuntimeException e) {
                log.info(e.getMessage());
            }
        }
    }
    ```
    ```
    @Service
    @Slf4j
    @RequiredArgsConstructor
    public class InnerService {
        private final TestObjectRepository testObjectRepository;

        @Transactional
        public void logic() {
            log.info("currentTransactionName : {}",
                    TransactionSynchronizationManager.getCurrentTransactionName());
            testObjectRepository.save(new TestObject("자식"));
            throw new RuntimeException();
        }
    }
    ```
    - 위와 같은 코드에서 OuterService가 InnerService에서 발생하는 RuntimeException을 try catch로 잡아서 처리하기 때문에 부모 트랜잭션이 커밋되는 오해를 하기 쉽다.
    - 스프링 트랜잭션은 트랜잭션에서 예외가 발생하면 rollback-only를 마킹하고 최종 커미승ㄹ 할 때 마킹이 되어 있어 롤백시켜버린다. 따라서 예외가 발생하면 해당 트랜잭션을 재사용할 수 없다.
2. REQUIRES_NEW
    - 무조건 새로운 트랜잭션을 만든다.
    - 부모 트랜잭션에 예외가 발생해도 자식 트랜잭션에는 꼭 커밋되어야 하는 상황에서 사용하면 좋다.
3. MANDATORY
    - 무조건 부모 트랜잭션에 합류시킨다. 부모 트랜잭션이 존재하지 않는다면 예외를 발생시킨다.
4. SUPPORTS
    - 메소드가 트랜잭션이 필요하진 않지만, 진행중인 트랜잭션이 존재한다면 트랜잭션을 사용한다. 진행중인 트랜잭션이 없더라도 정상적으로 동작한다.
5. NESTED
    - 부모 트랜잭션이 존재한다면 부모 트랜잭션에 중첩시키고, 존재하지 않는다면 새로운 트랜잭션을 생성한다. 부모 트랜잭션에 예외가 발생하면 자식 트랜잭션도 롤백시키고 자식 트랜잭션에서 롤백이 발생하면 부모 트랜잭션에선 자식 트랜잭션을 호출하는 지점까지만 롤백된다. 이후 부모 트랜잭션에 문제가 없으면 부모 트랜잭션은 커밋된다.
6. NEVER 
    - 메소드가 트랜잭션을 필요로 하지 않는다. 만약 진행중인 트랜잭션이 존재하면 예외가 발생한다.
    