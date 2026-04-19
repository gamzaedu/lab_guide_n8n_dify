---
title: "Lab 5. Webhook 노드로 외부 데이터 받기"
description: "Webhook 노드로 외부 HTTP 호출을 받아 워크플로우를 시작시키고, Query Parameter 처리와 Response 반환까지 실습합니다."
tags:
  - n8n
  - 기초실습
  - Webhook
---

# Lab 5. Webhook 노드로 외부 데이터 받기

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Webhook 노드로 외부 HTTP 호출을 받아 워크플로우를 실행시키고, Query Parameter 를 읽어 동적 응답을 반환할 수 있다 |
| 핵심 노드 | Webhook, Edit Fields (Set), Respond to Webhook |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 01~04 학습 완료, 웹 브라우저 사용 가능 환경 |
| 난이도 | ★★☆☆☆ |

!!! note "Test URL vs Production URL"
    Webhook 노드는 **두 가지 URL** 을 제공합니다.
    
    - **Test URL**: 캔버스에서 **[Listen for test event]** 를 누른 상태에서만 1회 동작. 개발·디버깅용.
    - **Production URL**: 워크플로우가 **Active** 상태일 때 상시 동작. 실제 서비스용.
    
    이번 실습은 **Test URL** 로 진행합니다.

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
    **Webhook 노드 1개** 만으로 구성된 워크플로우를 만드세요. Test URL 을 브라우저 주소창에 붙여넣어 호출했을 때, n8n 캔버스로 돌아와 Webhook 노드에 Output 이 생성되었는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud, 웹 브라우저
- **워크플로우 이름:** `Lab 05 - Webhook 기초`
- **Webhook 노드 설정:**

| 항목 | 값 |
|------|-----|
| HTTP Method | `GET` |
| Path | `hello` (또는 자동 생성 UUID 그대로) |
| Authentication | `None` |
| Respond | `Immediately` |

**구성 단계:**

1. 새 워크플로우 생성 → 이름 변경
2. **Webhook** 노드 추가, 위 표대로 설정
3. 노드 상단의 **Test URL** 복사 (예: `https://<your-n8n>/webhook-test/hello`)
4. Webhook 노드를 클릭한 상태로 **[Listen for test event]** 클릭 (상태: "Waiting for webhook call...")
5. 브라우저 새 탭에서 복사한 **Test URL** 접속
6. n8n 캔버스로 돌아와 Webhook 노드의 **Output 패널** 에서 수신 데이터 확인

??? note "예시"

    > **Webhook 노드 Output (빈 GET 요청)**
    >
    > ```json
    > [
    >   {
    >     "headers": {
    >       "host": "your-n8n-host",
    >       "user-agent": "Mozilla/5.0 ...",
    >       "accept": "text/html,application/xhtml+xml,..."
    >     },
    >     "params": {},
    >     "query": {},
    >     "body": {},
    >     "webhookUrl": "https://<your-n8n>/webhook-test/hello",
    >     "executionMode": "test"
    >   }
    > ]
    > ```
    >
    > **확인 포인트:**
    >
    > - `query`, `body`, `params` 는 모두 비어 있음 (GET 요청에 별도 데이터가 없으므로)
    > - `headers` 에 브라우저 정보가 담겨 있음
    > - `executionMode` 가 `"test"` → **Test URL** 로 호출되었다는 표시
    > - 브라우저 화면에는 **"Workflow got started"** 같은 간단한 메시지가 보일 수 있음

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
    Test URL 끝에 **Query Parameter 를 붙여** 호출하세요 (예: `?name=Alex&city=Seoul`). 그 뒤에 **Set 노드** 를 연결하여 Query 값을 Expression 으로 읽고, `greeting` 필드에 `"Hello, Alex from Seoul"` 같은 동적 메시지를 만드세요. Step 1(빈 호출)과 Output 을 비교 분석하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에 Set 노드 추가
- **호출 URL 예시:**
  `https://<your-n8n>/webhook-test/hello?name=Alex&city=Seoul`
- **Set 노드 (`Include Other Input Fields` = OFF, Manual Mapping):**

| Field Name | Type | Value (Expression) |
|------------|------|--------------------|
| name | String | `={{ $json.query.name }}` |
| city | String | `={{ $json.query.city }}` |
| greeting | String | `=Hello, {{ $json.query.name }} from {{ $json.query.city }}` |

**구성 순서:**

1. Step 1의 Webhook 노드 뒤에 **Edit Fields (Set)** 추가
2. 위 표대로 3개 필드 입력 (모두 Expression, `=` 접두사 필수)
3. Webhook 노드 선택 후 **[Listen for test event]** 다시 클릭
4. 브라우저에서 `?name=Alex&city=Seoul` 을 붙인 Test URL 호출
5. n8n 에서 Set 노드의 Output 확인

**(선택) 차이 분석 프롬프트:**

```
Webhook 의 Test URL 과 Production URL 의 차이를
"언제 활성화되는지 / 호출 횟수 제한 / 주로 쓰는 상황" 세 가지 기준으로
각 2줄씩 정리해줘.
```

**예시**

