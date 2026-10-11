# GitHub 협업 실전 가이드 (A&I)

> **대상:** A&I의 웹·모바일·백엔드·AI 트랙 구성원  
> **목적:** Issue → Branch → Commit → PR → Review → Merge의 흐름을 실제 작업에 적용합니다.  
> **성격:** 입문자를 위한 실습 문서입니다. 세부 규칙이 충돌할 경우 [PR Guide](PR_GUIDE.md), [PR & Code Review Guide](PR_REVIEW_GUIDE.md), [Commit Convention](COMMIT_CONVENTION.md)을 우선합니다.

## 1. 협업 흐름 한눈에 보기

```text
Issue 등록 → 작업 범위/완료 기준 확인 → 작업 브랜치 생성
→ 구현 및 검증 → Commit & Push → PR 작성 → Code Review
→ 피드백 반영 → 재검토 → Merge → Issue 상태 확인
```

- **Issue**는 왜, 무엇을 해야 하는지 정의합니다.
- **PR**은 무엇이 바뀌었고 어떻게 확인했는지 설명합니다.
- **Code Review**는 사람이 아니라 코드와 시스템을 함께 개선하는 대화입니다.
- **Merge**는 리뷰/검증 및 프로젝트별 병합 조건을 확인한 뒤 진행합니다.

> 화면 예시는 특정 프로젝트에서 촬영되었으므로 계정명, 브랜치명, 버튼 배치가 실제 저장소와 다를 수 있습니다. 이 문서에서 강조하는 것은 화면의 특정 프로젝트명이 아니라 GitHub 사용 방법입니다.

## 2. Issue: 작업과 완료 기준 정하기

저장소의 **Issues → New issue**에서 제목과 설명을 작성합니다. Assignees는 담당자, Labels는 분류 태그, Type은 제공되는 경우 작업 유형을 뜻합니다. Type을 반드시 채워야 하는 것은 아닙니다.

**예시**

```markdown
## 설명
로그인 성공 이후 사용자 정보가 화면에 반영되도록 수정합니다.

## 할 일
- [ ] 로그인 성공 시 사용자 상태 갱신
- [ ] 실패 시 오류 메시지 표시
- [ ] 성공/실패 시나리오 검증

## 참고
- 관련 디자인/명세 링크
```

할 일은 '테스트 개선'처럼 모호하게 쓰기보다 **완료 여부를 직접 확인할 수 있는 문장**으로 작성합니다. 기능·버그·문서 변경 등 모든 트랙에서 동일합니다.

## 3. Branch와 Commit

작업 전에 기준 브랜치를 최신화합니다. Fork 방식과 단일 저장소 방식은 서로 다릅니다.

```bash
# 단일 저장소 협업의 예시
git switch main
git pull --ff-only origin main
git switch -c feature/login-state
```

Fork를 사용하는 팀은 자신의 `origin`과 원본 `upstream`을 구분하고, **각 프로젝트의 실제 기준 브랜치**를 확인하세요. `develop`을 쓰는 팀은 PR의 base가 반드시 `main`인 것은 아닙니다.

A&I 권장 브랜치명은 `feature/<topic>`, `fix/<bug>`, `docs/<topic>`, `refactor/<topic>`입니다. 이슈 번호를 이름에 추가하는 별도 프로젝트 규칙이 있다면 해당 팀 규칙을 따릅니다.

```bash
git status
git diff
git add <변경한-파일>
git commit -m "feat(auth): add login state handling"
git push -u origin feature/login-state
```

**커밋 형식:** `<type>(<scope>): <subject>` (`scope` 생략 가능). `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore` 등은 [공통 커밋 규칙](COMMIT_CONVENTION.md)에 따릅니다. Commit 전에 비밀 키, 토큰, `.env`, 불필요한 파일이 포함되지 않았는지 확인합니다.

## 4. Pull Request 작성하기

**Compare**는 작업 브랜치, **Base**는 병합 대상 브랜치입니다. `Able to merge`는 자동 병합 충돌이 없다는 뜻이지 승인 완료를 의미하지 않습니다. 이미 연결된 PR이 있으면 `View pull request`가 표시될 수 있습니다.

A&I 공통 템플릿은 **변경 내용 (What)**, **확인 방법 (How to check)**, **체크리스트**, **참고 (선택)** 순서입니다.

```markdown
## 변경 내용 (What)
- 로그인 화면의 오류 상태를 정리했습니다.

## 확인 방법 (How to check)
- 테스트 계정으로 성공/실패 시나리오를 확인했습니다.
- 자동 테스트는 실행하지 않았습니다 (이유: ...).

## 체크리스트
- [x] PR 목적이 하나입니다
- [x] 변경 범위를 최소화했습니다
- [ ] 필요한 문서/가이드를 업데이트했습니다

## 참고 (선택)
- 관련 이슈: #19
```

실제로 수행하지 않은 테스트를 '통과'했다고 작성하지 않습니다. 필요할 경우 다음 증거를 첨부하세요.

| 트랙 | 변경 예시 | 검증 자료 예시 |
| --- | --- | --- |
| 웹 프론트엔드 | 로그인 폼/상태 관리 | 화면 캡처, 접근성·반응형 점검, 실행 결과 |
| 모바일(Flutter) | 네비게이션/상태 처리 | 에뮬레이터 화면, 플랫폼별 동작 확인 |
| 백엔드(Spring Boot) | API/권한/예외 처리 | 요청·응답 예시, 단위·통합 테스트 |
| AI/데이터 | 전처리/학습 코드 | 재현 명령, 샘플 데이터 결과, 평가 지표 |

