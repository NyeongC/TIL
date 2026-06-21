# Google OTP 구현 정리

## 1. Google OTP(One Time Password)란?

OTP(One Time Password)는 말 그대로 **일회용 비밀번호**이다.

일반 비밀번호는 항상 동일하지만 OTP는 일정 시간이 지나면 새로운 값으로 변경된다.

Google OTP는 스마트폰의 Google Authenticator 앱을 이용하여 OTP를 생성하는 방식이며, 서버와 사용자가 동일한 Secret Key를 공유하여 인증을 수행한다.

---

### TOTP(Time-based One Time Password)

Google OTP는 TOTP 방식을 사용한다.

TOTP는 현재 시간을 기반으로 OTP를 생성하는 방식이다.

일반적으로 30초마다 새로운 OTP가 생성된다.

예시

```text
12:00:00 ~ 12:00:29
→ 123456

12:00:30 ~ 12:00:59
→ 847291

12:01:00 ~ 12:01:29
→ 552183
```

사용자는 현재 표시된 6자리 숫자를 입력하고, 서버도 동일한 Secret Key와 현재 시간을 이용하여 OTP를 계산한다.

계산 결과가 일치하면 인증에 성공한다.

---

### Google OTP 동작 원리

```text
Secret Key 생성
↓
서버 DB 저장
↓
QR 생성
↓
Authenticator 앱 등록(TOTP를 지원하는 앱 다 가능)
↓
앱과 서버가 동일한 Secret Key 보유
↓
현재 시간을 기반으로 OTP 생성
↓
사용자 OTP 입력
↓
서버 OTP 검증
↓
인증 성공
```

중요한 점은 서버가 OTP 번호를 저장하는 것이 아니라는 것이다.

서버와 앱이 동일한 Secret Key를 가지고 있기 때문에 각각 독립적으로 OTP를 계산할 수 있다.

따라서 DB에는 OTP 번호가 아닌 Secret Key만 저장한다.

---

## 2. Google OTP 인증 흐름

```text
Secret Key 생성
↓
otpauth URI 생성
↓
QR 생성
↓
Google Authenticator 등록
↓
사용자 OTP 입력
↓
서버 OTP 검증
↓
인증 성공
```

---

## 3. 프로젝트에 사용한 라이브러리

### Gradle 의존성

```gradle
// Google OTP Secret Key 생성 및 OTP 코드 검증
implementation 'com.warrenstrange:googleauth:1.5.0'

// QR 코드 생성에 필요한 핵심 라이브러리
implementation 'com.google.zxing:core:3.5.3'

// BitMatrix를 PNG 이미지로 변환할 때 사용
implementation 'com.google.zxing:javase:3.5.3'
```

---

## 4. Secret Key 생성

### Secret Key란?

Secret Key는 Google OTP의 핵심 값이다.

서버와 Google Authenticator 앱은 동일한 Secret Key를 공유하게 되며, 이 값을 기준으로 OTP를 생성한다.

즉, OTP 번호 자체를 저장하는 것이 아니라 Secret Key를 저장하고, 서버와 앱이 각각 동일한 알고리즘으로 OTP를 계산하는 방식이다.

예시

```text
서버
Secret Key = ABCDEF123456
↓
OTP 생성
↓
123456

Google Authenticator
Secret Key = ABCDEF123456
↓
OTP 생성
↓
123456
```

따라서 OTP 인증 시에는 OTP 번호를 비교하는 것이 아니라, 동일한 Secret Key를 기준으로 생성된 OTP가 일치하는지 확인한다.

---

### Secret Key 생성 및 DB 저장

Google OTP 등록을 위해서는 먼저 사용자별 Secret Key를 생성해야 한다.

Secret Key는 Google Authenticator 앱과 서버가 함께 사용하는 기준값이다.

서버는 이 Secret Key를 DB에 저장하고, 사용자는 이 Secret Key가 포함된 QR을 Google Authenticator 앱에 등록한다.

```java
GoogleAuthenticatorKey key =
        googleAuthenticator.createCredentials();

GoogleOtp otp = GoogleOtp.builder()
        .user(user)
        .secretKey(key.getKey())
        .build();

return googleOtpRepository.save(otp);
```

흐름은 다음과 같다.

```text
GoogleAuthenticator가 Secret Key 생성
↓
현재 로그인 사용자와 Secret Key를 매핑
↓
DB 저장
↓
이후 QR 생성 시 사용
↓
OTP 검증 시 사용
```

즉, OTP 생성 시 사용자 정보를 Secret Key 생성 함수에 직접 넣는 것은 아니다.

대신 생성된 Secret Key를 특정 사용자와 연결하여 저장한다.

```text
User 1명
↓
Secret Key 1개 발급
↓
Google Authenticator 등록
↓
OTP 검증 기준값으로 사용
```

---

### Secret Key 생성 코드

```java
GoogleAuthenticator gAuth =
        new GoogleAuthenticator();

GoogleAuthenticatorKey key =
        gAuth.createCredentials();

String secretKey =
        key.getKey();
```

생성 예시

```text
JBSWY3DPEHPK3PXP
```

---

## 5. otpauth URI 생성

### 왜 필요한가?

Google Authenticator 앱은 Secret Key를 직접 입력할 수도 있지만, 일반적으로 QR을 스캔하여 등록한다.

하지만 QR에는 단순 Secret Key만 들어있는 것이 아니라 Google OTP 표준 규격인 `otpauth://` URI가 들어있다.

예시

```text
otpauth://totp/ALPS:test@test.com?secret=ABCDEF123456&issuer=ALPS
```

