# OAuth2 로그인 + JWT 인증 흐름 정리

## 1. 오늘 구현한 구조 한 줄 요약

Google OAuth2 로그인은 Spring Security가 대부분 자동으로 처리하고,  
나는 중간 지점에 다음 두 가지를 직접 연결했다.

1. Google 사용자 정보를 받은 뒤 DB에 저장하는 `CustomOAuth2UserService`
2. 로그인 성공 후 JWT를 발급하는 `OAuth2SuccessHandler`

그리고 로그인 이후 API 요청은 매번 `JwtAuthenticationFilter`를 거쳐서 JWT를 검증한다.

---

## 2. 전체 흐름

```text
[최초 로그인]
사용자
  ↓
프론트에서 /oauth2/authorization/google 요청
  ↓
Spring Security가 Google 로그인 페이지로 redirect
  ↓
Google 로그인 성공
  ↓
Google이 우리 백엔드 콜백 URL로 redirect
/login/oauth2/code/google?code=xxx&state=yyy
  ↓
Spring Security가 code로 Google에 토큰 요청
  ↓
Spring Security가 Google UserInfo API 호출
  ↓
CustomOAuth2UserService.loadUser() 실행
  ↓
DB에 사용자 저장 또는 업데이트
  ↓
OAuth2SuccessHandler.onAuthenticationSuccess() 실행
  ↓
accessToken, refreshToken 발급
  ↓
refreshToken DB 저장
  ↓
프론트로 token을 붙여 redirect
```

```text
[이후 API 요청]
프론트
  ↓
Authorization: Bearer {accessToken}
  ↓
JwtAuthenticationFilter
  ↓
JWT 검증
  ↓
SecurityContextHolder에 인증 정보 저장
  ↓
/api/** 컨트롤러 실행 가능
```

---

## 3. `/oauth2/authorization/google`은 내가 만든 API가 아니다

사용자가 Google 로그인을 시작할 때 프론트는 다음 URL로 이동시킨다.

```text
/oauth2/authorization/google
```

이 URL은 컨트롤러에 `@GetMapping`으로 직접 만든 API가 아니다.  
Spring Security OAuth2 Client가 기본으로 제공하는 로그인 시작 URL이다.

내 코드에서는 `SecurityConfig`에서 다음 설정을 했기 때문에 Spring Security가 OAuth2 로그인 흐름을 처리한다.

```java
.oauth2Login(oauth2 -> oauth2
        .userInfoEndpoint(userInfo ->
                userInfo.userService(customOAuth2UserService)
        )
        .successHandler(oAuth2SuccessHandler)
)
```

즉, 이 설정이 있기 때문에:

```text
/oauth2/authorization/google
```

로 접근하면 Spring Security가 자동으로 Google 로그인 페이지로 보낸다.

---

## 4. `/login/oauth2/code/google`도 내가 만든 API가 아니다

Google Cloud Console에 등록한 redirect URI는 다음과 같다.

```text
https://edgecut.xyz/login/oauth2/code/google
```

이 URL도 내가 직접 만든 컨트롤러가 아니다.

Spring Security OAuth2가 내부적으로 처리하는 기본 콜백 URL이다.

기본 규칙은 다음과 같다.

```text
/login/oauth2/code/{registrationId}
```

Google 설정의 registrationId가 `google`이므로 최종 URL은 다음이 된다.

```text
/login/oauth2/code/google
```

Google 로그인에 성공하면 Google은 사용자 정보를 바로 넘기는 것이 아니라, 보통 다음처럼 authorization code를 넘긴다.

```text
/login/oauth2/code/google?code=xxx&state=yyy
```

그 다음 Spring Security가 이 `code`를 사용해서 Google에 access token을 요청하고, 다시 Google UserInfo API를 호출해서 사용자 정보를 가져온다.

---

## 5. `CustomOAuth2UserService`는 언제 실행되는가?

Google 로그인 성공 후, Spring Security가 Google 사용자 정보를 가져오는 과정에서 실행된다.

```java
@Service
public class CustomOAuth2UserService extends DefaultOAuth2UserService {

    private final UserRepository userRepository;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2User oauth2User = super.loadUser(userRequest);

        saveOrUpdate(oauth2User);

        return oauth2User;
    }
}
```

여기서 중요한 부분은 다음 코드다.

```java
OAuth2User oauth2User = super.loadUser(userRequest);
```

이 부분이 Google에서 실제 사용자 정보, 예를 들면 `email`, `name` 같은 값을 가져오는 역할을 한다.

그 후 내가 만든 `saveOrUpdate()`에서 DB에 유저를 저장하거나 업데이트한다.

