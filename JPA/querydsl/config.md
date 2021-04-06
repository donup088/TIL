### Querydsl 사용
- build.gradle
    ```
    plugins {
        ...
        id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    }
    ...
    dependencies {
        ...
        implementation 'com.querydsl:querydsl-jpa'
    }
    ...
    def querydslDir = "/src/main/generated"
    querydsl {
        jpa = true
        querydslSourcesDir = querydslDir
    }
    sourceSets {
        main.java.srcDir querydslDir
    }
    configurations {
        querydsl.extendsFrom compileClasspath
    }
    compileQuerydsl {
        options.annotationProcessorPath = configurations.querydsl
    }
    ```

### JpaQueryFactory 주입
- 프로젝트에서 JpaQueryFactory를 주입받아 사용할 수 있도록 설정
```
@Configuration
public class QuerydslConfiguration {

    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
```

### QueryRepository 따로 생성해서 사용
- Querydsl Repository를 따로 만들어서 JPAQueryFactory를 주입받아서 사용한다.
```
@RequiredArgsConstructor
@Repository
public class RouteQueryRepository {
    private final JPAQueryFactory queryFactory;

    public Page<Route> findAllByCoordinate(Double maxX, Double minX, Double maxY, Double minY, Pageable pageable) {
        QueryResults<Route> results = queryFactory.selectFrom(route)
                .where(route.minX.goe(minX)
                        .and(route.maxX.loe(maxX))
                        .and(route.minY.goe(minY))
                        .and(route.maxY.loe(maxY)))
                .orderBy(route.id.desc())
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();
        List<Route> content = results.getResults();
        long total = results.getTotal();

        return new PageImpl<>(content, pageable, total);
    }
}
```