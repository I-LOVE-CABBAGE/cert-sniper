# 🔄 저격증 데이터·추천 로직 (etl)

공공데이터로 만든 **서비스용 데이터(CSV 8종)** 와, 그 데이터를 쓰는 **추천·자격증 정보 로직(Python)** 을 담고 있습니다.

* **DB 적재 원천**: `data/*.csv`를 MySQL 테이블에 그대로 적재합니다
* **백엔드 이식 기준**: `jeogyeokjeung/`의 함수 흐름과 `docs/API_CONTRACT.md`의 입출력을 Spring Boot로 옮깁니다
* **결과 검증 기준**: `tests/golden_recommendations.json`과 같은 입력을 넣어 Java 결과가 같은지 비교합니다

Python 3.9 이상, 외부 라이브러리 없음.

---

## 📦 구성

```
📁 etl/
├─ 📄 README.md
├─ 📄 .env.example                      공공데이터포털 인증키 자리 (실제 값은 .env, 커밋 금지)
├─ 📄 run.py                            명령줄 실행 도구 (확인·디버깅용)
│
├─ 📁 jeogyeokjeung/                    추천·자격증 정보 로직 (백엔드 이식 기준)
│  ├─ 📄 __init__.py
│  ├─ 📄 service.py                     선택 목록·추천·자격증 상세·트렌드·D-Day·입력 검증
│  ├─ 📄 eligibility.py                 학교 종류·학년·보유 자격별 응시 가능 시점 판정
│  └─ 📄 store.py                       data/*.csv 로딩 (DB 사용 시 이 모듈만 교체)
│
├─ 📁 data/                             서비스용 테이블 (UTF-8 BOM CSV, DB 적재 원천)
│  ├─ 📄 schools.csv                    학교·캠퍼스
│  ├─ 📄 majors.csv                     학과·전공, 대·중·소계열
│  ├─ 📄 faculties.csv                  학부
│  ├─ 📄 faculty_majors.csv             학부 소속 학과
│  ├─ 📄 qualifications.csv             자격증 613종 (시험정보·한 줄 소개·개요·수행직무)
│  ├─ 📄 major_qualifications.csv       학과 → 자격증 연결과 근거
│  ├─ 📄 exam_schedules.csv             2026 시험일정
│  ├─ 📄 pass_rates.csv                 연도·단계별 응시자·합격자
│  └─ 📄 MANIFEST.json                  행 수·체크섬·빌드 통계
│
├─ 📁 tools/                            데이터 생성 스크립트와 보정표
│  ├─ 📄 build_service_data.py          수집 파이프라인 결과 → data/ 생성
│  ├─ 📄 major_name_rules.csv           학과명 핵심어 보완 규칙
│  ├─ 📄 professional_fields.csv        국가전문자격 직무분야·관련부처 매핑
│  └─ 📄 qualification_texts.csv        자격증 한 줄 소개·개요·수행직무·진로 및 전망
│
├─ 📁 tests/                            테스트
│  ├─ 📄 __init__.py
│  ├─ 📄 test_data_integrity.py         테이블 PK·FK·행 수·체크섬 검사
│  ├─ 📄 test_service.py                추천·상세·트렌드·D-Day·입력 검증
│  └─ 📄 golden_recommendations.json    기준 추천 결과 (Java 이식 결과 비교용)
│
└─ 📁 docs/
   ├─ 📄 DATA_TABLES.md                 테이블 컬럼·PK·FK·출처
   ├─ 📄 API_CONTRACT.md                API 경로·입출력 형식·화면 표시 순서
   └─ 📄 qualification_summary_review.xlsx   한 줄 소개 검토 목록 (EXAM_INFO 224종)
```

---

## 🗂 데이터 (`data/`)

