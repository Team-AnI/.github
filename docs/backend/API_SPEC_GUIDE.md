# Backend API Specification Guide (A&I)

이 문서는 API 명세서를 처음 작성하는 백엔드 개발자를 위한 공통 온보딩 가이드입니다.
PRD / ERD / 화면에서 필요한 API를 찾고, 개발 전에 프론트엔드와 백엔드가 맞출 최소 계약을 작성하는 방법을 설명합니다.

처음부터 완벽한 명세를 만들려고 하지 않습니다. 구현과 테스트 이후 Swagger/OpenAPI와 비교하여 명세를 구체화하고, API 문서와 실제 코드의 차이를 줄입니다.

## Quick Links

- [Backend Docs Index](./README.md)
- [Backend API Naming Guide (A&I)](./API_NAMING.md)
- [Backend API Common Exception Handling Guide v1](./COMMON_EXCEPTION_HANDLING_GUIDE_V1.md)

---

## 0) 문서 범위

- `.github/docs/backend`: 모든 백엔드 프로젝트가 공통으로 참고하는 규칙과 작성 흐름을 둡니다.
- 각 서비스 Repository Wiki: 실제 API 상세 명세, 도메인별 정책, 서비스별 에러 코드를 관리합니다.

이 문서의 Template을 이용해 작성한 **서비스별 API Spec은 해당 서비스 Wiki에 저장**합니다. 아래 `groups` 경로, 필드, 인증 조건은 모두 **설명용 예시**이며 Team-AnI 서비스의 실제 API나 새로운 Organization 정책을 정의하지 않습니다.

---

## 1) API 명세의 작성 흐름

```text
PRD / 요구사항
    ↓
ERD / 도메인 관계
    ↓
Wireframe / 사용자 행동
    ↓
필요한 API 후보 추출
    ↓
초기 API 계약 작성
    ↓
Backend / Frontend 구현
    ↓
테스트
    ↓
Swagger / OpenAPI 검증
    ↓
최종 API 명세 확정
```

### 초기 API 명세

개발 전 프론트엔드와 백엔드가 서로 작업할 수 있도록 만드는 **최소 계약**입니다.
모든 예외와 Response를 처음부터 완벽하게 확정하는 것이 목적은 아닙니다.

최소한 아래 내용을 합의합니다.

- 기능
- HTTP Method
- Endpoint
- 인증/권한 여부
- Request
- Response
- 주요 HTTP Status

필드의 Type과 Required 여부도 함께 적어 서로 다르게 구현하지 않도록 합니다. 아직 결정되지 않은 제약이나 예외는 Notes에 확인할 내용으로 남깁니다.

### 구현 이후 명세

실제 Controller, Request/Response DTO, Validation, Exception 처리와 테스트가 완성되면 Swagger/OpenAPI와 비교하여 상세 내용을 확정합니다. 구현 중 계약이 바뀌면 프론트엔드와 공유하고 초기 명세에도 반영합니다.

아래 두 접근은 피합니다.

```text
API 명세서를 전부 완성 → 절대 변경하지 않음 → 구현
코드부터 전부 구현 → 마지막에 Swagger를 보고 처음 API 문서를 작성
```

권장 접근은 다음과 같습니다.

```text
최소 API 계약 → 구현 → 변경사항 반영 → 테스트 → Swagger/OpenAPI와 맞춰 최종 명세
```

---

## 2) PRD / ERD / 화면에서 API 찾기

먼저 PRD와 현재 Issue에서 이번 개발 범위와 완료 조건을 읽습니다. ERD에서는 필요한 Resource와 관계를 확인하고, 화면에서는 사용자 행동을 하나씩 나눕니다. Entity나 화면 이름만 보고 API를 정하지 않습니다.

```text
화면이 있다
↓
사용자가 무엇을 하는가?
↓
서버 데이터가 필요한 행동인가?
↓
서버에서는 어떤 처리가 필요한가?
↓
어떤 Resource를 다루는가?
↓
HTTP Method + Endpoint 결정
```

각 행동에 대해 서버에서 조회할 데이터, 저장할 데이터, 바꿀 상태가 있는지 적습니다. ERD의 관계는 어떤 Resource를 연결하거나 식별해야 하는지 판단할 때 사용합니다.

**설명용 예시:**

```text
사용자가 모임 이름과 설명을 입력한다
↓
"모임 생성" 버튼을 누른다
↓
서버에 새로운 모임을 저장해야 한다
↓
Group Resource 생성
↓
POST /v1/groups
```

### 화면의 모든 버튼이 API는 아니다

