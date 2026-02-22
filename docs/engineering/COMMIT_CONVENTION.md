# Git Commit Convention (A&I)

A&I는 협업 품질을 위해 **AngularJS 커밋 컨벤션**을 기본 규칙으로 사용합니다.

---

## 1) 커밋 메시지 구조

```txt
<type>(<scope>): <subject>

<body>

<footer>
```

- **type**: 변경 성격(필수)
- **scope**: 변경 영역(선택) — 예: `auth`, `api`, `ui`, `docs`
- **subject**: 변경 요약(필수)
- **body**: 무엇/왜 변경했는지(선택)
- **footer**: Breaking change / 이슈 링크(선택)

---

## 2) Type (필수)

| Type | 설명 |
| :--- | :--- |
| **feat** | 새로운 기능 추가 |
| **fix** | 버그 수정 |
| **docs** | 문서 수정 |
| **style** | 코드 포맷팅, 세미콜론 누락 등 (로직 변경 없음) |
| **refactor** | 코드 리팩토링 (기능 변경 없음) |
| **test** | 테스트 코드 추가/수정 |
| **chore** | 빌드 설정, 패키지 관리 등 (프로덕션 코드 변경 없음) |

---

## 3) Subject 규칙 (필수)

- 50자 이내로 간결하게 작성합니다.
- **명령문**으로 시작하는 것을 권장합니다.
- 끝에 마침표(.)를 찍지 않습니다.
- 영문인 경우 **동사 원형**으로 시작합니다. (예: `feat: Add login function`)

---

## 4) Body (선택)

- **무엇을(What)**, **왜(Why)** 변경했는지 설명합니다.
- 어떻게(How)는 코드에 드러나므로 필요할 때만 작성합니다.

---

## 5) Footer (선택)

- **BREAKING CHANGE**: 호환성 문제가 발생하는 큰 변경이 있을 때 작성합니다.
- **ISSUES**: 관련 이슈를 참조합니다. (예: `Closes #123`)

---

## ✅ Examples

```txt
feat(auth): add JWT refresh flow
fix(api): handle null response from user endpoint
docs(readme): update onboarding steps
refactor(core): extract validation logic
chore(ci): add lint workflow
```

---

## ✅ 좋은 커밋 체크리스트

- [ ] “한 커밋 = 한 목적”인가?
- [ ] 커밋 메시지만 읽어도 변경 의도가 보이는가?
- [ ] 불필요하게 큰 변경(여러 목적)이 섞이지 않았는가?