구성 요소

```text
issuer
→ 앱에 표시될 서비스 이름

email
→ 사용자 식별 정보

secret
→ OTP 생성에 사용할 Secret Key
```

Google Authenticator는 이 URI를 읽어 OTP 정보를 등록한다.

---

### URI 생성 코드

```java
String otpAuthUrl = String.format(
        "otpauth://totp/%s:%s?secret=%s&issuer=%s",
        encodedIssuer,
        encodedEmail,
        secret,
        encodedIssuer
);
```

---

## 6. ZXing을 이용한 QR 생성

ZXing은 QR 코드를 생성하기 위한 라이브러리이다.

생성한 otpauth URI를 QR 이미지로 변환하는 역할을 수행한다.

### QR 생성 코드

```java
QRCodeWriter writer =
        new QRCodeWriter();

BitMatrix bitMatrix =
        writer.encode(
                otpAuthUrl,
                BarcodeFormat.QR_CODE,
                250,
                250
        );
```

흐름

```text
otpauth URI
↓
QRCodeWriter
↓
BitMatrix
↓
QR 이미지
```

---

## 7. Base64 이미지 변환

### Base64를 사용하는 이유

QR 이미지를 서버 디스크에 저장하지 않고 브라우저에 바로 전달하기 위함이다.

```text
QR 생성
↓
PNG 이미지
↓
Base64 문자열 변환
↓
JSON 응답
↓
브라우저 출력
```

이미지 파일 저장 없이 즉시 화면에 출력할 수 있다는 장점이 있다.

---

### PNG → Base64 변환 코드

```java
ByteArrayOutputStream baos =
        new ByteArrayOutputStream();

MatrixToImageWriter.writeToStream(
        bitMatrix,
        "PNG",
        baos
);

return Base64.getEncoder()
        .encodeToString(
                baos.toByteArray()
        );
```

---

## 8. QR 생성 메서드 전체 코드

```java
public String generateQrBase64(
        String secret,
        String email
) {

    try {

        String encodedIssuer =
                URLEncoder.encode(
                        ISSUER,
                        StandardCharsets.UTF_8
                );

        String encodedEmail =
                URLEncoder.encode(
                        email,
                        StandardCharsets.UTF_8
                );

        String otpAuthUrl =
                String.format(
                        "otpauth://totp/%s:%s?secret=%s&issuer=%s",
                        encodedIssuer,
                        encodedEmail,
                        secret,
                        encodedIssuer
                );

        QRCodeWriter writer =
                new QRCodeWriter();

        BitMatrix bitMatrix =
                writer.encode(
                        otpAuthUrl,
                        BarcodeFormat.QR_CODE,
                        250,
                        250
                );

        ByteArrayOutputStream baos =
                new ByteArrayOutputStream();

        MatrixToImageWriter.writeToStream(
                bitMatrix,
                "PNG",
                baos
        );

        return Base64.getEncoder()
                .encodeToString(
                        baos.toByteArray()
                );

    } catch (Exception e) {

        throw new RuntimeException(
                "QR 코드 생성 실패",
                e
        );
    }
}
```

---

## 9. 프론트 연동

### 서버 응답

```json
{
  "qrBase64": "iVBORw0KGgoAAAANSUhEUgAA..."
}
```

또는

```json
{
  "qrBase64": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}
```

---

### 출력 부분 


#### 순수 HTML 
```HTML
<img th:src="'data:image/png;base64,' + ${qrBase64}" alt="OTP QR Code" width="220"/>
```

---

### 프론트 역할

프론트는 QR을 직접 생성하지 않는다.

서버가 생성한 QR 이미지를 받아 화면에 출력만 한다.

```text
백엔드
Secret Key 생성
↓
QR 생성
↓
Base64 변환
↓
응답

프론트
↓
img 태그 출력
↓
사용자가 QR 스캔
↓
OTP 입력
↓
OTP 검증 API 호출
```

---

## 10. OTP 검증

### 검증 코드

```java
boolean success =
        googleAuthenticator.authorize(
                secretKey,
                code
        );
```

---

### 검증 원리

```text
서버
Secret Key 보유
↓
현재 시간 기준 OTP 생성

Google Authenticator
Secret Key 보유
↓
현재 시간 기준 OTP 생성

두 값 비교
↓
일치
↓
인증 성공
```

---

## 11. DB에는 무엇을 저장하는가?

### 저장하는 것

* Secret Key
* OTP 활성화 여부
* 실패 횟수

### 저장하지 않는 것

* OTP 번호
* QR 이미지

---

## 12. 구현하며 알게 된 점

* OTP 번호를 저장하는 방식이 아니다.
* Secret Key만 저장한다.
* Google Authenticator는 Secret Key를 기반으로 OTP를 생성한다.
* 서버도 동일한 Secret Key와 시간을 기반으로 OTP를 계산한다.
* QR은 단순히 Secret Key를 전달하기 위한 수단이다.
* 프론트는 QR을 생성하지 않고 서버가 생성한 QR 이미지를 출력만 한다.

---

## 정리

Google OTP는 서버와 사용자가 동일한 Secret Key를 공유하고, 현재 시간을 기반으로 생성된 OTP를 비교하여 인증하는 방식이다.

실제 구현 시에는 다음 순서로 동작한다.

```text
Secret Key 생성
↓
사용자와 Secret Key 매핑 후 DB 저장
↓
otpauth URI 생성
↓
ZXing QR 생성
↓
Base64 변환
↓
프론트 출력
↓
Google Authenticator 등록
↓
OTP 검증
```
