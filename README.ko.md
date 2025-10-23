# React Firebase 배포 스킬

React + Firebase 풀스택 애플리케이션을 자동으로 생성하고 배포하는 Claude Code 스킬입니다.

## 개요

이 스킬은 다음을 완전 자동화합니다:
- React + Vite 프로젝트 생성
- Firebase 백엔드 인프라 구축
- Firestore 데이터베이스 및 보안 규칙 설정
- Firebase Hosting 배포
- 인증 구현 (이메일/비밀번호, Google 로그인)

**시간 절약**: 수동 설정 2-3시간 → 자동화로 10-15분

## 설치 방법

### Claude Code Plugin Marketplace에서 설치

```bash
# 1. Marketplace 추가
/plugin marketplace add jkf87/react-firebase-skill

# 2. 플러그인 설치
/plugin install react-firebase

# 3. Claude Code 재시작 (필요시)
# Cmd+Shift+P (Mac) 또는 Ctrl+Shift+P (Windows/Linux)
# "Developer: Reload Window" 선택
```

### 로컬에서 직접 설치

```bash
# 1. 저장소 클론
git clone https://github.com/jkf87/react-firebase-skill.git
cd react-firebase-skill

# 2. Claude Code 스킬 디렉토리에 복사
mkdir -p ~/.claude/skills/react-firebase
cp -r . ~/.claude/skills/react-firebase/

# 3. 설치 확인
/plugin list
```

### 설치 확인

스킬이 제대로 설치되었는지 확인:

```bash
# Claude Code에서
/plugin list

# 출력에서 "react-firebase" 확인
# ✓ react-firebase (3.2.0) - enabled
```

## 빠른 시작

### 사용 방법

다음과 같이 요청하면 스킬이 자동으로 실행됩니다:

**한국어**:
```
"React와 Firebase로 todo 앱 만들어줘"
"Firebase 백엔드 사용해서 채팅 앱 만들어줘"
"Firestore 데이터베이스 포함한 React 프로젝트 생성해줘"
"firebase skill을 사용해서 블로그 앱 만들어줘"
```

**English**:
```
"Create a React app with Firebase backend"
"Build a todo app using React and Firestore"
"Set up a React + Firebase project with authentication"
```

### 자동 실행 과정

스킬은 다음을 자동으로 수행합니다:

1. **Firebase 프로젝트 생성** (CLI 자동 실행)
   ```bash
   npx firebase projects:create your-app-1729680000
   ```

2. **Firebase 설정 추출** (실제 API 키 받아옴)
   ```bash
   npx firebase apps:sdkconfig WEB <app-id>
   ```

3. **React 프로젝트 생성** (Vite + Tailwind CSS)
   ```bash
   npm create vite@latest your-app -- --template react
   ```

4. **Firebase 통합** (실제 설정값으로)
   - `.env` 파일에 실제 API 키 작성
   - Firebase config 파일 생성
   - Firestore 보안 규칙 배포

5. **Firestore 배포**
   ```bash
   npx firebase deploy --only firestore
   ```

6. **Hosting 배포**
   ```bash
   npm run build
   npx firebase deploy --only hosting
   ```

7. **라이브 URL 제공**
   ```
   ✅ 배포 완료: https://your-app-1729680000.web.app
   ```

## 주요 기능

### 완전 자동화

- ✅ Firebase CLI 자동 설치 및 로그인 확인
- ✅ Firebase 프로젝트 자동 생성 (항상 새 프로젝트)
- ✅ Web 앱 자동 등록
- ✅ **실제 API 키 자동 추출** (플레이스홀더 없음!)
- ✅ React 프로젝트에 Firebase 설정 자동 통합
- ✅ Firestore 보안 규칙 자동 배포
- ✅ Firebase Hosting 자동 배포
- ✅ **한 번의 명령으로 완성**

### 포함된 기술 스택

- **프론트엔드**: React 19 + Vite 7
- **스타일링**: Tailwind CSS 3
- **백엔드**: Firebase Firestore
- **인증**: Firebase Authentication
- **호스팅**: Firebase Hosting
- **보안**: 사용자별 데이터 격리 규칙

### 생성되는 컴포넌트

스킬이 자동으로 생성하는 완전한 구현:

- 🔐 **인증 컴포넌트**
  - `Login.jsx` - 이메일/비밀번호 + Google 로그인
  - `SignUp.jsx` - 사용자 등록
  - `Header.jsx` - 네비게이션 + 로그아웃

- 📦 **레이아웃 컴포넌트**
  - 반응형 헤더
  - 인증 상태 관리

- 🔌 **커스텀 훅**
  - `useTodos.js` - Firestore CRUD 작업
  - 실시간 데이터 동기화
  - 에러 처리

- ⚙️ **Firebase 설정**
  - `src/firebase/config.js` - Firebase 초기화
  - 환경 변수 관리
  - Firestore, Auth 내보내기

