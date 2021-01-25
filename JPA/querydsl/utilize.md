### JPAQueryFactory 사용
- @Bean으로 등록해서 사용
```
private final EntityManager em;
private final JPAQueryFactory queryFactory;

public MemberJpaRepository(EntityManager em,JPAQueryFactory queryFactory) {
    this.em = em;
    this.queryFactory = queryFactory;
}
```
- 생성자에서 new 사용
```
private final EntityManager em;
private final JPAQueryFactory queryFactory;
public MemberJpaRepository(EntityManager em) {
    this.em = em;
    this.queryFactory = new JPAQueryFactory(em);
}
```

### 사용자 정의 repository
1. 사용자 정의 인터페이스 작성
2. 사용자 정의 인터페이스 구현
3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속


### 페이징 처리
- count 쿼리와 페이징 쿼리가 같이 나가는 경우
```
QueryResults<MemberTeamDto> result = queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();
List<MemberTeamDto> content = result.getResults();
long total = result.getTotal();

return new PageImpl<>(content, pageable, total);
```
- 페이징쿼리와 count 쿼리를 분리하는 경우
```
List<MemberTeamDto> content = queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

long count = queryFactory
        .selectFrom(member)
        .leftJoin(member.team, team)
        .where(usernameEq(condition.getUsername()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe()))
        .fetchCount();

return new PageImpl<>(content, pageable, count);
```