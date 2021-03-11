### Cors 문제 해결
- 와일드카드로 사용하기 위해서는   config.setAllowCredentials(false); 로 설정을 해야한다.
-   config.setAllowCredentials(true); 로 설정하였다면 명시적으로 url을 적어줘야한다.
```
@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.addAllowedOrigin("http://localhost:3000");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");

        source.registerCorsConfiguration("/api/**", config);
        return new CorsFilter(source);
    }
}
```