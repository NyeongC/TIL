# 🔥 @Transactional 내부 동작 정리

## ✅ 한 줄 정의

> @Transactional은 AOP 기반 프록시를 통해 메서드 실행 전 트랜잭션을 시작하고, 종료 시점에 commit 또는 rollback을 수행하는 방식으로 동작합니다.

---

## 🧠 핵심 개념 흐름

우리가 `@Transactional`을 붙이면 내부에서는 이런 일이 일어난다:

```
Controller → 프록시 객체 → 실제 Service
```

👉 중요한 포인트
- 우리가 사용하는 객체는 실제 객체가 아니라 **프록시 객체**
- 트랜잭션은 이 프록시가 담당

---

## ⚙️ 실제 동작 순서

1. 프록시 객체가 먼저 호출됨
2. 트랜잭션 시작 (Connection 획득, autoCommit=false)
3. 실제 서비스 로직 실행
4. 결과에 따라
   - 성공 → commit
   - 실패 → rollback

---

## 💡 내부 동작 (의사 코드)

```java
try {
    beginTransaction();

    target.method(); // 실제 비즈니스 로직

    commit();
} catch (Exception e) {
    rollback();
    throw e;
}
```

👉 이 과정을 우리가 직접 작성하지 않고, 스프링이 대신 해준다.

---

## 🔥 AOP 기반 구조

실제 스프링은 단순 프록시가 아니라 인터셉터 체인 구조로 동작한다.

```
Controller
  ↓
Proxy
  ↓
TransactionInterceptor
  ↓
실제 Service
```

👉 핵심
- `invocation.proceed()`를 통해 실제 메서드 실행
- 트랜잭션 로직은 `TransactionInterceptor`가 담당

---

## ⚠️ 롤백 규칙 (중요)

### 기본 동작

- RuntimeException → rollback
- Checked Exception → commit

```java
@Transactional
public void test() throws Exception {
    throw new Exception(); // ❌ rollback 안됨
}
```

---

### 해결 방법

```java
@Transactional(rollbackFor = Exception.class)
```

👉 모든 예외에 대해 rollback 처리 가능

---

## ⚠️ 프록시 기반의 한계 (실무 핵심)

### 1. 내부 호출 문제

```java
@Service
public class UserService {

    @Transactional
    public void outer() {
        inner(); // ❌ 트랜잭션 적용 안됨
    }

    @Transactional
    public void inner() {}
}
```

👉 이유
- 프록시를 거치지 않고 직접 호출되기 때문

---

### 2. private 메서드

```java
@Transactional
private void method() {}
```

👉 ❌ 적용 안됨

---

## 🔥 핵심 정리

- @Transactional 붙이면 스프링이 **프록시 객체 생성**
- 이 프록시가 트랜잭션을 대신 관리
- 메서드 실행 전에 트랜잭션 시작
- 실행 후 결과에 따라 commit / rollback

👉 즉

> 프록시가 트랜잭션을 열고 → 실제 로직 실행 → 끝나면 commit/rollback

---

## 🧠 한 줄 결론

> 우리가 @Transactional 스티커를 붙이는 순간, 스프링은 트랜잭션 처리 코드가 포함된 프록시 객체를 자동으로 만들어 주입해준다.

---

## 💥 실무 관점 포인트

- 트랜잭션 범위가 길어지면
  → DB 커넥션 오래 점유
  → 커넥션 풀 고갈 가능

👉 그래서
- 외부 API 호출
- 오래 걸리는 작업

➡️ 트랜잭션 밖으로 분리 고민 필요

---

## 🚀 최종 답변

> “@Transactional은 AOP 기반 프록시를 통해 메서드 실행 전에 트랜잭션을 시작하고, 실제 로직을 실행한 뒤 결과에 따라 commit 또는 rollback을 수행합니다. 또한 프록시 기반이기 때문에 내부 호출에서는 트랜잭션이 적용되지 않습니다.”