탭 이동, Modal 열기, 입력값의 화면 내부 변경은 Frontend Local State만으로 처리할 수 있으므로 API가 필요하지 않을 수 있습니다. 해당 행동에서 서버 데이터를 읽거나 저장해야 하는지 먼저 확인합니다.

### 화면 하나에 API 하나라는 규칙도 없다

한 화면에서도 목록 조회, 상세 조회, 생성, 상태 변경처럼 서로 다른 서버 작업이 필요하면 여러 API를 사용할 수 있습니다. 화면 개수보다 실제 서버 작업을 기준으로 API 후보를 나눕니다.

### CRUD를 기계적으로 만들지 않는다

Entity가 존재한다고 해서 `POST`, `GET`, `PATCH`, `DELETE`를 모두 만들지 않습니다. PRD와 실제 Use Case에서 필요한 동작만 정의합니다.

예를 들어 이번 Sprint Scope가 공지 작성뿐이라면 `POST /...`가 필요하다는 이유만으로 삭제 API까지 미리 설계하지 않습니다.

> "PRD 또는 현재 Issue에서 이 API가 실제로 필요한가?"

필요하지 않다면 미래 확장을 예상하여 먼저 만들지 않습니다.

---

## 3) API 하나를 설계하기 전 5가지 질문

1. 누가 사용하는 API인가?
2. 어떤 사용자 행동에서 호출되는가?
3. 어떤 Resource를 다루는가?
4. 클라이언트가 서버에 어떤 데이터를 보내야 하는가?
5. 성공 후 클라이언트가 어떤 데이터를 필요로 하는가?

이 질문에 답한 다음 Method와 Endpoint를 정합니다. Endpoint 네이밍은 [API Naming Guide](./API_NAMING.md)를 따릅니다.

---

## 4) 초기 API 목록 작성

상세 명세부터 쓰기보다 API 후보를 요약표로 작성합니다. 기능이 현재 범위에 필요한지, 서버 작업이 겹치지 않는지 프론트엔드와 확인한 뒤 실제 구현할 API를 선택합니다.

아래는 **설명용 예시**입니다. 세 기능이 모두 요구사항에 포함되어 있다고 가정하며, 실제 서비스의 API나 인증 정책을 뜻하지 않습니다.

| 기능 | Method | Endpoint | 설명 | 인증 |
| --- | --- | --- | --- | --- |
| 모임 생성 | POST | `/v1/groups` | 이름과 설명으로 모임 생성 | Required |
| 모임 목록 조회 | GET | `/v1/groups` | 화면에 표시할 모임 목록 조회 | Required |
| 모임 상세 조회 | GET | `/v1/groups/{groupId}` | 선택한 모임의 상세 정보 조회 | Required |

---

## 5) 상세 API 명세 Template

초기 목록에서 실제 구현할 API를 선택한 뒤 아래 Template으로 상세화합니다. Method와 성공 Status는 해당 작업에 맞게 바꿉니다.

빈 Section을 반드시 남길 필요는 없습니다. 사용하지 않는 Path/Query Parameters나 Request Body는 생략할 수 있습니다. 필드의 Type, Required 여부, 필요한 Validation 조건은 표나 설명으로 함께 적습니다.

````md
## [기능명]

### Endpoint

`POST /v1/...`

### Description

이 API가 수행하는 작업을 한두 문장으로 설명합니다.

### Authorization

- Required / Not Required
- 필요한 역할이 있다면 명시

### Path Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |

### Query Parameters

| Name | Type | Required | Description |
| --- | --- | --- | --- |

### Request Body

```json
{
}
```

### Response

#### Success

Status: `201 Created`

```json
{
}
```

### Error

| HTTP Status | Error Code | Description |
| --- | --- | --- |

### Notes

추가 제약사항 또는 구현 시 확인할 내용을 작성합니다.
````

Template의 빈 JSON은 작성 위치를 나타냅니다. 실제 응답은 공통 예외 처리 가이드의 Envelope를 사용합니다.

---

## 6) Request / Response 작성 기준

초기 명세에서는 필요한 필드부터 정의합니다.

- Request: **서버가 해당 작업을 수행하기 위해 실제 필요한 값**을 포함합니다.
- Response: **다음 화면이나 Client State에서 실제 필요한 값**을 우선 포함합니다.

Entity의 모든 Column을 그대로 Request/Response에 노출하지 않습니다. API 계약은 클라이언트와 주고받을 데이터를 정의하고, Database Entity는 저장 구조를 정의합니다.

**설명용 예시:** DB Entity에 아래 Column이 있다고 가정합니다.

```text
id
name
description
createdAt
updatedAt
deletedAt
```

그렇다고 생성 Request를 아래처럼 만들지 않습니다.