> **Webhook 노드 Output (Query 포함 호출)**
>
> ```json
> [
>   {
>     "headers": { "...": "..." },
>     "params": {},
>     "query": {
>       "name": "Alex",
>       "city": "Seoul"
>     },
>     "body": {},
>     "executionMode": "test"
>   }
> ]
> ```
>
> **Set 노드 Output**
>
> ```json
> [
>   {
>     "name": "Alex",
>     "city": "Seoul",
>     "greeting": "Hello, Alex from Seoul"
>   }
> ]
> ```
>
> **비교 분석표 — Test URL vs Production URL**
>
> | 기준 | Test URL | Production URL |
> |------|---------|----------------|
> | 활성화 조건 | **[Listen for test event]** 를 누른 순간부터 1회 | 워크플로우가 **Active** 인 동안 상시 |
> | 호출 횟수 | 1회 호출 후 리스닝 종료 (다시 누르면 재활성) | 무제한 |
> | 데이터 확인 | 캔버스 노드 Output 에 바로 표시 | Executions 탭에서 기록 확인 |
> | 주로 쓰는 시점 | 설계·디버깅 단계 | 배포·운영 단계 |
>
> **분석 결과:**
>
> - Query Parameter 는 Webhook Output 의 **`query`** 오브젝트에 자동으로 담긴다 → `{{ $json.query.필드명 }}` 으로 접근
> - Body / Headers / Params 도 각각 별도 오브젝트로 제공되므로 POST, Path Parameter, 인증 헤더 등 다양한 패턴으로 확장 가능
> - 개발 중엔 Test URL 로 빠르게 반복 테스트 → 완성되면 Active 토글 후 Production URL 로 전환하는 워크플로우가 표준

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
    **Respond to Webhook** 노드를 활용하여, 호출자에게 JSON 응답을 반환하는 **미니 API** 를 완성하세요. 브라우저에서 `?name=Alex` 로 호출했을 때 브라우저 화면에 `{"message": "Hello, Alex"}` 형태의 JSON 이 바로 표시되어야 합니다.

**활용 도구 & 데이터**

- **도구:** Step 2의 워크플로우를 확장
- **Webhook 노드 변경:**

| 항목 | 값 |
|------|-----|
| Respond | `Using 'Respond to Webhook' Node` |

- **Set 노드 (Manual Mapping, `Include Other Input Fields` = OFF):**

| Field Name | Type | Value (Expression) |
|------------|------|--------------------|
| message | String | `=Hello, {{ $json.query.name }}` |

- **Respond to Webhook 노드 설정:**

| 항목 | 값 |
|------|-----|
| Respond With | `JSON` |
| Response Body | `={{ $json }}` |
| Response Code | `200` |

**구성 순서:**

1. Webhook 노드의 **Respond** 옵션을 `Using 'Respond to Webhook' Node` 로 변경
2. Set 노드를 `message` 1개 필드만 남도록 단순화
3. Set 노드 뒤에 **Respond to Webhook** 노드 추가, 위 표대로 설정
4. Webhook 노드 선택 → **[Listen for test event]**
5. 브라우저에서 `<Test URL>?name=Alex` 호출
6. 브라우저 화면에 JSON 응답이 바로 표시되는지 확인

**예시**

> **완성 워크플로우: "미니 인사 API"**
>
> 흐름: `Webhook → Set(message 구성) → Respond to Webhook`
>
> **브라우저로 `?name=Alex` 호출 시 화면:**
>
> ```json
> {
>   "message": "Hello, Alex"
>   }
> ```
>
> **브라우저로 `?name=Sam` 호출 시 화면:**
>
> ```json
> {
>   "message": "Hello, Sam"
> }
> ```
>
> **확인 포인트:**
>
> - 브라우저가 더 이상 **"Workflow got started"** 같은 기본 메시지를 보이지 않고, 우리가 설계한 JSON 을 직접 반환함
> - Query 의 `name` 값이 바뀌면 응답도 즉시 달라짐 — 외부에서 값을 받아 **동적 응답** 을 내는 미니 API 완성
> - 실무에서는 이 패턴에 Code / HTTP Request / DB 조회 등을 추가해 풀스케일 API 로 확장 가능

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

> - **Webhook 의 HTTP Method 를 `POST` 로 바꾸고 JSON Body 를 받으려면 어떻게 구성할까?**
>     - Method 를 POST 로 변경 후, 브라우저 대신 httpbin 의 Form 이나 Lab 4 에서 배운 HTTP Request 노드로 POST 호출을 만들어 테스트해보세요. Output 의 `body` 필드에 값이 들어오는지 확인합니다.
> - **Respond Mode 의 `Immediately` / `When Last Node Finishes` / `Using 'Respond to Webhook' Node` 는 각각 언제 쓸까?**
>     - 세 옵션을 각각 설정해 응답 시점과 응답 본문이 어떻게 달라지는지 직접 비교해보세요. 긴 작업(예: AI 호출)이 뒤에 있을 때 특히 차이가 커집니다.
> - **Webhook + Schedule Trigger 를 한 워크플로우에 같이 두면 어떤 활용이 가능할까?**
>     - 동일한 로직을 "외부 호출로도, 정기 스케줄로도" 동시에 돌릴 수 있다. 두 트리거를 모두 둔 뒤 공통 로직을 Sub-workflow 로 빼는 설계를 상상해보세요.
> - **Production URL 로 전환하려면 어떤 체크가 필요할까?**
>     - 워크플로우 Active 토글, Authentication 설정, Rate Limit, 로그·모니터링 같은 운영 관점의 차이를 정리해보고 Test URL 과 나란히 비교표를 만들어보세요.
> - **Webhook Path 에 변수(Path Parameter)를 넣을 수 있을까?**
>     - Path 를 `hello/:userId` 같은 형태로 설정한 뒤 호출하면 Output 의 `params.userId` 로 접근할 수 있다. Query 방식과 어떤 장단이 있는지 비교해보세요.
