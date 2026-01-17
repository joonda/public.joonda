---
title: Spring Security 흐름 알아보기
description: spring-security-flow
tags:
  - Java
date: 2025-09-02
draft: false
---

## Spring Security 흐름
### 1. User enters credentials
사용자가 로그인 폼에 정보와 함께 인증 요청

### 2. Spring Security Filters
모든 인증/인가 요청은 **Filter Chain**을 통과 <br />
`AbstractAuthenticationProcessingFilter`를 구현한 `UsernamePasswordAuthenticationFilter`를 활용.
#### Role
- 인증 요청 감지 `/login POST`
- `UsernamePasswordAuthenticationToken` 인증용 객체 생성, `ProviderManager`에 전달
  - 요청으로 받은 `username`, `password`를 담음.
- `AuthenticationManager` 호출 -> 실제 인증 로직 위임

```java
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		username = (username != null) ? username.trim() : "";
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
		UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
				password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

### 3. AuthenticationManager
인증 요청을 실제 인증 로직으로 위임한다. <br />
`AuthenticationManager`를 구현한 `ProviderManager`로 위임 <br />

#### Role
- `UsernamePasswordAuthenticationToken`을 입력 받는다.
- 등록된 `AuthenticationProvider`에게 인증 요청 전달
- 인증 성공 시, `Authentication` 객체 반환, 실패 시 `AuthenticationException`발생

### 4. AuthenticationProvider
실제 인증 로직을 수행 <br />
`AuthenticationProvider` 인터페이스를 실제로 구현한다.

#### Role
- `AuthenticationManager`로부터 인증 요청을 받음
- `UserDetailsService`로 사용자 정보 조회
- `PasswordEncoder`로 비밀번호 검증
- 인증 성공 시, `Authentication` 객체 반환

```java title="SomeAuthenticationProvider"
@Component
@RequiredArgsConstructor
public class SomeAuthenticationProvider implements AuthenticationProvider {

  private final SomeUserDetailsService userDetailsService;
  private final PasswordEncoder passwordEncoder;

  @Override
  public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    String username = authentication.getName();
    String password = authentication.getCredentials().toString();
    userDetailsService.loadUserByUsername(username);
    UserDetails userDetails = userDetailsService.loadUserByUsername(username);

    if (passwordEncoder.matches(password, userDetails.getPassword())) {
      return new UsernamePasswordAuthenticationToken(username, password, userDetails.getAuthorities());
    } else {
      throw new BadCredentialsException("Invalid password!");
    }
  }

  @Override
  public boolean supports(Class<?> authentication) {
    return (UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication));
  }
}
```

### 5. UserDetailsService / UserDetailsManager
`SomeAuthenticationProvider` 에서 사용하기 위한 사용자 정보를 조회하는 서비스

#### Role
- `AuthenticationProvider`가 `username`을 전달
- DB 또는 저장소에서 사용자 정보조회
- UserDetails 객체 반환

```java title="SomeUserDetailsService"
@Service
@RequiredArgsConstructor
public class SomeUserDetailsService implements UserDetailsService {

  private final CustomerRepository customerRepository;

  @Override
  public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    Customer customer = customerRepository.findByEmail(username).orElseThrow(() -> new
      UsernameNotFoundException("User not found: " + username));

    List<GrantedAuthority> authorities = customer.getAuthorities().stream().map(authority -> new
      SimpleGrantedAuthority(authority.getName())).collect(Collectors.toList());
    return new User(customer.getEmail(), customer.getPwd(), authorities);
  }
}
```

### 6. PasswordEncoder
비밀번호 암호화 및 검증 수행 <br />
`PasswordEncoder`를 구현한 구현체를 주로 사용한다.
- `BCryptPasswordEncoder`
- `Pbkdf2PasswordEncoder`
- 그 외 등등등..
`password`를 `encode`하거나, 저장된 `password`와 비교하여 일치 여부를 확인한다.

### 7. Authentication Token
인증 요청 및 인증 결과를 담는 객체 <br />
`UsernamePasswordAuthenticationToken` (폼 로그인 기본) or `JwtAuthenticationToken` (JWT 기반 인증)
- 인증 요청 시 -> `UsernameAuthenticationToken` (`isAuthenticated` = `false`)
- 인증 성공 후 -> `Authentication` 객체 (`isAuthenticated` = `true`)

### 8. AuthenticationManager 결과 반환
- 인증 성공 시 -> `Authentication` 객체 반환
- 인증 실패 시 -> `AuthenticationException` 발생

### 9. Security Context
현재 인증된 사용자 정보 저장
- `SecurityContext`
- `SecurityContextHolder`
인증 성공 시, `SecurityContextHolder`에 `Authentication` 저장

### 10. Filter Chain 종료 & 응답 반환
인증 / 인가 결과에 따라 Controller 호출 또는 예외 처리
- `ExceptionTranslationFilter`
- `AccessDeniedHandler`
- `AuthenticationEntryPoint`

## Reference
- [Udemy Lecture](https://www.udemy.com/course/spring-security-6-jwt-oauth2-korean/?couponCode=KEEPLEARNING)
- [Spring Security의 흐름과 개념 설명](https://koseonje1222.tistory.com/12#1.%20HTTP%20Request%20%EC%88%98%EC%8B%A0-1-4)