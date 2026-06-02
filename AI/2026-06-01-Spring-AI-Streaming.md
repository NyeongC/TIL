# Spring AI - Flux와 스트리밍 응답 정리

Spring AI를 학습하면서 `chatModel.call()`과 `chatModel.stream()`의 차이가 궁금했다.

처음에는 단순히 반환 타입 차이로 생각했지만, 실제로는 LLM의 응답 생성 방식과 사용자 경험(UX)에 큰 차이가 있다는 것을 알게 되었다.

---

## LLM은 답변을 한 번에 생성하지 않는다

질문

```text
김치가 뭐야?
```

LLM 내부

```text
김치
 ↓
는
 ↓
한국의
 ↓
전통
 ↓
발효식품
 ↓
입니다
```

LLM은 완성된 문장을 만드는 것이 아니라 토큰(Token)을 하나씩 생성한다.

---

## ChatModel.call()

```java
ChatResponse response = chatModel.call(prompt);
```

동작 방식

```text
질문 전송
 ↓
LLM 전체 답변 생성
 ↓
응답 완성
 ↓
한 번에 반환
```

예시

```java
String answer =
    "김치는 한국의 전통 발효식품입니다.";
```

사용자는 답변이 완성될 때까지 기다려야 한다.

---

## ChatModel.stream()

```java
Flux<ChatResponse> response =
    chatModel.stream(prompt);
```

동작 방식

```text
질문 전송
 ↓
"김치"
 ↓
"는"
 ↓
"한국의"
 ↓
"전통"
 ↓
"발효식품"
 ↓
"입니다"
```

생성되는 즉시 응답을 전달한다.

---

## Flux란?

Flux는 Spring WebFlux의 Reactive Stream 객체이다.

쉽게 말하면

> 여러 개의 데이터를 시간차를 두고 흘려보내는 통로

라고 생각하면 된다.

---

## List와 Flux 차이

### List

```java
List<String> list =
    List.of("A", "B", "C");
```

```text
A, B, C
```

모든 데이터가 준비된 후 반환된다.

---

### Flux

```java
Flux<String> flux =
    Flux.just("A", "B", "C");
```

```text
A
 ↓
B
 ↓
C
```

데이터가 순차적으로 전달된다.

---

## Flux<String>은 계속 들어오는가?

그럴 수도 있고 아닐 수도 있다.

### Spring AI

```java
Flux<String>
```

```text
안녕
 ↓
하세요
 ↓
반갑습니다
 ↓
완료(onComplete)
```

GPT 응답이 끝나면 종료된다.

---

### 무한 스트림 예시

```java
Flux.interval(Duration.ofSeconds(1));
```

```text
0
 ↓
1
 ↓
2
 ↓
3
 ↓
...
```

직접 종료하기 전까지 계속 흘러온다.

---

## 현재 구현 구조

### Backend

```java
@PostMapping(
    produces = MediaType.APPLICATION_NDJSON_VALUE
)
public Flux<String> chat() {
    return service.generateStreamText(question);
}
```

```java
chatModel.stream(prompt)
```

↓

```java
Flux<String>
```

↓

HTTP 스트림 전송

---

### Frontend

```javascript
const response = await fetch(...);
```

```javascript
const reader =
    response.body.getReader();
```

브라우저가 응답 스트림을 읽는다.

---

## 핵심 코드

### 응답 읽기

```javascript
while (true) {

    const { value, done }
        = await reader.read();

    if (done) break;
}
```

동작

```text
청크 수신
 ↓
화면 출력
 ↓
청크 수신
 ↓
화면 출력
```

반복

---

## 왜 Flux + Stream을 사용하는가?

### 1. 사용자 경험 향상

call()

```text
5초 대기
 ↓
답변 출력
```

stream()

```text
1초 후 응답 시작
 ↓
계속 출력
 ↓
완료
```

사용자는 응답이 오고 있음을 즉시 확인할 수 있다.

---

### 2. 긴 답변에 유리

1000자 답변

call()

```text
전부 생성 후 출력
```

stream()

```text
생성 즉시 출력
```

---

### 3. ChatGPT 방식 구현 가능

현재 ChatGPT도 유사한 방식으로 동작한다.

```text
질문
 ↓
답
 ↓
변
 ↓
생
 ↓
성
```

실시간으로 보이는 이유가 스트리밍 때문이다.

---

## 정리

### call()

```java
String
```

- 전체 응답 생성 후 반환
- 구현이 단순
- 일반 REST API에 적합

---

### stream()

```java
Flux<String>
```

- 응답 생성 즉시 전달
- 실시간 UI 구현 가능
- AI 채팅 서비스에 적합

---

## 한 줄 정리

LLM은 토큰을 하나씩 생성한다. `chatModel.stream()`은 이 토큰들을 `Flux<String>` 형태로 흘려보내고, 프론트는 `response.body.getReader()`로 이를 읽어 실시간으로 화면에 출력한다.
