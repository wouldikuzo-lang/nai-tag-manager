# 작가 태그 서가 (NAI 작가 태그 관리 웹앱)

NovelAI 작가 태그를 검색·분류·복사할 수 있는 1인용 웹앱입니다.
Google 로그인(본인 계정만 허용) + Firebase Firestore로 멀티 디바이스 동기화를 지원합니다.

---

## 1. 준비물

- Node.js 18 이상
- Firebase 프로젝트
---

## 2. Firebase 콘솔 설정

### 2-1. 웹 앱 등록 & 설정값 확인
1. [Firebase 콘솔](https://console.firebase.google.com/) → 본인 프로젝트 선택
2. 왼쪽 위 톱니바퀴 → **프로젝트 설정** → 아래로 스크롤 → **내 앱**
3. 웹 앱이 없다면 `</>` 아이콘 클릭 → 앱 닉네임 입력 (예: `nai-tag-web`) → 등록
4. 표시되는 `firebaseConfig` 객체의 값들을 복사해둡니다 (`apiKey`, `authDomain`, `projectId` 등)

### 2-2. Authentication 활성화 (Google 로그인)
1. 왼쪽 메뉴 **Authentication** → **시작하기**
2. **Sign-in method** 탭 → **Google** 클릭 → **사용 설정** 토글 ON
3. 프로젝트 지원 이메일 선택 → **저장**

### 2-3. Firestore 데이터베이스 생성
1. 왼쪽 메뉴 **Firestore Database** → **데이터베이스 만들기**
2. 위치는 `asia-northeast3 (서울)` 추천
3. 처음엔 **테스트 모드**로 시작해도 되지만, 아래 3번 항목의 보안 규칙을 반드시 적용하세요.

---

## 3. 프로젝트 설정

### 3-1. 패키지 설치
```bash
npm install
```

### 3-2. 환경변수 설정
`.env.example` 파일을 복사해서 `.env` 파일을 만들고, Firebase 콘솔에서 복사한 값을 채워주세요.

```bash
cp .env.example .env
```

```env
VITE_FIREBASE_API_KEY=AIza...
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789
VITE_FIREBASE_APP_ID=1:123456789:web:abcdef

# 본인 구글 계정 이메일 (이 이메일로만 로그인 허용)
VITE_ALLOWED_EMAILS=you@gmail.com
```

> `VITE_ALLOWED_EMAILS`는 앱 화면 단(클라이언트)에서의 1차 방어입니다.
> 진짜 보안은 **Firestore 보안 규칙**(4번 항목)이 담당하므로 두 곳 모두 같은 이메일로 맞춰주세요.

### 3-2-1. (선택) 이미지 드래그앤드롭 업로드 설정 — ImgBB
컴퓨터에 있는 이미지 파일을 URL 없이 바로 드래그하거나 클릭해서 올리고 싶다면, 무료 이미지
호스팅 서비스인 ImgBB를 연동하세요. (설정하지 않아도 URL 직접 입력으로는 계속 사용 가능합니다.)

1. [api.imgbb.com](https://api.imgbb.com/) 접속 → 이메일로 무료 가입 (카드 등록 불필요)
2. 로그인 후 표시되는 **API Key** 복사
3. `.env`의 `VITE_IMGBB_API_KEY=` 뒤에 붙여넣기

```env
VITE_IMGBB_API_KEY=여기에_발급받은_키
```

### 3-3. Firestore 보안 규칙에 본인 이메일 반영
`firestore.rules` 파일을 열어 `YOUR_EMAIL@gmail.com` 부분을 본인 구글 이메일로 바꿔주세요.

```js
allow read, write: if request.auth != null
  && request.auth.token.email == "you@gmail.com";
```

여러 계정을 허용하고 싶다면:
```js
allow read, write: if request.auth != null
  && request.auth.token.email in ["you@gmail.com", "second@gmail.com"];
```

---

## 4. 로컬 실행

```bash
npm run dev
```

터미널에 뜨는 주소(기본 `http://localhost:5173`)로 접속하면 로그인 화면이 나옵니다.

---

## 5. Firebase에 배포하기

### 5-1. Firebase CLI 설치 & 로그인 (최초 1회)
```bash
npm install -g firebase-tools
firebase login
```

### 5-2. 프로젝트 연결 (최초 1회)
```bash
firebase use --add
```
→ 목록에서 본인 Firebase 프로젝트 선택 → 별칭은 `default` 정도로 입력

### 5-3. Firestore 보안 규칙 배포
```bash
firebase deploy --only firestore:rules
```

### 5-4. 빌드 후 호스팅 배포
```bash
npm run build
firebase deploy --only hosting
```

배포가 끝나면 `https://your-project.web.app` 형태의 주소가 출력됩니다. 이 주소로 어떤 기기에서든 접속 가능합니다.

> **참고**: Google 로그인 팝업이 배포된 도메인에서 정상 동작하려면, Firebase Authentication → Settings →
> **승인된 도메인**에 `your-project.web.app` / `your-project.firebaseapp.com`이 기본으로 포함되어 있는지 확인하세요
> (보통 자동으로 추가되어 있습니다).

---

## 6. 기능 요약

| 기능 | 설명 |
|---|---|
| Google 로그인 | 본인 계정 이메일만 접근 허용 (클라이언트 화이트리스트 + Firestore 규칙 이중 체크) |
| 실시간 동기화 | Firestore `onSnapshot`으로 여러 기기 간 즉시 반영 |
| 검색 | 태그명 실시간 검색 |
| 즐겨찾기 | ★ 토글, 목록 상단 고정 정렬 |
| 원클릭 복사 | 태그 카드의 복사 아이콘 클릭 시 클립보드 복사 |
| 정렬 | 최신순 / 오래된순 / 알파벳순 / 기타(드래그로 직접 순서 변경, Firestore에 저장됨) |
| 화풍/분류 태그 | 작가 태그당 다중 분류 태그 부여, 상단 칩 클릭으로 다중 필터 (AND 조건) |
| 예시 이미지 3슬롯 | `girl` / `boy` / `빈프롬(None)` 슬롯별 이미지, 스와이프/좌우 화살표로 전환 |
| 전역 슬롯 필터 | 상단 `전체/girl/boy/빈프롬` 토글 선택 시 모든 카드가 해당 슬롯 이미지로 고정 표시 |
| 이미지 업로드 | ImgBB API 키 설정 시 드래그앤드롭/클릭으로 이미지 파일 업로드 가능. 미설정 시 URL 직접 입력만 가능 |
| 작가 태그 탭 | 단일 작가 태그 관리 (이름, 화풍, 예시 이미지, 메모) |
| 그깍(조합) 탭 | 작가 태그 조합 관리 — 메인/부정 프롬프트, Steps·Prompt Guidance·Sampler 설정, Advanced(Rescale·Noise Schedule), 화풍/예시 이미지/즐겨찾기까지 작가 태그와 동일하게 지원. 별도 Firestore 컬렉션(`combos`)에 저장 |

---

## 7. 폴더 구조

```
src/
  components/     화면 구성 요소 (카드, 모달, 필터바, 캐러셀 등)
  hooks/          useAuth(로그인), useTags(Firestore CRUD)
  lib/            firebase.js(초기화), parseNaiMetadata.js(EXIF/PNG 메타데이터 파서)
  styles/         디자인 토큰 + 전체 스타일시트
firestore.rules    Firestore 보안 규칙
firebase.json      Hosting/Firestore 배포 설정
```

---

## 8. 자주 발생할 수 있는 문제

- **로그인 버튼이 비활성화되어 있어요** → `.env`의 Firebase 설정값이 비어있는지 확인하세요.
- **로그인은 되는데 "접근이 허용되지 않았습니다" 메시지가 떠요** → `.env`의 `VITE_ALLOWED_EMAILS`와 로그인한 구글 계정 이메일이 일치하는지 확인하세요.
- **이미지가 안 떠요** → 입력한 이미지 URL이 실제로 브라우저에서 직접 열리는 이미지 주소인지 확인하세요. (구글 드라이브 공유 링크처럼 리다이렉트되는 링크는 안 될 수 있습니다.)

---

## 9. 기존에 이미 배포하셨다면 (업데이트 안내)

업데이트가 진행된 경우, 변경된 파일을 다운로드받아 기존 파일을 교체하고, 업데이트 안내에 따라 `npm run build`→ `firebase deploy --only hosting`로 재배포

기존에 업로드한 내용들은 그대로 남아 있습니다.
