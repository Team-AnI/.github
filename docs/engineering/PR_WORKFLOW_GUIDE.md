# Pull Request & Code Review Guide (A&I) — 제출부터 Merge까지

> **대상:** A&I 웹·모바일·백엔드·AI 트랙 구성원  
> **목적:** 작은 변경을 올리고, 검증 근거를 공유하고, 존중하는 리뷰로 품질을 높입니다.

이 문서는 A&I 프로젝트의 PR 작성과 리뷰 과정을 안내합니다. 아래 실습은 개인 Fork와 원본 저장소의 `main` 브랜치를 기준으로 설명합니다. [A&I 공통 PR 가이드](PR_GUIDE.md), [리뷰 운영 가이드](PR_REVIEW_GUIDE.md), [커밋 컨벤션](COMMIT_CONVENTION.md)과 함께 읽어 주세요. **공통 규칙은 관련 가이드를 따르고 프로젝트별 자동 검사 및 별도 운영 규칙은 해당 저장소 문서를 따릅니다.**

## 1. 전체 흐름

```mermaid
sequenceDiagram
    autonumber
    actor Author as 작성자
    participant Issue as GitHub Issue
    participant Fork as 개인 Fork / 작업 브랜치
    participant PR as Pull Request
    actor Reviewer as 리뷰어
    Author->>Issue: 작업 목적 및 완료 기준 등록
    Author->>Fork: upstream 동기화 후 작업 브랜치 생성
    Author->>Fork: 구현 · 테스트 · Commit · Push
    Author->>PR: 원본 저장소 main 대상으로 PR 작성
    PR-->>Reviewer: 리뷰 요청 및 검증 결과 공유
    Reviewer->>PR: Diff · CI · 요구사항 검토
    alt 수정 요청이 있는 경우
        Reviewer-->>Author: 근거와 개선 방향 전달
        Author->>Fork: 수정 · 테스트 · Push
        Author->>PR: Reply 및 재검토 요청
        Reviewer->>PR: 재확인
    end
    Reviewer->>PR: Approve
    Note over Author,PR: 필수 피드백·CI·승인 조건 확인
    Author->>PR: Merge
    Author->>Issue: 자동 종료 여부 확인
```

- **Issue:** 무엇을, 왜 바꾸는가?
- **PR:** 무엇을 바꿨으며 어떻게 확인했는가?
- **Review:** 요구사항·안전성·변경 범위를 함께 점검했는가?
- **Merge:** 프로젝트에서 정한 승인·검증 조건을 충족했는가?

## 2. Branch와 Commit

Fork 기반으로 협업할 때는 개인 Fork에서 작업하고 원본 저장소로 PR을 제출합니다. 이 실습의 병합 대상은 `main`이며 실제 작업에서는 해당 저장소의 협업 방식과 대상 브랜치를 확인합니다.

```bash
# 개인 Fork를 clone하고 upstream을 등록했다고 가정
# 첫 작업 전 remote 상태 확인
git remote -v
git switch main
git pull --ff-only upstream main
git switch -c 'feature/#12-login'
```

작업 브랜치 예시는 `feature/#번호-내용`, `fix/#번호-내용`, `docs/#번호-내용`입니다. **A&I 기본 작업 브랜치는 유형 + Issue 번호 + 작업 요약 형식으로 작성합니다.** 저장소별 자동화나 별도 합의가 있다면 해당 프로젝트 README를 우선합니다.

```bash
git status
git diff
git add <변경한-파일>
git commit -m 'feat: 로그인 상태 처리 추가 (#12)'
git push -u origin 'feature/#12-login'
```

커밋 메시지는 [A&I 커밋 가이드](COMMIT_CONVENTION.md)에 따라 `<type>: <subject> (#이슈번호)` 형식과 한글 제목으로 작성합니다. **프로젝트에 이미 적용된 자동 검증**을 먼저 확인하세요. Merge 전에 불필요한 파일, `.env`, 접근 토큰, 개인 정보가 없는지 점검합니다.

![작업 브랜치 목록 예시](assets/github-collaboration/04-branch-commits.png)

## 3. Pull Request 생성

PR 화면에서 **base**는 병합 대상, **compare**는 변경한 브랜치입니다. 이 실습에서는 원본 저장소의 `main`을 base로 선택합니다. `Able to merge`는 자동 병합 가능 여부를 표시합니다. 코드 검토 승인은 별도로 확인합니다. 기존 PR이 있다면 `View pull request`가 표시될 수 있습니다.

