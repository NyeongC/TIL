# 쿠버네티스 첫 오브젝트 정리

## Namespace

쿠버네티스 안의 논리적 공간(폴더)

Deployment, Service, ConfigMap 등의 오브젝트를 묶어서 관리한다.

예시

anotherclass-123

---

## Deployment

Pod를 생성하고 관리하는 오브젝트

- Pod 생성
- Pod 개수 유지
- 무중단 배포(Rolling Update)
- 장애 시 Pod 재생성

실무에서는 직접 Pod를 만들기보다 Deployment를 사용한다.

---

## ReplicaSet

Deployment가 내부적으로 사용하는 오브젝트

설정한 Pod 개수를 유지한다.

예시

replicas: 2

→ Pod가 죽으면 다시 생성해서 항상 2개 유지

---

## Pod

쿠버네티스의 최소 실행 단위

실제 컨테이너(Docker Container)가 실행되는 곳

쉽게 말하면

서버 프로세스가 실제로 떠있는 공간

---

## Service

Pod로 트래픽을 연결하는 오브젝트

Pod는 생성/삭제될 때 IP가 변경될 수 있기 때문에

사용자는 Service를 통해 접근한다.

예시

브라우저
↓
Service
↓
Pod

---

## ConfigMap

일반 설정값 저장소

환경변수나 설정파일을 관리한다.

예시

spring profile
application role

등

---

## Secret

민감정보 저장소

비밀번호, 토큰, 인증서 등을 저장한다.

예시

DB 계정
DB 비밀번호
JWT Secret Key

---

## PV (Persistent Volume)

실제 저장공간

노드의 디스크와 연결된다.

예시

/root/k8s-local-volume/1231

---

## PVC (Persistent Volume Claim)

PV 사용 신청서

Pod는 PV를 직접 사용하지 않고 PVC를 통해 연결한다.

흐름

Pod
↓
PVC
↓
PV
↓
디스크

---

## HPA (Horizontal Pod Autoscaler)

Pod 자동 확장 기능

CPU 사용량 등을 보고 Pod 개수를 자동으로 늘리거나 줄인다.

예시

Pod 2개
↓
트래픽 증가
↓
Pod 4개

---

# 전체 흐름

Namespace
 ├─ Deployment
 │   └─ ReplicaSet
 │        └─ Pod(실제 컨테이너)
 │
 ├─ Service
 │      └─ Pod 연결
 │
 ├─ ConfigMap
 │      └─ 설정값 제공
 │
 ├─ Secret
 │      └─ 민감정보 제공
 │
 ├─ PVC
 │      └─ PV 연결
 │            └─ 실제 디스크
 │
 └─ HPA
        └─ Pod 자동 확장