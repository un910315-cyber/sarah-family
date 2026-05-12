# Sarah Family — 우리가족 가계부

가족 전용 가계부 웹앱. Google 로그인으로 화이트리스트된 가족 구성원만 접근 가능.

- **Live**: https://un910315-cyber.github.io/sarah-family/
- **Firebase**: `sarah-family` 프로젝트 (Auth + Firestore)

## 주요 기능

- Google 로그인 + 가족 화이트리스트 인증
- 월별 수입/지출/잔액 집계
- 거래 등록·수정·삭제 (카테고리, 금액, 메모)
- 가족 구성원 관리 (admin 전용)

## 기술 스택

- 단일 HTML 파일 (CSS/JS 인라인)
- Firebase Authentication (Google)
- Firebase Firestore (`familyMembers`, `transactions` 컬렉션)
- GitHub Pages 배포
