# 요청 흐름 전체 정리 (Client ~ Spring Boot 내부)

## 📌 전체 구조

```text
Client
  ↓
Nginx (웹서버 / 로드밸런서)
  ↓
Spring Boot (내장 Tomcat)
  ↓
DispatcherServlet
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
DB
```

---

## 🔥 1️⃣ 요청 흐름 (외부 → 내부)

### 1. 사용자가 요청
```text
GET /user
```

---

### 2. Nginx (웹서버)

- 요청 최초 수신
- HTTPS 처리 (SSL)
- 정적 파일이면 바로 응답
- 아니면 Spring 서버로 전달

```text
Client → Nginx → Spring Boot
```

---

### 3. Tomcat (WAS)

- HTTP 요청 수신
- Servlet으로 전달

👉 스프링으로 요청 넘기는 역할

---

### 4. DispatcherServlet (스프링 입구)

- 모든 요청의 시작점
- 어떤 Controller로 보낼지 결정

---

### 5. Controller 실행

```java
@GetMapping("/user")
public String getUser() {
    return "user";
}
```

---

### 6. Service / Repository

- 비즈니스 로직 처리
- DB 접근

---

### 7. 응답 생성

```text
Controller → DispatcherServlet → Tomcat → Nginx → Client
```

---

## 🔁 2️⃣ 응답 흐름

```text
DB
 ↓
Repository
 ↓
Service
 ↓
Controller
 ↓
DispatcherServlet
 ↓
Tomcat
 ↓
Nginx
 ↓
Client
```

---

## 🔥 3️⃣ Spring Boot 실행 흐름

```bash
java -jar app.jar
```

---

### 내부 동작

```text
1. Spring Boot 실행
2. 내장 Tomcat 생성
3. DispatcherServlet 등록
4. Controller, Bean 생성 (IoC)
5. 서버 포트 열림 (예: 8080)
6. 요청 대기 상태 진입
```

---

## 💡 핵심 포인트

### ✔️ 역할 분리

| 구성 | 역할 |
|------|------|
| Nginx | 요청 수신, SSL, 로드밸런싱 |
| Tomcat | HTTP 요청 처리 (WAS) |
| DispatcherServlet | 요청 분배 |
| Controller | 요청 처리 |
| Service | 비즈니스 로직 |
| Repository | DB 접근 |

---

### ✔️ 핵심 흐름 요약

```text
Client → Nginx → Tomcat → DispatcherServlet → Controller
```

---

## 한 줄 정리

> 사용자의 요청은 Nginx에서 최초로 수신되어 Spring Boot 서버로 전달되고, Tomcat이 이를 받아 DispatcherServlet으로 전달합니다. 이후 DispatcherServlet이 적절한 Controller로 요청을 분배하여 비즈니스 로직을 수행하고 응답을 반환합니다.

---

### 최종 흐름
```text
1. 사용자가 naver.com 입력
2. 브라우저가 DNS 서버에 IP 요청
3. DNS → 퍼블릭 IP 반환
4. 브라우저가 해당 IP로 HTTP 요청
5. 요청이 Nginx(웹서버)에 도착
6. Nginx가 요청 판단
   - 정적 파일 → 바로 응답
   - API 요청 → Spring 서버로 전달
7. Tomcat이 요청 수신
8. DispatcherServlet 호출
9. Controller 실행
10. Service → DB 조회/저장
11. 결과 반환
12. 응답이 다시 사용자에게 전달
```