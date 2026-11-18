# 데이터 테이블 정의 (data/*.csv, UTF-8 BOM, 헤더 1행)

CSV 파일 1개 = MySQL 테이블 1개. PK·FK는 아래와 같으며 `tests/test_data_integrity.py`가 검사합니다.

| 테이블 | PK | 컬럼 | FK |
|---|---|---|---|
| schools | school_id | school_name, campus, school_type | |
| majors | major_id | school_id, major_name, degree_course, college_name, parent_name, academic_category, unit_type, discipline(전공 계열 판정), merged_major_ids(합쳐진 동명 코드 ID, `;` 구분), category_large/category_mid/category_small(대·중·소계열) | school_id → schools |
| faculties | faculty_scope_id | school_id, faculty_name, resolution | school_id → schools |
| faculty_majors | (faculty_scope_id, major_id) | | → faculties, → majors |
| qualifications | jmcd | name, qualgbcd/qualgbnm(국가기술·국가전문), seriescd/seriesnm(등급), obligfldcd/obligfldnm(직무분야 대=관심분야), mdobligfldcd/mdobligfldnm(직무분야 중=희망직무), conducting_agency(시행처), related_majors(관련학과), exam_subjects(시험과목), exam_method(검정방법), pass_criteria(합격기준), eligibility(응시자격), acquisition_method(원문 전체), exam_trend(출제경향), field_source(직무분야 출처: QNET / MAPPED / 빈칸), related_ministry(국가전문자격 관련부처, 국가기술은 빈칸), summary(화면용 한 줄 소개), summary_basis(QNET_TEXT / EXAM_INFO), overview(개요), job_description(수행직무), career_outlook(진로 및 전망) | |
| major_qualifications | (major_id, jmcd) | link_rank(학과 안 순서), basis(existing_direct_links/recommended/same_subfield), link_basis, recommendation_type | → majors, → qualifications |
| exam_schedules | (jmcd, exam_stage, exam_start_date, exam_end_date) | exam_year(2026), exam_round, registration_start_date, registration_end_date, pass_announcement_date, source_type(OFFICIAL_ROUNDS/QNET_SCHEDULE_API), source_url, source_note | jmcd → qualifications |
| pass_rates | (jmcd, exam_year, exam_stage) | examinees(실제 응시자), passed, source_type(QNET_OFFICIAL/QNET_STATS_API), source_url | jmcd → qualifications |

## 국가전문자격 직무분야 보완
- Q-Net 원장에서 국가전문자격 100종은 직무분야 코드가 비어 있다. `tools/professional_fields.csv`(자격 계열 seriesnm → 대·중분류 매핑표)로 98종을 채우고 `field_source=MAPPED`. 국가기술자격에 쓰이는 코드만 쓰고, 맞는 중분류가 없으면 대분류만(세무사·관세사 등). 경비지도사 2종은 맞는 분야가 없어 빈칸.
- 관련부처 `related_ministry`: Q-Net 국가자격 종목별 상세정보(crf005)의 국가전문자격 '관련부처' 분류 그대로(18개 분류, 100종 전부). 같은 칸 `tools/professional_fields.csv`의 ministry. 입력값이 아니라 상세 화면 표시용.
- 효과: 관심분야·희망직무 점수와 트렌드 관심분야 필터에 국가전문자격도 잡힘.

## 서비스 범위
- 학부·전문대 과정만. 학교 유형에 '대학원'이 들어가는 기관(일반·특수·전문대학원) 소속 학과와 `degree_course`에 석사·박사가 들어가는 학과(전문대 전문기술석사 포함)는 제외. 범위·제외 수는 `data/MANIFEST.json`의 `service_scope`.
- schools 는 학부 학과가 1개 이상 남는 학교만, faculties 는 그 학교 것만, faculty_majors 는 남는 학과만.
- faculties·faculty_majors 는 화면 입력(대학교 → 전공)에는 쓰지 않고, 전공 목록의 학부 이름 항목을 골랐을 때 소속 전공 합산 추천에만 쓴다.

