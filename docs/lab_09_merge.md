---
title: "Lab 9. Merge 노드로 데이터 병합"
description: "Merge 노드의 Append / Combine / Choose Branch 모드를 비교하며 두 경로의 데이터를 합치는 패턴을 익힙니다."
tags:
  - n8n
  - 중급실습
  - Merge
  - 제어흐름
---

# Lab 9. Merge 노드로 데이터 병합

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Merge 노드의 Append / Combine / Choose Branch 모드를 상황에 맞게 선택하고, 두 갈래의 데이터를 목적에 맞게 병합할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Merge |
| 소요 시간 | 약 50분 |
| 사전 준비 | Lab 06~07 (IF / Switch) 완료 권장 |
| 난이도 | ★★★☆☆ |

!!! note "Merge 모드 3종 개요"
    - **Append**: 두 입력의 아이템을 순서대로 이어붙임 (합집합 리스트)
    - **Combine**: 공통 키를 기준으로 두 입력의 필드를 한 아이템으로 결합 (SQL JOIN 과 유사)
    - **Choose Branch**: 조건에 따라 한쪽 입력만 통과

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
    Manual Trigger 하나에서 두 갈래로 Set 노드를 두고(A 쪽은 `label: "Top"`, B 쪽은 `label: "Bottom"`), Merge 노드의 **Append** 모드로 합쳐 Output 에 두 아이템이 나란히 나오는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 09 - Merge 기초`
- **구성:**

```
Manual Trigger
  ├─ Set A (label: "Top",    value: 10)
  └─ Set B (label: "Bottom", value: 20)
            → Merge (Mode: Append)
```

- **Merge 노드 설정:**

| 항목 | 값 |
|------|-----|
| Mode | `Append` |
| Input 1 연결 | Set A |
| Input 2 연결 | Set B |

**구성 단계:**

1. Manual Trigger 뒤에 Set 노드 2개를 **나란히** 배치 (트리거의 + 버튼을 두 번 눌러 두 갈래 생성)
2. **Merge** 노드 추가, Input 1 에 Set A, Input 2 에 Set B 연결
3. Merge 노드의 Mode 를 `Append` 로 설정
4. **[Execute Workflow]** 실행 → Output 패널에서 2개 아이템이 순서대로 보이는지 확인

??? note "예시"

    > **Merge 노드 Output (Append 모드)**
    >
    > ```json
    > [
    >   { "label": "Top",    "value": 10 },
    >   { "label": "Bottom", "value": 20 }
    > ]
    > ```
    >
    > **확인 포인트:** Input 1 의 아이템이 먼저, Input 2 의 아이템이 뒤에 이어집니다. 총 개수는 두 입력의 개수 합.

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
    Merge 노드의 Mode 를 **Append** → **Combine (Combine by Matching Fields)** → **Choose Branch** 순서로 바꿔가며 같은 입력 조합에서 Output 이 어떻게 달라지는지 비교 분석표를 만드세요. 특히 Combine 모드에서 **매칭 키** 를 지정하는 방법을 집중 관찰하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에서 Set 노드의 필드를 아래로 변경

- **Set A (공통 키 `id` 포함):**

| Field | Type | Value |
|-------|------|-------|
| id | Number | `1` |
| name | String | `Alex` |

- **Set B (같은 `id` + 추가 필드):**

| Field | Type | Value |
|-------|------|-------|
| id | Number | `1` |
| email | String | `alex@example.com` |

- **Combine 모드 설정:**

| 항목 | 값 |
|------|-----|
| Mode | `Combine` |
| Combine By | `Combine by Matching Fields` |
| Fields to Match — Input 1 Field | `id` |
| Fields to Match — Input 2 Field | `id` |

- **Choose Branch 모드 설정:**

| 항목 | 값 |
|------|-----|
| Mode | `Choose Branch` |
| Output | `Input 1 Data` (또는 `Input 2 Data`) |

**(선택) 비교 프롬프트:**

```
Merge 노드의 Append / Combine / Choose Branch 세 모드를
"Output 아이템 개수 / 필드 결합 여부 / 주요 용도" 기준으로 표로 만들어줘.
```

**예시**

> **모드별 Output 비교표 — 입력: Set A `{id:1, name:"Alex"}`, Set B `{id:1, email:"alex@example.com"}`**
>
> | 모드 | Output | 특징 | 대표 용도 |
> |------|--------|------|---------|
> | **Append** | `[ {id:1, name:"Alex"}, {id:1, email:"..."} ]` (2개) | 두 입력을 순서대로 이어붙임 | 여러 소스의 기록을 모아 나열 |
> | **Combine** (by `id`) | `[ {id:1, name:"Alex", email:"..."} ]` (1개) | 매칭 키(id) 로 필드 결합 | DB JOIN 류, 다른 API 결과 합치기 |
> | **Choose Branch** (Input 1) | `[ {id:1, name:"Alex"} ]` | 한쪽만 통과, 다른쪽 버림 | IF/Switch 후 어느 경로를 쓸지 선택 |
>
> **분석 결과:**
>
> - **Append** 는 가장 단순 — 그냥 리스트 이어붙이기
> - **Combine** 은 SQL 의 INNER / LEFT JOIN 과 유사 — **매칭 키**가 핵심, 키가 없으면 빈 Output 가능
> - **Choose Branch** 는 "두 갈래 중 하나만 쓸 때" — 분기 후 재수렴보다는 선택적 통과 용도
> - 실무에서 "다른 API 로 받은 사용자 정보를 하나로 합치기" → Combine 이 정답

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
    IF 노드와 Merge 노드를 **조합** 하여, "분기 후 각 경로에서 다른 가공을 한 뒤 하나로 다시 합치는" 파이프라인을 완성하세요. 예: 금액이 큰 주문과 작은 주문을 나눠 각기 다른 `note` 를 붙인 뒤, Merge(Append) 로 최종 리스트를 만드는 구조.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우
- **구성:**

```
Manual Trigger → Set (샘플 주문 데이터 여러 건)
  → IF (amount >= 1000)
       ├─ True  → Set (note: "대형 주문 - 즉시 알림")
       └─ False → Set (note: "일반 주문 - 일일 집계")
                  → Merge (Mode: Append)
