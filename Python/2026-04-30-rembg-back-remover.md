### rembg 라이브러리(배경 제거)

### 기능

이미지에서 사람 또는 객체를 분리하여 배경을 제거하는 라이브러리
딥러닝 모델(U-2-Net)을 사용하여 자동으로 배경을 투명 처리

---

### 구현법

1. 라이브러리 설치

```bash
pip install rembg pillow
pip uninstall rembg -y
pip install rembg[cpu]
```

2. 동작 방식

* 최초 실행 시 AI 모델(약 170MB) 다운로드
* 이후부터는 로컬에 저장된 모델 재사용
* 기본적으로 CPU 기반으로 동작 (GPU 없이도 가능)

3. 실행 코드

```python
from rembg import remove

input_path = 'input.jpg'
output_path = 'output.png'

with open(input_path, 'rb') as i:
    with open(output_path, 'wb') as o:
        o.write(remove(i.read()))

print("완료!")
```

---

### 현재 구조

* 같은 폴더 기준으로 입력/출력 파일 처리

```text
프로젝트 폴더
 ├ main.py
 ├ input.jpg
 ├ output.png
```

---

### 실행 흐름

1. input.jpg 읽기
2. rembg로 배경 제거
3. output.png로 저장

---

### 특징

* CPU만으로 실행 가능하지만 처리 속도는 느림
* 이미지 크기가 클수록 처리 시간 증가
* 최초 실행 시 모델 다운로드로 시간이 오래 걸림
* 이후 실행부터는 속도 개선됨

---

### 한계

* 무료 서버 환경에서는 처리 속도가 더 느림
* 대량 처리 시 워커 분리 및 비동기 구조 필요

---

### 향후 개선 방향

* 이미지 리사이즈 후 처리로 성능 개선
* MQ 기반 워커 구조로 확장
* S3 업로드 연동
* Spring API와 비동기 처리 구조 구성