```java
private User saveOrUpdate(OAuth2User oauth2User) {
    Map<String, Object> attributes = oauth2User.getAttributes();

    String email = (String) attributes.get("email");
    String name = (String) attributes.get("name");

    User user = userRepository.findByEmail(email)
            .map(entity -> entity.update(name))
            .orElse(User.builder()
                    .email(email)
                    .nickname(name)
                    .build());

    return userRepository.save(user);
}
```

정리하면:

```text
Google 로그인 성공
  ↓
Spring Security가 Google 사용자 정보 조회
  ↓
CustomOAuth2UserService.loadUser()
  ↓
DB에 사용자 저장/업데이트
```

---

## 6. 로그인 성공 후 JWT 발급 핸들러는 왜 실행되는가?

이유는 `SecurityConfig`에 성공 핸들러로 등록했기 때문이다.

```java
.oauth2Login(oauth2 -> oauth2
        .userInfoEndpoint(userInfo ->
                userInfo.userService(customOAuth2UserService)
        )
        .successHandler(oAuth2SuccessHandler)
)
```

여기서 이 부분이 핵심이다.

```java
.successHandler(oAuth2SuccessHandler)
```

이 설정 때문에 OAuth2 로그인이 성공하면 Spring Security가 기본 성공 처리 대신 내가 만든 `OAuth2SuccessHandler`를 실행한다.

---

## 7. `OAuth2SuccessHandler`가 하는 일

OAuth2 로그인이 성공하면 다음 메서드가 실행된다.

```java
@Override
public void onAuthenticationSuccess(
        HttpServletRequest request,
        HttpServletResponse response,
        Authentication authentication
) throws IOException {

    OAuth2User oAuth2User = (OAuth2User) authentication.getPrincipal();

    String email = (String) oAuth2User.getAttributes().get("email");

    User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new IllegalArgumentException("OAuth user not found"));

    String accessToken = jwtTokenProvider.generateAccessToken(user);
    String refreshToken = jwtTokenProvider.generateRefreshToken(user);

    user.updateRefreshToken(refreshToken);
    userRepository.save(user);

    String targetUrl = UriComponentsBuilder.fromUriString(redirectUri)
            .queryParam("token", accessToken)
            .queryParam("refreshToken", refreshToken)
            .build()
            .toUriString();

    getRedirectStrategy().sendRedirect(request, response, targetUrl);
}
```

이 코드의 역할은 다음과 같다.

```text
1. 로그인 성공한 OAuth2User에서 email 추출
2. email로 DB User 조회
3. accessToken 생성
4. refreshToken 생성
5. refreshToken을 DB에 저장
6. 프론트엔드 URL로 redirect
7. URL에 token, refreshToken을 붙여 전달
```

즉, JWT는 Google이 발급하는 것이 아니라, Google 로그인 성공 후 우리 백엔드가 직접 발급한다.

---

## 8. JWT는 어떻게 만들어지는가?

`JwtTokenProvider`에서 accessToken과 refreshToken을 만든다.

```java
public String generateAccessToken(User user) {
    return generateToken(user, Duration.ofHours(2));
}

public String generateRefreshToken(User user) {
    return generateToken(user, Duration.ofDays(14));
}
```

현재 구조에서는:

| 토큰 | 만료 시간 | 용도 |
|---|---:|---|
| accessToken | 2시간 | API 요청마다 사용 |
| refreshToken | 14일 | accessToken 재발급용 |

JWT 내부에는 다음 정보가 들어간다.

```java
return Jwts.builder()
        .setHeaderParam(Header.TYPE, Header.JWT_TYPE)
        .setIssuer(jwtProperties.getIssuer())
        .setIssuedAt(now)
        .setExpiration(expiry)
        .setSubject(user.getEmail())
        .claim("id", user.getId())
        .claim("nickname", user.getNickname())
        .signWith(getSigningKey(), SignatureAlgorithm.HS256)
        .compact();
```

정리하면 JWT 안에는 다음 값들이 들어간다.

```text
issuer      : 발급자
issuedAt    : 발급 시간
expiration  : 만료 시간
subject     : 사용자 email
id          : 사용자 DB id
nickname    : 사용자 nickname
signature   : secretKey 기반 서명
```

---

## 9. 이후 모든 요청마다 JWT 필터를 거치는가?

결론부터 말하면, 백엔드로 들어오는 대부분의 요청은 `JwtAuthenticationFilter`를 거친다.

이유는 `SecurityConfig`에서 다음처럼 필터를 Security Filter Chain에 등록했기 때문이다.

