# JWT 생성 필터

사용자가 로그인을 시도하고 **성공적으로 완료될 때마다 JWT를 생성**하기 위한 JWT Filter를 구현해보자.

### JWTTokenGeneratorFilter

```java
public class JWTTokenGeneratorFilter extends OncePerRequestFilter {
		@Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (null != authentication) {
            SecretKey key = Keys.hmacShaKeyFor(SecurityConstants.JWT_KEY.getBytes(StandardCharsets.UTF_8));
            String jwt = Jwts.builder().issuer("Eazy Bank").subject("JWT Token")
                    .claim("username", authentication.getName())
                    .claim("authorities", populateAuthorities(authentication.getAuthorities()))
                    .issuedAt(new Date())
                    .expiration(new Date((new Date()).getTime() + 30000000))
                    .signWith(key).compact();
            response.setHeader(SecurityConstants.JWT_HEADER, jwt);
        }

        filterChain.doFilter(request, response);
    }

    @Override
    protected boolean shouldNotFilter(HttpServletRequest request) {
        return !request.getServletPath().equals("/user");
    }

    private String populateAuthorities(Collection<? extends GrantedAuthority> collection) {
        Set<String> authoritiesSet = new HashSet<>();
        for (GrantedAuthority authority : collection) {
            authoritiesSet.add(authority.getAuthority());
        }
        return String.join(",", authoritiesSet);
    }
}
```

해당 필터는 ***각 요청당 한 번만 실행***돼야 하기 때문에 Spring의 `OncePerRequestFilter`를 상속받아 구현한다. 따라서, `OncePerRequestFilter` 내부의 `doFilterInternal` 추상 메서드를 구현한다. 해당 메서드 안에 JWT를 생성하는 로직을 작성하게 된다. 

로직의 단계를 보면 다음과 같다.

1.  *해당 JWT 필터는 로그인이 성공된 후에 적용시킬 필터*이기 때문에, 인증이 성공적으로 완료되어 **SecurityContextHolder**에 저장된 사용자의 정보를 가져와 `Athentication` 객체에 저장한다.
2. 해당 객체가 null이 아니라면, **SecurityConstants** 내의 비밀 값를 기반으로 **SecretKey**를 생성한다. 따라서, 아래와 같은 별도의 `SecurityConstants` 인터페이스를 생성하자.

### **SecurityConstants**

```java
public interface SecurityConstants {
		String JWT_KEY = "jxgEQeXHuPq8VdbyYFNkANdudQ53YUn4";
		String JWT_HEADER = "Authorization";
}
```

- **JWT_KEY** := 비밀 키 값을 가진 JWT Key
    - 개발 단계에서는 위와 같이 직접 명시하지만, 실제 배포 환경에서는 Jenkins와 같은 DevOps 툴에서 환경 변수를 통해 주입하는 것이 안전하고 이상적이다.
1. Key를 생성하고 `Jwts.builder`를 이용해 JWT를 만든다.
    - `issuer` := Issuer는 ***해당 JWT를 발행하는 주체(발행자)***를 나타내며, JWT의 “**iss**” 클레임을 설정한다.
    - `subject` := Subject는 토큰의 주체로, ***해당 토큰이 발행된 사용자***를 나타내며, JWT의 “**sub**” 클레임을 설정한다.
    - `claim` := claim 메서드를 통해 로그인된 사용자의 ID, 권한과 같은 사용자에 관한 정보를 저장할 수 있다.

```java
private String populateAuthorities(Collection<? extends GrantedAuthority> collection) {
    Set<String> authoritiesSet = new HashSet<>();
    for (GrantedAuthority authority : collection) {
        authoritiesSet.add(authority.getAuthority());
    }
    return String.join(",", authoritiesSet);
}
```

해당 필터 내의 populateAuthorities 메서드는 사용자의 모든 권한을 받아와 쉼표(,)로 구분된 문자열을 반환하는 메서드이다.

- `issuedAt` := ***JWT가 발행된 날짜를 설정***할 수 있다.
- `expiration` := ***JWT의 만료 시간을 설정***할 수 있다.
- `signWith` := JWT의 모든 내용에 위에서 생성한 ***비밀키(SecretKey)를 통해 디지털 서명을 진행***할 수 있다.
- `compact` := 해당 메서드를 통해 JWT를 생성한다.
1. 생성된 JWT를 응답 헤더의 “**Authorization**” 필드에 추가한다.
2. `doFilter` 메서드를 통해 다음 필터를 호출한다.

### shouldNotFilter

```java
@Override
protected boolean shouldNotFilter(HttpServletRequest request) {
    return !request.getServletPath().equals("/user");
}
```

`OncePerRequestFilter`의 위 메서드를 구현함으로서, ***해당 조건에 따라 필터를 실행하지 않도록 설정***할 수 있다.

구현 내용은, 해당 JWT 생성 필터는 오직 로그인 과정 중에만 실행되어야 하고, 이후의 요청에서 토큰이 계속해서 생성되는 것을 막기 위한 내용이다. 즉, “/user” 경로에서만 필터가 실행되게 된다.

## Spring Security 설정 파일에 Filter 추가하기

해당 Filter를 사용하기 위해, 이전에 작업한 Spring Security 설정 파일에 `addFilterAfter` 메서드를 이용해 Filter를 추가하자.

```java
http...
		.addFilterAfter(new JWTTokenGeneratorFilter(), BasicAuthenticationFilter.class)
		...
```

로그인이 성공적으로 이루어진 이후, 해당 필터를 실행해야 하기 때문에 `addFilterAfter` 를 사용했다.