# Spring 3계층 아키텍처 (3-Tier Architecture)

## 한줄 정의
👉 Controller / Service / Repository로 역할을 나눠 책임을 분리하는 구조

---

## 구조

```
Controller → Service → Repository → DB
```

---

## 계층 역할

### 1. Controller
- HTTP 요청/응답 처리
- DTO 변환
👉 입구 역할 (점원)

---

### 2. Service
- 비즈니스 로직 처리
- 트랜잭션 관리
👉 핵심 계층(매니저)

---

### 3. Repository
- DB 접근
- CRUD 처리
👉 데이터 전담(창고지기)

---

## 왜 이렇게까지 할까?

👉 처음엔 한 파일에 다 짜는 게 편해보임
👉 근데 이 구조는 미래를 위한 보험

---

### 1. 전문성 (SRP)

- 통짜 코드 → 어디 수정해야 할지 모름 (스파게티 코드)

- 3계층 구조
  - 화면 문제 → Controller
  - 로직 문제 → Service
  - DB 문제 → Repository

👉 버그 찾는 속도 빨라짐

---

### 2. 재사용성

- 웹 + 앱 동시에 필요할 때

- 통짜 코드 → 복붙 지옥
- 3계층 → Service 재사용

👉 로직은 한 곳에서만 관리 

WebController에서 쓰던 JoinService를 그대로   
AppController에서 재사용 가능

---

### 3. 안전성

- Repository 직행 → 바로 DB 변경 (위험)

- Service 경유
```java
if (!user.isAdmin()) {
    throw new IllegalArgumentException("관리자만 가능");
}
```

👉 Service = 방화벽 역할

---

## ❌ 하지 말아야 할 것

### 1. 계층 역류

```java
// ❌ Repository → Service 호출
```

👉 구조 깨짐 + 순환참조 위험

---

### 2. 계층 스키핑

```java
// ❌ Controller → Repository 직접 호출
```

👉 로직 분산 + 유지보수 어려움

---

### 3. Controller에 로직 작성 ❌

👉 요청/응답만 처리해야 함

---

### 4. Repository에 비즈니스 로직 작성 ❌

👉 DB 접근만 해야 함

---

### 5. God Service ❌

👉 Service 하나에 다 때려넣기 금지

---

## 흐름 원칙

```
Controller → Service → Repository
```

👉 무조건 위 → 아래 방향

---

## 실무 감각

### 좋은 구조
- Service에 비즈니스 로직 집중
- 공통 로직 관리 (캐시, retry 등)

---

### 나쁜 구조
- Controller에서 외부 API 호출
- Repository에서 조건 처리

---

## 한줄 정리

👉 Controller = 입구
👉 Service = 핵심
👉 Repository = DB

👉 "계층 건너뛰지 말고, 역류하지 말 것"

👉 이 규칙 하나가 유지보수 난이도를 결정한다