## 워크플로우

### Workflow 1: 새 프로젝트 생성

**빈 디렉토리에서 시작 → 라이브 URL까지 완전 자동화**

```
User: "Firebase로 todo 앱 만들어줘"

Claude:
1. 프로젝트 이름 물어봄
2. Firebase CLI 확인/설치
3. Firebase 프로젝트 생성 ← 먼저!
4. Web 앱 등록
5. Firebase config 받기
6. React 프로젝트 생성 ← config 준비된 상태
7. 의존성 설치
8. .env 파일에 실제 값 작성
9. Firebase 설정 파일 생성
10. Firestore 보안 규칙 배포
11. 컴포넌트 구현 (App, Login, Header 등)
12. 빌드 및 Hosting 배포

결과: https://todo-app-1729680000.web.app ✅
```

**소요 시간**: 10-15분 (자동)

### Workflow 2: 기존 프로젝트에 Firebase 추가

기존 React 앱에 Firebase 백엔드 통합:

```bash
# 기존 React 프로젝트 디렉토리에서
"이 프로젝트에 Firebase 백엔드 추가해줘"
```

스킬이 자동으로:
- Firebase 프로젝트 생성
- Firebase SDK 설치
- Firestore 설정
- 인증 컴포넌트 추가
- 배포

**소요 시간**: 5-10분

### Workflow 3: TypeScript 변환

JavaScript 구현을 TypeScript로 마이그레이션:

```bash
"이 프로젝트를 TypeScript로 변환해줘"
```

**소요 시간**: 15-30분

## 실제 사용 예시

### Todo 앱 만들기

```
User: "Firebase 사용해서 todo list 앱 만들어주세요. 백엔드, db, 배포 전부 처리해주세요"

Claude:
✅ npx firebase projects:create todo-app-1729680000 --display-name "Todo App"
✅ npx firebase apps:create WEB "Todo App Web"
✅ npx firebase apps:sdkconfig WEB 1:xxx:web:xxx
   → API 키 받아옴
✅ npm create vite@latest todo-app -- --template react
✅ npm install firebase
✅ Write(.env) with real API keys
✅ Write(src/firebase/config.js) with actual config
✅ Write(src/App.jsx) with todo CRUD logic
✅ npx firebase deploy --only firestore:rules
✅ npm run build
✅ npx firebase deploy --only hosting

Result: https://todo-app-1729680000.web.app
```

### 채팅 앱 만들기

```
User: "실시간 채팅 앱 만들어줘, Firebase 사용해서"

Claude:
✅ Firebase 프로젝트 생성
✅ React 프로젝트 생성
✅ 실시간 메시지 컴포넌트 구현
✅ Firestore 실시간 리스너 설정
✅ 배포

Result: 라이브 채팅 앱
```

## Firebase-First 워크플로우 (v3.2.0)

### 왜 Firebase를 먼저 만드나요?

**이전 방식 (비효율적)**:
```
1. React 프로젝트 생성 (더미 코드)
2. Firebase 프로젝트 생성
3. Firebase config 받기
4. 더미 파일 덮어쓰기 ← 낭비!
```

**현재 방식 (효율적)**:
```
1. Firebase 프로젝트 생성 ← 먼저!
2. Firebase config 받기
3. React 프로젝트 생성 ← config 준비된 상태
4. .env 파일 한 번에 작성 ← 덮어쓰기 없음!
```

**장점**:
- ✅ 파일 작업 50% 감소
- ✅ 플레이스홀더 값 없음 (처음부터 실제 값)
- ✅ 더 빠른 실행
- ✅ 더 깔끔한 워크플로우

## 핵심 패턴

### Vite 환경 변수

```javascript
// ✅ 올바른 방법 (Vite)
const apiKey = import.meta.env.VITE_FIREBASE_API_KEY;

// ❌ 잘못된 방법 (Node.js 패턴)
const apiKey = process.env.VITE_FIREBASE_API_KEY;
```

### Firestore 타임스탬프

```javascript
import { serverTimestamp } from 'firebase/firestore';

// ✅ 올바른 방법 - 서버 시간 (타임존 안전)
await addDoc(collection(db, 'todos'), {
  createdAt: serverTimestamp()
});

// ❌ 잘못된 방법 - 클라이언트 시간 (타임존 문제)
await addDoc(collection(db, 'todos'), {
  createdAt: new Date()
});
```

### 실시간 리스너 정리

```javascript
// ✅ 항상 unsubscribe로 메모리 누수 방지
useEffect(() => {
  const unsubscribe = onSnapshot(q, (snapshot) => {
    setData(snapshot.docs.map(doc => doc.data()));
  });

  return () => unsubscribe(); // 필수!
}, []);
```

### 사용자별 데이터 격리

