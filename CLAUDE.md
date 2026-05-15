# Sarah Family — 우리가족 가계부

가족 전용 가계부 웹앱. 단일 HTML 파일에 모든 기능 내장.

## 배포

- **Remote**: `github.com/un910315-cyber/sarah-family` (master 브랜치)
- **Live**: `https://un910315-cyber.github.io/sarah-family/`
- Push → GitHub Pages 자동 빌드, 약 1~2분 내 반영
- **배포는 항상 Claude가 직접 수행** (수정 → 검증 → commit → push). 사용자에게 푸시 단계를 미루지 말 것.

### ⚠️ PWA 배포 시 필수 절차

**`sw.js` 상단의 `CACHE_VERSION` 문자열을 매번 갱신해야 사용자에게 새 버전이 전달됨.**

```js
const CACHE_VERSION = 'v1-2026-05-15';  // ← 배포할 때마다 변경
```

권장 형식: `vN-YYYY-MM-DD` (예: `v2-2026-05-20`). 하루에 여러 번이면 `v2-2026-05-20-2`처럼 suffix.

**갱신을 빠뜨리면 어떻게 되나**: 가족들 기기에 이미 등록된 SW가 옛 `index.html`을 계속 캐시 응답으로 내보냄. 새 기능을 푸시해도 보이지 않음. SW 파일 내용이 바뀌어야 브라우저가 update를 트리거하므로 `CACHE_VERSION` 한 글자라도 바꾸는 것이 가장 확실.

**index.html / icons / manifest 중 하나라도 수정하면** → `CACHE_VERSION` 갱신 필수.
**이외 파일(README.md, CLAUDE.md 등)만 수정** → 갱신 불필요.

체크리스트:
1. 코드 수정
2. (앱 셸에 영향 있으면) `sw.js`의 `CACHE_VERSION` 갱신
3. `node --check sw.js` + manifest JSON 유효성 확인
4. 한국어 커밋 → push

## 디렉토리

```
/index.html              # 가계부 앱 본체 (단일 HTML, CSS/JS 인라인)
/manifest.webmanifest    # PWA manifest (이름, 아이콘, theme color 등)
/sw.js                   # 서비스 워커 (앱 셸 캐시) — CACHE_VERSION 갱신 주의
/icons/icon-192.png      # PWA 아이콘 (홈 화면, 작은 크기)
/icons/icon-512.png      # PWA 아이콘 (스플래시, 큰 크기)
/README.md               # 사용자용 소개
/CLAUDE.md               # 이 문서
/.gitignore
```

## Firebase 설정

- **Project ID**: `sarah-family`
- **Console**: https://console.firebase.google.com/project/sarah-family/overview
- **사용 서비스**: Authentication (Google), Firestore Database

`firebaseConfig`는 `index.html` 내 `<script type="module">` 상단에 인라인. API key는 공개되어도 보안 영향 없음 (Firebase는 보안 규칙으로 제어).

## Firestore 컬렉션

### `familyMembers/{email}` — 가족 화이트리스트
- `email`: 이메일 (문서 ID와 동일)
- `name`: 표시 이름
- `role`: `admin` | `member`
- `addedAt`, `addedBy`: 메타

**처음 사용 시 본인을 admin으로 등록해야 함.** Firebase Console > Firestore Database > 컬렉션 시작 → `familyMembers` 컬렉션 생성 → 문서 ID = 본인 Gmail, 필드 `email`/`name`/`role: admin` 입력.

### `transactions/{auto-id}` — 거래 내역
- `type`: `income` | `expense`
- `amount`: number (원)
- `category`: 한글 카테고리명
- `date`: `YYYY-MM-DD`
- `monthKey`: `YYYY-MM` (월별 쿼리용 — 인덱스)
- `memo`: string
- `createdAt`, `createdBy`, `updatedAt`, `updatedBy`: 메타

월별 쿼리는 `where('monthKey', '==', 'YYYY-MM')` — Firestore 단일 필드 인덱스 자동 생성.

## Firestore 보안 규칙

Firebase Console > Firestore Database > 규칙 탭에 아래 내용 게시 (현재 코드 기준 권장):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isFamily() {
      return request.auth != null
        && exists(/databases/$(database)/documents/familyMembers/$(request.auth.token.email));
    }
    function isAdmin() {
      return isFamily()
        && get(/databases/$(database)/documents/familyMembers/$(request.auth.token.email)).data.role == 'admin';
    }
    match /familyMembers/{email} {
      allow read: if isFamily();
      allow write: if isAdmin();
    }
    match /transactions/{id} {
      allow read, write: if isFamily();
    }
  }
}
```

⚠️ **부트스트랩 문제**: 본인이 처음 admin 문서를 만들 때는 위 규칙(`isAdmin()`)이 아직 통과하지 못함. 그래서 **첫 admin 등록은 Firebase Console에서 수동으로** 진행. 한 번만 하면 됨.

## 카테고리

코드 내 `CATEGORIES` 상수로 하드코딩. **이중 언어(한/태) 매핑** 구조:

```js
{ ko: '식비', th: 'อาหาร' }
```

DB에는 `ko` 값(예: `'식비'`)만 저장. 화면 표시는 `getCategoryLabel(ko)` 헬퍼가 `'식비 / อาหาร'` 형식으로 변환.

- 지출 11종: 식비, 교통, 생활용품, 의료, 교육, 문화, 통신, 공과금, 주거, 쇼핑, 기타
- 수입 5종: 급여, 용돈, 부수입, 환급, 기타

향후 가족별 커스텀 카테고리 원하면 Firestore에 별도 컬렉션 추가 필요.

## 이중 언어 (한국어 / 태국어)

**모든 UI 텍스트가 한국어와 태국어 동시 표시.** 가족 구성원 중 두 언어 사용자가 섞여 있어서 적용.

- 정적 라벨/버튼/카테고리/요일/월: 코드 내 인라인 (`'수입 / รายรับ'` 또는 `<span class="th">รายรับ</span>` 패턴)
- 사용자 자유 입력(메모)은 그대로 저장·표시 (자동 번역 X — 1단계 결정)
- 폰트: Inter + Noto Sans KR + **Noto Sans Thai** (CDN)
- 알림(showNotif), confirm/alert도 두 언어 병기

향후 자동 번역(MyMemory API 등) 추가하려면 `saveTx` 함수 내 `memo` 처리에 fetch 호출 추가하고 `memoTh` 필드 별도 저장 권장.

## 디자인 톤

다크 테마. accent 컬러는 amber(`#f59e0b`). 수입은 green, 지출은 red.

KGM 성수 사이트와 동일한 모노스페이스(`JetBrains Mono`)로 금액 표시, 한글은 `Noto Sans KR`, 영문은 `Inter`.

**절제된 UI 변경 선호** (사용자 일반 원칙): 화려한 강조 X, 인지될 정도의 최소 변화만.

## 알려진 제약

- 카테고리 커스터마이즈 불가 (코드 수정 필요)
- 다중 통화 미지원 (KRW 고정)
- 이미지/영수증 첨부 미지원 (요구사항: "사진 업로드 없음")
- 가족 인원 제한 없음 (Firestore 비용 무시할 수준)