Issue를 참조하는 `Refs #19` 같은 표현과 자동 종료 키워드 `Closes #19`, `Fixes #19`는 구분합니다. 종료 키워드는 GitHub의 병합 조건을 충족할 때 관련 Issue를 자동 종료할 수 있습니다.

## 5. Code Review: 어디를 확인할까?

1. **Conversation**: 관련 Issue, 작업 목적, 테스트 결과, 기존 논의를 읽습니다.
2. **Files changed**: 초록색 추가 줄, 빨간색 삭제 줄과 주변 맥락을 함께 살펴봅니다.
3. **Checks**: 자동화 검사와 실행 결과를 확인하되, 통과 여부만으로 품질을 확정하지 않습니다.
4. **댓글**: 문제가 있는 줄 또는 여러 줄을 선택해 근거·영향·개선 방향을 설명합니다.

트랙 공통 체크 포인트는 **요구사항 충족, 기존 설계/컨벤션 일관성, 예외·경계 조건, 테스트의 의미, 변경 범위**입니다.

- **프론트엔드/모바일**: 로딩·빈 상태·오류 상태, 접근성, 반응형/디바이스별 화면, 상태 관리와 화면 전환.
- **백엔드**: 권한, 요청 검증, API 계약, 데이터 모델, 트랜잭션, 오류 응답.
- **AI/데이터**: 데이터 누수, 전처리 일관성, 실행 재현성, 평가 기준.

### 리뷰 댓글 톤

조직의 [리뷰 가이드](PR_REVIEW_GUIDE.md)에 맞춰 다음 접두어를 권장합니다.

| 표현 | 의미 |
| --- | --- |
| `blocking:` | Merge 전에 반드시 해결해야 할 문제 |
| `suggestion:` | 개선 제안 |
| `question:` | 의도 확인 질문 |
| `nit:` | 사소하고 선택적인 의견 |

**예시:** `question: 오류 응답에서 사용자에게 표시할 메시지를 구분한 이유가 있을까요?`

**예시:** `blocking: 권한 검증 없이 수정 요청을 처리할 수 있어 보입니다. 해당 경로의 인가 검증과 테스트가 필요합니다.`

확신이 없으면 오류로 단정하지 말고 질문합니다. 이번 PR과 직접 관계없는 대규모 리팩터링은 후속 Issue로 분리할 수 있습니다.

### 리뷰 제출

`Finish your review`에서 `Comment`(의견), `Approve`(승인), `Request changes`(수정 요청)를 선택하고 `Submit review`를 누릅니다. `Request changes`가 실제 Merge를 차단하는지는 저장소의 규칙/보호 설정에 따라 다릅니다. `Start a review`로 댓글을 모아 제출할 수도 있습니다.

## 6. 피드백 반영과 Merge

1. 댓글의 의도와 현재 코드를 비교합니다.
2. 필요한 부분만 수정하고 관련 테스트를 수행합니다.
3. Commit & Push 후, 무엇을 수정했고 어떻게 검증했는지 **Reply**로 알립니다.
4. 논의가 끝났을 때 **Resolve conversation**을 사용합니다.
5. 필수 항목·테스트·리뷰어 승인 및 대상 브랜치를 다시 확인합니다.

`No conflicts with base branch`나 Merge 버튼의 활성화는 코드 품질 또는 팀 승인 기준을 충족했다는 뜻이 아닙니다. A&I 공통 가이드에서는 **최소 1명 리뷰를 권장**하지만, 승인/CI 강제 여부는 각 저장소 설정과 팀 합의를 확인해야 합니다.

PR이 Merge되고 Closed 상태가 되더라도 연결된 Issue가 언제나 자동 종료되는 것은 아닙니다. 반드시 Issue의 상태를 별도로 확인합니다.

## 7. 처음 참여할 때 체크리스트

- [ ] Issue의 목적과 확인 가능한 완료 조건을 이해했습니다.
- [ ] 기준 브랜치와 작업 브랜치를 올바르게 선택했습니다.
- [ ] 한 PR에 한 목적만 담았습니다.
- [ ] 커밋·PR 제목이 공통 컨벤션에 맞습니다.
- [ ] PR에 변경 내용과 실제 확인 방법을 작성했습니다.
- [ ] UI/API/모델 등 변경 유형에 맞는 증거를 남겼습니다.
- [ ] 리뷰 댓글에 존중하는 표현과 구체적인 이유를 적었습니다.
- [ ] 필수 피드백, CI, 비밀정보 포함 여부를 확인했습니다.
- [ ] Merge 후 관련 Issue 상태를 확인했습니다.

## 8. 더 읽기

- [A&I PR 작성 가이드](PR_GUIDE.md)
- [A&I 리뷰 운영 가이드](PR_REVIEW_GUIDE.md)
- [A&I 커밋 컨벤션](COMMIT_CONVENTION.md)
- [A&I 클린 코드 가이드](CLEAN_CODE.md)
- [GitHub 공식 문서: Pull Request 생성](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request)
- [GitHub 공식 문서: 리뷰 제출](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/reviewing-proposed-changes-in-a-pull-request)

> **이미지 자료:** 별도로 준비된 GitHub 화면 예시 15장을 문서와 같은 저장소의 `assets/github-collaboration/`에 함께 추가하는 작업이 남아 있습니다. 이미지가 저장소에 반영되기 전까지 깨진 링크를 만들지 않도록 이 초안에는 이미지 태그를 넣지 않았습니다.