## 동명 코드 합치기
- 같은 학교·단과대학·학부·이름이면 한 줄. 대표 = 연결 자격증이 가장 많은 코드(같으면 major_id 순), 연결은 합집합. 나머지 ID는 `merged_major_ids` → 서비스는 예전 ID도 대표 ID로 받아준다. DB 이관 시 별도 alias 테이블(major_id_alias: old_major_id → major_id)로 빼면 된다.

## 학과명 보완 규칙
- `major_qualifications.link_basis`가 "학과명 핵심어 'X' 보완 규칙"인 행. 동명 코드를 합친 뒤, 기존 연결이 전혀 없는 학과에만 적용. 규칙 파일 `tools/major_name_rules.csv`(keyword, discipline, mode, exclude, note).

## 동계열 묶음
- 소계열(대·중·소 3단 키) 학과 5개 이상, 학과 절반 이상 공통, 기술사·기능장 제외, 최대 5개 → 빠진 학과에 `basis=same_subfield` 행 추가. 제외 소계열: 군사・국방・안보. 계열 출처: 학과_계열매핑.csv(대학알리미 공식 계열 중심).

## 출처 (수집 파이프라인 결과 파일 → 서비스 테이블)
- schools, majors: `data/serving/majors.csv` + `data/reference/kcue/program_reviews_2026.csv`(선택 제외 판정) + Stage3 `discipline`
- faculties, faculty_majors: `tools/faculty_scope.build_faculty_scopes`
- qualifications: `data/source/qnet/qualification_master.csv` + `data/curated/qualification_profile.csv`
- major_qualifications: 수집 파이프라인 `RecommendationProfileService`가 쓰는 Stage3 `final_build_result`의 `existing_direct_links`·`recommended` 중 613종 안의 것
- exam_schedules: `data/curated/exam_schedules/official_exam_rounds.csv` + `data/curated/qualification_exam_schedules_2026.csv` (같은 종목·단계·날짜는 1행). 국가전문자격은 설명의 `(N회 1차/2차/면접)`을 단계명으로 사용
- pass_rates: `data/curated/pass_rates/qnet_annual_pass_rates.csv` + `data/curated/qualification_statistics_2021_2025.csv`(앞 파일에 없는 연도·단계만)

## qualifications 설명 칸 나누기
- `qualification_profile.csv`의 `acquisition_method` 원문을 항목 이름(시행처·관련학과·시험과목·검정방법·합격기준·응시자격) 기준으로 나눔. 원문은 `acquisition_method`에 그대로 보존.
- 채워진 종목 수(613 중): 시행처 483, 관련학과 294, 시험과목 540, 검정방법 473, 합격기준 480, 응시자격 76. 국가전문자격 상당수는 원문 형식이 달라 비어 있음 → `acquisition_method` 원문 표시.
- 합격기준 뒤에 붙던 '작업형 실기시험 기본정보(안전등급 등)'는 잘라냄. `&middot;` 등 HTML 기호는 문자로 변환.

## 자격증 소개 문구(overview·job_description·career_outlook)
- 출처: 공공데이터포털「한국산업인력공단_국가기술자격 종목별 시험정보」(data.go.kr 3038404, 2025-12-31 기준) → `tools/qualification_texts.csv`.
- 채워진 종목 수(613 중): 개요 391, 수행직무 389, 진로 및 전망 217. 국가기술 105종(원천 내용 비어 있음)과 국가전문 100종은 빈칸.
- 원문은 PDF 줄바꿈 흔적으로 단어 중간 띄어쓰기가 있음(예: '필요 하다').
- `summary`: 화면 카드용 한 줄 소개(예: '소프트웨어의 기획·분석·설계·구현과 정보시스템 관리 능력을 평가하는 자격증이에요.'). 613종 전부 서비스 작성 문구(공식 문구 아님). `summary_basis` = QNET_TEXT(공단 수행직무 근거, 389종) / EXAM_INFO(수행직무 없음 → 시험과목·직무분야 근거, 224종. 개요만 있는 종목은 제정 목적만 담겨 있어 근거로 보지 않음). 공식 문구로 바꿀 때는 `tools/qualification_texts.csv`만 고치고 빌드. 검토 목록: `docs/qualification_summary_review.xlsx`(EXAM_INFO 224종).
