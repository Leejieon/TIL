# CORS

> Cross Origin Resource Sharing의 약어로, 두 가지의 다른 출처(Domain)가 자신들의 자원을 공유할 때 일어나는 보안 정책
> 
- WEB 내의 보안 위험으로부터의 보호 정책

# CORS 해결 방법

## 1. @CrossOrigin 어노테이션

> Controller layer에서 직접 @CrossOrigin 어노테이션을 사용해서 해결하는 방법
> 
- `@CrossOrigin(origins = “http://localhost:4200”);` ⇒ 특정 도메인에 한하여 허용
- `@CrossOrigin(origins = “*”);` ⇒ 모든 도메인에 대해 허용

⇒ 하지만, 실제 애플리케이션의 경우, Controller로 개수가 매우 많아 일일이 지정하는 것은 현실적으로 권장되지 않는 방법이다.

## 2. @Configuration 설정 파일에서 `http.cors()` 사용

> Spring Security 설정 파일의 메서드 안에서 Securtiy FilterChain의 bean을 생성할 때, CORS 설정을 정의하는 방법
> 

```java
@Configuration
public class ProjectSecurityConfig {
	@Bean
	SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
	    http.cors(corsCustomizer -> corsCustomizer.configurationSource(new CorsConfigurationSource() {
              @Override
              public CorsConfiguration getCorsConfiguration(HttpServletRequest request) {
                  CorsConfiguration config = new CorsConfiguration();
                  config.setAllowedOrigins(Collections.singletonList("http://localhost:4200"));
                  config.setAllowedMethods(Collections.singletonList("*"));
                  config.setAllowCredentials(true);
                  config.setAllowedHeaders(Collections.singletonList("*"));
                  config.setMaxAge(3600L);
                  return config;
              }
            }))
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests((requests) -> requests
                    .requestMatchers("/myAccount", "/myBalance", "/myLoans", "/myCards", "/user").authenticated()
                    .requestMatchers("/notices", "/contact", "/register").permitAll())
            .formLogin(withDefaults())
            .httpBasic(withDefaults());
	    return http.build();
	}
	
	@Bean
	public PasswordEncoder passwordEncoder() {
	    return new BCryptPasswordEncoder();
	}
}

```

- http.cors()에서 configurationSource 메서드에 `CorsConfigurationSource`의 객체를 전달해야 한다. ⇒ 익명 클래스를 선언한다.