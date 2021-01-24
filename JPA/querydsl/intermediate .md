### 프로젝션
- select 대상 지정
- 하나면 타입을 명확하게 지정할 수 있다.
- 둘 이상이면 튜플이나 DTO로 조회한다.

### DTO 조회
- Setter 사용
    - Projection.bean 사용
    ```
    List<MemberDto> result = queryFactory
                .select(Projections.bean(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();
    ```
- field 사용
    - Projection.fields 사용
    ```
    List<MemberDto> result = queryFactory
                .select(Projections.fields(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();
    ```
- constructor 사용
    - Projections.constructor 사용
    ```
    List<MemberDto> result = queryFactory
                .select(Projections.constructor(MemberDto.class,
                        member.username,
                        member.age))
                .from(member)
                .fetch();
    ```
- Dto 클래스와 엔티티 조회하는 필드이름이 다를 때 as를 사용해서 맞춰줄 수 있다.
- @QueryProjection 사용
    - dto 클래스 생성자에 해당 애노테이션을 붙이고 querydsl 컴파일을 하면 Q파일이 생성된다.
    - 생성된 Q파일을 querydsl 사용하는 곳에서 new 명령어로 사용하면 된다.
    - 컴파일 시점에 오류를 잡아주지만 dto 클래스가 querydsl을 의존하게 되는 단점이 있다.

### 동적 쿼리
- BooleanBuilder 사용
    ```
    BooleanBuilder builder = new BooleanBuilder();
        if (usernameParam != null) {
            builder.and(member.username.eq(usernameParam));
        }
        if (ageParam != null) {
            builder.and(member.age.eq(ageParam));
        }

        return queryFactory
                .selectFrom(member)
                .where(builder)
                .fetch();
    ```
- where절 사용
    ```
    private List<Member> searchMember2(String usernameParam, Integer ageParam) {
        return queryFactory
                .selectFrom(member)
                .where(usernameEq(usernameParam), ageEq(ageParam))
                .fetch();
    }

    private Predicate ageEq(Integer ageParam) {
        return ageParam != null ? member.age.eq(ageParam) : null;
    }

    private Predicate usernameEq(String usernameParam) {
        return usernameParam != null ? member.username.eq(usernameParam) : null;
    }
    ```
    - 메소드를 조합해서 사용할 수 있다.
    ```
    private Predicate allEq(String usernameCond, Integer ageCond) {
        return usernameEq(usernameCond).and(ageEq(ageCond));
    }
    ```
    - 다른 쿼리에서도 재활용 할 수 있다.
    - 쿼리의 가독성이 높아진다.

### 벌크 연산
- 벌크 연산을 수행하면 영속성 컨텍스트와 DB의 상태가 일치하지 않게 된다.
- em.flush()와em.clear()를 사용해서 초기화를 시켜준다.
```
long count = queryFactory
                .update(member)
                .set(member.username, "비회원")
                .where(member.age.lt(20))
                .execute();
```
