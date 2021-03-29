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