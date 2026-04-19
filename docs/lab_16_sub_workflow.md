---
title: "Lab 16. Sub-workflow 로 공통 로직 재사용"
description: "Execute Workflow 노드로 공통 로직을 Sub-workflow 로 분리하고, 여러 메인에서 재사용하며 데이터를 주고받습니다."
tags:
  - n8n
  - 심화실습
  - Sub-workflow
  - 재사용
---

# Lab 16. Sub-workflow 로 공통 로직 재사용

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Execute Workflow 노드로 공통 로직을 Sub-workflow 로 분리하고, 메인 워크플로우와 데이터를 주고받으며 재사용할 수 있다 |
| 핵심 노드 | Manual Trigger, Execute Workflow, Execute Workflow Trigger, Set, Code |
| 소요 시간 | 약 50분 |
| 사전 준비 | Lab 12 (Code) 완료 권장 |
| 난이도 | ★★★★☆ |

!!! note "Sub-workflow 의 2가지 구성 요소"
    - **Execute Workflow (호출하는 쪽, 메인)**: 일반 노드처럼 사용하며 다른 워크플로우를 호출
    - **Execute Workflow Trigger (호출되는 쪽, 서브)**: Sub-workflow 의 시작 지점. 메인에서 넘긴 데이터를 받음

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
    가장 단순한 Sub-workflow 를 만드세요. 서브는 `Execute Workflow Trigger → Set (greeting 출력)` 로 구성하고, 메인에서는 `Manual Trigger → Execute Workflow` 로 서브를 호출하여 결과를 받아오세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud (워크플로우 2개)
- **서브 워크플로우 이름:** `Lab 16 - Greeting (sub)`
- **메인 워크플로우 이름:** `Lab 16 - Main`

**① 서브 워크플로우 구성:**

```
Execute Workflow Trigger → Set (greeting: "Hello from sub!")
```

- Execute Workflow Trigger 설정: 기본값 (모든 input 모드 유지)
- Set 노드 필드:

| Field | Type | Value |
|-------|------|-------|
| greeting | String | `Hello from sub!` |

**② 메인 워크플로우 구성:**

```
Manual Trigger → Execute Workflow (Sub-workflow 선택)
```

- Execute Workflow 노드 설정:

| 항목 | 값 |
|------|-----|
| Source | `Database` |
| Workflow | `Lab 16 - Greeting (sub)` 선택 |

**구성 단계:**

1. 서브 워크플로우 먼저 생성하고 **저장** (ID 부여 필요)
2. 메인 워크플로우 생성 → Execute Workflow 노드 추가 → 서브 선택
3. 메인에서 **[Execute Workflow]** 실행
4. 메인의 Execute Workflow 노드 Output 에 서브 결과가 나오는지 확인

??? note "예시"

    > **메인의 Execute Workflow 노드 Output**
    >
    > ```json
    > [ { "greeting": "Hello from sub!" } ]
    > ```
    >
    > **확인 포인트:**
    >
    > - 메인은 Execute Workflow 노드 **하나만** 으로 서브의 전체 결과를 받음
    > - 서브는 메인에서 호출될 때 자동으로 실행됨 (Active 일 필요 없음)
    > - 캔버스에서 Execute Workflow 노드를 더블클릭하면 **서브 워크플로우로 바로 이동** 가능

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
    메인에서 서브로 **파라미터를 전달** 하도록 구성하세요. 메인 Set 노드에서 `name` 필드를 만들어 Execute Workflow 에 넘기고, 서브에서 그 값을 읽어 `greeting: "Hello, Alex!"` 같은 맞춤 메시지를 반환하도록 변경하세요.

**활용 도구 & 데이터**

- **메인 수정:**

```
Manual Trigger
  → Set (name: "Alex")
  → Execute Workflow (서브 호출)
```

- **서브 수정:**

```
Execute Workflow Trigger
  → Set (greeting: "=Hello, {{ $json.name }}!")
```

- **서브의 Set 노드 필드:**

| Field | Type | Expression |
|-------|------|-----------|
| greeting | String | `=Hello, {{ $json.name }}!` |

**관찰 포인트:**

1. 서브 워크플로우 안에서 `$json` 은 **Execute Workflow Trigger 의 Output** → 메인이 넘긴 데이터
2. 여러 아이템(예: 이름 3개)을 넘기면 서브 역시 3번 실행됨 (Implicit Loop 가 서브로 이어짐)
3. 서브가 반환한 마지막 노드의 Output 이 메인의 Execute Workflow 노드 Output 으로 돌아옴

**(선택) 분석 프롬프트:**

```
Sub-workflow 와 Code 노드 모두 "공통 로직 재사용" 에 쓸 수 있는데,
어느 경우에 Sub-workflow 가, 어느 경우에 Code 가 더 유리한지
"재사용 범위 / 가시성 / 권한 관리" 기준으로 비교해줘.
```

**예시**

