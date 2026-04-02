## @ControllerAdvice

### 1. 정의

* 컨트롤러 전역에서 발생하는 예외를 공통으로 처리하는 컴포넌트
* `@ExceptionHandler`를 통해 특정 예외를 잡아 HTTP 응답으로 변환

👉 컨트롤러마다 예외 처리 코드를 작성하지 않도록 해줌

---

### 2. 사용하는 이유

#### ✔ 1) 중복 제거

* 각 컨트롤러마다 try-catch 작성할 필요 없음

#### ✔ 2) 응답 통일

* 모든 API의 에러 응답 형식을 일관되게 유지 가능

#### ✔ 3) 역할 분리

* Service → 예외 발생
* Controller → 요청 처리
* ControllerAdvice → 응답 변환

#### ✔ 4) 유지보수 용이

* 예외 처리 정책 변경 시 한 곳만 수정

---

### 3. 기본 사용 예시

#### 📌 커스텀 예외

```java
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

---

#### 📌 서비스

```java
public User findUser(Long id) {
    return userRepository.findById(id)
        .orElseThrow(() -> new UserNotFoundException("유저 없음"));
}
```

---

#### 📌 ControllerAdvice

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handle(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("USER_NOT_FOUND", ex.getMessage()));
    }
}
```

---

### 4. 처리 흐름

```text
클라이언트 요청
 ↓
DispatcherServlet
 ↓
Controller
 ↓
Service (예외 발생)
 ↓
(예외 전파)
 ↓
DispatcherServlet
 ↓
HandlerExceptionResolver
 ↓
@ControllerAdvice (@ExceptionHandler)
 ↓
HTTP 응답 반환
```

---

### 5. 핵심 개념

#### ✔ 1) 예외는 위로 전파된다

* Service에서 발생한 예외도 ControllerAdvice에서 처리됨

#### ✔ 2) RuntimeException 사용

* try-catch 없이 throw 가능
* 전역 처리 구조와 잘 맞음

#### ✔ 3) Controller에서는 try-catch 안쓴다

* 예외는 던지고, 처리 책임은 ControllerAdvice가 가짐

---

### 6. 한 줄 정리

👉
Controller에서 발생한 예외를 전역에서 처리하여
일관된 API 응답으로 변환하는 구조

---

### 7. 핵심 답변

👉
"@ControllerAdvice는 DispatcherServlet의 예외 처리 과정에서 HandlerExceptionResolver를 통해 호출되며, 컨트롤러 및 하위 계층에서 발생한 예외를 전역적으로 처리하고 일관된 API 응답으로 변환하는 역할을 합니다."
