# A&I 공통 API 에러 처리 가이드 v1

A&I 백엔드 프로젝트의 API 응답 형식, HTTP 상태코드, 에러 코드와 예외 처리 기준을 정의합니다. 프로젝트별 도메인 코드와 상세 정책은 해당 프로젝트 문서에서 관리합니다.

## 1. 기본 원칙

- JSON 응답은 `success`, `data`, `error`, `timestamp` 형식을 사용합니다. 본문이 없는 `204 No Content`에는 이 형식을 적용하지 않습니다.
- HTTP 상태코드는 요청 처리 결과를, `error.code`는 클라이언트의 세부 분기 기준을 나타냅니다.
- `error.code`는 대문자 스네이크 케이스로 작성합니다. 메시지나 내부 구현이 바뀌어도 의미가 같으면 코드 값은 유지합니다.
- 공통 코드부터 사용하고, 부족한 경우에만 도메인 코드를 추가합니다. 새 코드의 의미가 기존 코드와 겹치지 않도록 정의합니다.
- 서비스 계층은 도메인 예외를 던지고, HTTP 응답 변환은 전역 예외 처리기에서 수행합니다.
- 응답에는 사용자에게 보여줄 메시지만 담습니다. 내부 예외 원문, SQL, 스택트레이스, 토큰은 노출하지 않습니다.

## 2. 응답 형식

- 성공: `success=true`, `data!=null`, `error=null`
- 실패: `success=false`, `data=null`, `error!=null`
- `timestamp`: 응답 생성 시각을 ISO-8601 형식으로 작성합니다.

### 성공 응답

```json
{
  "success": true,
  "data": {},
  "error": null,
  "timestamp": "2026-03-11T10:00:00+09:00"
}
```

### 실패 응답

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "NOT_FOUND",
    "message": "요청한 리소스를 찾을 수 없습니다."
  },
  "timestamp": "2026-03-11T10:00:00+09:00"
}
```

### 검증 오류 응답 (`details` 포함)

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "요청값 검증에 실패했습니다.",
    "details": [
      {
        "field": "name",
        "reason": "must not be blank",
        "rejectedValue": ""
      }
    ]
  },
  "timestamp": "2026-03-11T10:00:00+09:00"
}
```

`details`는 선택 필드이며 주로 `VALIDATION_ERROR`의 필드별 오류를 전달할 때 사용합니다. 위 예시는 `field`, `reason`, `rejectedValue`로 구성합니다. 토큰·비밀번호·개인정보는 넣지 않으며, 클라이언트는 `details`가 없어도 오류를 처리해야 합니다.

## 3. HTTP 상태코드와 공통 에러 코드

### HTTP 상태코드

| HTTP | 사용 시점 |
| --- | --- |
| 200 OK | 조회/수정/삭제/생성 성공 |
| 201 Created | 신규 리소스 생성 성공 |
| 204 No Content | 본문 없이 처리 성공 |
| 400 Bad Request | 요청값/문법/형식 오류 |
| 401 Unauthorized | 인증 실패 |
| 403 Forbidden | 권한 부족 |
| 404 Not Found | 리소스 없음 |
| 405 Method Not Allowed | 허용되지 않은 메서드 |
| 409 Conflict | 중복/충돌 상태 |
| 415 Unsupported Media Type | 지원하지 않는 Content-Type |
| 422 Unprocessable Entity | 상태상 처리 불가 |
| 429 Too Many Requests | 요청 과다 |
| 500 Internal Server Error | 내부 예외 |
| 503 Service Unavailable | 외부 의존성 장애 |

### 공통 에러 코드

아래 표의 상태코드와 기본 메시지를 공통 기준으로 사용합니다. 메시지를 바꿀 때도 사용자에게 노출 가능한 내용인지 확인합니다.

