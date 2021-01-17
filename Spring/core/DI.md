## 의존관계 주입 방법
- 생성자 주입
- setter 주입
- 필드 주입
- 일반 메서드 주입

### 생성자 주입
- 생성자 호출시점에 1번만 호출된다.
- 불변, 필수 의존관계에 사용
```
 @Autowired
public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

### setter 주입
- 선택, 변경 가능성이 있는 의존관계에 사용
```
@Autowired
public void setMemberRepository(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
}
```

### 필드 주입
- 외부에서 변경이 불가능해서 테스트 하기 힘들다.
- DI 프레임워크가 없으면 아무것도 할 수 없다.
- 필드주입은 사용하지 말고 수정자 주입을 사용하자.
- 테스트코드에는 사용해도 된다.
```
@Autowired
private MemberRepository memberRepository;
```

### 의존관계 주입을 할 때 조회되는 빈이 2개이상일 때
- NoUniqueBeanDefinitionException 오류가 발생한다.
- 해결방법
    - @Autowired 필드 명 매칭
        - 타입 매칭
        - 타입 매칭의 결과가 2개 이상일 때 필드 명으로 빈 이름 매칭
    - @Quilifier 사용 
        ```
        @Component
        @Qualifier("fixDiscountPolicy")
        public class FixDiscountPolicy implements DiscountPolicy{
            ...
        }
        ```
        ```
        @Autowired
        public OrderServiceImpl(MemberRepository memberRepository,
                @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
            this.memberRepository = memberRepository;
            this.discountPolicy = discountPolicy;
        }
        ```
    - @Primary 사용
        - 빈이 여러개 조회될 때 @Primary가 붙은 것이 먼저 조회된다.
    - 애노테이션을 직접 만들어서 사용도 가능하다.
