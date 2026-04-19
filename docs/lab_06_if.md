---
title: "Lab 6. IF 노드로 조건 분기"
description: "IF 노드로 데이터를 True/False 두 경로로 나누고, 조건식을 조합하여 워크플로우의 분기 로직을 구성합니다."
tags:
  - n8n
  - 중급실습
  - IF
  - 제어흐름
---

# Lab 6. IF 노드로 조건 분기

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | IF 노드로 데이터를 True/False 분기하고, 숫자·문자·Boolean 비교 연산과 AND/OR 조합을 사용할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), IF |
| 소요 시간 | 약 50분 |
| 사전 준비 | Lab 01~03 학습 완료 (Set 노드·Expression 이해 필수) |
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
    Manual Trigger → Set 노드(`score: 85`) → IF 노드 구조의 워크플로우를 만들어, **score 가 70 이상이면 True, 미만이면 False** 로 분기하도록 구성하세요. True 와 False 각 브랜치 뒤에 Set 노드를 하나씩 붙여 서로 다른 메시지를 출력하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 06 - IF 조건 분기`
- **노드 구성:**

```
Manual Trigger → Set (score 입력) → IF (score >= 70)
                                     ├─ True  → Set (result: "PASS")
                                     └─ False → Set (result: "FAIL")
```

- **IF 노드 조건 (Conditions):**

| 항목 | 값 |
|------|-----|
| Value 1 | `={{ $json.score }}` |
| Operation | `Larger or Equal` |
| Value 2 | `70` |
| Combinator | (조건 1개이므로 불필요) |

**구성 단계:**

1. Manual Trigger 추가
2. Set 노드 추가, 필드 `score` (Number) = `85`
3. **IF** 노드 추가, **Number > Larger or Equal** 연산 선택, 위 표대로 입력
4. IF 노드의 True 출력에 Set 노드 연결 → `result` (String) = `PASS`
5. IF 노드의 False 출력에 Set 노드 연결 → `result` (String) = `FAIL`
6. **[Execute Workflow]** 실행 → IF 노드에서 True 경로로 흘러가는지 확인
7. Set 노드의 `score` 값을 `50` 으로 바꾼 후 다시 실행 → False 경로로 전환되는지 확인

??? note "예시"

    > **score = 85 일 때 Output (True 분기)**
    >
    > - IF 노드 Output: `true` 브랜치에 1 item
    > - 마지막 Set 노드 Output:
    >
    > ```json
    > [ { "score": 85, "result": "PASS" } ]
    > ```
    >
    > **score = 50 일 때 Output (False 분기)**
    >
    > - IF 노드 Output: `false` 브랜치에 1 item
    > - 마지막 Set 노드 Output:
    >
    > ```json
    > [ { "score": 50, "result": "FAIL" } ]
    > ```
    >
    > **확인 포인트:** IF 노드는 두 개의 출력 포트(초록색 True, 빨간색 False)를 가집니다. 조건이 맞는 아이템은 True 포트로만, 맞지 않는 아이템은 False 포트로만 흐릅니다.

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
    IF 노드의 조건을 **2개 이상 조합** 하여 AND / OR 의 차이를 직접 관찰하세요. 데이터 타입별(Number / String / Boolean) 로 서로 다른 연산을 1번씩은 사용해야 하며, 조건을 덧붙일 때 결과가 어떻게 달라지는지 비교표를 만들어 정리하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에서 Set 노드 필드와 IF 조건을 확장
- **Set 노드 필드:**

| Field | Type | Value |
|-------|------|-------|
| score | Number | `85` |
| country | String | `KR` |
| isMember | Boolean | `true` |

- **IF 조건 (세 가지 버전을 차례로 실험):**

| 버전 | 조건 구성 | Combinator |
|------|---------|-----------|
| A | `score >= 70` AND `country equals "KR"` | AND |
| B | `score >= 70` AND `country equals "KR"` AND `isMember is true` | AND |
| C | `score >= 90` OR `isMember is true` | OR |

**변경 방법:**

1. IF 노드에서 **[Add Condition]** 클릭하여 2번째, 3번째 조건 추가
2. String 비교 시 Operation: `Equals` / `Not Equals` 선택 (대소문자 구분 옵션도 확인)
3. Boolean 비교 시 Operation: `Is True` / `Is False` 선택
4. Combinator 드롭다운에서 `AND` / `OR` 전환
5. 각 버전마다 실행 후 True/False 어느 쪽으로 흐르는지 기록

**(선택) 분석 프롬프트:**

```
아래 IF 조건 3가지에 대해 (score=85, country="KR", isMember=true) 데이터를 넣었을 때
각각의 True/False 결과를 예측하고, AND 와 OR 의 작동 방식을 2줄로 설명해줘.
```

**예시**

> **Data: `{ score: 85, country: "KR", isMember: true }`**
>
> | 버전 | 조건 | 평가 과정 | 결과 |
> |------|------|----------|------|
> | A | `score>=70 AND country="KR"` | `true AND true` | **True** |
> | B | `score>=70 AND country="KR" AND isMember=true` | `true AND true AND true` | **True** |
> | C | `score>=90 OR isMember=true` | `false OR true` | **True** |
>
> **Data 변경: `{ score: 60, country: "US", isMember: false }`**
>
> | 버전 | 평가 과정 | 결과 |
> |------|---------|------|
> | A | `false AND false` | **False** |
> | B | `false AND false AND false` | **False** |
> | C | `false OR false` | **False** |
>
> **분석 결과:**
>
> - **AND**: 조건 하나라도 false 면 전체 False — "모든 조건이 동시에 만족할 때만 통과"
> - **OR**: 조건 하나라도 true 면 전체 True — "어느 하나만 만족해도 통과"
> - 조건이 많아질수록 **디버깅 난이도가 빠르게 올라간다** → IF 여러 개로 쪼개거나 Switch 노드(Lab 07) 사용 검토
> - String 비교는 대소문자 구분이 기본이므로 `"KR"` 과 `"kr"` 은 다르게 취급됨 — 필요 시 Operation 옆 톱니 아이콘에서 Case Sensitive 옵션 조정

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
    본인이 정한 주제로 **3단계 이상의 분기 로직** 을 IF 노드만으로 구성하세요. 예: "결제 금액 + VIP 여부" 조합으로 `할인율` 을 3~4단계로 나누는 로직. IF 노드를 중첩(한 브랜치 뒤에 또 다른 IF)하여 계층적 분기 구조를 만드세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우 또는 Step 1~2의 워크플로우 확장
- **설계 포인트:**

| 항목 | 설명 |
|------|------|
| 입력 데이터 | 최소 2개 필드 (Number + Boolean 또는 String) |
| 분기 단계 | IF 노드 최소 2개를 연결한 **중첩 분기** 구조 |
| 출력 | 각 분기마다 서로 다른 최종 Set 노드로 다른 결과 출력 |

**예시**

> **완성 워크플로우: "할인율 결정기"**
>
> **입력:** `{ amount: 15000, isVip: true }`
>
> **구조:**
>
> ```
> Set (입력 데이터)
>   └─ IF #1: amount >= 10000 ?
>        ├─ True  ─ IF #2: isVip is true ?
>        │              ├─ True  → Set (discount: 20)
>        │              └─ False → Set (discount: 10)
>        └─ False ─ IF #3: isVip is true ?
>                       ├─ True  → Set (discount: 5)
>                       └─ False → Set (discount: 0)
> ```
>
> **4가지 입력별 예상 결과:**
>
> | amount | isVip | 경로 | discount |
> |--------|-------|------|----------|
> | 15000 | true | True → True | 20 |
> | 15000 | false | True → False | 10 |
> | 5000 | true | False → True | 5 |
> | 5000 | false | False → False | 0 |
>
> **최종 Output (amount=15000, isVip=true 경로):**
>
> ```json
> [ { "amount": 15000, "isVip": true, "discount": 20 } ]
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