| 파일 | 행 수 | 내용 |
|---|---|---|
| `schools.csv` | 539 | 학교·캠퍼스, 학제 |
| `majors.csv` | 21,730 | 학부·전문대 학과 (폐지·개칭·대학원 제외, 동명 코드 합침), 대·중·소계열 |
| `faculties.csv` / `faculty_majors.csv` | 3,551 / 3,535 | 학부와 소속 학과 (학부 이름으로 고를 때 합산 추천에 사용) |
| `qualifications.csv` | 613 | Q-Net 자격증 (국가기술 513 · 국가전문 100), 시험정보·한 줄 소개·개요·수행직무 |
| `major_qualifications.csv` | 41,707 | 학과 → 자격증 연결과 근거 (직접 연결 · 전공 연관 · 동계열 공통) |
| `exam_schedules.csv` | 3,041 | 2026 시험일정 (회차·단계별 접수·시험·발표일) |
| `pass_rates.csv` | 13,016 | 연도·시험 단계별 실제 응시자·합격자 |

테이블 컬럼과 PK·FK는 [docs/DATA_TABLES.md](docs/DATA_TABLES.md)에 있습니다.

### 원천 데이터

| 원천 | 쓰인 곳 |
|---|---|
| 한국대학교육협의회 대학·학과 정보 (+ 학교 공식 홈페이지 검증) | schools, majors, faculties |
| 한국산업인력공단 Q-Net 국가자격 종목·상세정보 | qualifications |
| 한국산업인력공단 시험일정 (Q-Net 공식 회차표 + 공공데이터 API) | exam_schedules |
| 한국산업인력공단 검정현황·통계 (Q-Net 공식 + 공공데이터 API) | pass_rates |
| 공공데이터포털 「국가기술자격 종목별 시험정보」 | qualifications 개요·수행직무·진로 및 전망 |
| 대학알리미 학과 계열 분류 | majors 대·중·소계열, 동계열 공통 자격 |

공공데이터 API 수집과 학과 검증은 **별도 수집 파이프라인**에서 실행하고, 이 폴더에는 그 결과를 정리한 CSV만 둡니다.

---

## ⚙️ 추천 로직 요약

1. **후보**: 선택한 전공에 연결된 자격증 (`major_qualifications`). 학부 이름을 고르면 소속 학과 연결을 합산
2. **응시 시점**: 학교 종류·학년·등급으로 지금 가능 / N학년부터 / 졸업·경력 후 / 개별 확인 / 제한 없음 판정
3. **점수**: 학과 연결(+30~40) · 희망 직무(+30) · 관심 분야(+20) · 응시 시점(0~+20) · 올해 남은 시험(+10) · 등급 적합(+5 / −20)
4. **결과**: 전공 맞춤 추천·관심 분야 탐색 후보(화면에는 갈래별 상위 5개), 카드 정보. 학년별 로드맵·보유 자격 상위 등급도 계산하지만 화면 적용은 미정

자세한 입출력은 [docs/API_CONTRACT.md](docs/API_CONTRACT.md)에 있습니다.

---

## 🔧 활용

```bash
python run.py check                                  # 데이터 무결성 + 전체 테스트
python run.py recommend profile.json                 # 추천 결과 JSON (입력 형식: docs/API_CONTRACT.md 2장)
python run.py qualification 1320                     # 자격증 상세
python run.py options majors --school-id <학교 ID>   # 선택 목록
```

### 데이터를 다시 만들 때

```bash
python tools/build_service_data.py --source <수집 파이프라인 경로>
python run.py check
```

* 골든 결과(`tests/golden_recommendations.json`)가 달라지면 의도한 변경인지 확인한 뒤 갱신합니다
* 자격증 한 줄 소개를 고칠 때는 `tools/qualification_texts.csv`의 `summary`만 바꾸고 다시 빌드합니다

---

## ⚠️ 알아둘 점

* 추천의 응시 시점은 **진로 안내**이며 개인 응시자격 확정이 아닙니다 (`personal_eligibility_verified: false`)
* 합격률은 **실제 응시자 기준**입니다 (접수자 아님)
* 자격증 한 줄 소개(`summary`)는 서비스가 작성한 문구입니다. 공단 수행직무 근거(`QNET_TEXT`, 389종)와 시험과목·직무분야 근거(`EXAM_INFO`, 224종)로 구분하며, `EXAM_INFO`는 [docs/qualification_summary_review.xlsx](docs/qualification_summary_review.xlsx)로 검토합니다
* CSV는 엑셀에서 바로 열 수 있게 UTF-8 BOM으로 저장합니다
