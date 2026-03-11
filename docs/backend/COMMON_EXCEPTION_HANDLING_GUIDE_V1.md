# Backend API Common Exception Handling Guide v1

> 본 문서는 **Team-AnI 백엔드 서비스 공통 예외처리 표준안**입니다.
> 각 서비스(Auth, Report, Web, Gateway, Blog 등)는 본 문서를 기준으로 자체 구현을 맞춥니다.
>
> 대표 구현 대상 예시:
>
> - `common/error/ErrorCode.kt`
> - `common/error/ErrorResponseFactory.kt`
> - `common/error/GlobalExceptionHandler.kt`
> - `common/config/SwaggerConfig.kt`
>
> 서비스별 상세 예외 시나리오와 도메인 전용 에러 코드는 각 레포 wiki에서 관리합니다.

---

## Quick Links

### Common Docs

- [Backend Docs Index](./README.md)
- [Backend API Naming Guide (A&I)](./API_NAMING.md)

### Service Error Models

- [Auth Error-Model-v1](https://github.com/Team-AnI/A-AND-I-AUTH-SERVER/wiki/Error-Model-v1)
- [Report Error-Model-v1](https://github.com/Team-AnI/A-AND-I-REPORT-SERVER/wiki/Error-Model-v1)
- [Web Error-Model-v1](https://github.com/Team-AnI/A-AND-I-WEB-SERVER/wiki/Error-Model-v1)
- [Gateway Error-Model-v1](https://github.com/Team-AnI/A-AND-I-GATEWAY-SERVER/wiki/Error-Model-v1)
- [Blog Error-Model-v1](https://github.com/Team-AnI/A-AND-I-TECH-BLOG-SERVER/blob/main/wiki/Error-Model-v1.md)

### Service API Specs

- [Auth API-Spec-v1](https://github.com/Team-AnI/A-AND-I-AUTH-SERVER/wiki/API-Spec-v1)
- [Report API-Spec-v1](https://github.com/Team-AnI/A-AND-I-REPORT-SERVER/wiki/API-Spec-v1)
- [Web API-Spec-v1](https://github.com/Team-AnI/A-AND-I-WEB-SERVER/wiki/API-Spec-v1)
- [Gateway API-Spec-v1](https://github.com/Team-AnI/A-AND-I-GATEWAY-SERVER/wiki/API-Spec-v1)
- [Blog API-Spec-v1](https://github.com/Team-AnI/A-AND-I-TECH-BLOG-SERVER/blob/main/wiki/API-Spec-v1.md)

---

## 0. 문서 목적

이 문서는 백엔드 API에서 발생하는 예외를 **일관된 응답 포맷**, **예측 가능한 HTTP 상태코드**, **안정적인 에러 코드 체계**로 통합하기 위한 공통 기준 문서입니다.

핵심 목적은 아래와 같습니다.

1. 프론트엔드가 `success`, `error.code`, `error.message`만으로 안정적으로 분기할 수 있어야 한다.
2. 각 서비스가 도메인 예외, 요청 검증 오류, 인증/인가 실패, 시스템 장애를 같은 방식으로 응답해야 한다.
3. Swagger/OpenAPI 문서에 공통 오류 응답이 일관되게 노출되어야 한다.
4. 운영 시 `X-Request-Id` 기준으로 로그와 오류 응답을 쉽게 연결할 수 있어야 한다.
5. 사용자에게는 안전한 메시지만 노출하고, 내부 스택트레이스나 민감한 정보는 숨겨야 한다.

---

## 0.1 문서 Depth 원칙

이 문서는 **공통 규칙(Depth 1)** 만 정의한다.

- `.github` 공통 문서:
  - 공통 응답 모델
  - 공통 HTTP status 정책
  - 공통 `error.code` taxonomy
  - 공통 Swagger/OpenAPI 문서화 기준
- 서비스별 레포 wiki:
  - Auth/Report/Blog/Gateway 도메인 전용 예외 코드
  - 세부 예외 메시지 정책
  - 서비스 특화 운영/장애 대응 규칙

즉, 클라이언트가 우선 분기해야 하는 **공통 에러 모델**은 이 문서에서 정의하고, 서비스별 상세 케이스는 각 레포 wiki로 내려보낸다.

빠른 이동:

- 공통 인덱스: [Backend Docs Index](./README.md)
- Auth 상세: [Auth Error-Model-v1](https://github.com/Team-AnI/A-AND-I-AUTH-SERVER/wiki/Error-Model-v1)
- Report 상세: [Report Error-Model-v1](https://github.com/Team-AnI/A-AND-I-REPORT-SERVER/wiki/Error-Model-v1)
- Web 상세: [Web Error-Model-v1](https://github.com/Team-AnI/A-AND-I-WEB-SERVER/wiki/Error-Model-v1)
- Gateway 상세: [Gateway Error-Model-v1](https://github.com/Team-AnI/A-AND-I-GATEWAY-SERVER/wiki/Error-Model-v1)
- Blog 상세: [Blog Error-Model-v1](https://github.com/Team-AnI/A-AND-I-TECH-BLOG-SERVER/blob/main/wiki/Error-Model-v1.md)

---

## 1. 설계 원칙

### 1.1 응답 포맷은 항상 동일해야 한다

성공/실패 여부와 관계없이 응답 envelope 구조는 고정한다.

- 성공 시: `success=true`, `data!=null`, `error=null`
- 실패 시: `success=false`, `data=null`, `error!=null`

### 1.2 HTTP 상태코드는 의미대로 사용한다

오류를 모두 `400` 또는 `500`으로 뭉개지 않는다.

- 요청 자체가 잘못되었는가 -> `400`
- 인증이 없는가 -> `401`
- 권한이 없는가 -> `403`
- 대상 리소스가 없는가 -> `404`
- 중복/충돌 상태인가 -> `409`
- 문법은 맞지만 현재 상태상 처리 불가인가 -> `422`
- 서버 내부 문제인가 -> `500`

### 1.3 `error.code`는 프론트 계약이다

`error.code`는 프론트 분기 기준이므로 문자열을 자주 바꾸면 안 된다.

권장 원칙:

- 코드 문자열은 대문자 스네이크 케이스 사용
- 메시지는 바뀔 수 있어도 코드 값은 최대한 유지
- 내부 구현이 바뀌어도 같은 의미면 같은 코드 유지

### 1.4 사용자 메시지와 내부 로그 메시지는 분리한다

외부 응답에는 사용자 친화적인 메시지를 주고, 내부 로그에는 상세 원인과 스택트레이스를 남긴다.

예:

- 사용자 응답: `요청 본문(JSON) 형식이 올바르지 않습니다.`
- 서버 로그: Jackson 파싱 실패 stack trace, field path, request body 일부

### 1.5 예외는 Controller 단이 아니라 전역에서 수집한다

각 Controller에서 try-catch로 직접 응답을 만들지 않고, `@RestControllerAdvice`에서 공통 처리한다.

---

## 2. 표준 응답 포맷

### 2.1 성공 응답

```json
{
  "success": true,
  "data": {},
  "error": null,
  "timestamp": "2026-03-11T10:00:00+09:00"
}
```

### 2.2 실패 응답

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

### 2.3 확장 가능한 실패 응답(권장)

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

### `details` 사용 원칙

- 선택 필드로 둔다.
- 주로 `VALIDATION_ERROR`에서 사용한다.
- 민감정보(토큰, 비밀번호, 개인정보)는 넣지 않는다.
- 프론트는 `details`가 없어도 동작해야 한다.

---

## 3. 헤더 규칙

오류/성공과 무관하게 서버는 아래 헤더를 일관되게 반환하는 것을 권장한다.

- `Content-Type: application/json`
- `X-Request-Id: <request-id>`

### `X-Request-Id` 운영 원칙

- 클라이언트가 요청에 `X-Request-Id`를 넣으면 그대로 전달하거나 검증 후 사용
- 없으면 서버에서 생성
- 응답 헤더에 반드시 포함
- 로그 MDC에도 동일 값 저장

---

## 4. HTTP 상태코드 표준

| HTTP | 이름 | 사용 시점 | 대표 `error.code` |
|---|---|---|---|
| 200 | OK | 조회/수정/삭제/생성 성공 | - |
| 201 | Created | 신규 리소스 생성 성공 | - |
| 204 | No Content | 본문 없이 처리 성공 | - |
| 400 | Bad Request | 요청값/문법/형식 오류 | `VALIDATION_ERROR`, `INPUT_ERROR`, `JSON_PARSE_ERROR`, `ENUM_MISMATCH`, `MISSING_REQUIRED_VALUE`, `BAD_REQUEST` |
| 401 | Unauthorized | 인증 실패 | `UNAUTHORIZED` |
| 403 | Forbidden | 권한 부족 | `FORBIDDEN` |
| 404 | Not Found | 리소스 없음 | `NOT_FOUND` |
| 405 | Method Not Allowed | 허용되지 않은 메서드 | `METHOD_NOT_ALLOWED` |
| 409 | Conflict | 중복/충돌 상태 | `CONFLICT` |
| 415 | Unsupported Media Type | 지원하지 않는 Content-Type | `UNSUPPORTED_MEDIA_TYPE` |
| 422 | Unprocessable Entity | 상태상 처리 불가 | `UNPROCESSABLE_ENTITY` |
| 429 | Too Many Requests | 요청 과다 | `TOO_MANY_REQUESTS` |
| 500 | Internal Server Error | 내부 예외 | `INTERNAL_ERROR` |
| 503 | Service Unavailable | 외부 의존성 장애 | `SERVICE_UNAVAILABLE` |

---

## 5. 공통 `ErrorCode` 설계 가이드

### 5.0 공통 에러 코드 분류 체계

클라이언트는 개별 서비스 구현이 아니라 아래 공통 코드군을 기준으로 먼저 분기한다.

#### 1) 요청/입력 오류

- `VALIDATION_ERROR`
- `INPUT_ERROR`
- `JSON_PARSE_ERROR`
- `ENUM_MISMATCH`
- `MISSING_REQUIRED_VALUE`
- `BAD_REQUEST`

#### 2) 인증/인가 오류

- `UNAUTHORIZED`
- `AUTH_TOKEN_EXPIRED`
- `AUTH_TOKEN_INVALID`
- `FORBIDDEN`

#### 3) 리소스/상태 오류

- `NOT_FOUND`
- `CONFLICT`
- `UNPROCESSABLE_ENTITY`

#### 4) 파일/이미지 오류

- `FILE_UPLOAD_FAILED`
- `IMAGE_TOO_LARGE`
- `IMAGE_UNSUPPORTED_FORMAT`

#### 5) 외부 의존성/네트워크 오류

- `SMTP_SEND_FAILED`
- `EXTERNAL_API_FAILED`
- `UPSTREAM_TIMEOUT`
- `SERVICE_UNAVAILABLE`

#### 6) 시스템 오류

- `INTERNAL_ERROR`

공통 문서에서는 여기까지 정의하고, 서비스별 세부 코드는 각 레포 wiki에서 확장한다.

```kotlin
enum class ErrorCode(
    val status: HttpStatus,
    val code: String,
    val message: String,
) {
    VALIDATION_ERROR(HttpStatus.BAD_REQUEST, "VALIDATION_ERROR", "요청값 검증에 실패했습니다."),
    INPUT_ERROR(HttpStatus.BAD_REQUEST, "INPUT_ERROR", "요청값 형식이 올바르지 않습니다."),
    JSON_PARSE_ERROR(HttpStatus.BAD_REQUEST, "JSON_PARSE_ERROR", "요청 본문(JSON) 형식이 올바르지 않습니다."),
    ENUM_MISMATCH(HttpStatus.BAD_REQUEST, "ENUM_MISMATCH", "허용되지 않은 enum 값입니다."),
    MISSING_REQUIRED_VALUE(HttpStatus.BAD_REQUEST, "MISSING_REQUIRED_VALUE", "필수 요청값이 누락되었습니다."),
    BAD_REQUEST(HttpStatus.BAD_REQUEST, "BAD_REQUEST", "잘못된 요청입니다."),
    UNAUTHORIZED(HttpStatus.UNAUTHORIZED, "UNAUTHORIZED", "인증이 필요하거나 토큰이 유효하지 않습니다."),
    AUTH_TOKEN_EXPIRED(HttpStatus.UNAUTHORIZED, "AUTH_TOKEN_EXPIRED", "토큰이 만료되었습니다."),
    AUTH_TOKEN_INVALID(HttpStatus.UNAUTHORIZED, "AUTH_TOKEN_INVALID", "토큰이 올바르지 않습니다."),
    FORBIDDEN(HttpStatus.FORBIDDEN, "FORBIDDEN", "요청을 수행할 권한이 없습니다."),
    NOT_FOUND(HttpStatus.NOT_FOUND, "NOT_FOUND", "요청한 리소스를 찾을 수 없습니다."),
    METHOD_NOT_ALLOWED(HttpStatus.METHOD_NOT_ALLOWED, "METHOD_NOT_ALLOWED", "허용되지 않은 HTTP 메서드입니다."),
    CONFLICT(HttpStatus.CONFLICT, "CONFLICT", "리소스 충돌이 발생했습니다."),
    UNSUPPORTED_MEDIA_TYPE(HttpStatus.UNSUPPORTED_MEDIA_TYPE, "UNSUPPORTED_MEDIA_TYPE", "지원하지 않는 Content-Type 입니다."),
    UNPROCESSABLE_ENTITY(HttpStatus.UNPROCESSABLE_ENTITY, "UNPROCESSABLE_ENTITY", "현재 상태에서는 요청을 처리할 수 없습니다."),
    FILE_UPLOAD_FAILED(HttpStatus.BAD_REQUEST, "FILE_UPLOAD_FAILED", "파일 업로드에 실패했습니다."),
    IMAGE_TOO_LARGE(HttpStatus.BAD_REQUEST, "IMAGE_TOO_LARGE", "이미지 용량이 허용 범위를 초과했습니다."),
    IMAGE_UNSUPPORTED_FORMAT(HttpStatus.BAD_REQUEST, "IMAGE_UNSUPPORTED_FORMAT", "지원하지 않는 이미지 형식입니다."),
    SMTP_SEND_FAILED(HttpStatus.SERVICE_UNAVAILABLE, "SMTP_SEND_FAILED", "메일 전송에 실패했습니다."),
    EXTERNAL_API_FAILED(HttpStatus.SERVICE_UNAVAILABLE, "EXTERNAL_API_FAILED", "외부 연동 처리에 실패했습니다."),
    UPSTREAM_TIMEOUT(HttpStatus.SERVICE_UNAVAILABLE, "UPSTREAM_TIMEOUT", "외부 서비스 응답 시간이 초과되었습니다."),
    TOO_MANY_REQUESTS(HttpStatus.TOO_MANY_REQUESTS, "TOO_MANY_REQUESTS", "요청이 너무 많습니다."),
    INTERNAL_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "INTERNAL_ERROR", "서버 내부 오류가 발생했습니다."),
    SERVICE_UNAVAILABLE(HttpStatus.SERVICE_UNAVAILABLE, "SERVICE_UNAVAILABLE", "일시적으로 서비스를 사용할 수 없습니다."),
}
```

### 설계 규칙

- `code`는 외부 계약으로 본다.
- 공통 코드 중심으로 시작하고, 도메인별 세분화는 꼭 필요할 때만 한다.
- 내부 예외 원문, SQL, stack trace, 토큰 값은 응답에 노출하지 않는다.
- 공통 코드만으로 부족한 도메인 전용 케이스는 각 서비스 wiki에 정의하되, 공통 코드와 중복되지 않게 설계한다.

---

## 6. 예외 계층 구조 권장안

```kotlin
open class ApiException(
    val errorCode: ErrorCode,
    override val message: String? = errorCode.message,
) : RuntimeException(message)
```

상태별 예외:

```kotlin
class BadRequestException(message: String) : ApiException(ErrorCode.BAD_REQUEST, message)
class UnauthorizedException(message: String = ErrorCode.UNAUTHORIZED.message) : ApiException(ErrorCode.UNAUTHORIZED, message)
class ForbiddenException(message: String = ErrorCode.FORBIDDEN.message) : ApiException(ErrorCode.FORBIDDEN, message)
class NotFoundException(message: String) : ApiException(ErrorCode.NOT_FOUND, message)
class ConflictException(message: String) : ApiException(ErrorCode.CONFLICT, message)
class UnprocessableEntityException(message: String) : ApiException(ErrorCode.UNPROCESSABLE_ENTITY, message)
```

---

## 7. `ErrorResponseFactory` 책임 정의

`ErrorResponseFactory`는 예외를 받아 응답 body를 만드는 단일 진입점이어야 한다.

### 책임

1. `ErrorCode`로부터 HTTP 상태와 기본 메시지 결정
2. override message가 있으면 우선 사용
3. `success=false`, `data=null`, `error!=null` 형태 보장
4. `timestamp` 생성
5. 필요 시 `details` 주입
6. requestId를 헤더와 로그 문맥에서 동일하게 유지

예시:

```kotlin
data class ErrorResponse(
    val success: Boolean = false,
    val data: Any? = null,
    val error: ErrorBody,
    val timestamp: OffsetDateTime = OffsetDateTime.now(),
)

data class ErrorBody(
    val code: String,
    val message: String,
    val details: List<Map<String, Any?>>? = null,
)

object ErrorResponseFactory {
    fun of(errorCode: ErrorCode, message: String? = null): ErrorResponse {
        return ErrorResponse(
            error = ErrorBody(
                code = errorCode.code,
                message = message ?: errorCode.message,
            )
        )
    }
}
```

---

## 8. 전역 예외처리기(`@RestControllerAdvice`) 매핑 표준

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

예시:

```kotlin
@RestControllerAdvice
class GlobalExceptionHandler {

    @ExceptionHandler(ApiException::class)
    fun handleApiException(ex: ApiException): ResponseEntity<ErrorResponse> {
        val body = ErrorResponseFactory.of(ex.errorCode, ex.message)
        return ResponseEntity.status(ex.errorCode.status).body(body)
    }

    @ExceptionHandler(MethodArgumentNotValidException::class)
    fun handleValidation(ex: MethodArgumentNotValidException): ResponseEntity<ErrorResponse> {
        val firstError = ex.bindingResult.fieldErrors.firstOrNull()
        val message = firstError?.let { "${it.field}: ${it.defaultMessage}" }
            ?: ErrorCode.VALIDATION_ERROR.message

        return ResponseEntity.badRequest()
            .body(ErrorResponseFactory.of(ErrorCode.VALIDATION_ERROR, message))
    }

    @ExceptionHandler(HttpMessageNotReadableException::class)
    fun handleUnreadable(ex: HttpMessageNotReadableException): ResponseEntity<ErrorResponse> =
        ResponseEntity.badRequest()
            .body(ErrorResponseFactory.of(ErrorCode.JSON_PARSE_ERROR))

    @ExceptionHandler(AccessDeniedException::class)
    fun handleAccessDenied(ex: AccessDeniedException): ResponseEntity<ErrorResponse> =
        ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(ErrorResponseFactory.of(ErrorCode.FORBIDDEN))

    @ExceptionHandler(Exception::class)
    fun handleUnexpected(ex: Exception): ResponseEntity<ErrorResponse> =
        ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ErrorResponseFactory.of(ErrorCode.INTERNAL_ERROR))
}
```

---

## 9. Swagger/OpenAPI 문서화 기준

`SwaggerConfig.kt`에서는 공통 오류 응답을 재사용 가능하게 등록한다.

### 목표

- 모든 API가 같은 에러 스키마를 사용한다고 문서화
- 자주 쓰는 `400/401/403/404/409/422/500`을 공통 response component로 등록
- Controller에서는 필요한 것만 참조

### 공통 스키마 항목

- `ApiResponse<T>`
- `ErrorResponse`
- `ErrorBody`
- `ValidationDetail`

### 공통 response 예시

- `BadRequestResponse`
- `UnauthorizedResponse`
- `ForbiddenResponse`
- `NotFoundResponse`
- `ConflictResponse`
- `UnprocessableEntityResponse`
- `InternalServerErrorResponse`

---

## 10. 프론트엔드 계약 가이드

프론트는 아래 순서로 처리하는 것을 권장한다.

1. HTTP 상태코드를 먼저 본다.
2. body의 `success === false`를 확인한다.
3. 세부 분기는 `error.code` 기준으로 한다.
4. 사용자 표시 문구는 `error.message`를 사용한다.
5. `details`가 있으면 필드별 오류 UI에 연결한다.

권장 분기:

- `401 UNAUTHORIZED` -> 재로그인 / 토큰 갱신
- `403 FORBIDDEN` -> 권한 없음 안내
- `404 NOT_FOUND` -> 없는 리소스 안내
- `409 CONFLICT` -> 중복/충돌 수정 유도
- `422 UNPROCESSABLE_ENTITY` -> 선행 상태 점검 유도
- `500 INTERNAL_ERROR` -> 일반 오류 토스트 + `X-Request-Id` 수집

---

## 11. 로깅/모니터링 기준

### 로그 레벨

- `400`, `401`, `403`, `404`, `409`, `422` -> `warn` 또는 `info`
- `500`, `503` -> `error`

### 반드시 남길 로그 항목

- `requestId`
- URI / HTTP method
- 사용자 식별자(가능한 경우)
- 에러 코드
- HTTP 상태코드
- 예외 클래스명
- stack trace (`5xx`는 필수)

### 남기면 안 되는 값

- Access Token 전체값
- Refresh Token 전체값
- 비밀번호
- 민감 개인정보
- 대용량 원본 body 전체

---

## 12. 구현 순서 제안

1. `ErrorCode` 정비
2. 공통 예외 클래스 정비
3. `ErrorResponseFactory` 일원화
4. `GlobalExceptionHandler` 정비
5. 서비스/도메인 예외 교체
6. Swagger 공통 응답 등록
7. 테스트 추가

---

## 13. 테스트 체크리스트

- 상태코드가 기대값과 일치하는가
- `success=false`가 들어가는가
- `data=null`인가
- `error.code`가 계약과 일치하는가
- `error.message`가 사용자 노출 가능 문구인가
- `timestamp`가 들어가는가
- `X-Request-Id`가 응답 헤더에 존재하는가
- Validation 오류 시 `details` 형식이 일관적인가

---

## 14. 최종 권장 표준 요약

### 반드시 지킬 것

- 오류 응답은 모두 같은 JSON envelope 사용
- HTTP 상태코드와 `error.code`를 의미에 맞게 분리
- 서비스 계층은 도메인 예외를 던지고, HTTP 변환은 전역 예외처리기에서 수행
- 프론트 계약은 `error.code` 기준으로 안정적으로 유지
- `500`에서는 내부 상세를 노출하지 않음
- `X-Request-Id`를 응답과 로그에 함께 남김

### 공통 권장 코드 세트

- `VALIDATION_ERROR`
- `INPUT_ERROR`
- `JSON_PARSE_ERROR`
- `ENUM_MISMATCH`
- `MISSING_REQUIRED_VALUE`
- `BAD_REQUEST`
- `UNAUTHORIZED`
- `AUTH_TOKEN_EXPIRED`
- `AUTH_TOKEN_INVALID`
- `FORBIDDEN`
- `NOT_FOUND`
- `METHOD_NOT_ALLOWED`
- `CONFLICT`
- `UNSUPPORTED_MEDIA_TYPE`
- `UNPROCESSABLE_ENTITY`
- `FILE_UPLOAD_FAILED`
- `IMAGE_TOO_LARGE`
- `IMAGE_UNSUPPORTED_FORMAT`
- `SMTP_SEND_FAILED`
- `EXTERNAL_API_FAILED`
- `UPSTREAM_TIMEOUT`
- `SERVICE_UNAVAILABLE`
- `INTERNAL_ERROR`

---

## 15. 결론

백엔드 공통 예외처리는 아래 5축으로 맞추는 것이 가장 안정적이다.

- `ErrorCode`
- `ApiException`
- `ErrorResponseFactory`
- `GlobalExceptionHandler`
- `Swagger 공통 응답 정의`

이 기준을 맞추면:

- 백엔드는 예외 처리 중복을 줄일 수 있고
- 프론트는 안정적으로 오류를 분기할 수 있으며
- 운영은 `X-Request-Id` 기준으로 장애를 추적하기 쉬워진다.

---

## Related Docs

- [Backend Docs Index](./README.md)
- [Backend API Naming Guide (A&I)](./API_NAMING.md)
- [Auth Error-Model-v1](https://github.com/Team-AnI/A-AND-I-AUTH-SERVER/wiki/Error-Model-v1)
- [Report Error-Model-v1](https://github.com/Team-AnI/A-AND-I-REPORT-SERVER/wiki/Error-Model-v1)
- [Web Error-Model-v1](https://github.com/Team-AnI/A-AND-I-WEB-SERVER/wiki/Error-Model-v1)
- [Gateway Error-Model-v1](https://github.com/Team-AnI/A-AND-I-GATEWAY-SERVER/wiki/Error-Model-v1)
- [Blog Error-Model-v1](https://github.com/Team-AnI/A-AND-I-TECH-BLOG-SERVER/blob/main/wiki/Error-Model-v1.md)
