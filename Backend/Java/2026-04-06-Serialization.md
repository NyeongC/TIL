# 📌 직렬화 / 역직렬화 (Serialization / Deserialization)

## 📖 한줄 정의

- 직렬화 → 객체를 전송/저장 가능한 형태(JSON, byte 등)로 변환  
- 역직렬화 → 변환된 데이터를 다시 객체로 복원  

---

## 📌 왜 필요한가?

객체는 메모리에만 존재하는 구조라서  
👉 그대로는 **네트워크 전송 / 파일 저장 불가능**

그래서  
👉 **텍스트 or 바이트 형태로 변환 필요**

---

## 📌 언제 쓰냐 (실무 기준)

- API 요청/응답 (JSON)
- Redis 캐싱
- 파일 저장
- 메시지 큐 (Kafka 등)

👉 거의 모든 서버 개발에서 사용됨

---

## 📌 흐름

[객체] → (직렬화) → JSON/byte → (전송/저장) → JSON/byte → (역직렬화) → [객체]

---

## 📌 Java 예제 (Jackson)

### 🔹 1. 객체 → JSON (직렬화)

```java
import com.fasterxml.jackson.databind.ObjectMapper;

public class Main {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();

        User user = new User("user", 30);

        String json = objectMapper.writeValueAsString(user);

        System.out.println(json);
    }
}
```

👉 결과
```json
{"name":"user","age":30}
```

---

### 🔹 2. JSON → 객체 (역직렬화)

```java
import com.fasterxml.jackson.databind.ObjectMapper;

public class Main {
    public static void main(String[] args) throws Exception {
        ObjectMapper objectMapper = new ObjectMapper();

        String json = "{\"name\":\"user\",\"age\":30}";

        User user = objectMapper.readValue(json, User.class);

        System.out.println(user.getName());
    }
}
```

---

### 🔹 User 클래스

```java
public class User {
    private String name;
    private int age;

    public User() {} // 기본 생성자 필수

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // getter/setter
}
```

---

## 📌 핵심 포인트

### 1️⃣ JSON만 있는게 아님
- JSON
- XML
- byte stream

👉 전부 직렬화 대상

---

### 2️⃣ API에서는 거의 JSON
👉 Spring Boot + Jackson 자동 처리

```java
@GetMapping("/user")
public User getUser() {
    return new User("user", 30);
}
```

👉 자동 직렬화됨 (객체 → JSON)

---

### 3️⃣ 역직렬화 시 기본 생성자 필요

👉 Jackson은 객체 생성 후 필드 세팅 방식

```java
public User() {} // 없으면 에러
```

---

## 📌 한줄 요약

👉  
객체를 네트워크/저장 가능한 형태로 바꾸는게 직렬화, 다시 객체로 만드는게 역직렬화

---

## 🔥 실무 감각 한줄

👉  
Spring에서는 Controller에서 객체를 반환하면 자동으로 JSON 직렬화되고, 요청 body는 자동으로 역직렬화된다.
