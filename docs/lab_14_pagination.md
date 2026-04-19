---
title: "Lab 14. HTTP Request Pagination 처리"
description: "HTTP Request 노드의 Pagination 옵션으로 여러 페이지에 걸친 API 응답을 한 번의 실행으로 모두 수집합니다."
tags:
  - n8n
  - 심화실습
  - HTTP
  - Pagination
---

# Lab 14. HTTP Request Pagination 처리

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | HTTP Request 노드의 Pagination 옵션으로 페이지 번호 기반·응답 기반 페이징을 자동 반복하고, 결과를 하나로 집계할 수 있다 |
| 핵심 노드 | Manual Trigger, HTTP Request, Code |
| 소요 시간 | 약 50분 |
| 사전 준비 | Lab 04·10·12 완료 (HTTP / 반복 / Code) |
| 난이도 | ★★★★☆ |

!!! note "이번 실습에서 사용할 공개 API"
    **JSONPlaceholder** 의 `/comments` 엔드포인트 — 500개 코멘트를 `_page` / `_limit` 파라미터로 페이징 가능:
    
    - `https://jsonplaceholder.typicode.com/comments?_page=1&_limit=10` — 1페이지 10개
    - `https://jsonplaceholder.typicode.com/comments?_page=2&_limit=10` — 2페이지 10개

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
    HTTP Request 로 **1페이지 10개** 만 받아오는 워크플로우를 만드세요. `_page=1`, `_limit=10` 을 Query Parameter 로 넣고, Output 에 10개 코멘트 아이템이 들어오는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 14 - Pagination`
- **HTTP Request 노드 설정:**

| 항목 | 값 |
|------|-----|
| Method | `GET` |
| URL | `https://jsonplaceholder.typicode.com/comments` |
| Send Query Parameters | ON |
| Query Parameters | `_page=1`, `_limit=10` |

**구성 단계:**

1. Manual Trigger → HTTP Request
2. URL / Query Parameters 위 표대로 입력
3. **[Execute Workflow]** 실행
4. Output 패널에서 아이템 개수 = **10** 확인

??? note "예시"

    > **Output JSON (10개 중 첫 2개)**
    >
    > ```json
    > [
    >   {
    >     "postId": 1,
    >     "id": 1,
    >     "name": "id labore ex et quam laborum",
    >     "email": "Eliseo@gardner.biz",
    >     "body": "laudantium enim quasi est..."
    >   },
    >   {
    >     "postId": 1,
    >     "id": 2,
    >     "name": "quo vero reiciendis velit similique earum",
    >     "email": "Jayne_Kuhic@sydney.com",
    >     "body": "est natus enim nihil est dolore omnis..."
    >   }
    > ]
    > ```
    >
    > **확인 포인트:** Output 총 아이템 = **10** (페이지 1페이지만 받았음). 전체 500 개를 받으려면 50 페이지를 반복 호출해야 함.

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
    HTTP Request 노드의 **Pagination 옵션** 을 ON 하여, `_page` 파라미터를 자동으로 1→2→3...로 증가시키며 **최대 5페이지(50개)** 까지 수집하도록 설정하세요. 한 번의 실행으로 50개 아이템이 Output 에 쌓이는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 HTTP Request 노드 설정 수정
- **Pagination 옵션 설정:**

| 항목 | 값 |
|------|-----|
| Pagination | ON |
| Pagination Mode | `Update a Parameter in Each Request` |
| Parameter Type | `Query` |
| Parameter Name | `_page` |
| Parameter Value | `={{ $pageCount + 1 }}` |
| Pagination Complete When | `Receive Specific Status Code` → `No response` 또는 `Response is Empty` |
| Limit Pages Fetched | `5` |
| Interval Between Requests (ms) | `200` (API 부하 방지) |

!!! tip "`$pageCount` 내장 변수"
    Pagination 모드에서만 사용 가능한 변수로, 지금까지 완료된 호출 횟수입니다. 첫 호출은 `$pageCount = 0` → `_page = 1` 이 됩니다.

**변경 방법:**

1. HTTP Request 노드의 **Options > Pagination** 을 ON
2. 위 표대로 입력
3. **[Execute Workflow]** 실행
4. 실행 로그에 "5 requests sent" 같은 메시지 확인, Output 총 아이템 = **50**

**(선택) 분석 프롬프트:**

```
n8n HTTP Request 의 Pagination 옵션에서
"Update a Parameter in Each Request" vs "Response Contains Next URL"
두 모드의 차이를 2줄씩 정리해줘.
```

**예시**

