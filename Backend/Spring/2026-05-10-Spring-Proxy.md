## Spring Proxy(프록시)란?

### 개념

```text
실제 객체를 직접 호출하지 않고,
중간에 대리 객체(Proxy)를 하나 두어
부가 기능을 처리하는 기술
```

스프링은 주로 AOP, 트랜잭션, 보안, 로깅 등을 위해 프록시를 사용한다.

---

### 왜 프록시를 사용할까?

```text
공통 기능을 실제 비즈니스 코드와 분리하기 위해
```

예를 들어:

- 로그 출력
- 트랜잭션 시작/커밋
- 권한 체크
- 시간 측정
- 예외 처리

이런걸 서비스마다 직접 넣으면 중복이 심해짐

---

### 프록시 없이 구현하면?

```java
public void order() {

    System.out.println("트랜잭션 시작");

    try {
        // 실제 비즈니스 로직
        System.out.println("주문 처리");

        System.out.println("커밋");
    } catch (Exception e) {
        System.out.println("롤백");
    }
}
```

서비스마다 반복됨

---

### 프록시 사용 구조

```text
사용자
 ↓
Proxy 객체
 ↓
실제 Service 객체
```

프록시가 중간에서 공통 작업 처리

---

## 스프링에서 대표적인 프록시 사용 예시

### 1. @Transactional

가장 대표적

```java
@Transactional
public void order() {
    // 비즈니스 로직
}
```

실제로는:

```text
Proxy가 메소드 시작 전:
→ 트랜잭션 시작

메소드 정상 종료:
→ commit

예외 발생:
→ rollback
```

---

### 동작 흐름

```text
Controller
 ↓
Transaction Proxy
 ↓
OrderService 실제 객체
 ↓
DB 작업
```

---

### 실제 서비스

```java
@Service
public class OrderService {

    @Transactional
    public void order() {
        System.out.println("주문 처리");
    }
}
```

---

### 내부적으로는 느낌상 이런 구조

```java
public class OrderServiceProxy {

    private OrderService target;

    public void order() {

        System.out.println("트랜잭션 시작");

        try {
            target.order();

            System.out.println("commit");
        } catch (Exception e) {

            System.out.println("rollback");
        }
    }
}
```

---

# 프록시를 통해 얻는 이점

### 1. 공통 기능 분리

비즈니스 로직 집중 가능

```text
핵심 로직만 작성 가능
```

---

### 2. 중복 제거

트랜잭션 코드 반복 제거

---

### 3. 유지보수 쉬움

공통 기능 수정 시 한곳만 수정

---

### 4. 객체지향적

관심사 분리 (AOP)

```text
주문 처리
≠
로그 처리
≠
트랜잭션 처리
```

---

### 스프링에서 프록시로 처리하는 것들

### 대표 사례

| 기능 | 설명 |
|---|---|
| `@Transactional` | 트랜잭션 |
| Spring Security | 권한 체크 |
| `@Async` | 비동기 |
| AOP | 로깅, 시간측정 |
| 캐시 | `@Cacheable` |

---

## JDK 동적 프록시 vs CGLIB

### JDK Dynamic Proxy

```text
인터페이스 기반 프록시
```

```java
public interface OrderService
```

---

### CGLIB

```text
클래스를 상속해서 프록시 생성
```

```java
public class OrderService
```

스프링부트는 대부분 CGLIB 사용

---

## 중요한 특징

### 프록시는 스프링 빈에만 적용됨

```java
new OrderService()
```

직접 생성하면 프록시 적용 안됨

반드시:

```java
@Autowired
private OrderService orderService;
```

이렇게 스프링이 관리해야 함

---

## 한줄 정리

### 프록시란?

```text
실제 객체 앞에서 공통 기능을 대신 처리하는 대리 객체
```

---

### 스프링이 프록시를 쓰는 이유

```text
트랜잭션, 로깅, 보안 같은 공통 기능을
비즈니스 로직과 분리하기 위해 사용한다.
```

---

### 핵심 흐름 한방 정리

```text
사용자 요청
 ↓
프록시 객체
 ↓
트랜잭션 시작
 ↓
실제 서비스 호출
 ↓
commit / rollback
```
