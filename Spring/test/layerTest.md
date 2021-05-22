## Layer 별 테스트

### Domain
- 다른 프레임워크의 도움없이 AssertJ를 사용하면 테스트할 수 있다.

### Repository
- @DataJpaTest 슬라이스 테스트를 활용한다.
- @DataJpaTest 테스트는 JPA 연관 관계의 구성, Repository 메서드의 구현을 확인하는 것이 목적이다.

### Service
- Service layer 테스트는 데이터의 CRUD를 Repository에 위임하고 트랜잭션 관리를 확인하는 것이 목적이다.
- 단위테스트를 하기위해 Mockito를 사용한다. JUnit5에서 Mockito를 사용하기 위해 @ExtendWith(MockitoExtension.class)를 사용한다. @InjectMocks의 사용으로 주입받을 객체를 정하고 @Mock의 사용으로 @InjectMocks를 사용한 객체에 필요한 의존성을 주입시킨다.
    ```
    @ExtendWith(MockitoExtension.class)
    class UserServiceTest {
        @InjectMocks
        private UserService userService;
        @Mock
        private FileService fileService;
        @Mock
        private UserRepository userRepository;
        @Mock
        private PasswordEncoder passwordEncoder;
        ...
    }
    ```
- 생성, 조회, 삭제하는 경우에는 메서드 호출 여부를 verify를 사용하여 확인하면 된다.
- 수정할 경우에는 save()나 saveAndFlush() 가 호출되지 않을 수 있으므로 verify로는 테스트가 불가능하기 때문에 AssertJ를 사용하여 엔티티나 DTO의 변경을 확인한다.

### Controller
- 컨트롤러의 로직이 많지 않은 경우 보통 Mock을 사용하는 것보다 실제 서비스 및 도메인 계층을 대상으로 통합테스트를 하는 경우가 좋을 수도 있다.