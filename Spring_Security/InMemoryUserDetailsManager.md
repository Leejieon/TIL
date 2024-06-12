@Configuration 클래스에 사용자를 정의할 수 있다.

# 첫번째 방법

```java
@Bean
public InMemoryUserDetailsManager userDetailsService() {
	UserDetails admin = User.withDefaultPasswordEncoder()
	        .username("admin")
	        .password("12345")
	        .authorities("admin")
	        .build();
	
	UserDetails user = User.withDefaultPasswordEncoder()
	        .username("user")
	        .password("12345")
	        .authorities("read")
	        .build();
	
	return new InMemoryUserDetailsManager(admin, user);
}
```

⇒ `User`는 `UserDetails` 를 구현하고 있기 때문에, User를 통해 사용자를 만들어 생성자로 사용자를 추가하는 방식

# 두번째 방법

```java
@Bean
public InMemoryUserDetailsManager userDetailsService() {
  UserDetails admin = User.withUsername("admin")
          .password("12345")
          .authorities("admin")
          .build();

  UserDetails user = User.withUsername("user")
          .password("12345")
          .authorities("read")
          .build();

  return new InMemoryUserDetailsManager(admin, user);
}

/* 
이 방식을 사용하기 위해 반드시 필요한 메서드
PasswordEncoder를 반환하는 메서드를 Bean으로 등록해야 한다.
*/
@Bean
public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
}
```

### `NoOpPasswordEncoder`

> Spring Security 내에서 사용 가능한 가장 간단한 PasswordEncoder
> 
- 비밀번호의 암호화 및 해싱을 수행하지 않는다.
    - 비밀번호 값을 일반 텍스트로 저장 ⇒ ***프로덕션 환경에서는 권장되지 않음***

`User`의 첫번째 메서드는 반드시 `withUsername` 혹은 `withDefaultPasswordEncoder`로 시작해야 한다.