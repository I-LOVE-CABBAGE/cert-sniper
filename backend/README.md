# 🖥 backend (Spring Boot)

기능 명세서 v1.4.0 기준 패키지 구조입니다. 패키지 하나 = 기능 하나이며, 담당자는 자기 패키지 안에서만 작업합니다. [← 처음으로](../README.md)

```
📁 backend/
└─ 📁 src/
   ├─ 📁 main/
   │  ├─ 📁 java/com/certsniper/
   │  │  ├─ 📁 global/                공통 (모든 기능이 사용)
   │  │  │  ├─ 📁 config/                보안·CORS·JPA·메일 설정
   │  │  │  ├─ 📁 security/              JWT 발급·검증, 로그인 사용자 정보
   │  │  │  ├─ 📁 exception/             예외 처리 (400 VALIDATION_ERROR · 500)
   │  │  │  ├─ 📁 common/                공통 응답·페이지 형식, 생성·수정 시각
   │  │  │  └─ 📁 util/                  날짜(Asia/Seoul) 등 공통 도구
   │  │  │
   │  │  ├─ 📁 auth/                  회원가입·로그인·이메일 인증·비밀번호 재설정/변경   AUTH-01·02·05·06
   │  │  ├─ 📁 user/                  회원정보 조회·변경, 탈퇴                             AUTH-03·04
   │  │  │
   │  │  ├─ 📁 option/                대학교·전공·학년·직무·관심 분야·자격증 검색 목록     CF-01-01·02·07, CF-02-09
   │  │  ├─ 📁 university/            대학·학부·전공 데이터                                CF-01-01·07
   │  │  ├─ 📁 job/                   희망 직무·관심 분야 데이터                           CF-01-02
   │  │  ├─ 📁 recommendation/        자격증 추천 (전공 맞춤 · 관심 분야 탐색), 추천 조건    CF-01-03~12
   │  │  ├─ 📁 qualification/         자격증 상세·시험과목·응시자격·합격률·트렌드           CF-02-01~09
   │  │  ├─ 📁 examschedule/          시험일정·회차 상태·D-Day 계산                         CF-02-10, CF-03-04
   │  │  │
   │  │  ├─ 📁 favorite/              찜·관심 자격증                                       CF-03-01·02·09
   │  │  ├─ 📁 targetexam/            목표 자격증·목표 시험일                               CF-03-03, CF-04-04·09~11
   │  │  ├─ 📁 planner/               플래너 다이어리 (일정바·당일 일정·개인 일정·6개 제한, 오늘의 일정 패널)   CF-04-03·05~07·12~16·18·19, CF-00-04
   │  │  ├─ 📁 memo/                  자격증별 메모                                        CF-04-01·08·17
   │  │  ├─ 📁 aiplanner/             AI 플래너 (7단계 설문, Qwen2.5 호출, 최종 플랜 저장)  CF-06-01~05
   │  │  │
   │  │  ├─ 📁 passstory/             저격왕 합격수기 (작성·조회·과목별 체감 난이도)        CF-05
   │  │  ├─ 📁 report/                신고 접수·즉시 숨김·단계별 정지                      CF-05, ADMIN-05·06
   │  │  ├─ 📁 rank/                  저격 등급·승급 알림                                  CF-05, ADMIN-08·09
   │  │  │
   │  │  ├─ 📁 support/               공지사항·FAQ·1:1 문의                                CS-01~05
   │  │  └─ 📁 admin/                 관리자 대시보드·회원·공지·문의·신고·저격왕 글 관리   ADMIN-02~11
   │  │
   │  └─ 📁 resources/
   │     └─ 📁 db/migration/          DB 변경 SQL (V1__…sql, Flyway 도입 시 그대로 사용)
   │
   └─ 📁 test/java/com/certsniper/    기능별 테스트 (main과 같은 패키지 구조)
```

## 기능 패키지 안의 구조 (모든 패키지 공통)

```
📁 {기능}/
├─ 📁 controller/    API 입구: 요청을 받고 응답을 돌려줌
├─ 📁 service/       처리 로직: 검증·계산·트랜잭션
├─ 📁 repository/    DB 조회 (Spring Data JPA)
├─ 📁 entity/        DB 테이블과 1:1로 대응하는 클래스
└─ 📁 dto/           요청·응답 형식
```

## 이름 규칙

| 대상 | 규칙 | 예시 |
|---|---|---|
| 패키지 | 소문자 한 단어 | `recommendation`, `passstory` |
| 클래스 | 대문자로 시작, 역할을 뒤에 붙임 | `PassStoryController`, `PassStoryService` |
| 엔티티 | 테이블 이름의 단수형 | `pass_stories` → `PassStory` |
| DTO | 요청 `~Request`, 응답 `~Response` | `SignupRequest`, `RecommendationResponse` |
| API 경로 | `/api/v1/` + 복수형 소문자, 단어 사이 `-` | `/api/v1/pass-stories` |

## 지켜 주세요

* 다른 기능 패키지의 클래스를 고치지 않습니다. 필요하면 담당자에게 이슈로 요청하고, 여러 기능이 같이 쓰는 것은 `global/`에 둡니다
* DB는 `db/migration`의 SQL 파일로만 바꿉니다. 이미 병합된 SQL 파일은 고치지 않고 새 번호 파일을 추가합니다
* DB 계정·SMTP 앱 비밀번호·AI API 키는 환경변수(`.env`)나 `application-local.yml`에 둡니다 (커밋 금지)
* 추천·자격증 로직(`recommendation`, `qualification`, `examschedule`)의 기준 구현은 [etl/](../etl/README.md)에 있습니다. 같은 입력에 같은 결과가 나와야 합니다
