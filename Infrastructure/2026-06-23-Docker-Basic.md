# Docker

## Docker란?

Docker는 리눅스 컨테이너 기술을 쉽게 사용할 수 있도록 만든 오픈소스 플랫폼이다.

원래 리눅스에는 컨테이너를 만들 수 있는 기능이 있었지만 사용이 복잡했다.

Docker는 이러한 기능을 쉽게 사용할 수 있도록 다음과 같은 기능을 제공한다.

- Docker Engine
- Docker Image
- Docker Container
- Docker Hub
- Docker Compose
- Docker Desktop

실제로 컨테이너를 생성하고 실행하는 핵심은 Docker Engine이다.

---

## 과거 가상화 방식(VM)

과거에는 하이퍼바이저(Hypervisor) 기반 가상화를 사용했다.

```text
Host OS
 ├ VM1
 │ └ Guest OS
 ├ VM2
 │ └ Guest OS
 └ VM3
   └ Guest OS
```

각 VM은 독립적인 운영체제를 가진다.

예를 들어

- Ubuntu VM
- Rocky Linux VM
- Windows VM

모두 각각 자신의 OS를 가지고 실행된다.

### 단점

- Guest OS 설치 필요
- 메모리 사용량 큼
- 부팅 느림
- 디스크 사용량 큼
- 성능 오버헤드 존재

---

## Docker 컨테이너 방식

Docker는 VM처럼 OS 전체를 가상화하지 않는다.

리눅스 커널 기능을 이용하여 프로세스를 격리한다.

사용되는 주요 기술

- Namespace
- Cgroups
- Chroot

### Namespace

프로세스가 서로를 보지 못하게 함

예)

```text
Container A
 └ Process 1

Container B
 └ Process 1
```

서로 자신의 프로세스만 볼 수 있다.

### Cgroups

CPU, Memory 사용량 제한

예)

```text
Container A
CPU 2 Core
Memory 2GB

Container B
CPU 1 Core
Memory 1GB
```

### Chroot

컨테이너마다 다른 루트 디렉토리를 사용

```text
Container A
/

Container B
/
```

둘 다 / 를 보지만 실제로는 서로 다른 공간이다.

---

## Docker가 빠른 이유

VM

```text
Host OS
 ↓
Hypervisor
 ↓
Guest OS
 ↓
Application
```

Docker

```text
Host OS Kernel
 ↓
Container
 ↓
Application
```

Docker는 Guest OS를 따로 만들지 않는다.

Host OS의 Kernel을 공유한다.

따라서

- 실행 속도 빠름
- 메모리 사용량 적음
- 이미지 크기 작음

---

## 커널을 공유한다는 의미

VM은 Guest OS 자체를 포함한다.

예)

```text
Ubuntu ISO
+ Library
+ Application
```

그래서 수 GB 단위가 된다.

반면 Docker 이미지는

```text
Library
+ Application
```

위주로 구성된다.

커널은 Host OS의 커널을 사용한다.

그래서 이미지 크기가 훨씬 작다.

---

## 격리(Isolation)란?

처음에는

```text
localhost:5432
```

로 접속되는데 왜 격리라고 하는지 헷갈렸다.

실제로는 컨테이너는 기본적으로 격리되어 있다.

격리되는 것

- 프로세스
- 파일시스템
- 네트워크

### 프로세스 격리

컨테이너 내부 PostgreSQL 프로세스는 실행 중이지만

윈도우 작업관리자에서는 postgres.exe 로 보이지 않는다.

### 파일시스템 격리

컨테이너 내부에서 파일 생성

```bash
touch test.txt
```

해도 내 C:\ 드라이브에서는 보이지 않는다.

### 네트워크 격리

기본적으로 컨테이너는 외부 접근이 불가능하다.

---

## 포트 매핑

```bash
docker run -p 5432:5432 postgres
```

의 의미

```text
Host PC
localhost:5432
        ↓
Container:5432
```

원래는 접근할 수 없는 컨테이너에

-p 옵션으로 통로를 만들어 준 것이다.

즉

```text
격리되어 있음
+
포트 공개
=
외부 접근 가능
```

---

## Docker Image

Docker Image는 컨테이너를 만들기 위한 설계도이다.

예)

```text
JDK17
Spring Boot
실행 명령어
```

를 하나로 묶어 둔 것

이미지는 실행되지 않는다.

---

## Docker Container

Docker Image를 실행한 결과물

```text
Image
 ↓
Run
 ↓
Container
```

예)

```bash
docker run my-app
```

```text
my-app Image
 ↓
Container 생성
```

---

## Image와 Container 관계

붕어빵 비유

```text
Image
=
붕어빵 틀

Container
=
실제 붕어빵
```

이미지 하나로 여러 컨테이너 생성 가능

```bash
docker run my-app
docker run my-app
docker run my-app
```

```text
Container A
Container B
Container C
```

---

## Docker Hub

Docker 이미지 저장소

GitHub와 비슷한 개념

| GitHub | Docker Hub |
|----------|----------|
| 소스코드 저장 | 이미지 저장 |
| git push | docker push |
| git clone | docker pull |

흐름

```text
소스코드 작성
 ↓
Image 생성
 ↓
Docker Hub 업로드
 ↓
다른 서버에서 Pull
 ↓
Container 실행
```

---

## Docker Desktop

Docker Desktop은 Docker Engine을 쉽게 사용하기 위한 GUI 도구이다.

실제로 컨테이너를 만드는 것은 Docker Engine이다.

```text
Docker Desktop
 ├ GUI
 ├ WSL2 연동
 ├ 설정화면
 └ Docker Engine
```

Windows에서

```bash
docker ps
docker images
docker run
```

명령어가 동작하는 이유는

Docker Desktop 내부에서 Docker Engine이 실행되고 있기 때문이다.

---

## 정리

Docker = 컨테이너 플랫폼

Docker Engine = 실제 컨테이너 생성

Docker Desktop = Engine을 쉽게 사용하기 위한 GUI

Image = 컨테이너 설계도

Container = 실행 중인 인스턴스

Docker Hub = 이미지 저장소

Container는 Host OS의 Kernel을 공유하면서

프로세스, 파일시스템, 네트워크를 격리한다.