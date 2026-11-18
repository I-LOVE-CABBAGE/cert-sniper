# 🎨 frontend (React)

기능 명세서 v1.4.0의 화면 ID(VIEW-01~19) 기준 구조입니다. 화면 하나 = `pages/` 폴더 하나입니다. [← 처음으로](../README.md)

```
📁 frontend/
└─ 📁 src/
   ├─ 📁 api/                       백엔드 호출 (backend 기능 패키지 하나에 파일 하나)
   ├─ 📁 components/
   │  ├─ 📁 common/                 여러 화면이 같이 쓰는 부품 (헤더·푸터·버튼·모달·검색 선택·칩 선택)
   │  ├─ 📁 qualification/          자격증 부품 (추천 카드·합격률 그래프·시험일정 표·검색 결과 팝업)
   │  ├─ 📁 planner/                플래너 부품 (타임라인·일정바·마일스톤·일정/메모 팝업·작은 캘린더)
   │  └─ 📁 passstory/              저격왕 부품 (합격수기 카드·등급 배지·신고 팝업)
   ├─ 📁 pages/
   │  ├─ 📁 Main/                   VIEW-01  초기 메인 (로그인 전·후, 오늘의 일정 패널)
   │  ├─ 📁 Auth/                   VIEW-02  회원가입·가입 완료 팝업
   │  │                             VIEW-03  로그인
   │  │                             VIEW-04  비밀번호 재설정
   │  ├─ 📁 Recommend/              VIEW-05  자격증 추천
   │  ├─ 📁 QualificationDetail/    VIEW-06  자격증 상세 (검색·조회)
   │  ├─ 📁 Planner/                VIEW-07  플래너 다이어리
   │  ├─ 📁 AiPlanner/              VIEW-08  AI 플래너 생성 (설문·로딩)
   │  ├─ 📁 PassStory/              VIEW-09  저격왕 게시판
   │  │                             VIEW-10  합격수기 작성·수정·상세·신고
   │  ├─ 📁 MyPage/                 VIEW-11  마이페이지 (회원정보·비밀번호·관심 자격증·내 글·신고내역·탈퇴)
   │  ├─ 📁 Support/                VIEW-12  공지사항·FAQ
   │  │                             VIEW-13  1:1 문의
   │  └─ 📁 Admin/                  VIEW-14~19  관리자 (대시보드·회원·공지·문의·신고·저격왕 글)
   ├─ 📁 router/                    화면 경로, 로그인 필요 화면 보호
   ├─ 📁 hooks/                     여러 화면에서 쓰는 로직 (로그인 상태·검색 입력 지연)
   ├─ 📁 store/                     전역 상태 (로그인 정보)
   ├─ 📁 utils/                     날짜·숫자 형식, D-Day 표시
   ├─ 📁 constants/                 고정 값 (학년 목록·자격구분·등급)
   └─ 📁 styles/                    공통 스타일
```

## 기능 하나를 만들 때 파일이 놓이는 곳

| 위치 | 넣는 것 | 예 (자격증 추천) |
|---|---|---|
| `pages/{화면}/` | 화면 파일 `~Page.jsx` | `pages/Recommend/RecommendPage.jsx` |
| `components/{기능}/` | 그 화면의 부품 | `components/qualification/RecommendCard.jsx` |
| `api/` | 백엔드 호출 `~Api.js` | `api/recommendationApi.js` |

## 이름 규칙

| 대상 | 규칙 | 예시 |
|---|---|---|
| 화면 폴더 | 대문자로 시작 | `Recommend/`, `PassStory/` |
| 화면 파일 | `~Page.jsx` | `RecommendPage.jsx` |
| 부품 파일 | 대문자로 시작, 역할이 보이게 | `PassRateChart.jsx` |
| API 파일 | 소문자로 시작, `~Api.js` | `qualificationApi.js` |
| 훅 | `use`로 시작 | `useAuth.js` |

## 지켜 주세요

* 다른 기능의 화면·부품을 고치지 않습니다. 공통 부품이 필요하면 `components/common/`에 새로 만들고 PR에 적어 주세요
* 백엔드 주소 같은 설정은 `.env`에 두고, 공유할 예시는 `.env.example`로 올립니다
