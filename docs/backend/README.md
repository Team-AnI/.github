# Backend Docs Index

이 문서는 Team-AnI 백엔드 공통 문서와 서비스별 상세 문서의 진입점입니다.

---

## 1. 문서 Depth 원칙

백엔드 문서는 아래 depth로 나눕니다.

### Depth 1. Organization Common Docs (`.github`)

모든 백엔드 서비스가 공통으로 따라야 하는 기준만 둡니다.

- API 네이밍 규칙
- 공통 예외처리/에러 모델
- 공통 엔지니어링 규칙

### Depth 2. Service Docs (각 레포 Wiki)

각 서비스의 도메인/구현/운영 특화 내용은 개별 레포 wiki에 둡니다.

- 도메인별 에러 코드 상세
- 서비스별 API 스펙
- 도메인 상태 전이 규칙
- 마이그레이션/운영 메모

즉, `.github`는 **공통 계약**, 각 서비스 wiki는 **서비스 상세 구현 규칙**을 담당합니다.

---

## 2. Common Backend Docs

- [Backend API Naming Guide (A&I)](./API_NAMING.md)
- [Backend API Common Exception Handling Guide v1](./COMMON_EXCEPTION_HANDLING_GUIDE_V1.md)

---

## 3. Service Wiki Links

- Auth
  - [Wiki Home](https://github.com/Team-AnI/A-AND-I-AUTH-SERVER/wiki)
  - [API-Spec-v1](https://github.com/Team-AnI/A-AND-I-AUTH-SERVER/wiki/API-Spec-v1)
  - [Error-Model-v1](https://github.com/Team-AnI/A-AND-I-AUTH-SERVER/wiki/Error-Model-v1)
- Report
  - [Wiki Home](https://github.com/Team-AnI/A-AND-I-REPORT-SERVER/wiki)
  - [API-Spec-v1](https://github.com/Team-AnI/A-AND-I-REPORT-SERVER/wiki/API-Spec-v1)
  - [Error-Model-v1](https://github.com/Team-AnI/A-AND-I-REPORT-SERVER/wiki/Error-Model-v1)
- Web
  - [Wiki Home](https://github.com/Team-AnI/A-AND-I-WEB-SERVER/wiki)
  - [API-Spec-v1](https://github.com/Team-AnI/A-AND-I-WEB-SERVER/wiki/API-Spec-v1)
  - [Error-Model-v1](https://github.com/Team-AnI/A-AND-I-WEB-SERVER/wiki/Error-Model-v1)
- Gateway
  - [Wiki Home](https://github.com/Team-AnI/A-AND-I-GATEWAY-SERVER/wiki)
  - [API-Spec-v1](https://github.com/Team-AnI/A-AND-I-GATEWAY-SERVER/wiki/API-Spec-v1)
  - [Error-Model-v1](https://github.com/Team-AnI/A-AND-I-GATEWAY-SERVER/wiki/Error-Model-v1)
- Blog
  - [Docs Home](https://github.com/Team-AnI/A-AND-I-TECH-BLOG-SERVER/blob/main/wiki/Home.md)
  - [API-Spec-v1](https://github.com/Team-AnI/A-AND-I-TECH-BLOG-SERVER/blob/main/wiki/API-Spec-v1.md)
  - [Error-Model-v1](https://github.com/Team-AnI/A-AND-I-TECH-BLOG-SERVER/blob/main/wiki/Error-Model-v1.md)

---

## 4. 예외처리 문서 적용 원칙

공통 예외처리 문서에서는 아래만 정의합니다.

- 공통 응답 envelope
- 공통 HTTP status 정책
- 공통 `error.code` taxonomy
- 공통 Swagger 문서화 기준

아래는 각 서비스 wiki에서 정의합니다.

- 서비스별 도메인 예외 코드
- 서비스별 상세 예외 메시지
- 서비스별 예외 샘플 응답
- 서비스별 예외 흐름/운영 대응 방식

예:

- 공통 문서: `IMAGE_TOO_LARGE`
- Auth wiki: 프로필 이미지 업로드 정책
- Report wiki: 과제 제출 첨부파일 정책
- Blog wiki: 썸네일 업로드 정책

---

## 5. 권장 운영 방식

1. 공통 변경은 `.github` 문서를 먼저 수정한다.
2. 서비스별 상세 정책은 각 레포 wiki에서 이어서 정의한다.
3. 클라이언트는 우선 공통 `error.code`를 기준으로 분기한다.
4. 서비스 전용 예외는 필요한 경우에만 서비스 wiki와 API 문서에 추가한다.
