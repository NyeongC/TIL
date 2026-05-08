# Filter / Interceptor / AOP

## 개념

### Filter

HTTP 요청 자체를 가장 앞단에서 가로채는 기능  
(Servlet Container 레벨)

---

### Interceptor

스프링 MVC에서 Controller 호출 전/후를 제어하는 기능

---

### AOP

비즈니스 메서드의 공통 기능을 특정 시점에 끼워넣는 기능(포인트컷)

---

## 전체 흐름
```
클라이언트 요청
    ↓
[ Filter ]          ← Servlet Container 레벨, Spring 이전
    ↓
DispatcherServlet
    ↓
[ Interceptor ]     ← Spring MVC 레벨, Controller 전후
    ↓
Controller
    ↓
[ AOP ]             ← Spring Bean 레벨, 메서드 실행 시점
    ↓
Service
    ↓
Repository
    ↓
응답 반환
```

---

## 언제 사용하는가

### Filter

* 인증 / 인가
* Spring Security
* 인코딩 처리
* 요청 로깅
* 모든 HTTP 요청 공통 처리

---

### Interceptor

* 로그인 체크
* 관리자 권한 확인
* Controller 실행 시간 측정
* Controller 공통 처리

---

### AOP

* 트랜잭션 처리
* 공통 로깅
* 성능 측정
* 예외 처리
* 중복 코드 제거

---

## 차이점

| 구분 | Filter | Interceptor | AOP |
|---|---|---|---|
| 동작 위치 | Servlet Container | Spring MVC | Spring Bean |
| 실행 시점 | 가장 먼저 | Controller 전후 | 메서드 실행 시 |
| 주요 대상 | HTTP 요청 | Controller | Service/Bean |
| 대표 사용 | Security | 로그인 체크 | 트랜잭션 |

---

## Spring Security가 Filter인 이유

가장 앞단에서 인증되지 않은 요청을 빠르게 차단하기 위해 사용한다.

즉,

* Controller까지 가지 않음
* 불필요한 비즈니스 로직 수행 방지
* 보안적으로 안전
* 서버 리소스 절약 가능

---

## 핵심 정리

Filter
: HTTP 요청 자체를 가장 앞단에서 처리

Interceptor
: Controller 호출 흐름 제어

AOP
: 비즈니스 메서드 공통 기능 분리