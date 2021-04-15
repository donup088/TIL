### AuditorAware 사용
- 등록자데이터를 자동으로 생성할 수 있다.
```
@Component
public class LoginUserAuditorAware  implements AuditorAware<String> {
    @Override
    public Optional<String> getCurrentAuditor() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (null == authentication || !authentication.isAuthenticated()) {
            return null;
        }
        User user = (User) authentication.getPrincipal();
        return Optional.of(user.getUserId());
    }
}
```
```
@Configuration
@EnableJpaAuditing
public class AuditingConfig {

}
```
- 엔티티에 @CreatedBy 사용으로 등록한 유저 데이터를 생성할 수 있다.

### Soft Delete
- soft delete 란 DB 데이터를 실제로 삭제하지 않고, 삭제 여부를 나타내는 컬럼을 사용하는 방식이다.
- 일반적인 삭제 대신 removed,deleted_at, is_deleted 컬럼을 사용한다. 컬럼의 자료형은 boolean 또는 datatime을 사용한다.
- 삭제되지 않은 데이터의 removed 컬럼값은 NULL 이고 삭제된 데이터는 removed값이 true 이거나 삭제된 날짜를 기록해 놓는다.
- 데이터를 복구하거나 에전기록을 확인하고자 할 때 편리하다.
- 다른 테이블과 join 시에 항상 removed 점검을 해야하므로 불편하고 속도가 느려지는 단점이 있다.
- Jpa 에서는 Soft Delete 를 구현할 때 변경감지나 @SQLDelete를 사용한다.
    - delete 쿼리가 보내지는 것을 update로 바꿔서 사용한다.
    ```
    //Enity class
    @Where(clause = "deleted_at IS NULL")
    @SQLDelete(sql = "UPDATE routes_reviews SET deleted_at=CURRENT_TIMESTAMP WHERE `id`=?")
    ```