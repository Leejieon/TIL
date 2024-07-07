# JWT dependency 추가

JWT 사용을 위해 build.gradle 파일에 아래 의존성을 추가한다.

```java
implementation 'io.jsonwebtoken:jjwt-api:0.12.3'
runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.12.3'
runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.12.3'
```

# JWT 적용 전 설정

JWT를 사용하기 전, 기존 Spring Security 설정 파일을 살펴보면 다음과 같은 설정이 적용되어 있었다.

```java
http.securityContext((context) -> context.requireExplicitSave(false))
    .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.ALWAYS))
    .cors(corsCustomizer -> corsCustomizer.configurationSource(new CorsConfigurationSource() {
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
    }))...
```

위 설정을 통해, 매번 JSESSIONID를 생성하고 해당 ID를 클라이언트에 전송하도록 했다. 이를 통해, 초기 로그인 후 클라이언트가 요청을 보낼 때마다 동일한 JSESSIONID를 활용할 수 있었다.

# JWT 적용 설정

하지만, 이제 더이상 Spring Security에서 생성된 JSESSIONID를 사용하지 않고 직접 JWT를 생성하고 사용할 것이다. 

```java
http.sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
    .cors(corsCustomizer -> corsCustomizer.configurationSource(new CorsConfigurationSource() {
		    @Override
		    public CorsConfiguration getCorsConfiguration(HttpServletRequest request) {
		        CorsConfiguration config = new CorsConfiguration();
		        config.setAllowedOrigins(Collections.singletonList("http://localhost:4200"));
		        config.setAllowedMethods(Collections.singletonList("*"));
		        config.setAllowCredentials(true);
		        config.setAllowedHeaders(Collections.singletonList("*"));
		        config.setExposedHeaders(Arrays.asList("Authorization"));
		        config.setMaxAge(3600L);
		        return config;
		    }
    }))...
```

위 `STATELESS` 설정으로 **Spring Security의 JSESSIONID나 HTTP Session 생성을 막는다**.

다음으로, 추가적인 CORS 설정이 필요하다. 

JWT를 생성하면 해당 JWT를 클라이언트로 전송해야 한다. 이때 응답 헤더(Response Header)의 ***“Authorization”*** 필드를 이용한다. 따라서, 이에 대한 CORS 설정이 추가적으로 필요하다.

### setExposedHeaders & Authorization

**Exposed Headers**는

> 클라이언트가 CORS 요청의 응답 헤더에 접근할 수 있도록 허용된 헤더 목록을 설정한다.
> 

이때, 인증 토큰을 전달할 때 사용되는 `Authorization` 헤더를 통해 토큰을 전달한다.