> - **IF 노드 대신 Set 노드의 Expression `={{ $json.x > 70 ? "PASS" : "FAIL" }}` 를 써도 될까?**
>     - Expression 삼항 연산자로 동일한 분기를 표현할 수 있지만, 뒤 노드가 공통이면 Set 이 편하고 **뒤 노드가 다르면 IF 가 필수**. 두 방식의 가독성·유지보수성을 비교해보세요.
> - **IF 노드 중첩과 Switch 노드는 언제 무엇을 선택할까?**
>     - 분기가 3개 이상이면 IF 중첩은 금방 지저분해진다. Lab 07 에서 Switch 를 배운 뒤, 어느 기준에서 전환할지 직접 판단 기준을 만들어보세요.
> - **IF 노드가 배열(items 여러 개)을 받으면 어떻게 동작할까?**
>     - IF 는 아이템마다 개별 평가한다. Set 노드로 여러 아이템을 만들어 보낸 뒤 True 브랜치에 몇 개, False 브랜치에 몇 개가 가는지 확인해보세요.
> - **"거의 같지만 하나만 다른" 분기가 반복될 때 DRY 하게 만들려면?**
>     - Sub-workflow(Lab 16)로 공통 로직을 빼거나, 분기 뒤 노드를 Merge 로 다시 합치는 패턴을 떠올려보세요.
