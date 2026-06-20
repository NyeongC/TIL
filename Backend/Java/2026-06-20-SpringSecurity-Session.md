# Spring Security - 세션 기반(Form Login) 인증 실습 정리

## 현재 방식

현재 프로젝트는

```java
.formLogin(...)
```

을 사용하므로

```text
Spring Security
+
세션 기반 인증(Session Authentication)
+
폼 로그인(Form Login)
```

구조이다.

JWT 방식이 아니라 서버가 로그인 상태를 관리하는 방식이다.

---

# 필수 구성 요소

## 1. UserDetails

사용자 정보를 Spring Security가 사용할 수 있도록 제공하는 객체

```java
public class User implements UserDetails
```

대표적으로 구현해야 하는 메소드

```java
getUsername()
getPassword()
getAuthorities()
```

시큐리티는 인증 시 UserDetails 객체의 정보를 사용한다.

---

## 2. UserDetailsService

로그인 시 사용자 조회 담당

```java
@Service
@RequiredArgsConstructor
public class UserDetailService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String email) {
        return userRepository.findByEmail(email)
                .orElseThrow(() ->
                        new IllegalArgumentException(email));
    }
}
```

로그인 시 호출된다.

```text
사용자 입력 이메일
↓
loadUserByUsername()
↓
DB 조회
↓
UserDetails 반환
```

---

## 3. PasswordEncoder

비밀번호 암호화 및 비교

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

회원가입 시

```java
passwordEncoder.encode(password)
```

로그인 시

```java
passwordEncoder.matches(
    입력비밀번호,
    DB암호화비밀번호
)
```

비교한다.

실질적으로 필수이다.

---

# SecurityConfig

## 인증 제외 URL

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers(
        "/login",
        "/signup",
        "/user"
    ).permitAll()
    .anyRequest().authenticated()
)
```

의미

```text
/login
/signup
/user

인증 없이 접근 가능

그 외 모든 URL

로그인 필요
```

---

## 로그인 페이지 지정

```java
.formLogin(formLogin -> formLogin
    .loginPage("/login")
    .defaultSuccessUrl("/articles")
)
```

의미

```text
GET /login

사용자가 만든 로그인 페이지 사용
```

로그인 성공 시

```text
/articles
```

로 이동

---

## 로그아웃

```java
.logout(logout -> logout
    .logoutSuccessUrl("/login")
    .invalidateHttpSession(true)
)
```

의미

```text
로그아웃
↓
세션 무효화
↓
인증정보 삭제
↓
/login 이동
```

---

# 로그인 과정

## 로그인 화면 요청

```text
GET /login
```

컨트롤러가 처리

```java
@GetMapping("/login")
public String login() {
    return "login";
}
```

---

## 로그인 시도

```text
POST /login
```

컨트롤러가 없음

Spring Security가 제공하는

```text
UsernamePasswordAuthenticationFilter
```

가 자동 처리

---

# 로그인 인증 흐름

```text
사용자
↓
POST /login
↓
UsernamePasswordAuthenticationFilter
↓
AuthenticationManager
↓
DaoAuthenticationProvider
↓
UserDetailsService
↓
UserRepository
↓
UserDetails 반환
↓
PasswordEncoder.matches()
↓
인증 성공
↓
Authentication 생성
↓
SecurityContext 저장
↓
HttpSession 저장
↓
JSESSIONID 발급
↓
/articles 이동
```

---

# Authentication

로그인 성공 후 생성되는 객체

```text
Authentication
 ├─ principal
 ├─ authorities
 └─ authenticated
```

예시

```text
Authentication
 ├─ principal = test@test.com
 ├─ authorities = ROLE_USER
 └─ authenticated = true
```

---

# SecurityContext

Authentication 를 저장하는 객체

```text
SecurityContext
 └─ Authentication
```

---

# HttpSession

사용자별 서버 저장 공간

시큐리티는 내부적으로

```java
session.setAttribute(
    "SPRING_SECURITY_CONTEXT",
    securityContext
);
```

와 유사한 작업을 수행한다.

---

# JSESSIONID

브라우저가 보관하는 세션 키

예시

```http
Cookie:
JSESSIONID=A123
```

실제 인증정보는 서버에 있고

브라우저는 세션 번호만 보관한다.

---

# 로그인 이후 요청 흐름

```text
GET /articles
↓
Cookie: JSESSIONID=A123
↓
서버
↓
HttpSession 조회
↓
SecurityContext 조회
↓
Authentication 조회
↓
인증 확인
↓
접근 허용
```

---

# 세션 저장 구조

```text
JSESSIONID=A123
↓
HttpSession
↓
SPRING_SECURITY_CONTEXT
↓
Authentication
↓
UserDetails
```

---

# 핵심 요약

```text
UserDetails
= 사용자 정보

UserDetailsService
= 사용자 조회

PasswordEncoder
= 비밀번호 검증

UsernamePasswordAuthenticationFilter
= 로그인 요청 처리

Authentication
= 인증 결과 객체

SecurityContext
= 인증정보 저장

HttpSession
= 사용자별 서버 저장소

JSESSIONID
= 세션 식별자

현재 프로젝트
= Spring Security + Form Login + Session Authentication
```
