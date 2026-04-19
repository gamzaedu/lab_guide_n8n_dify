---
title: "Lab 3. Edit Fields(Set) 노드로 데이터 가공"
description: "Set 노드로 필드를 추가·수정·삭제하고, Expression 문법을 처음 사용하여 앞 노드 데이터를 동적으로 가공합니다."
tags:
  - n8n
  - 기초실습
  - Set노드
  - Expression
---

# Lab 3. Edit Fields(Set) 노드로 데이터 가공

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Set 노드로 필드를 추가·수정·삭제하고, `{{ $json.xxx }}` Expression 문법으로 앞 노드 데이터를 참조·계산할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set) × 2 |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 01~02 학습 완료 권장 |
| 난이도 | ★★☆☆☆ |

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
    Manual Trigger 뒤에 **Set 노드 1개** 를 붙여, 가상의 상품 데이터(`product`, `price`, `currency`) 3개 필드를 생성하세요. Type 을 올바르게 지정해 Output JSON 의 `price` 가 숫자로, 나머지는 문자열로 저장되도록 하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 03 - 데이터 가공`
- **Set 노드 필드 (Manual Mapping):**

| Field Name | Type | Value |
|------------|------|-------|
| product | String | `Notebook` |
| price | Number | `1200` |
| currency | String | `USD` |

**구성 단계:**

1. 새 워크플로우 생성 → 이름 변경
2. **Manual Trigger** 추가
3. **Edit Fields (Set)** 연결, Mode: `Manual Mapping`
4. 위 표대로 필드 3개 추가
5. **[Execute Workflow]** 실행, Output 에서 `price` 에 따옴표가 없는지 (숫자인지) 확인

??? note "예시"

    > **Output JSON**
    >
    > ```json
    > [
    >   {
    >     "product": "Notebook",
    >     "price": 1200,
    >     "currency": "USD"
    >   }
    > ]
    > ```
    >
    > **확인 포인트:** `price` 값이 `"1200"` 이 아니라 `1200` 으로 출력되어야 합니다. 따옴표가 보이면 Type 이 String 으로 설정된 것이므로 Number 로 바꾸세요.

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
    Step 1 뒤에 **두 번째 Set 노드** 를 연결하세요. 두 번째 Set 노드에서 **Expression 문법** 을 사용하여 다음 3개 필드를 동적으로 생성하세요:
    (1) `priceWithTax` = 원래 가격의 110% (숫자 연산), (2) `label` = `"Notebook - 1200 USD"` 형태의 조합 문자열, (3) `isExpensive` = price 가 1000 초과이면 `true`.
    그 뒤 `Keep Other Fields` 옵션을 켜고 끄면서 Output 이 어떻게 달라지는지 비교하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에 Set 노드 1개를 추가 연결
- **두 번째 Set 노드 필드 (Manual Mapping):**

| Field Name | Type | Value (Expression) |
|------------|------|--------------------|
| priceWithTax | Number | `={{ $json.price * 1.1 }}` |
| label | String | `={{ $json.product }} - {{ $json.price }} {{ $json.currency }}` |
| isExpensive | Boolean | `={{ $json.price > 1000 }}` |

!!! tip "Expression 입력 방법"
    Value 칸에서 **[Expression]** 탭을 선택(또는 값 앞에 `=` 추가)한 뒤 `{{ }}` 안에 n8n 변수를 작성합니다. 입력창 아래 실시간 미리보기에서 실제 평가 결과가 보입니다.

**실험 항목:**

1. 위 3개 필드를 모두 추가하고 실행 → Output 확인
2. 두 번째 Set 노드의 **[Options]** 에서 `Include Other Input Fields` (Keep Other Fields) 를 **ON** / **OFF** 로 바꿔가며 실행
3. Expression 에 오타를 내면 (예: `$json.prrice`) 어떤 에러가 나는지 관찰

**(선택) 차이 정리 프롬프트:**

```
아래 두 Output JSON 을 비교해서,
"Include Other Input Fields" 옵션이 Set 노드의 출력에
어떤 영향을 주는지 2줄로 설명해줘.

