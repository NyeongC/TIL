## Vue 기본 프로젝트 세팅

### 1. 프로젝트 생성

```bash
mkdir my-vue-app
cd my-vue-app
npm create vue@latest
```

---

### 2. 기본 설정 선택

```bash
◇  Project name (target directory):
│  vue-project
│
◇  Use TypeScript?
│  No
│
◇  Select features to include in your project: (↑/↓ to navigate, space to select, a to 
│  toggle all, enter to confirm)
│  Router (SPA development), Linter (error prevention)
│
◇  Select experimental features to include in your project: (↑/↓ to navigate, space to 
│  select, a to toggle all, enter to confirm)
│  none
│
◇  Skip all example code and start with a blank Vue project?
│  No
```

---

### 3. 실행

```bash
cd vue-project
npm install
npm run dev
```

---

### 4. 실행 확인

```bash
VITE vX.X.X ready

Local:   http://localhost:5173/
```

브라우저에서 접속하여 기본 화면 확인

---

### 5. 체크 포인트

- `npm run dev` 실행 전 반드시 `npm install` 필요
- `package.json` 위치에서 실행해야 함
- `node_modules` 생성 여부 확인

---

### 6. 폴더 구조 (핵심만)

```plaintext
src/
 ├─ views/        # 페이지
 ├─ components/   # 공통 컴포넌트
 ├─ router/       # 라우팅
 ├─ App.vue
 └─ main.js
```

---

### 7. 목적

- Vue 학습이 아니라
- Spring API와 연결하여 전체 흐름 구성
