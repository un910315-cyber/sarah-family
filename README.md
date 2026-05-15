# OurNest — 우리가족 가계부

가족 전용 가계부 웹앱. Google 로그인으로 화이트리스트된 가족 구성원만 접근 가능.

- **Live**: https://un910315-cyber.github.io/sarah-family/
- **Firebase**: `sarah-family` 프로젝트 (Auth + Firestore)

## 주요 기능

- Google 로그인 + 가족 화이트리스트 인증
- 월별 수입/지출/잔액 집계
- 거래 등록·수정·삭제 (카테고리, 금액, 메모, 한↔태 자동 번역)
- 카테고리별 월 예산 + 진행률 + 도넛/추이 차트
- 회사 일정·특이사항 캘린더
- 고정지출 관리 (월별 자동 발생)
- 가족 구성원 관리 (admin 전용)
- PWA 설치(홈 화면) + 앱 셸 오프라인

## 기술 스택

- 단일 HTML 파일 (CSS/JS 인라인) + manifest + service worker
- Firebase Authentication (Google), Firestore
- Chart.js (CDN) — 도넛/라인 차트
- GitHub Pages 배포
