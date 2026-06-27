# Jenkins + Docker + Kubernetes 배포 컨셉

## 전체 컨셉

기존에는 서버에 직접 JAR를 복사해서 실행하는 방식이었다.

```text
개발자
   │
git push
   ▼
GitHub
   │
Jenkins
   │
JAR 생성
   │
scp 전송
   ▼
운영 서버
   │
java -jar app.jar
```

하지만 Kubernetes 환경에서는 JAR를 직접 서버에 복사하지 않는다.

JAR를 Docker Image로 만든 뒤 Image Registry(Docker Hub 등)에 업로드하고, Kubernetes가 해당 이미지를 내려받아 Pod를 생성한다.

```text
개발자
   │
GitHub
   │
Jenkins
   │
Docker Image 생성
   │
Docker Hub Push
   │
kubectl 배포
   ▼
Kubernetes
   │
Image Pull
   │
Pod 생성
   ▼
서비스 운영
```

---

# 서버 구성

보통 서버는 역할을 나누어 운영한다.

## 1. Jenkins 서버

Jenkins 서버는 **빌드와 배포를 담당**한다.

보통 설치되는 프로그램

- Git
- JDK
- Gradle
- Docker
- Jenkins
- kubectl

각각의 역할

| 프로그램 | 역할 |
|----------|------|
| Git | GitHub에서 소스 다운로드 |
| JDK | Java 컴파일 |
| Gradle | JAR 생성 |
| Docker | Docker Image 생성 |
| Jenkins | CI/CD 자동화 |
| kubectl | Kubernetes에 배포 명령 전달 |

---

## 2. Kubernetes 서버(Cluster)

Kubernetes 서버는 **애플리케이션을 실제 운영**하는 역할이다.

마스터 노드

- API Server
- Scheduler
- Controller Manager
- etcd

워커 노드

- kubelet
- containerd
- Pod 실행

여기에는 JDK나 Gradle이 필요하지 않다.

이미 Docker Image 안에 실행에 필요한 JDK와 애플리케이션이 모두 포함되어 있기 때문이다.

---

# Jenkins에서 하는 일

### 1. GitHub에서 최신 소스 가져오기

```bash
git pull
```

---

### 2. Gradle Build

```bash
./gradlew clean build
```

Gradle은 내부적으로 JDK를 이용하여

- Java 컴파일
- 테스트 수행
- JAR 생성

을 진행한다.

결과

```
build/libs/app.jar
```

---

### 3. Docker Image 생성

Dockerfile을 이용하여

```
JDK
+
app.jar
```

를 하나의 Docker Image로 만든다.

예시

```dockerfile
FROM eclipse-temurin:17-jdk
COPY app.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

---

### 4. Docker Hub 업로드

```bash
docker push my-api:1.0
```

Docker Hub에는 이제

```
my-api:1.0
```

이라는 이미지가 저장된다.

---

### 5. Kubernetes 배포

Jenkins는 마지막으로

```bash
kubectl apply -f deployment.yml
```

또는

```bash
kubectl set image ...
```

명령을 실행한다.

여기서 중요한 점은

**Jenkins가 Pod를 만드는 것이 아니다.**

Jenkins는 단순히

> "쿠버네티스야, 새로운 이미지를 사용해서 배포해."

라고 요청하는 역할만 한다.

---

# Kubernetes는 무엇을 할까?

kubectl 명령을 받은 Kubernetes는 내부적으로

1. Deployment를 확인한다.
2. Worker Node를 선택한다.
3. Docker Hub에서 이미지를 다운로드(Pull)한다.
4. Pod를 생성한다.
5. Pod 안에서 Container를 실행한다.

즉,

```text
kubectl
   │
   ▼
Kubernetes
   │
Deployment 확인
   │
Image Pull
   │
Pod 생성
   │
Container 실행
```

이 모든 작업은 Kubernetes가 자동으로 수행한다.

---

# Replica가 2개라면?

Deployment

```yaml
replicas: 2
```

라면 Kubernetes는 동일한 Pod를 두 개 실행한다.

```text
          Service
        ┌────┴────┐
        ▼         ▼
     Pod 1     Pod 2
```

사용자의 요청은 Service가 두 Pod로 분산한다.

---

# 전체 흐름

```text
Developer
    │
git push
    ▼
GitHub
    │
    ▼
Jenkins
    │
Git Pull
    │
Gradle Build
    │
Docker Image 생성
    │
Docker Hub Push
    │
kubectl apply
    ▼
Kubernetes
    │
Image Pull
    │
Pod 생성
    │
Container 실행
    ▼
Service
    │
사용자 요청 처리
```

---

# 핵심 정리

- Jenkins는 **빌드와 배포를 담당**한다.
- Gradle은 **JDK를 이용하여 JAR를 생성**한다.
- Docker는 **JAR를 Image로 만든다.**
- Docker Hub는 **Image 저장소**이다.
- Jenkins는 **kubectl 명령으로 Kubernetes에 배포를 요청**한다.
- **Pod를 생성하는 주체는 Jenkins가 아니라 Kubernetes**이다.
- Kubernetes는 이미지를 Pull하여 Pod와 Container를 생성한다.
- Service는 여러 Pod로 트래픽을 분산한다.