```java
.addFilterBefore(
        jwtAuthenticationFilter,
        UsernamePasswordAuthenticationFilter.class
)
```

이 설정은 내가 만든 `JwtAuthenticationFilter`를 Spring Security의 인증 필터 앞에 추가한다는 뜻이다.

즉, 요청이 들어오면 다음 흐름을 탄다.

```text
HTTP 요청
  ↓
Security Filter Chain
  ↓
JwtAuthenticationFilter
  ↓
나머지 Spring Security 필터
  ↓
Controller
```

---

## 10. JWT 필터는 매번 JWT를 검증하는가?

Authorization 헤더에 Bearer 토큰이 있으면 매번 검증한다.

```java
String token = resolveToken(request);

if (token != null && jwtTokenProvider.validToken(token)) {
    String email = jwtTokenProvider.getEmail(token);

    User user = userRepository.findByEmail(email).orElse(null);

    if (user != null) {
        UsernamePasswordAuthenticationToken authentication =
                new UsernamePasswordAuthenticationToken(
                        user,
                        null,
                        List.of(new SimpleGrantedAuthority("ROLE_USER"))
                );

        authentication.setDetails(
                new WebAuthenticationDetailsSource().buildDetails(request)
        );

        SecurityContextHolder.getContext().setAuthentication(authentication);
    }
}

filterChain.doFilter(request, response);
```

여기서 핵심은 다음이다.

```java
if (token != null && jwtTokenProvider.validToken(token))
```

토큰이 있고, 유효하면 인증 정보를 만든다.

토큰이 없거나 유효하지 않으면 인증 정보를 만들지 않는다.

하지만 필터는 마지막에 항상 다음 필터로 넘긴다.

```java
filterChain.doFilter(request, response);
```

즉, JWT 필터가 직접 401을 내는 구조가 아니라, 인증 정보를 넣을 수 있으면 넣고, 아니면 그냥 통과시킨다.

그 다음에 Spring Security의 인가 설정에서 `/api/**`는 인증이 없으면 막힌다.

---

## 11. JWT가 있어야만 실행되는 것은 어느 쪽인가?

JWT가 있어야 하는지는 `SecurityConfig`의 인가 설정에서 결정된다.

```java
.authorizeHttpRequests(auth -> auth
        .requestMatchers(
                "/",
                "/login",
                "/oauth2/**",
                "/login/oauth2/**",
                "/api/auth/refresh"
        ).permitAll()

        .requestMatchers("/api/**").authenticated()

        .anyRequest().permitAll()
)
```

여기서 핵심은 이 부분이다.

```java
.requestMatchers("/api/**").authenticated()
```

즉:

```text
/api/** 요청은 인증된 사용자만 가능하다.
```

그리고 JWT 기반 구조에서 “인증된 사용자”가 되려면 `JwtAuthenticationFilter`가 JWT를 검증해서 `SecurityContextHolder`에 인증 정보를 넣어줘야 한다.

그래서 흐름은 다음과 같다.

```text
/api/images/upload 요청
  ↓
JwtAuthenticationFilter 실행
  ↓
Authorization 헤더 확인
  ↓
JWT 유효성 검증
  ↓
유효하면 SecurityContextHolder에 Authentication 저장
  ↓
/api/** authenticated 조건 통과
  ↓
컨트롤러 실행
```

반대로 JWT가 없으면:

```text
/api/images/upload 요청
  ↓
JwtAuthenticationFilter 실행
  ↓
Authorization 헤더 없음
  ↓
SecurityContextHolder 비어 있음
  ↓
/api/** authenticated 조건 실패
  ↓
401 Unauthorized
```

---

## 12. permitAll URL은 JWT가 없어도 된다

다음 URL들은 JWT가 없어도 접근 가능하다.

```java
.requestMatchers(
        "/",
        "/login",
        "/oauth2/**",
        "/login/oauth2/**",
        "/api/auth/refresh"
).permitAll()
```

각각의 의미는 다음과 같다.

| URL | 의미 |
|---|---|
| `/` | 기본 페이지 |
| `/login` | 로그인 관련 페이지 |
| `/oauth2/**` | OAuth2 로그인 시작 URL |
| `/login/oauth2/**` | OAuth2 콜백 URL |
| `/api/auth/refresh` | accessToken 재발급 API |

특히 `/api/auth/refresh`는 accessToken이 만료됐을 때 호출해야 하므로 accessToken 없이 접근 가능해야 한다.

---

## 13. accessToken 만료 시 refreshToken으로 재발급하는 이유

accessToken은 API 요청마다 사용된다.

하지만 accessToken을 너무 오래 유지하면 탈취됐을 때 위험하다.

