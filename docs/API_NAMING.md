# API Naming Guide (A&I)

이 문서는 A&I Organization의 API 경로/이름 규칙을 통일하기 위한 가이드입니다.

> 목표
> - 경로만 보고 리소스/권한 범위를 바로 이해할 수 있게
> - 프론트/백엔드/게이트웨이 간 계약 혼선을 줄이기
> - 버저닝/레거시 호환 시 충돌을 줄이기

---

## 0) 공통 원칙

- Base Path는 명시적 버전으로 시작한다.
  - 예: `/v1/...`
- 리소스는 **명사**로 표현한다.
- 컬렉션은 **복수형**을 기본으로 한다.
  - 예: `/users`, `/courses`, `/assignments`
- URI에는 행위를 넣지 않는다.
  - `publish`, `deliveries`처럼 도메인 액션이 필요한 경우 하위 리소스로 표현한다.

---

## 1) Prefix 규칙

- Auth: `/v1/auth/*`
- Me(User self): `/v1/me*`
- Admin: `/v1/admin/**`
- Report: `/v1/report/**`
- Course Query: `/v1/courses/**`
- Blog(Post): `/v1/posts/**`
- Internal 전용: `/internal/v1/**`

> 권한 정책은 Prefix 단위로 먼저 정의하고, 세부 권한은 엔드포인트에서 보강한다.

---

## 2) Path Segment 규칙

- Segment는 `kebab-case` 사용
  - 예: `/invite-mail`, `/reset-password`
- Path Variable은 `lowerCamelCase` 사용
  - 예: `{courseSlug}`, `{assignmentId}`, `{userId}`
- 축약어보다 의미가 분명한 이름을 우선
  - `id`는 식별자 문맥이 명확할 때만 사용

---

## 3) HTTP Method 규칙

- `GET`: 조회
- `POST`: 생성/트리거
- `PATCH`: 부분 수정/상태 전이
- `PUT`: 전체 치환(필요 시)
- `DELETE`: 삭제/아카이브

예시
- `POST /v1/admin/courses/{courseSlug}/assignments/{assignmentId}/publish`
- `POST /v1/admin/courses/{courseSlug}/assignments/{assignmentId}/deliveries`

---

## 4) Query Parameter 네이밍

- 필터 파라미터는 의미 중심 소문자 카멜로 통일
  - `status`, `phase`, `track`, `weekNo`
- 호환성 때문에 별칭이 필요하면 문서에 명시
  - 예: `week`/`weekNo` 동시 지원

---

## 5) Request/Response 필드 네이밍

- JSON 필드: `lowerCamelCase`
- 날짜: `yyyy-MM-dd`
- 일시: ISO-8601
  - 요청: `Z`/`+09:00` 허용
  - 응답: 서비스 정책 기준(팀 내 표준은 KST 직렬화 권장)

---

## 6) 버전/레거시 정책

- 신규 표준은 `/v1` 기준으로 먼저 정의한다.
- 레거시 alias(`/v2/...`)를 유지할 경우:
  - 문서에 `rewrite` 경로를 반드시 표기
  - sunset 일정(제거 예정일)과 영향 범위를 명시

예시
- `/v2/post/courses/**` -> `/v1/courses/**`
- `/v2/post/admin/courses/**` -> `/v1/admin/courses/**`

---

## 7) 금지 패턴

- 동사형 URI 남용
  - 나쁨: `/v1/getUsers`
  - 좋음: `/v1/users`
- 의미 없는 약어 사용
  - 나쁨: `/v1/crs`
  - 좋음: `/v1/courses`
- 권한 Prefix 혼합
  - 나쁨: `/v1/courses/admin/...`
  - 좋음: `/v1/admin/courses/...`

---

## 8) PR 체크리스트

- [ ] Prefix가 정책과 일치하는가? (`/v1/admin`, `/v1/courses` 등)
- [ ] Segment/Path Variable 네이밍 규칙을 지켰는가?
- [ ] 메서드 의미가 REST 규칙과 맞는가?
- [ ] 레거시 alias가 있다면 rewrite/sunset을 문서화했는가?
- [ ] 프론트/백엔드 공통 API 스펙 문서를 함께 업데이트했는가?