![PR base / compare 비교](assets/github-collaboration/05-pr-base-compare.png)

![PR 상세 화면](assets/github-collaboration/06.png)

### PR 내용은 무엇을 써야 하나요?

PR 본문 항목과 작성 기준은 [PR 작성 가이드](PR_GUIDE.md)와 [공통 PR 템플릿](../../PULL_REQUEST_TEMPLATE.md)을 참고하세요. 검증 자료에는 작업에 필요한 화면, API 요청·응답, 로그 등을 첨부하며 자료가 없으면 해당 항목을 생략할 수 있습니다.

```markdown
## 📌 관련 이슈
- Refs #12

## ✨ 작업 내용
- 로그인 성공/실패 상태를 구분해 표시

## 🧪 테스트 및 검증
- 실행 명령: flutter test
- 결과: (실제로 확인한 결과만 작성)
- 수동 확인: 로그인 성공/실패 시나리오

## 📎 검증 자료 (선택)
- 변경 전/후 또는 화면 녹화 링크

## 💬 리뷰 요청 사항
- 리뷰어: @멘토아이디, @협업자아이디
- 실패 시 상태 전환 로직 확인 부탁드립니다.

```

체크리스트는 위 공통 PR 템플릿의 항목을 사용합니다.

이것은 **설명용 예시**이며 실제 작성 시 저장소에 자동 표시되는 템플릿을 우선합니다. 테스트 결과는 실제 실행한 내용을 기준으로 작성합니다. Issue 완료 시 자동 종료가 필요하면 `Refs` 대신 올바른 종료 키워드를 사용합니다.

| 작업 유형 | 리뷰어에게 보여줄 근거 |
| --- | --- |
| 웹·Flutter UI | 변경 전/후, 로딩·오류·빈 상태, 사용 기기/브라우저 |
| 백엔드 API | 요청/응답 예시, 권한·유효성 검사, 테스트 결과 |
| AI·데이터 | 실행 조건, 재현 단계, 샘플 결과 및 평가 지표 |
| 공통 문서·설정 | 링크/명령 확인 내용, 영향 범위 |

### 프로젝트별 검증

프로젝트에서 사용하는 테스트·빌드·정적 분석 명령과 CI는 해당 저장소의 README에서 확인합니다. 수행한 검증 방법과 실제 결과를 PR에 기록합니다.

## 4. 리뷰어가 확인할 것

### Conversation

관련 Issue·설명·검증 결과·이전 논의를 확인합니다.

![PR Conversation 예시](assets/github-collaboration/07-pr-conversation.png)

### Files changed

추가(초록색)·삭제(빨간색) 줄과 주변 맥락을 함께 검토합니다.

![파일별 Diff](assets/github-collaboration/08-pr-files-changed.png)

공통 리뷰 기준은 [코드 리뷰 원칙](PR_REVIEW_GUIDE.md), 코드 품질과 가독성 기준은 [클린 코드 가이드](CLEAN_CODE.md)를 확인합니다.

### 한 줄 또는 여러 줄에 코멘트

![라인 코멘트 편집기](assets/github-collaboration/09-inline-comment-editor.png)

![여러 줄 선택](assets/github-collaboration/09a-code-line-selection.png)

![코드 범위 코멘트 입력](assets/github-collaboration/09b-code-range-comment-editor-corrected.png)

댓글 구분은 [코드 리뷰 원칙의 표기 기준](PR_REVIEW_GUIDE.md)을 따릅니다. 필요하면 사소한 의견을 `[참고]`로 구분할 수 있지만 필수 여부는 분명히 밝혀야 합니다.

> `[필수]` 권한 검증 없이 데이터 변경 요청이 처리될 수 있습니다. 인가 검증과 실패 테스트가 필요합니다.
>
> `[질문]` 이 상태를 별도로 분리한 이유가 있을까요?

필수 수정에는 **문제의 원인·사용자 영향·수정 방향**을 함께 적습니다. 확인이 필요한 내용은 질문으로 남깁니다.

## 5. 리뷰 제출 및 피드백 반영

### 리뷰어 지정 및 승인 상태 확인

PR 작성자는 [PR 작성 가이드](PR_GUIDE.md)에 따라 오른쪽 **Reviewers에 멘토와 이번 작업의 협업자를 반드시 지정**해 리뷰를 요청합니다. PR 본문의 **리뷰 요청 사항**에는 리뷰어를 `@GitHub아이디`로 태그하고 확인받을 내용을 적습니다.