```javascript
// ✅ 항상 userId 포함 (보안 규칙 적용)
await addDoc(collection(db, 'todos'), {
  text: 'Buy milk',
  userId: user.uid,  // 필수!
  createdAt: serverTimestamp()
});
```

## 보안 규칙

스킬이 자동으로 적용하는 보안 규칙:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{collection}/{document} {
      // 인증된 사용자만 자신의 데이터 읽기
      allow read: if request.auth != null &&
                     resource.data.userId == request.auth.uid;

      // 자신의 데이터만 생성
      allow create: if request.auth != null &&
                       request.resource.data.userId == request.auth.uid;

      // 자신의 데이터만 수정/삭제
      allow update, delete: if request.auth != null &&
                               resource.data.userId == request.auth.uid;
    }
  }
}
```

**효과**:
- ✅ 사용자 A는 사용자 B의 데이터를 볼 수 없음
- ✅ 인증되지 않은 사용자는 아무것도 할 수 없음
- ✅ 모든 컬렉션에 적용

## 문제 해결

### Tailwind CSS 작동 안 함

**증상**: 스타일이 적용되지 않음

**원인**: Tailwind v4 설치됨 (CLI 호환성 문제)

**해결**: 스킬이 자동으로 v3 설치
```bash
npm install -D tailwindcss@^3
```

### Firestore API 활성화 안됨

**증상**: `HTTP Error: 403, Cloud Firestore API has not been used`

**해결**: `firebase deploy --only firestore` 실행하면 자동 해결

### 인증 설정 안됨

**증상**: `auth/configuration-not-found`

**원인**: Firebase Console에서 인증 제공업체 활성화 필요

**해결**: 스킬이 제공하는 직접 링크 사용
```
🔗 https://console.firebase.google.com/project/<project-id>/authentication/providers
```

**수동 작업 시간**: 2-3분

### Google 로그인 팝업 차단

**해결**: 브라우저에서 localhost와 Firebase 도메인 팝업 허용

## 시간 비교

| 작업 | 수동 설정 | 이 스킬 사용 |
|------|----------|-------------|
| 프로젝트 생성 | 30분 | 2분 |
| Firebase 설정 | 45분 | 3분 (자동) |
| Firestore 설정 | 30분 | 2분 (자동) |
| 인증 구현 | 60분 | 5분 |
| 배포 설정 | 30분 | 2분 (자동) |
| **총 시간** | **2-3시간** | **10-15분** |

**절약 시간**: 최소 1.5시간

## 의존성

테스트된 버전:
- Node.js 18+
- React 19
- Vite 7
- Firebase SDK 12
- Firebase CLI 14+
- Tailwind CSS 3

## 버전 히스토리

### 최신 버전: 3.2.0 (2025-10-23)

**주요 개선**:
- 🚀 Firebase-First 워크플로우
- ✅ 파일 덮어쓰기 제거 (효율성 2배)
- ✅ 항상 새 프로젝트 생성
- ✅ 실제 API 키 처음부터 작성

**이전 버전**:
- **3.1.0** - 첫 프롬프트에서 스킬 트리거 수정
- **3.0.0** - 명령형 워크플로우 형식 (실제 실행)
- **2.0.0** - 공식 스킬 형식 적용
- **1.0.0** - 초기 릴리스

자세한 내용: [CHANGELOG.md](CHANGELOG.md)

## 제한사항

### Firebase Authentication 수동 활성화

**문제**: Firebase CLI로 Authentication 제공업체를 자동 활성화할 수 없음 (OAuth2 요구사항)

**해결**:
1. 스킬이 자동으로 프로젝트 생성
2. 스킬이 직접 링크 제공:
   ```
   🔗 https://console.firebase.google.com/project/<project-id>/authentication/providers
   ```
3. 링크 클릭 → Email/Password 활성화 → 저장
4. 스킬이 자동으로 재배포

**수동 작업 시간**: 2-3분만 필요

## 라이선스

MIT License - 자유롭게 사용, 수정, 배포 가능

## 기여

이 스킬은 [Anthropic Skill Best Practices](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/best-practices)를 따릅니다:
- 간결성 (Claude의 기존 지식 제거)
- 점진적 공개 (메인 파일에서 상세 가이드 참조)
- 명확한 활성화 트리거
- 각 워크플로우의 검증 체크리스트

## 지원

문제 또는 질문:
1. `SKILL.md` 확인 (완전한 워크플로우)
2. `firebase-cli.md` 확인 (CLI 명령 참조)
3. `component-patterns.md` 확인 (구현 세부사항)
4. `guides/` 디렉토리의 관련 가이드 확인

## 링크

- **저장소**: https://github.com/jkf87/react-firebase-skill
- **이슈**: https://github.com/jkf87/react-firebase-skill/issues
- **문서**: [SKILL.md](SKILL.md)
- **변경사항**: [CHANGELOG.md](CHANGELOG.md)

---

**Made with** ❤️ **for rapid React + Firebase development**