[옵션 OFF] ...
[옵션 ON] ...
```

**예시**

> **두 번째 Set 노드 Output — `Include Other Input Fields` **OFF**
>
> ```json
> [
>   {
>     "priceWithTax": 1320,
>     "label": "Notebook - 1200 USD",
>     "isExpensive": true
>   }
> ]
> ```
>
> **두 번째 Set 노드 Output — `Include Other Input Fields` **ON**
>
> ```json
> [
>   {
>     "product": "Notebook",
>     "price": 1200,
>     "currency": "USD",
>     "priceWithTax": 1320,
>     "label": "Notebook - 1200 USD",
>     "isExpensive": true
>   }
> ]
> ```
>
> **비교 분석표**
>
> | 옵션 | Output 필드 개수 | 특징 |
> |------|---------------|------|
> | OFF (기본값) | 3개 (새로 만든 것만) | 기존 필드 제거, 깔끔한 새 구조 |
> | ON | 6개 (원본 + 새것) | 기존 필드 유지, 파이프라인 뒤쪽에서 원본 참조 가능 |
>
> **분석 결과:**
>
> - Expression `={{ $json.xxx }}` 는 **앞 노드의 Output** 을 그대로 참조한다 — 따라서 두 번째 Set 노드 입장에서 `$json.price` 는 첫 번째 Set 노드가 만든 `1200`
> - `=` 가 없으면 문자열 그대로 저장되고, `=` 가 있어야 Expression 으로 평가된다
> - `Include Other Input Fields` 는 파이프라인에서 **중간 Set 노드가 원본 데이터를 덮을지 보존할지**를 결정하는 핵심 옵션

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
    본인이 직접 선택한 주제(가상 주문 / 도서 정보 / 날씨 로그 등)로 **원본 3~5개 필드 + 가공 3개 필드** 구조의 워크플로우를 완성하세요. 가공 필드 중 **최소 1개는 산술 연산**, **최소 1개는 문자열 조합**, **최소 1개는 Boolean 조건** 을 포함해야 합니다.

**활용 도구 & 데이터**

- **도구:** Step 1~2의 워크플로우를 복제(Duplicate)하여 수정, 또는 새로 구성
- **설계 포인트:**

| 단계 | 무엇을 정할지 |
|------|-------------|
| 주제 | 가상 주문 / 사용자 프로필 / 도서 / 날씨 / 운동 기록 중 자유 |
| 원본 필드 | 3~5개, 서로 다른 Type 섞기 |
| 가공 필드 | 산술(예: 합계·세율), 문자열 조합(예: 요약 라벨), Boolean(예: 조건 판별) 각 1개 이상 |

**설계 순서:**

1. 주제 결정 후 원본 필드 목록을 메모에 작성
2. 첫 번째 Set 노드에서 원본 필드 입력
3. 두 번째 Set 노드에서 Expression 으로 가공 필드 생성
4. 두 번째 Set 노드의 `Include Other Input Fields` 를 용도에 맞게 설정
5. 실행 후 Output 이 의도한 구조인지 확인, 저장

**예시**

> **완성 워크플로우: "가상 주문 요약 파이프라인"**
>
> **첫 번째 Set 노드 (원본):**
>
> | Field | Type | Value |
> |-------|------|-------|
> | orderId | String | `ORD-2025-0412` |
> | customer | String | `Alex` |
> | itemCount | Number | `3` |
> | unitPrice | Number | `250` |
> | isMember | Boolean | `true` |
>
> **두 번째 Set 노드 (가공, `Include Other Input Fields` = ON):**
>
> | Field | Type | Expression |
> |-------|------|-----------|
> | subtotal | Number | `={{ $json.itemCount * $json.unitPrice }}` |
> | discount | Number | `={{ $json.isMember ? 50 : 0 }}` |
> | total | Number | `={{ $json.itemCount * $json.unitPrice - ($json.isMember ? 50 : 0) }}` |
> | summary | String | `=Order {{ $json.orderId }} by {{ $json.customer }}: {{ $json.itemCount }} items` |
> | needsReview | Boolean | `={{ $json.itemCount * $json.unitPrice > 500 }}` |
>
> **최종 Output JSON:**
>
> ```json
> [
>   {
>     "orderId": "ORD-2025-0412",
>     "customer": "Alex",
>     "itemCount": 3,
>     "unitPrice": 250,
>     "isMember": true,
>     "subtotal": 750,
>     "discount": 50,
>     "total": 700,
>     "summary": "Order ORD-2025-0412 by Alex: 3 items",
>     "needsReview": true
>   }
> ]
> ```

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

> - **Set 노드의 Mode 를 `JSON` 으로 바꾸면 Expression 과 어떻게 공존할까?**
>     - JSON 모드에서 `{ "total": "={{ $json.a + $json.b }}" }` 처럼 문자열 안에 Expression 을 넣을 수 있다. Manual Mapping 과 어느 쪽이 더 편한지 상황별로 비교해보세요.
> - **중첩 객체(예: `user.address.city`)에 접근하려면 어떻게 해야 할까?**
>     - `{{ $json.user.address.city }}` 같은 Dot Notation 을 쓴다. 첫 번째 Set 노드에서 중첩 오브젝트를 만들고 두 번째 노드에서 접근해보세요.
> - **`$json` 외에 어떤 내장 변수들이 있을까?**
>     - `$now`, `$today`, `$runIndex`, `$workflow.name`, `$node["노드이름"].json` 등. 각 변수의 출력을 Set 노드로 내보내어 확인해보세요.
> - **Expression 의 `=` 접두사는 내부적으로 어떤 의미일까?**
>     - `=` 가 없으면 리터럴 문자열, `=` 가 있으면 Expression 평가로 동작한다. `={{ 1 + 1 }}` 과 `{{ 1 + 1 }}` 의 결과 차이를 Output 에서 직접 확인해보세요.