> **Pagination ON 후 Output 요약**
>
> - 총 아이템 수: **50** (5페이지 × 10개)
> - 실행 시간: 약 1~2초 (각 호출 간 200ms 인터벌 + API 응답 시간)
> - 노드 하나가 내부적으로 **5번의 HTTP 호출** 을 수행
>
> **Pagination 모드 비교표**
>
> | 모드 | 동작 | 대표 API 예 |
> |------|------|------------|
> | Off | 1회만 호출 | 단일 엔드포인트 |
> | Update a Parameter in Each Request | 지정 파라미터(`_page`, `offset` 등)를 매 호출마다 증가 | JSONPlaceholder, REST 표준 |
> | Response Contains Next URL | 응답 본문의 `next` 필드에서 다음 URL 추출 | GitHub, Notion API |
>
> **분석 결과:**
>
> - **Pagination 옵션이 없을 때** 는 Lab 10 의 Split In Batches / Loop 로 직접 반복을 짜야 했다 — 코드 많음, 복잡
> - **Pagination 옵션** 은 그 반복 로직을 HTTP Request 노드 안에 내장 → 워크플로우가 **1개 노드** 로 단순해짐
> - `Limit Pages Fetched` 는 무한루프 방지 장치 — 실제 서비스에서는 넉넉히 잡되 반드시 설정
> - `Interval Between Requests` 는 API 의 Rate Limit 준수 — 설정 안 하면 API 가 429 응답 줄 수 있음

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
    Pagination 으로 **전체 500개 코멘트** 를 수집하고, 뒤에 Code 노드를 붙여 **이메일 도메인별 코멘트 개수** 를 집계하세요. 결과는 `{ domain: "xxx.com", count: 12 }` 형태의 아이템 배열로, count 내림차순 정렬되어야 합니다.

**활용 도구 & 데이터**

- **도구:** Step 2 의 워크플로우에 Code 노드 추가
- **Pagination 변경:** `Limit Pages Fetched` = `50` (500개 전체)
- **Code 노드 (All Items):**

```javascript
const counts = {};

for (const item of items) {
  const email = item.json.email;
  const domain = email.split("@")[1];
  counts[domain] = (counts[domain] || 0) + 1;
}

const result = Object.entries(counts)
  .map(([domain, count]) => ({ json: { domain, count } }))
  .sort((a, b) => b.json.count - a.json.count);

return result;
```

**구성 순서:**

1. HTTP Request (Pagination Limit=50) → Code 노드 연결
2. Code 노드에 위 스크립트 입력
3. **[Execute Workflow]** 실행 (전체 500 아이템 수집에 수 초 소요)
4. Code 노드 Output 에서 도메인별 집계 확인

**예시**

> **완성 워크플로우: "이메일 도메인별 코멘트 집계"**
>
> **Output JSON (상위 5개만 발췌)**
>
> ```json
> [
>   { "domain": "sydney.com",  "count": 7 },
>   { "domain": "kory.org",    "count": 6 },
>   { "domain": "example.com", "count": 5 },
>   { "domain": "april.biz",   "count": 4 },
>   { "domain": "conroy.com",  "count": 4 }
> ]
> ```
>
> (실제 값은 JSONPlaceholder 데이터에 따라 달라질 수 있음)
>
> **구현 포인트:**
>
> - 1개 HTTP Request 노드가 50번의 호출을 **자동** 수행 → Split In Batches 없이도 OK
> - Code 노드가 500개 items 를 `reduce` 로 집계 → 1개 도메인 = 1개 아이템
> - 이 패턴은 **"API 전체 긁어오기 → 통계"** 의 표준 — 뒤에 Slack/Sheets 를 붙이면 대시보드 자동화

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

> - **커서(cursor) 기반 페이징은 어떻게 구성할까?**
>     - `Response Contains Next URL` 또는 `Next URL in Response Body` 모드를 써본다. GitHub Search API(`link` 헤더의 `rel="next"`) 를 예시로 실험해보세요.
> - **특정 조건에서 페이징을 멈추려면?**
>     - `Pagination Complete When` 의 여러 옵션(`Response is Empty`, `Receive Specific Status Code`, `Other` 등)을 비교해보세요. 응답이 빈 배열인지 어떻게 감지하는지도 체크.
> - **Pagination 과 Split In Batches(Lab 10) 를 같이 쓰면 어떤 의미가 될까?**
>     - Pagination 은 "한 노드 내부의 반복", Split In Batches 는 "뒤 노드를 배치로 돌리기" — 조합 시 시나리오를 상상해보세요.
> - **Limit Pages Fetched 를 설정하지 않으면 어떤 위험이 있을까?**
>     - 무한 페이지 API 에서 영원히 호출, Rate Limit 초과, 크레딧 소모. 안전장치로 어떤 조합을 기본으로 쓸지 본인 규칙을 정해보세요.
> - **헤더로 페이지 정보를 주는 API(예: `X-Total-Count`) 는 어떻게 활용할까?**
>     - 응답 헤더를 파싱해 IF 분기, 또는 Pagination 의 Complete When 조건에 활용. HTTP Request 의 응답 헤더를 Output 에서 확인하는 방법도 찾아보세요.