| `error.code` | HTTP | 기본 메시지 |
| --- | --- | --- |
| `VALIDATION_ERROR` | 400 | 요청값 검증에 실패했습니다. |
| `INPUT_ERROR` | 400 | 요청값 형식이 올바르지 않습니다. |
| `JSON_PARSE_ERROR` | 400 | 요청 본문(JSON) 형식이 올바르지 않습니다. |
| `ENUM_MISMATCH` | 400 | 허용되지 않은 enum 값입니다. |
| `MISSING_REQUIRED_VALUE` | 400 | 필수 요청값이 누락되었습니다. |
| `BAD_REQUEST` | 400 | 잘못된 요청입니다. |
| `UNAUTHORIZED` | 401 | 인증이 필요하거나 토큰이 유효하지 않습니다. |
| `AUTH_TOKEN_EXPIRED` | 401 | 토큰이 만료되었습니다. |
| `AUTH_TOKEN_INVALID` | 401 | 토큰이 올바르지 않습니다. |
| `FORBIDDEN` | 403 | 요청을 수행할 권한이 없습니다. |
| `NOT_FOUND` | 404 | 요청한 리소스를 찾을 수 없습니다. |
| `METHOD_NOT_ALLOWED` | 405 | 허용되지 않은 HTTP 메서드입니다. |
| `CONFLICT` | 409 | 리소스 충돌이 발생했습니다. |
| `UNSUPPORTED_MEDIA_TYPE` | 415 | 지원하지 않는 Content-Type 입니다. |
| `UNPROCESSABLE_ENTITY` | 422 | 현재 상태에서는 요청을 처리할 수 없습니다. |
| `FILE_UPLOAD_FAILED` | 400 | 파일 업로드에 실패했습니다. |
| `IMAGE_TOO_LARGE` | 400 | 이미지 용량이 허용 범위를 초과했습니다. |
| `IMAGE_UNSUPPORTED_FORMAT` | 400 | 지원하지 않는 이미지 형식입니다. |
| `SMTP_SEND_FAILED` | 503 | 메일 전송에 실패했습니다. |
| `EXTERNAL_API_FAILED` | 503 | 외부 연동 처리에 실패했습니다. |
| `UPSTREAM_TIMEOUT` | 503 | 외부 서비스 응답 시간이 초과되었습니다. |
| `TOO_MANY_REQUESTS` | 429 | 요청이 너무 많습니다. |
| `INTERNAL_ERROR` | 500 | 서버 내부 오류가 발생했습니다. |
| `SERVICE_UNAVAILABLE` | 503 | 일시적으로 서비스를 사용할 수 없습니다. |

## 4. 예외 처리 구현

구현 언어와 프레임워크에 맞춰 아래 책임을 분리하고, 실패 응답은 공통 응답 생성기로 통일합니다. Spring MVC에서는 `@RestControllerAdvice`로 전역 예외 처리를 구성합니다.

| 구성 요소 | 책임 |
| --- | --- |
| `ErrorCode` | HTTP 상태코드, 코드 문자열, 기본 메시지 관리 |
| 공통 예외 (`ApiException`) | 도메인 오류의 에러 코드와 사용자 메시지 전달 |
| 공통 응답 생성기 (`ErrorResponseFactory`) | 기본 메시지 또는 안전한 대체 메시지로 실패 응답 생성. `timestamp`와 필요한 `details` 포함 |
| 전역 예외 처리기 (`GlobalExceptionHandler`) | 발생 예외를 공통 코드에 매핑하고 HTTP 응답으로 변환 |

구현은 에러 코드 → 공통 예외 → 응답 생성기 → 전역 처리기 순서로 맞춘 뒤, 도메인 예외·OpenAPI 문서·테스트에 반영합니다. Controller마다 같은 응답을 만드는 try-catch를 반복하지 않습니다.

### Spring MVC 예외 매핑

| 발생 예외 | 권장 HTTP | `error.code` |
|---|---|---|
| `ApiException` | `errorCode.status` | `errorCode.code` |
| `MethodArgumentNotValidException` | 400 | `VALIDATION_ERROR` |
| `BindException` | 400 | `VALIDATION_ERROR` |
| `ConstraintViolationException` | 400 | `VALIDATION_ERROR` |
| `MethodArgumentTypeMismatchException` | 400 | `INPUT_ERROR` 또는 `ENUM_MISMATCH` |
| `HttpMessageNotReadableException` | 400 | `JSON_PARSE_ERROR` 또는 `ENUM_MISMATCH` |
| `MissingServletRequestParameterException` | 400 | `MISSING_REQUIRED_VALUE` |
| `MissingRequestHeaderException` | 400 | `MISSING_REQUIRED_VALUE` |
| `MissingPathVariableException` | 400 | `MISSING_REQUIRED_VALUE` |
| `HttpRequestMethodNotSupportedException` | 405 | `METHOD_NOT_ALLOWED` |
| `HttpMediaTypeNotSupportedException` | 415 | `UNSUPPORTED_MEDIA_TYPE` |
| `AccessDeniedException` | 403 | `FORBIDDEN` |
| 인증 관련 예외 | 401 | `UNAUTHORIZED` |
| `DataIntegrityViolationException` | 409 또는 500 | `CONFLICT` 또는 `INTERNAL_ERROR` |
| 그 외 `Exception` | 500 | `INTERNAL_ERROR` |