```json
{
  "id": 1,
  "name": "...",
  "description": "...",
  "createdAt": "...",
  "updatedAt": "...",
  "deletedAt": null
}
```

이 예시에서 식별자와 생성/수정 시각은 서버가 결정합니다. 클라이언트는 생성에 필요한 `name`, `description`만 보내고, 서버는 성공 후 화면에 필요한 값을 응답합니다. JSON 필드 네이밍과 날짜/일시 표현은 API Naming Guide를 따릅니다.

---

## 7) HTTP Method / Endpoint

[API Naming Guide](./API_NAMING.md)를 Source of Truth로 사용합니다. Method의 기본 의미는 다음과 같습니다.

- GET: 조회
- POST: 생성 또는 Trigger
- PATCH: 부분 수정 / 상태 변경
- PUT: 전체 치환이 필요한 경우
- DELETE: 삭제 또는 Archive

> URI 규칙, Prefix, Version, Path naming 등 상세 규칙은 `API_NAMING.md`를 우선한다.

설명용 경로를 실제 서비스에 그대로 적용하지 않고, 해당 서비스의 Prefix와 권한 범위를 기존 가이드에서 확인합니다.

---

## 8) Error Response

공통 응답 Envelope, HTTP Status 정책, Error Code taxonomy는 [Common Exception Handling Guide v1](./COMMON_EXCEPTION_HANDLING_GUIDE_V1.md)을 따릅니다. 별도의 Error Model을 만들지 않습니다.

API 명세에는 해당 Endpoint에서 실제 발생할 수 있는 대표적인 Domain Error와 요청 검증/인증 오류를 발생 조건과 함께 적습니다. 서비스 전용 코드와 도메인별 상세 정책은 해당 서비스 Wiki에서 정의합니다.

아래는 발생 조건을 검토하기 위한 예시이며, 모든 API에 무조건 적용하는 목록이 아닙니다.

```text
400 - 잘못된 Request
401 - 인증 필요
403 - 권한 없음
404 - Resource 없음
409 - 현재 상태와 충돌
```

실제로 발생하는 Error만 문서화합니다. 특히 중복/충돌(`409`)과 상태상 처리 불가(`422`)의 구분은 공통 가이드를 확인합니다. 공통 Swagger 응답을 재사용할 때도 해당 API에 필요한 항목만 참조합니다.

---

## 9) Swagger / OpenAPI의 역할

Swagger는 API를 설계해주는 도구가 아니라 **실제 구현된 API 계약을 확인하고 공유하는 도구**입니다. Swagger UI에서 표현되는 OpenAPI 내용이 코드와 일치하는지도 검증해야 합니다.

```text
초기 API 계약
↓
Controller / DTO 구현
↓
Validation / Exception 구현
↓
Test
↓
Swagger 확인
↓
초기 명세와 비교
↓
변경된 계약 반영
↓
명세 확정
```

Method, Endpoint, 인증/권한, 필드 Type과 Required 여부, Validation, 성공/오류 Status와 응답을 실제 코드·테스트 결과·Swagger/OpenAPI·서비스 Wiki 명세에서 비교합니다.

코드와 API 문서가 다르면 실제 구현을 확인한 뒤 **의도된 변경인지 검토**합니다. 의도된 계약 변경이면 프론트엔드와 공유하고 Swagger/OpenAPI와 Wiki를 동기화합니다. 구현이 합의한 계약에서 벗어난 경우에는 구현을 수정하고 다시 테스트합니다. 검증된 계약으로 최종 명세를 확정하며, 이후 변경에도 문서를 함께 갱신합니다.

---

## 10) API 변경 시 원칙

API는 프론트엔드와 백엔드 사이의 계약입니다. 다음 변경은 사용하는 화면과 Client State에 미치는 영향을 확인합니다.

- Endpoint 변경
- HTTP Method 변경
- Request Field 변경
- Response Field 변경
- Field Type 변경
- Required 여부 변경
- HTTP Status 변경
- Error Code 변경

이미 Client가 사용하는 API를 수정할 때는 **Backend 내부 구현 변경**과 **외부 API Contract 변경**을 구분합니다. 내부 로직을 바꿔도 요청과 응답 계약이 같다면 내부 구현 변경입니다. 클라이언트가 보내거나 받는 값, 호출 방식, 오류 분기가 달라지면 Contract 변경입니다.

Contract가 변경되면 프론트엔드와 먼저 공유하여 영향과 적용 시점을 맞추고 API 문서도 함께 수정합니다. 버전이나 레거시 호환 처리의 상세 규칙은 API Naming Guide를 따릅니다.

---

## 11) API 명세 작성 예제

