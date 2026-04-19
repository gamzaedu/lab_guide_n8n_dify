---
title: "Lab 4. HTTP Request 노드로 외부 API 호출"
description: "HTTP Request 노드로 공개 테스트 API 를 호출하고, GET / POST, Query / Header / Body 를 실전으로 다룹니다."
tags:
  - n8n
  - 기초실습
  - HTTP
  - API
---

# Lab 4. HTTP Request 노드로 외부 API 호출

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | HTTP Request 노드로 외부 API 를 GET/POST 로 호출하고, Query Parameter / Header / Body / JSON 응답을 다룰 수 있다 |
| 핵심 노드 | Manual Trigger, HTTP Request, Edit Fields (Set) |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 01~03 학습 완료, 인터넷 연결 |
| 난이도 | ★★☆☆☆ |

!!! note "이번 실습에서 사용할 공개 테스트 API (모두 무료·인증 불필요)"
    - `https://api.github.com/zen` — GitHub 랜덤 명언 (짧은 텍스트 응답)
    - `https://jsonplaceholder.typicode.com/users/1` — 가상 사용자 1명의 상세 JSON
    - `https://httpbin.org/post` — 보낸 요청을 그대로 에코해주는 엔드포인트

---

### Step 1. 탐색하기 (Exploring)

??? note "해당 단계 사용법 살펴보기"

    !!! tip "원칙"
        남이 알려준 정답보다 나의 시도가 더 가치 있습니다.

    **[의미]**

    수업에서 배운 내용을 직접 적용해보는 단계입니다.

    **[지침]**

    - 가능한 직접 손으로 타이핑해보고 직접 생각해봅니다.
        - 5분 정도는 AI 에서 손을 떼고 펜을 들어보세요!
    - 한번의 완벽한 결과보다 다양한 시도를 통한 점진적 개선을 추구합니다.
    - 실패를 두려워하지 마세요. 빠른 실패는 오히려 빠른 성장을 가져옵니다!

!!! abstract "과제"
    Manual Trigger 뒤에 **HTTP Request 노드** 를 연결하고, `GET https://api.github.com/zen` 을 호출하여 랜덤 명언 텍스트를 받아 Output 패널에서 확인하세요. 실행을 3~4회 반복하여 응답이 매번 달라지는지 관찰하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 04 - HTTP Request`
- **HTTP Request 노드 설정:**

| 항목 | 값 |
|------|-----|
| Method | `GET` |
| URL | `https://api.github.com/zen` |
| Authentication | `None` |
| Send Query Parameters | OFF |
| Send Headers | OFF |
| Send Body | OFF |

**구성 단계:**

1. 새 워크플로우 생성 → 이름 변경
2. **Manual Trigger** 추가
3. **HTTP Request** 노드 연결, 위 표대로 설정
4. **[Execute Workflow]** 실행, Output 패널의 `data` 필드에 짧은 영어 문장이 담겨 있는지 확인
5. 3~4회 재실행하여 문장이 매번 달라지는지 확인

??? note "예시"

    > **Output JSON (한 번의 실행 결과)**
    >
    > ```json
    > [
    >   {
    >     "data": "Design for failure."
    >   }
    > ]
    > ```
    >
    > **재실행 시 예시:**
    >
    > ```json
    > [ { "data": "Non-blocking is better than blocking." } ]
    > [ { "data": "Approachable is better than simple." } ]
    > ```
    >
    > **확인 포인트:** `data` 필드 값이 매번 달라지면 GitHub Zen API 가 랜덤 응답을 반환하고 있다는 뜻입니다. 응답이 짧은 plain text 라서 n8n 이 자동으로 `data` 키로 감싸 줍니다.

---

### Step 2. 분석과 성찰 (Analyzing & Reflecting)

??? note "해당 단계 사용법 살펴보기"

    !!! tip "원칙"
        객관적인 피드백을 구하되, 판단은 스스로 합니다.

    **[의미]**

    Step 1 에서 만든 결과물을 한 발 물러서서 객관적으로 바라보는 단계입니다.
    혼자서는 보이지 않던 것들이, 다른 시선을 통해 드러납니다.

    **[지침]**

    - 1단계 작업물에 대해, 객관적인 피드백을 얻습니다
    - 다른 탐색하기 결과물을 확인합니다
        - 다른 동료의 작업 결과물 | AI 의 생성 결과물 | 웹검색 등
    - AI 를 사용할 땐 반드시 비판적인 자세를 유지합니다.
        - AI 가 생성한 결과물 및 지침은 그대로 받아들이지 않고, 의심합니다.

