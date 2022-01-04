### querydsl 에서 Result Aggregation 사용하기
- Result Aggregation 이란 특정 키를 기준으로 삼아서 그룹화하는 것을 의미한다.

```
import static com.querydsl.core.group.GroupBy.groupBy;
import static com.querydsl.core.group.GroupBy.list;

public List<Family> findFamily() {
    Map<Parent, List<Child>> transform = queryFactory
        .from(parent)
        .leftJoin(parent.children, child)
        .transform(groupBy(parent).as(list(child)));

    return transform.entrySet().stream()
        .map(entry -> new Family(entry.getKey().getName(), entry.getValue()))
        .collect(Collectors.toList());
    }
```

- transform을 사용하면 groupBy를 기준 Key로 list를 통해 모으고 싶은 대상을 지정할 수 있다.
- 예시
    ```
    @Override
    public List<PostApplicantResponse.ApplicantInfo> findAllByPostAndDoDate(Long postId, ApplyStatus applyStatus, LocalDate doDate) {
        Map<PostDoDate, List<PostApplicantDto.UserInfo>> transform = queryFactory
                .from(postApplicant)
                .join(postApplicant.user, user)
                .join(postApplicant.postDoDate, postDoDate)
                .join(postDoDate.post, post).fetchJoin()
                .where(post.id.eq(postId)
                        .and(postDoDate.doDate.year().eq(doDate.getYear())
                                .and(postDoDate.doDate.month().eq(doDate.getMonthValue()))
                                .and(postDoDate.doDate.dayOfMonth().eq(doDate.getDayOfMonth())))
                        .and(byApplyStatus(applyStatus)))
                .orderBy(postDoDate.doDate.asc(), postApplicant.createdAt.asc())
                .transform(groupBy(postDoDate).as(list(
                                new QPostApplicantDto_UserInfo(
                                        user.id, user.name, user.email, user.birth, user.gender, user.phoneNum, user.job, post.doTime, postApplicant.applyStatus, postApplicant.businessFinish, postDoDate.doDate, postApplicant.evaluatedBusiness
                                )
                        ))
                );
        return transform.entrySet()
                .stream().map(entry ->
                        new PostApplicantResponse.ApplicantInfo(
                                entry.getKey().getId(),
                                entry.getKey().getDoDate(),
                                entry.getKey().getDoDate().plusMinutes(entry.getKey().getPost().getDoTime()),
                                entry.getValue()))
                .collect(Collectors.toList());
    }
    ```
    - transform 결과를 dto로 바꾸는 과정에서 다른 테이블의 값을 사용하여 Lazy Loading이 발생한다.

- 또 다른 방법으로 entity를 직접 조회와 left outer join을 사용하는 방법이 있다.
```
public List<Family> findFamily() {
    List<Parent> parents = queryFactory
        .selectFrom(parent)
        .leftJoin(parent.children, child).fetchJoin()
        .fetch();

    return parents.stream()
        .map(p -> new Family(p.getName(), p.getChildren()))
        .collect(Collectors.toList());
}
```

참고 링크 : https://jojoldu.tistory.com/342