아래는 하나의 생성 API를 상세화하는 **설명용 예시**입니다. 로그인한 사용자가 모임 이름과 설명을 입력해 생성하며, 성공 후 클라이언트가 생성된 모임의 식별자와 이름을 사용한다고 가정합니다. 필드의 필수 여부와 오류 조건도 이 예시의 가정입니다.

### 사용자 행동 → 필요한 API

```text
이름과 설명 입력 → 모임 생성 버튼 클릭 → 서버에 모임 저장 → POST /v1/groups
```

### 초기 요약

| 기능 | Method | Endpoint | 설명 | 인증 |
| --- | --- | --- | --- | --- |
| 모임 생성 | POST | `/v1/groups` | 입력한 이름과 설명으로 모임 생성 | Required |

### 상세 API 명세: 모임 생성

#### Endpoint

`POST /v1/groups`

#### Description

로그인한 사용자가 입력한 이름과 설명으로 새 모임을 생성합니다.

#### Authorization

- Required
- 이 예시에서는 로그인한 사용자에게 추가 역할을 요구하지 않습니다.

#### Request Body

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| name | string | Yes | 모임 이름. 공백만 입력할 수 없음 |
| description | string | Yes | 모임 설명 |

```json
{
  "name": "백엔드 스터디",
  "description": "API 설계를 함께 공부하는 모임"
}
```

#### Response

##### Success

Status: `201 Created`

```json
{
  "success": true,
  "data": {
    "groupId": 1,
    "name": "백엔드 스터디"
  },
  "error": null,
  "timestamp": "2026-03-11T10:00:00+09:00"
}
```

`data.groupId`는 생성된 모임의 식별자(number), `data.name`은 모임 이름(string)이며 이 예시에서는 둘 다 반환합니다. 공통 Envelope는 기존 공통 예외 처리 가이드의 성공 응답 구조를 따릅니다.

#### Error

| HTTP Status | Error Code | Description |
| --- | --- | --- |
| 400 | `VALIDATION_ERROR` | 이름이 공백이거나 필수 입력값이 누락되어 검증에 실패한 경우 |
| 401 | `UNAUTHORIZED` | 인증되지 않은 경우 |

위 표는 이 예시에서 합의한 주요 오류입니다. 구현 이후 파싱 오류 등 실제 발생하는 다른 오류도 공통 가이드에 따라 확인하여 반영합니다. 오류 응답 본문은 공통 가이드의 실패 Envelope를 사용합니다.

#### Notes

Path/Query Parameters는 사용하지 않아 생략했습니다. 이름과 설명의 길이 제한처럼 아직 합의되지 않은 제약은 구현 전에 확인하고, 확정되면 Validation과 명세에 함께 반영합니다. 구현과 테스트를 마친 뒤 Swagger/OpenAPI에서 위 계약과 일치하는지 확인합니다.

---

## 12) API 명세 작성 체크리스트

### 개발 전

- [ ] PRD / Issue의 개발 범위를 확인했는가?
- [ ] 사용자 행동을 기준으로 API를 추출했는가?
- [ ] 실제 서버 처리가 필요한 기능인가?
- [ ] 필요하지 않은 CRUD를 미리 만들지 않았는가?
- [ ] API Naming Guide를 확인했는가?
- [ ] Frontend와 필요한 최소 Request/Response를 합의했는가?

### 구현 중

- [ ] 실제 DTO가 초기 API 계약과 달라진 부분은 없는가?
- [ ] Validation 조건이 변경되지 않았는가?
- [ ] HTTP Status가 변경되지 않았는가?
- [ ] Error Response가 추가되거나 변경되지 않았는가?

### 구현 완료

- [ ] 테스트가 정상 통과하는가?
- [ ] Swagger에서 Request가 실제 구현과 일치하는가?
- [ ] Swagger에서 Response가 실제 구현과 일치하는가?
- [ ] Error Response가 문서화되어 있는가?
- [ ] 서비스별 API Spec 문서가 최신 상태인가?
- [ ] Frontend와 변경사항을 공유했는가?

---

## References

- [GitHub REST API Documentation](https://docs.github.com/en/rest): Method / Endpoint / Parameters / Request / Response 구조를 참고하기 좋은 실제 API 문서입니다.
- [Swagger Petstore](https://petstore.swagger.io/): Swagger UI에서 API 문서가 어떤 구조로 표현되는지 확인하기 좋은 예제입니다.
- [Stripe API Reference](https://docs.stripe.com/api): 실제 서비스의 상세 API Reference 구성을 참고하기 좋은 예시입니다.

외부 자료는 문서 작성 방식을 참고하기 위한 자료입니다. 외부 자료의 규칙을 Team-AnI의 API Naming Guide나 공통 예외 처리 규칙보다 우선하지 않습니다.