!!! abstract "과제"
    URL 을 `https://jsonplaceholder.typicode.com/users/1` 로 바꿔 복잡한 JSON 응답을 받고, 그 뒤에 **Set 노드** 를 연결하여 `name` / `email` / `city` 3개 필드만 **추출**하세요. Output 을 Step 1(짧은 텍스트)과 비교하여 "응답 구조가 복잡할수록 노드 구성이 어떻게 달라져야 하는지" 정리하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에 Set 노드를 추가 연결
- **HTTP Request 노드 변경:**

| 항목 | 값 |
|------|-----|
| URL | `https://jsonplaceholder.typicode.com/users/1` |

- **Set 노드 (`Include Other Input Fields` = OFF, Manual Mapping):**

| Field Name | Type | Value (Expression) |
|------------|------|--------------------|
| name | String | `={{ $json.name }}` |
| email | String | `={{ $json.email }}` |
| city | String | `={{ $json.address.city }}` |

**관찰 포인트:**

1. HTTP Request 의 Output 은 중첩 구조(예: `address.city`, `company.name`)를 가진다
2. Set 노드에서 Dot Notation (`$json.address.city`)으로 내부 값에 접근한다
3. Set 뒤 단계는 깔끔한 3-필드 구조로 정리된다 — 실무에서 매우 자주 쓰는 패턴

**(선택) 응답 구조 분석 프롬프트:**

```
아래는 JSONPlaceholder 에서 받은 Output 이야.
실무에서 뒷 단계(예: 메일 전송, Sheets 기록)에 쓸 때
꼭 필요한 필드 3~5개만 추려줘. 그리고 그 이유도 설명해줘.

[Output] ...
```

**예시**

> **HTTP Request 노드 Output (일부 발췌)**
>
> ```json
> {
>   "id": 1,
>   "name": "Leanne Graham",
>   "username": "Bret",
>   "email": "Sincere@april.biz",
>   "address": {
>     "street": "Kulas Light",
>     "city": "Gwenborough",
>     "zipcode": "92998-3874"
>   },
>   "phone": "1-770-736-8031",
>   "company": { "name": "Romaguera-Crona" }
> }
> ```
>
> **Set 노드로 추출 후 Output**
>
> ```json
> [
>   {
>     "name": "Leanne Graham",
>     "email": "Sincere@april.biz",
>     "city": "Gwenborough"
>   }
> ]
> ```
>
> **비교 분석표 — 응답 구조 복잡도 vs 필요한 후처리**
>
> | 엔드포인트 | 응답 형태 | 후처리 필요성 | 권장 노드 구성 |
> |----------|---------|--------------|--------------|
> | `/zen` | plain text 한 줄 | 거의 없음 | HTTP Request 만으로 OK |
> | `/users/1` | 중첩 JSON 10+ 필드 | 필요 필드 추출 필수 | HTTP Request → Set(추출) |
>
> **분석 결과:**
>
> - 외부 API 응답은 대부분 "필요 이상으로 많은 필드"를 포함한다 → **Set 노드로 좁혀 두는 것이 기본 패턴**
> - 중첩 오브젝트는 Dot Notation 으로 접근: `$json.address.city`
> - 응답 구조를 먼저 Output 패널에서 눈으로 확인한 뒤 Expression 을 쓰는 습관이 중요

---

### Step 3. 창조하기 (Creating)

??? note "해당 단계 사용법 살펴보기"

    !!! tip "원칙"
        완성품이 아니라, 작동하는 시작품을 만듭니다.

    **[의미]**

    앞서 얻은 통찰을 바탕으로, 나만의 결과물을 만들어 봅니다.

    **[지침]**

    - 2단계에서 얻은 피드백을 원하는 만큼 반영합니다.
        - 중요한 점은, 받아들일 수 있는 만큼만 받아들이는 것입니다.
    - 내 상황과 스타일에 맞게 최적화된 결과물을 만들어 봅니다.
    - 처음부터 완성하는 것이 아닌 계속해서 개선해 나가는 것입니다.
        - 가장 작은 단위부터 프로토타입으로 만들어 보세요

!!! abstract "과제"
    **POST 요청** 을 실습하세요. `https://httpbin.org/post` 에 JSON Body 를 보내고, 사용자 정의 Header(`X-Source: n8n-lab`) 를 함께 전송하세요. httpbin 은 받은 요청을 그대로 에코해주므로, Output 의 `json`·`headers` 필드에서 본인이 보낸 데이터가 정확히 되돌아오는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우 또는 Step 1~2의 HTTP Request 노드 재활용
- **HTTP Request 노드 설정:**

| 항목 | 값 |
|------|-----|
| Method | `POST` |
| URL | `https://httpbin.org/post` |
| Send Headers | ON |
| Send Body | ON |
| Body Content Type | `JSON` |
| Specify Body | `Using Fields Below` |

