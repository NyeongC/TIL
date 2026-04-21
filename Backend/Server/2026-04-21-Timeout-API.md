# API 연동시 Timeout 

## 1. 왜 타임아웃이 중요한가?

외부 API(PG사, 타 시스템 등)를 호출할 때
응답이 늦거나 끊길 수 있다.

-> 타임아웃이 없으면:

* 스레드가 계속 대기 (Thread Pool 고갈)
* 전체 서비스 장애로 확산
* 중복 요청/중복 결제 발생

-> 따라서 타임아웃은 **성능이 아니라 “생존 장치”**

---

## 2. 타임아웃 종류

### ✔️ Connection Timeout

* 서버와 **연결 자체를 맺는 시간 제한**
* TCP 연결 (3-way handshake)

```text
“서버 연결이 안되는데 언제까지 기다릴 것인가?”
```

-> 발생 상황

* 서버 다운
* 네트워크 단절
* 포트 차단

---

### ✔️ Read Timeout

* 연결 이후 **응답 데이터를 기다리는 시간 제한**

```text
“요청은 보냈는데 응답이 안 오네, 언제 포기할까?”
```

-> 발생 상황

* 서버 처리 지연
* DB 느림
* 외부 시스템 장애

---

## 3. 한줄 핵심 정리

```text
Connection Timeout = 연결 실패
Read Timeout = 응답 지연
```

---

## 4. RestTemplate 적용 (기본)

```java
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

public class RestTemplateConfig {

    public static RestTemplate create() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();

        factory.setConnectTimeout(3000); // 3초
        factory.setReadTimeout(5000);    // 5초

        return new RestTemplate(factory);
    }
}
```

---

## 5. RestTemplate (HttpClient 기반 - 실무 추천)

-> 커넥션 풀 지원

```java
import org.apache.hc.client5.http.config.RequestConfig;
import org.apache.hc.client5.http.impl.classic.HttpClients;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;

public class RestTemplateConfig {

    public static RestTemplate create() {

        RequestConfig config = RequestConfig.custom()
                .setConnectTimeout(3000)
                .setResponseTimeout(5000)
                .build();

        var httpClient = HttpClients.custom()
                .setDefaultRequestConfig(config)
                .build();

        return new RestTemplate(new HttpComponentsClientHttpRequestFactory(httpClient));
    }
}
```

---

## 6. WebClient 적용 (요즘 스타일, Reactive)

-> Spring에서 점점 권장되는 방식

```java
import io.netty.channel.ChannelOption;
import reactor.netty.http.client.HttpClient;
import org.springframework.web.reactive.function.client.WebClient;

import java.time.Duration;

public class WebClientConfig {

    public static WebClient create() {

        HttpClient httpClient = HttpClient.create()
                .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000) // 연결 타임아웃
                .responseTimeout(Duration.ofSeconds(5));            // 응답 타임아웃

        return WebClient.builder()
                .clientConnector(new org.springframework.http.client.reactive.ReactorClientHttpConnector(httpClient))
                .build();
    }
}
```
---
## 7. 예외처리

```java
try {
    ResponseEntity<String> response =
            restTemplate.exchange(url, HttpMethod.POST, request, String.class);

    return response.getBody();

} catch (ResourceAccessException e) {

    if (e.getCause() instanceof java.net.SocketTimeoutException) {
        // Read timeout
        throw new RuntimeException("PG 응답 지연");
    }

    if (e.getCause() instanceof java.net.ConnectException) {
        // Connection timeout
        throw new RuntimeException("PG 서버 연결 실패");
    }

    throw new RuntimeException("외부 API 호출 실패");
}
```
---

## 8. 실무에서 꼭 같이 가는 것

```text
Timeout + Retry + CircuitBreaker
```

* Timeout → 기본 방어
* Retry → 일시 장애 대응
* CircuitBreaker → 장애 확산 방지

---

## 9. 실무 기준 정리

* 외부 API 호출에는 반드시 타임아웃 설정
* 공통 모듈에서 관리하는 것이 좋음
* 타임아웃 없으면 잠재 장애 상태

---

## 10. 정리

> 외부 API 호출 시 Connection/Read Timeout을 설정하지 않으면
> 스레드 점유로 인해 전체 서비스 장애로 이어질 수 있기 때문에
> 필수적으로 적용해야 한다고 생각합니다.

---

