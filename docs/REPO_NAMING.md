# Repository Naming Guide (A&I)

이 문서는 A&I Organization에서 레포지토리를 **일관된 규칙**으로 관리하기 위한 네이밍 가이드입니다.

> ✅ 목표  
> - 레포 이름만 보고도 “기수/용도/팀/서비스”가 바로 식별되게  
> - 검색/정렬/자동화(CI, 템플릿 적용 등) 하기 쉽게

---

## 0) 공통 규칙

- 기본 포맷은 **대문자 + 하이픈(-)** 을 사용합니다.
- `<Period>`, `<Team>`, `<Service>` 값은 **한 번 정하면 기수 동안 고정**합니다.
- 레포명은 “짧고 명확하게”를 우선합니다.

`<Period>` 표기 예시(택 1, 조직 내 통일)
- `4TH`
- `2026-4TH`
- `2026-S1` / `2026-S2`

---

## 1) Code Lab (실습/과제)

### ✅ 규칙
- `A-AND-I-<Period>-CODE-LAB`

### ✅ 의미
- 기초 문법 / 알고리즘 / 트랙별 실습 / 과제 / 예제 코드를 **한 곳에 모아두는 레포**

### ✅ 예시
- `A-AND-I-4TH-CODE-LAB`
- `A-AND-I-2026-4TH-CODE-LAB`

---

## 2) Projects (팀 프로젝트)

### ✅ 규칙 (둘 중 하나)
- 팀 단위: `A-AND-I-<Period>-PROJECT-<Team>`
- 서비스 단위: `A-AND-I-<Period>-PROJECT-<Service>`

### ✅ 예시
- `A-AND-I-4TH-PROJECT-TEAM-A`
- `A-AND-I-4TH-PROJECT-TEAM-B`
- `A-AND-I-4TH-PROJECT-ATTENDANCE`
- `A-AND-I-4TH-PROJECT-FOOD-RECOMMENDER`

> `<Team>` / `<Service>` 값은 가능하면 **짧고 유니크하게** 추천합니다.

---

## 3) Docs (가이드/규칙)

문서/가이드는 레포 네이밍 규칙 대신, 폴더 구조로 관리합니다.

- `docs` : 규칙/가이드/회고 아카이브
- `.github` : Organization 프로필 및 공통 템플릿

---

## 4) (선택) GitHub Topics 권장

검색성과 가시성을 위해 Topics를 붙이는 것을 권장합니다.

추천 Topics 예시:
- `a-and-i`, `aandi`
- `code-lab`, `assignments`, `study`
- `project`, `team-project`
- `period-4th` (또는 `cohort-4`)