그래서 accessToken은 짧게 가져가고, refreshToken은 더 길게 가져간다.

현재 구조에서는:

```text
accessToken  : 2시간
refreshToken : 14일
```

accessToken이 만료되면 프론트는 다음 API를 호출한다.

```text
POST /api/auth/refresh
```

이 요청은 `permitAll()`로 열려 있으므로 accessToken 없이 호출 가능하다.

대신 refreshToken 자체를 검증하고, DB에 저장된 refreshToken과 비교해야 한다.

```text
요청 refreshToken == DB refreshToken
```

이 비교를 하는 이유는 서버에서 refreshToken을 무효화할 수 있게 하기 위해서다.

JWT는 기본적으로 한 번 발급되면 만료 전까지 서버가 강제로 취소하기 어렵다.

하지만 refreshToken을 DB에 저장해두면 로그아웃이나 탈취 의심 상황에서 DB 값을 삭제하거나 변경해서 재발급을 막을 수 있다.

---

## 14. 내가 이해한 핵심 정리

### 질문 1. 성공하면 JWT 발행하는 핸들러는 Config에서 설정되어 있어서 실행되는가?

맞다.

`SecurityConfig`에서 다음처럼 등록했기 때문에 OAuth2 로그인 성공 후 실행된다.

```java
.successHandler(oAuth2SuccessHandler)
```

그래서 로그인 성공 시 `OAuth2SuccessHandler.onAuthenticationSuccess()`가 실행되고, 여기서 JWT를 발급한다.

---

### 질문 2. 이후 요청마다 필터는 매번 JWT를 검증하는가?

맞다.

백엔드 요청이 들어오면 Security Filter Chain을 지나고, 그 안에 등록된 `JwtAuthenticationFilter`가 실행된다.

Authorization 헤더에 Bearer 토큰이 있으면 매번 JWT 유효성을 검증한다.

```java
if (token != null && jwtTokenProvider.validToken(token)) {
    ...
}
```

---

### 질문 3. 앞으로 백엔드에 들어오는 건 전부 필터를 거치는가?

대부분 거친다고 보면 된다.

`JwtAuthenticationFilter`는 Spring Security Filter Chain에 등록되어 있으므로, Security가 처리하는 요청은 이 필터를 지난다.

다만 `permitAll()`이라고 해서 필터를 안 타는 것은 아니다.

중요한 차이는 다음이다.

```text
permitAll()
  → 필터는 탈 수 있음
  → 하지만 인증이 없어도 최종 접근 허용

authenticated()
  → 필터를 탐
  → 인증 정보가 없으면 최종 접근 거부
```

---

### 질문 4. JWT가 있어야지만 실행되는 건 어느 쪽인가?

`/api/**` 쪽이다.

```java
.requestMatchers("/api/**").authenticated()
```

이 설정 때문에 `/api/**` 요청은 인증이 필요하다.

그리고 인증은 `JwtAuthenticationFilter`가 JWT를 검증해서 `SecurityContextHolder`에 넣어줘야 인정된다.

---

## 15. 최종 암기용 흐름

```text
/oauth2/authorization/google
  → Spring Security가 Google 로그인으로 보냄

/login/oauth2/code/google
  → Google 로그인 성공 후 돌아오는 콜백 URL
  → 내가 만든 API 아님
  → Spring Security가 처리

CustomOAuth2UserService
  → Google 사용자 정보 가져온 뒤 DB 저장/업데이트

OAuth2SuccessHandler
  → 로그인 성공 후 JWT 발급
  → Config에서 successHandler로 등록했기 때문에 실행됨

JwtAuthenticationFilter
  → 이후 요청마다 Authorization 헤더 확인
  → JWT 유효하면 SecurityContextHolder에 인증 정보 저장

/api/**
  → authenticated 설정 때문에 JWT 인증이 있어야 실행 가능
```

---

## 16. 핵심 결론

오늘 구현한 구조는 세션 로그인 방식이 아니라, OAuth2 로그인 성공 후 JWT를 발급해서 프론트가 보관하고, 이후 API 요청마다 JWT를 보내는 방식이다.

Spring Security OAuth2는 Google 로그인 시작, 콜백 처리, 사용자 정보 조회를 자동으로 처리해준다.

내가 직접 구현한 부분은 다음이다.

```text
1. Google 사용자 정보를 DB에 저장하는 CustomOAuth2UserService
2. 로그인 성공 후 JWT를 발급하는 OAuth2SuccessHandler
3. 요청마다 JWT를 검증하는 JwtAuthenticationFilter
4. /api/** 는 인증 필요하도록 설정한 SecurityConfig
```

