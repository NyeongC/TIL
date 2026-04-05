## CI/CD 첫 배포 삽질 정리 (GitHub Actions + Spring Boot)

---

## 1. 기존 프로세스 종료 방식 (중요🔥)

### ❌ 문제 상황

* 수동 `nohup`으로 실행 → PID 관리 안됨
* CI/CD에서 기존 프로세스 못 죽임
* 옛날 jar 계속 실행됨

---

### ✅ 해결 (PID 기반 관리)

```bash
if [ -f app.pid ]; then
  kill -15 $(cat app.pid) || true
  rm -f app.pid
fi
```

```bash
nohup java -jar app.jar > app.log 2>&1 < /dev/null &
echo $! > app.pid
disown
```

👉 핵심

* `pkill` 쓰지 말 것 (전체 java 죽음)
* PID 파일로 정확하게 관리

---

## 2. plain.jar 문제

### ❌ 문제 상황

* `ls *.jar` → plain.jar 먼저 잡힘
* 실행 → 바로 죽음
* 서버 안뜸

---

### 🔥 plain.jar란?

* 실행 불가 jar (라이브러리용)
* 톰캣 없음

---

### ✅ 해결

```gradle
bootJar {
    archiveFileName = 'app.jar'
}

jar {
    enabled = false
}
```

👉 결과

```
app.jar 하나만 생성
```

---

## 3. 자동 배포 vs 수동 배포

### ✅ 자동 배포

```yaml
on:
  push:
    branches:
      - main
```

👉 main push 시 자동 배포

---

### ✅ 수동 배포

```yaml
on:
  workflow_dispatch:
```

👉 GitHub Actions → Run workflow 버튼

---

### ✅ 혼합 (추천)

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

👉 자동 + 수동 둘 다 가능

---

## 4. deploy.yml 핵심 구성

```yaml
- name: 서버 실행
  uses: appleboy/ssh-action@v0.1.6
  with:
    host: ${{ secrets.SERVER_IP }}
    username: ubuntu
    key: ${{ secrets.SSH_KEY }}
    script: |
      cd /home/ubuntu/app

      if [ -f app.pid ]; then
        kill -15 $(cat app.pid) || true
        rm -f app.pid
      fi

      nohup java -jar app.jar > app.log 2>&1 < /dev/null &

      echo $! > app.pid
      disown
```

---

## 5. GitHub Secrets 설정

### 위치

```
Settings → Secrets and variables → Actions
→ Repository secrets
```

---

### 필요한 값

| 이름        | 값           |
| --------- | ----------- |
| SERVER_IP | 서버 공인 IP    |
| SSH_KEY   | 개인키 (전체 복붙) |

---

### ⚠️ SSH_KEY 주의

```
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

* 전체 포함
* 줄바꿈 유지

---

## 6. 배포 흐름

```
git push
 → GitHub Actions 실행
 → build
 → jar 생성
 → 서버 전송
 → 기존 프로세스 종료
 → 새 jar 실행
```

---

## 7. 실행 확인 방법

```bash
ps -ef | grep java
```

```bash
ss -tuln | grep 8080
```

```bash
curl localhost:8080/hello
```

---

## 💥 한줄 요약

* pkill ❌ → PID 관리 ⭕
* plain.jar ❌ → bootJar만 사용 ⭕
* 자동/수동 배포 둘 다 가능
* Secrets로 서버 접근

---

## 🚀 느낀점

* 배포는 “코드 문제가 아니라 구조 문제”
* CI/CD는 실행보다 “프로세스 관리”가 더 중요
* jar 하나로 통일하는게 안정성 핵심
