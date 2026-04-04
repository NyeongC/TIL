# 자주 사용하는 네트워크 명령어

> 네트워크 문제 발생 시 DNS → 네트워크 → 포트 → 서버 → 프로세스 → API 순으로 확인한다.

---

## nslookup → DNS 정상 여부 확인
```bash
nslookup google.com
```
👉 도메인이 어떤 IP로 변환되는지 확인  
👉 DNS 문제인지 빠르게 체크할 때 사용

---

## ping → 서버 네트워크 확인
```bash
ping google.com
```
👉 해당 서버까지 네트워크가 살아있는지 확인  
👉 응답 없으면 네트워크/방화벽 문제 의심

---

## nc → 포트 열려있는지 확인
```bash
nc -zv google.com 80
```
👉 특정 서버의 포트가 열려있는지 확인  
👉 연결 성공하면 포트 정상 (telnet 대신 사용)

---

## netstat / ss → 서버가 LISTEN 상태인지 확인
```bash
netstat -tulnp
```
또는
```bash
ss -tuln
```
👉 서버가 해당 포트를 열고 있는지 확인  
👉 LISTEN 상태 없으면 서버 안 떠있거나 설정 문제

---

## lsof → 해당 포트를 사용하는 프로세스 확인
```bash
lsof -i :8080
```
👉 어떤 프로세스가 포트를 점유하고 있는지 확인  
👉 포트 충돌이나 서버 실행 여부 확인

---

## curl → 실제 API 응답 확인
```bash
curl http://localhost:8080
```
👉 실제 HTTP 요청 보내서 응답 확인  
👉 서버 정상 동작 여부 최종 체크

## 패키지 설치 기본 흐름 (Ubuntu)

### 1. 패키지 목록 최신화
sudo apt update
→ 설치 가능한 패키지 목록을 최신 상태로 갱신

### 2. 패키지 설치

sudo apt install netcat-openbsd
→ netcat(nc) 명령어 사용 가능 (포트/네트워크 테스트용)

sudo apt install dnsutils
→ nslookup, dig 명령어 사용 가능 (DNS 조회용)

※ Ubuntu는 최소 설치 환경이라 기본적으로 많은 명령어가 포함되어 있지 않음
→ 필요한 도구는 직접 설치해야 함

### 💡 개념 정리
- sudo: 관리자(root) 권한으로 명령 실행
- apt: Ubuntu 패키지 관리 도구
- update: 설치 가능한 패키지 목록 갱신
- install: 실제 패키지 설치