- **Headers:**

| Name | Value |
|------|-------|
| X-Source | `n8n-lab` |

- **Body Parameters:**

| Name | Value |
|------|-------|
| product | `Notebook` |
| quantity | `2` |

**구성 순서:**

1. HTTP Request 노드에서 Method 를 `POST` 로, URL 을 변경
2. **Send Headers** 토글 ON → Header 1개 추가
3. **Send Body** 토글 ON → Body Content Type `JSON`, Body 방식은 `Using Fields Below` 선택 후 필드 2개 입력
4. **[Execute Workflow]** 실행
5. Output 의 `json`(내가 보낸 Body), `headers`(내가 보낸 Header), `url`(호출 URL) 을 확인

**예시**

> **httpbin 응답 Output (일부 발췌)**
>
> ```json
> {
>   "args": {},
>   "data": "{\"product\":\"Notebook\",\"quantity\":\"2\"}",
>   "headers": {
>     "Content-Type": "application/json",
>     "Host": "httpbin.org",
>     "X-Source": "n8n-lab",
>     "User-Agent": "axios/..."
>   },
>   "json": {
>     "product": "Notebook",
>     "quantity": "2"
>   },
>   "url": "https://httpbin.org/post"
> }
> ```
>
> **확인 포인트:**
>
> - `json.product` = `"Notebook"`, `json.quantity` = `"2"` — 내가 보낸 Body 가 그대로 돌아옴
> - `headers.X-Source` = `"n8n-lab"` — 내가 추가한 Header 가 서버에 도달함
> - `url` = `"https://httpbin.org/post"` — 호출한 엔드포인트가 확인됨
>
> POST 요청의 세 요소 (URL / Header / Body) 가 모두 정상 작동했음을 Output 으로 증명할 수 있습니다.

!!! tip ""
    결과가 만족스럽지 않다면 Step 1이나 Step 2로 돌아가 반복할 수 있습니다.

---

### Beyond Level. 연구하기

!!! info ""
    ※ 해당 단계는 Step3를 일찍 완료하셨거나, 수업 후 자기 주도 학습을 해보시고 싶은 분들께 추천 드립니다.

??? note "해당 단계 사용법 살펴보기"

    !!! tip "원칙"
        더 좋은 답을 찾기 위해, 좋은 질문을 만들어 봅니다.

    **[의미]**

    실습의 가이드라인을 넘어, 스스로 질문을 만들고 탐구하는 단계입니다.

    **[지침]**

    - 실무에 적용할 때 어떤 점들을 조정해야 하는지 생각해 보세요.
    - 왜 좋고 왜 나쁠까? 라는 화두를 갖고, 더 깊은 레벨까지 탐구해 보세요

!!! abstract "과제"
    아래 탐구 질문 중 관심 있는 주제를 골라 직접 실험해보세요. 또는 실습 중 궁금했던 점을 스스로 질문으로 만들어 탐구해보세요.

**활용 도구 & 데이터**

자유롭게 활용

**예시**

> - **Query Parameter 와 Path Parameter 는 어떻게 다를까?**
>     - `https://jsonplaceholder.typicode.com/users?_limit=2` 처럼 Query 로 보내는 방식과, `https://jsonplaceholder.typicode.com/users/2` 처럼 Path 로 넣는 방식이 있다. `Send Query Parameters` 옵션을 켜서 두 방식의 응답 차이를 비교해보세요.
> - **Authentication 옵션의 종류(Basic / Header / OAuth2 / Predefined Credential)는 각각 언제 쓸까?**
>     - 각 옵션 화면을 열어 어떤 필드를 요구하는지 관찰하고, 실무 시나리오를 1~2개씩 떠올려보세요.
> - **응답이 배열(`[...]`)로 올 때는 어떻게 처리할까?**
>     - `https://jsonplaceholder.typicode.com/users` (s 붙음) 을 호출하면 사용자 10명이 배열로 온다. 뒤 노드가 "여러 아이템"을 어떻게 다루는지 Output 에서 확인해보세요.
> - **Pagination 옵션은 어떤 상황에 필요할까?**
>     - HTTP Request 노드의 `Pagination` 을 켜면 커서/페이지 번호 기반 자동 반복 호출이 가능하다. `_limit` `_page` 파라미터로 실험해보세요.
> - **에러(404 / 500 / 타임아웃)가 났을 때 워크플로우는 어떻게 동작할까?**
>     - 일부러 잘못된 URL 로 호출해본 뒤, 노드의 `Settings > On Error` 옵션을 바꿔가며 동작 차이를 관찰해보세요.
