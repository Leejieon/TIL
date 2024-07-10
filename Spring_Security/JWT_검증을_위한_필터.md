이전에 JWT 생성을 위한 필터를 구현했다. 이제 클라이언트로부터 받은 JWT의 유효성을 검증하는 필터를 작성해보자.

# JWT 검증 필터

클라이언트로부터 받는 모든 후속 요청에 필요한 필터이다.

```java
public class JWTTokenValidatorFilter  extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        String jwt = request.getHeader(SecurityConstants.JWT_HEADER);
        if (null != jwt) {
            try {
                SecretKey key = Keys.hmacShaKeyFor(
                        SecurityConstants.JWT_KEY.getBytes(StandardCharsets.UTF_8));

                Claims claims = Jwts.parser()
                        .verifyWith(key)
                        .build()
                        .parseSignedClaims(jwt)
                        .getPayload();

                String username = String.valueOf(claims.get("username"));
                String authorities = (String) claims.get("authorities");
                Authentication auth = new UsernamePasswordAuthenticationToken(username, null,
                        AuthorityUtils.commaSeparatedStringToAuthorityList(authorities));
                SecurityContextHolder.getContext().setAuthentication(auth);
            } catch (Exception e) {
                throw new BadCredentialsException("Invalid Token received!");
            }

        }
        filterChain.doFilter(request, response);
    }

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) {
        return request.getServletPath().equals("/user");
    }
}
```

해당 필터 또한, 요청 당 한 번만 수행되어야 하는 필터이기 때문에, `OncePerRequestFilter`를 상속받아 사용한다.

1. 먼저, 헤더의 ***“Authorization”*** 필드에서 클라이언트 요청과 함께 전달된 JWT를 저장한다.
2. 해당 JWT가 null이 아니라면, hmacShaKeyFor 메서드를 사용해 **SecretKey**를 생성한다.
3. `Jwts`의 여러 메서드를 통해 서명된 JWT를 파싱해 **JWT의 Claims를 추출**한다.
4. 추출한 Claims를 이용해 **저장된 사용자의 정보**를 받아온다.
5. 받아온 username과 authorities를 `UsernamePasswordAuthenticationToken`클래스를 통해 인증 객체로 변환한다. 이 과정을 통해 Spring Security에게 JWT가 유효하다는 것을 알리게 된다.
6. 이 인증 객체를 **SecurityContextHolder**에 저장한다.
7. `doFilter` 메서드를 통해 다음 필터에 전달한다.

### shouldNotFilter

```java
@Override
protected boolean shouldNotFilter(HttpServletRequest request) {
    return request.getServletPath().equals("/user");
}
```

JWT 생성 필터와는 반대로, 검증 필터에서는 로그인을 제외한 요청에 대해 필터를 실행시켜야 하기 때문에 위와 같이 설정해준다.

## Spring Security 설정 파일에 Filter 추가하기

해당 Filter를 사용하기 위해, 이전에 작업한 Spring Security 설정 파일에 `addFilterBefore` 메서드를 이용해 Filter를 추가하자.

```java
http...
		.addFilterBefore(new JWTTokenValidatorFilter(), BasicAuthenticationFilter.class)
		...
```

Spring Security의 인증 유효성 검사 이전에 실행되어야 하는 필터이기 때문에, `addFilterBefore` 메서드를 사용하는 것이다.