리뷰 요청과 승인 완료는 다릅니다. 필수 승인자와 승인 수, 병합 차단 여부는 해당 저장소의 `CODEOWNERS`, 브랜치 보호 규칙 및 팀 운영 정책을 따릅니다. 봇 리뷰와 일반 댓글은 지정 리뷰어의 Approve와 구분합니다.

| 화면 상태 | 의미 | 다음 행동 |
| --- | --- | --- |
| 노란색 점 또는 리뷰 대기 표시 | 리뷰가 요청됐지만 해당 리뷰어의 최종 결과가 아직 표시되지 않은 상태일 수 있음 | 리뷰 완료를 기다리거나 담당 리뷰어에게 확인 |
| 초록색 체크 | 해당 리뷰어의 **Approve** 표시 | 필수 승인 수, 다른 수정 요청 및 CI 상태도 확인 |
| 빨간색 변경 요청 표시 | 해당 리뷰어의 **Request changes** 표시 | 지적 사항 수정·검증·Push 후 재검토 요청 |

**① 리뷰 대기 및 충돌이 있는 PR 예시**

![리뷰 대기 표시와 별도의 Merge conflicts 배지](assets/github-collaboration/review-pending-and-conflicts.png)

노란색 표시가 보이면 해당 리뷰어의 리뷰 완료 여부를 확인합니다. 상단의 `Merge conflicts`는 **리뷰 승인과 별개인 브랜치 충돌 문제**입니다.

**② 리뷰 승인이 표시된 PR 예시**

![Reviewers 영역의 초록색 승인 체크](assets/github-collaboration/review-approved.png)

초록색 체크는 해당 리뷰어의 승인을 나타냅니다. `Ready to merge`는 GitHub의 병합 가능 상태를 나타냅니다. Merge 전에는 팀의 승인·검증 조건도 확인합니다.

**③ 수정 요청과 승인이 함께 보이는 PR 예시**

<img width="1770" height="690" alt="review-changes-requested" src="https://github.com/user-attachments/assets/86f5feb3-beab-488c-80a9-eaafae9c2d7c" />


한 리뷰어가 승인했더라도 다른 리뷰어의 수정 요청이 남아 있을 수 있습니다. 수정 요청의 해결 여부, 재승인 필요 여부, 보호 규칙을 확인한 다음 Merge를 판단합니다.

`Finish your review`에서는 `Comment`(의견), `Approve`(승인), `Request changes`(수정 요청) 중 하나를 선택해 `Submit review`합니다. `Start a review`로 여러 댓글을 묶을 수도 있습니다. `Request changes`가 병합을 차단하는지는 저장소 보호 규칙에 따라 달라집니다.

![리뷰 제출 옵션](assets/github-collaboration/10-review-submit-options-corrected.png)

수정 요청을 받았다면 해당 코드를 확인하고 필요한 수정·검증을 거쳐 Push합니다. 댓글에 변경 사항을 설명한 뒤 논의가 끝나면 `Resolve conversation`을 사용합니다.

![리뷰 스레드 및 Resolve](assets/github-collaboration/11-review-thread-resolve-private.png)

## 6. Merge 전 확인

- [ ] Issue의 완료 기준에 부합하는가?
- [ ] 변경 범위·민감 정보·관련 문서를 확인했는가?
- [ ] 해당 저장소의 필수 CI/테스트가 통과했는가?
- [ ] 리뷰어와 합의한 필수 수정 사항을 해결했는가?
- [ ] 팀에서 요구하는 승인과 대상 브랜치를 확인했는가?

Merge 담당자와 필수 승인·CI 조건은 해당 저장소의 운영 규칙과 설정을 따릅니다.

![충돌 여부·수정 요청·Merge 버튼](assets/github-collaboration/12-merge-readiness.png)

`No conflicts with base branch`는 충돌 여부를 나타냅니다. Merge 전에는 리뷰 완료와 프로젝트의 승인·검증 조건을 함께 확인합니다.

![PR 병합 완료 화면](assets/github-collaboration/13-merged-issue-status.png)

PR 병합 후에는 종료 키워드·대상 저장소·기본 브랜치 병합 조건과 Issue 상태를 최신화합니다.

## 관련 문서

- [Issue 작성 실전 가이드](ISSUE_GUIDE.md)
- [A&I PR 작성 가이드](PR_GUIDE.md)
- [A&I PR/리뷰 운영 가이드](PR_REVIEW_GUIDE.md)
- [A&I 커밋 컨벤션](COMMIT_CONVENTION.md)