> **메인 → 서브 → 메인 왕복 데이터 흐름**
>
> - 메인 Set Output: `[{ "name": "Alex" }]` → 서브로 전달
> - 서브 Execute Workflow Trigger Output: `[{ "name": "Alex" }]`
> - 서브 Set Output: `[{ "greeting": "Hello, Alex!" }]` → 메인으로 반환
> - 메인 Execute Workflow 노드 Output: `[{ "greeting": "Hello, Alex!" }]`
>
> **메인 입력을 `[{name:"Alex"},{name:"Sam"},{name:"Kelly"}]` 3개로 확장하면:**
>
> ```json
> [
>   { "greeting": "Hello, Alex!" },
>   { "greeting": "Hello, Sam!" },
>   { "greeting": "Hello, Kelly!" }
> ]
> ```
>
> **Sub-workflow vs Code 노드 비교표**
>
> | 기준 | Sub-workflow | Code 노드 |
> |------|-------------|---------|
> | 재사용 범위 | 여러 워크플로우에서 공유 | 한 워크플로우 내부만 |
> | 가시성 | 캔버스에서 흐름 확인 가능 | 코드 읽어야 이해 가능 |
> | 구성 단위 | 여러 노드 조합 가능 | JS 함수 단위 |
> | 권한 관리 | 워크플로우별 권한 부여 가능 | 워크플로우에 종속 |
> | 디버깅 | 각 노드 Output 개별 확인 | 로그·throw 로만 가능 |
>
> **분석 결과:**
>
> - **여러 메인이 공통으로 쓰는 로직** 이면 Sub-workflow (예: "Slack 에 알림 보내는 표준 포맷")
> - **단일 워크플로우 안의 복잡한 변환** 이면 Code 노드
> - Sub-workflow 는 **"마이크로서비스 같은 재사용"** 에, Code 는 **"인라인 함수"** 에 비유 가능

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
    실용적인 Sub-workflow 를 완성하세요. 서브는 **"가격 + 세율 입력 → 세금 포함 가격과 세액을 반환"** 하는 순수 계산 로직을, 메인은 **3개의 서로 다른 가격 데이터를 서브에 넘겨 일괄 계산** 하는 구조로 만드세요. 서브를 재사용성 있게 설계하는 것이 핵심입니다.

**활용 도구 & 데이터**

- **도구:** 새 서브 워크플로우 + 새 메인 워크플로우
- **서브 (`Lab 16 - Tax Calc (sub)`):**

```
Execute Workflow Trigger
  → Code (가격·세율 → 세금·총액 계산)
```

- **서브 Code 노드:**

```javascript
return items.map(item => {
  const { price, taxRate } = item.json;
  const tax = Math.round(price * taxRate);
  const total = price + tax;
  return { json: { price, taxRate, tax, total } };
});
```

- **메인 (`Lab 16 - Invoice Main`):**

```
Manual Trigger
  → Set (JSON, 3개 가격 아이템)
  → Execute Workflow (Lab 16 - Tax Calc)
```

- **Set (JSON) 입력:**

```json
[
  { "product": "A", "price": 10000, "taxRate": 0.1 },
  { "product": "B", "price": 25000, "taxRate": 0.1 },
  { "product": "C", "price": 88000, "taxRate": 0.1 }
]
```

**구성 순서:**

1. 서브 워크플로우 생성, Code 노드까지 저장
2. 메인에서 Set(JSON) 3개 → Execute Workflow 로 서브 호출
3. Execute Workflow 의 Options 에서 `Mode: Run Once for All Items` 확인
4. **[Execute Workflow]** 실행, 결과에서 3개 아이템 각각 `tax`, `total` 이 계산되어 있는지 확인

!!! tip "`product` 필드 보존"
    위 Code 는 `product` 를 버린다. 보존하려면 `return items.map(item => ({ json: { ...item.json, tax, total } }))` 로 변경.

**예시**

> **완성 워크플로우: "세금 계산 서브"**
>
> **메인의 Execute Workflow Output**
>
> ```json
> [
>   { "price": 10000, "taxRate": 0.1, "tax": 1000, "total": 11000 },
>   { "price": 25000, "taxRate": 0.1, "tax": 2500, "total": 27500 },
>   { "price": 88000, "taxRate": 0.1, "tax": 8800, "total": 96800 }
> ]
> ```
>
> **재사용성 증명 시나리오:**
>
> - 다른 메인 워크플로우(예: 일일 정산, 월간 리포트)에서 동일한 서브를 호출해 세금 계산 결과를 받을 수 있다
> - 세율 정책이 바뀌면 **서브만 고치면 됨** → 모든 메인에 일괄 반영
> - 이 패턴이 커지면 "인증 표준", "알림 표준", "데이터 정제 표준" 같은 **사내 공통 서브-라이브러리** 로 진화

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

> - **Execute Workflow 의 `Mode` 옵션 (Run Once for All Items / Run Once for Each Item) 은 서브에 어떻게 영향을 줄까?**
>     - Each Item 모드는 아이템마다 서브를 "별도 실행" 으로 호출 — 실행 기록이 N개 생긴다. 같은 입력으로 두 모드를 비교해 Executions 기록 수 차이를 관찰해보세요.
> - **서브가 에러를 냈을 때 메인은 어떻게 동작할까?**
>     - Execute Workflow 노드 Settings 의 On Error 옵션은 다른 노드와 동일. 서브 내부 에러를 메인에서 catch 하려면 `Continue (using error output)` 필요. 실험해보세요.
> - **Sub-workflow 와 Error Trigger(Lab 15) 를 결합하려면?**
>     - 공통 "에러 알림 보내기" 로직을 Sub-workflow 로 만들고, 각 메인의 Error Workflow 가 그 서브를 호출하도록 구성 가능. 2단계 추상화의 장단을 고민해보세요.
> - **Input Data Mode 에서 "Define Below" / "Accept All Data" 는 어떻게 다를까?**
>     - Execute Workflow Trigger 의 Input 모드에 따라 메인에서 어떤 데이터를 넘길 수 있는지 제약이 달라진다. 실무에서 입력 스키마 강제 vs 자유 전달을 어떻게 선택할지 생각해보세요.
> - **너무 깊은 서브 호출 체인(메인 → 서브 → 서브2 → 서브3) 은 어떤 문제를 낳을까?**
>     - 가시성·디버깅·실행 시간 추적의 어려움. 실무에서 허용할 깊이 한계를 본인 기준으로 정해보세요.