```

!!! tip "여러 아이템을 Manual Trigger 뒤에 만들려면"
    Set 노드의 Mode 를 **JSON** 으로 바꾸고, `[{amount:2000,...}, {amount:300,...}, {amount:1500,...}]` 형태로 배열을 입력하면 여러 아이템이 한 번에 생성됩니다.

**설계 순서:**

1. 첫 Set 노드에서 `amount` 가 다양한 샘플 3~5개 생성 (JSON 모드)
2. IF 노드: `amount >= 1000`
3. True/False 각 브랜치에 Set 노드를 붙여 `note` 필드 추가
4. 두 브랜치를 Merge(Append) 로 합치기
5. 실행 후 Output 에 **모든 아이템이 note 와 함께** 돌아오는지 확인

**예시**

> **완성 워크플로우: "주문 분류 → 재결합"**
>
> **입력 (Set, JSON 모드):**
>
> ```json
> [
>   { "orderId": "O-1", "amount": 2000 },
>   { "orderId": "O-2", "amount": 300 },
>   { "orderId": "O-3", "amount": 1500 }
> ]
> ```
>
> **IF True 쪽 Set 후:** `{ orderId, amount, note: "대형 주문 - 즉시 알림" }`
> **IF False 쪽 Set 후:** `{ orderId, amount, note: "일반 주문 - 일일 집계" }`
>
> **Merge (Append) 최종 Output:**
>
> ```json
> [
>   { "orderId": "O-1", "amount": 2000, "note": "대형 주문 - 즉시 알림" },
>   { "orderId": "O-3", "amount": 1500, "note": "대형 주문 - 즉시 알림" },
>   { "orderId": "O-2", "amount": 300,  "note": "일반 주문 - 일일 집계" }
> ]
> ```
>
> **구현 포인트:**
>
> - True 경로의 아이템이 먼저(Input 1), False 경로가 뒤(Input 2) — 순서를 뒤집고 싶으면 Merge 의 Input 연결을 바꾸기
> - Merge 뒤에 하나의 공통 알림 노드(예: Slack) 를 붙이면 **DRY 한 파이프라인** 완성

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

> - **Combine 모드의 Join 종류(Keep Matches / Keep Non-Matches / Enrich) 는 각각 SQL 의 어떤 JOIN 과 대응될까?**
>     - Keep Matches = INNER, Enrich Input 1 = LEFT, Enrich Input 2 = RIGHT 에 가깝다. 동일 데이터로 각 옵션을 실행해 Output 차이를 관찰해보세요.
> - **Merge 대신 Set 노드로 `={{ $json }}` 과 `={{ $node["NodeX"].json }}` 을 합쳐도 되지 않을까?**
>     - 특정 상황에서는 가능하지만 **아이템 수가 많을 때** 는 Merge 가 안정적이다. 두 방식의 성능·가독성 차이를 실험해보세요.
> - **Input 1 과 Input 2 의 아이템 개수가 다를 때 Combine 은 어떻게 동작할까?**
>     - 매칭 키 기준으로 남는 아이템은 버려지거나 유지된다(Join 옵션에 따라). 의도적으로 불균형한 입력을 만들어 확인해보세요.
> - **"Wait For All Inputs" 같은 옵션은 어떤 상황에 켜야 할까?**
>     - 비동기 트리거(예: Webhook 2개)에서 양쪽이 모두 도착한 뒤 합치고 싶을 때 유용. 해당 옵션을 켜고 한쪽만 도착시켜 보면 어떻게 되는지 관찰해보세요.
