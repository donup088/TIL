### Q타입 사용
1. QMember m = new QMember("m")
2. QMember m= QMember.member (static import 가능)
3. 쿼리문안에서 바로 static import 사용해서 사용
```
Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"))
                .fetchOne();
```
- 3번째 방법을 사용하고 같은 테이블을 조인할 때만 1,2 번을 사용하자.

### 검색 조건
- where안에 ,로 이어 가면 and로 처리된다.
    ```
    Member findMember = queryFactory.selectFrom(member)
                    .where(member.username.eq("member1"), member.age.eq(10))
                    .fetchOne();       
    ```
- 다양한 조건
    ```
    member.username.eq("member1") // username = 'member1'
    member.username.ne("member1") //username != 'member1'
    member.username.eq("member1").not() // username != 'member1'
    member.username.isNotNull() //이름이 is not null
    member.age.in(10, 20) // age in (10,20)
    member.age.notIn(10, 20) // age not in (10, 20)
    member.age.between(10,30) //between 10, 30
    member.age.goe(30) // age >= 30
    member.age.gt(30) // age > 30
    member.age.loe(30) // age <= 30
    member.age.lt(30) // age < 30
    member.username.like("member%") //like 검색
    member.username.contains("member") // like ‘%member%’ 검색
    member.username.startsWith("member") //like ‘member%’ 검색
    ```

### 결과 조회
- fetch(): 리스트조회
- fetchOne(): 단 건 조회
- fetchResults(): 페이징 정보 포함, total count 쿼리 추가 실행
- fetchCount(): count 쿼리로 변경해서 count 수 조회

### 정렬
- desc(): 내림차순
- asc(): 오름차순
- nullsLast(): null은 마지막
```
List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(100))
                .orderBy(member.age.desc(), member.username.asc().nullsLast())
                .fetch();
```

### 페이징
- 성능상 count 쿼리를 분리할 때도 있다.
```
QueryResults<Member> result = queryFactory.selectFrom(member)
                .orderBy(member.username.desc())
                .offset(1)
                .limit(2)
                .fetchResults();
```

### 집합
- count,sum,avg,max,min 모두 사용가능
- group by 사용가능
    ```
    List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team) //member의 team과 team을 조인
                .groupBy(team.name) //having도 사용가능
                .fetch();
    ```
- 여러개를 조회할 때는 결과가 tuple로 나온다.
- 결과를 tuple로 받기보단 dto로 받는 것이 좋다.


### 조인
- join을 하면 기본적으로 innerJoin이 된다.
- join을 leftJoin 으로 바꿀 수도 있다.
```
List<Member> result = queryFactory
                .selectFrom(member)
                .join(member.team, team)
                .where(team.name.eq("teamA"))
                .fetch();
```
- 세타 조인
    - from 절에 여러 엔티티를 선택해서 세타 조인
    - 외부 조인을 하기 위해서는 on을 사용해야한다.
- on절 사용
    - 조인 대상 필터링
    - 연관관계 없는 엔티티 외부 조인
        - 일반 조인: leftJoin(member.team,team)
        - on조인: from(member).leftJoin(team).on(...)
    - 내부조인을 사용하면 where 절을 사용하고 외부조인이 필요한 경우에만 사용하자.
    ```
    List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(member.team, team).on(team.name.eq("teamA"))
                .fetch();
    ```
    cf) join vs leftjoin
    - join: 조인하는 대상이 null이라면 데이터를 가져오지 않는다.
    - leftjoin 조인하는 대상이 null이라도 데이터를가져온다.
- 패치 조인
    ```
    Member findMember = queryFactory
                    .selectFrom(member)
                    .join(member.team, team).fetchJoin()
                    .where(member.username.eq("member1"))
                    .fetchOne();
    ```

### 서브 쿼리
- 서브 쿼리에서 사용할 별칭을 하나 더 만든다.
- JPAExpressions를 사용한다.
```
QMember memberSub = new QMember("memberSub");

List<Member> result = queryFactory
        .selectFrom(member)
        .where(member.age.eq(
                JPAExpressions
                        .select(memberSub.age.max())
                        .from(memberSub)
        ))
        .fetch();
```
- from 절의 서브쿼리는 사용하지 못한다.
    - join으로 변경한다.
    - 쿼리를 2번 분리해서 실행한다.
    - nativeSQL을 사용한다.