## 5. 요청 추적과 로그

- JSON 응답은 `Content-Type: application/json`을 사용합니다.
- 성공·실패 모두 `X-Request-Id`를 응답 헤더에 포함합니다.
- 요청에 `X-Request-Id`가 있으면 그대로 전달하거나 검증 후 사용하고, 없으면 서버에서 생성합니다.
- 응답 헤더와 로그 문맥(MDC)에는 같은 요청 ID를 사용합니다.

| 로그 항목 | 기준 |
| --- | --- |
| 레벨 | `400/401/403/404/409/422`는 `warn` 또는 `info`, `500/503`은 `error` |
| 기록할 값 | 요청 ID, URI, HTTP 메서드, 사용자 식별자(가능한 경우), 에러 코드, HTTP 상태코드, 예외 클래스명 |
| 스택트레이스 | `5xx`에서는 필수 |
| 제외할 값 | Access/Refresh Token 전체값, 비밀번호, 민감 개인정보, 대용량 원본 body 전체 |

사용자 메시지와 상세 로그를 분리합니다. 예를 들어 JSON 파싱 오류 응답에는 “요청 본문(JSON) 형식이 올바르지 않습니다.”를 표시하고, 내부 로그에는 민감정보를 제외한 파싱 원인과 필드 경로를 남깁니다.

## 6. OpenAPI 문서화

- API마다 같은 응답 스키마를 재사용합니다: `ApiResponse<T>`, `ErrorResponse`, `ErrorBody`, `ValidationDetail`.
- 자주 쓰는 `400/401/403/404/409/422/500` 응답을 공통 component로 등록하고, 각 API에서는 필요한 응답만 참조합니다.
- 응답 component 예시: `BadRequestResponse`, `UnauthorizedResponse`, `ForbiddenResponse`, `NotFoundResponse`, `ConflictResponse`, `UnprocessableEntityResponse`, `InternalServerErrorResponse`.

## 7. 클라이언트 처리

HTTP 상태코드 → 본문의 `success` → `error.code` 순서로 확인합니다. 사용자 안내에는 `error.message`를 사용하고, `details`가 있으면 필드별 오류 표시에 연결합니다.

| 오류 | 처리 예시 |
| --- | --- |
| `401 UNAUTHORIZED` | 재로그인 또는 토큰 갱신 |
| `403 FORBIDDEN` | 권한 없음 안내 |
| `404 NOT_FOUND` | 없는 리소스 안내 |
| `409 CONFLICT` | 중복·충돌 수정 안내 |
| `422 UNPROCESSABLE_ENTITY` | 선행 상태 확인 안내 |
| `500 INTERNAL_ERROR` | 일반 오류 안내와 `X-Request-Id` 수집 |

## 8. 검증 체크리스트

- [ ] HTTP 상태코드와 `error.code`가 계약과 일치하는가?
- [ ] 실패 응답이 `success=false`, `data=null`, `error!=null` 형식인가?
- [ ] `error.message`가 사용자에게 노출 가능한 문구인가?
- [ ] `timestamp`가 포함되어 있는가?
- [ ] `X-Request-Id`가 응답 헤더와 로그에 같은 값으로 남는가?
- [ ] 검증 오류의 `details` 형식이 일관적이고, 없어도 클라이언트가 처리하는가?
- [ ] 응답과 로그에 토큰·비밀번호·민감 개인정보가 없는가?
- [ ] OpenAPI의 오류 응답과 실제 응답이 일치하는가?
