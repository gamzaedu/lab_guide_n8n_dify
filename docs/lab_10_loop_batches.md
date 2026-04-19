---
title: "Lab 10. Loop / Split In Batches 로 반복 처리"
description: "Loop Over Items 와 Split In Batches 로 여러 아이템을 반복 처리하고, 배치 크기를 조절해 API 쿼터를 관리합니다."
tags:
  - n8n
  - 중급실습
  - Loop
  - 반복처리
---

# Lab 10. Loop / Split In Batches 로 반복 처리

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | 여러 아이템을 반복 처리하는 n8n 의 기본 모델(Implicit Loop)과 Split In Batches 로 배치 크기를 제어하는 방법을 설명할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Split In Batches, HTTP Request (선택) |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 03~04 완료 필수 (Expression·HTTP Request 이해) |
| 난이도 | ★★★☆☆ |

!!! note "n8n 의 기본 반복 모델 — Implicit Loop"
    n8n 의 모든 노드는 **입력 아이템마다 자동으로 한 번씩 실행** 됩니다. 즉, 아이템이 5개 들어오면 뒤 노드도 5번 실행됩니다. 이것이 **Implicit Loop (묵시적 반복)** 입니다. 별도의 반복 노드 없이도 n8n 은 기본적으로 루프처럼 동작합니다.

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
    Set 노드의 **JSON 모드** 로 아이템 5개를 한 번에 만들고, 뒤에 Set 노드를 하나 더 붙여 각 아이템에 `processedAt` 필드를 추가하세요. 실행 후 두 번째 Set 노드가 **몇 번 실행되었는지** Output 패널에서 확인하세요 (Implicit Loop 관찰).

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 10 - 반복 처리`
- **첫 번째 Set 노드 (Mode: JSON):**

```json
[
  { "id": 1, "name": "A" },
  { "id": 2, "name": "B" },
  { "id": 3, "name": "C" },
  { "id": 4, "name": "D" },
  { "id": 5, "name": "E" }
]
```

- **두 번째 Set 노드 (Manual Mapping, `Include Other Input Fields` = ON):**

| Field | Type | Expression |
|-------|------|-----------|
| processedAt | String | `={{ $now.toFormat("HH:mm:ss.SSS") }}` |

**구성 단계:**

1. Manual Trigger → Set(JSON) 구성, 위 배열 입력
2. 두 번째 Set 노드 연결, `processedAt` 필드 추가
3. **[Execute Workflow]** 실행
4. 두 번째 Set 노드를 클릭 → Output 패널에서 아이템 개수와 `processedAt` 값 확인

??? note "예시"

    > **Output JSON (5개 아이템)**
    >
    > ```json
    > [
    >   { "id": 1, "name": "A", "processedAt": "14:32:05.101" },
    >   { "id": 2, "name": "B", "processedAt": "14:32:05.102" },
    >   { "id": 3, "name": "C", "processedAt": "14:32:05.103" },
    >   { "id": 4, "name": "D", "processedAt": "14:32:05.104" },
    >   { "id": 5, "name": "E", "processedAt": "14:32:05.105" }
    > ]
    > ```
    >
    > **확인 포인트:**
    >
    > - 두 번째 Set 노드 Output 에 **5개 아이템** 이 있음 → 노드가 5번 실행됨 (Implicit Loop)
    > - `processedAt` 값이 ms 단위로 조금씩 다름 → 각 아이템이 개별 실행되었다는 증거
    > - 반복 노드 없이도 n8n 은 **기본적으로 루프처럼 동작** — 이 개념을 이해해야 뒤 노드가 "내가 생각한 것과 다르게 여러 번 실행되는" 당혹감을 피할 수 있음

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
    아이템 5개를 **Split In Batches** 로 `Batch Size = 2` 로 쪼개 처리하세요. 루프 한 바퀴마다 몇 개씩 처리되는지, 마지막 배치의 개수가 어떻게 되는지, Output 패널에서 단계별로 관찰하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에서 두 번째 Set 노드 앞에 Split In Batches 삽입
- **구조:**

```
Manual Trigger → Set(JSON 5개)
  → Split In Batches (Batch Size: 2)
  → Set (processedAt 추가)
  → (Split In Batches 의 "loop" 출력으로 되돌아가는 선 연결)
```

- **Split In Batches 노드 설정:**

| 항목 | 값 |
|------|-----|
| Batch Size | `2` |
| Options | (기본값 유지) |

**연결 방법:**

1. `Set(JSON)` → **Split In Batches** → `Set(processedAt)` 순으로 연결
2. `Set(processedAt)` 노드의 출력을 **Split In Batches 노드의 위쪽 "loop" 입력** 으로 다시 연결 (되돌림)
3. Split In Batches 노드의 **"done" 출력** 에는 아무것도 연결하지 않거나, 최종 집계 노드를 연결
4. **[Execute Workflow]** 실행 → Split In Batches 의 Output 이 배치마다 어떻게 쌓이는지 관찰

**(선택) 분석 프롬프트:**

```
아이템 5개를 Batch Size 2 로 Split In Batches 처리하면
총 몇 바퀴가 돌고 각 바퀴에서 몇 개가 처리되는지 순서대로 설명해줘.
```

**예시**

> **배치별 동작 예측 vs 실제**
>
> | 바퀴 | 해당 배치 아이템 | Set 노드가 받는 개수 |
> |------|----------------|-------------------|
> | 1 | id=1, id=2 | 2 |
> | 2 | id=3, id=4 | 2 |
> | 3 | id=5 | 1 (마지막 배치) |
>
> **각 바퀴에서 Set 노드 Output (processedAt 시각이 다름):**
>
> ```
> 바퀴 1: [{id:1,...,processedAt:"14:35:01.010"}, {id:2,...,processedAt:"14:35:01.012"}]
> 바퀴 2: [{id:3,...,processedAt:"14:35:01.050"}, {id:4,...,processedAt:"14:35:01.052"}]
> 바퀴 3: [{id:5,...,processedAt:"14:35:01.090"}]
> ```
>
> **분석 결과:**
>
> - **Implicit Loop** 는 "아이템마다 순차 실행" — 개수 제어 불가
> - **Split In Batches** 는 "N개씩 묶어 반복" — **외부 API 쿼터(Rate Limit)** 관리에 필수
> - 마지막 배치는 Batch Size 보다 작을 수 있음 (나머지 처리)
> - Split In Batches 는 "done" 출력과 "loop" 출력이 있음 — done 은 모든 배치 완료 후 1회만 실행

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
    아이템 10개를 만든 뒤, **Split In Batches (Batch Size 3)** 와 **Wait 노드 (1초)** 를 조합하여 "3개씩 처리 후 1초 대기" 를 반복하는 파이프라인을 완성하세요. 마지막에 모든 배치가 끝난 뒤 1회만 실행되는 done 출력에 최종 집계 Set 노드를 붙여 `totalCount` 를 출력하세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우
- **구조:**

```
Manual Trigger
  → Set (JSON 10개)
  → Split In Batches (Batch Size: 3)
      ├─ (loop)  → Set(processedAt) → Wait(1초) → 다시 Split In Batches
      └─ (done)  → Set (totalCount: 10)
```

- **Wait 노드 설정:**

| 항목 | 값 |
|------|-----|
| Resume | `After Time Interval` |
| Wait Amount | `1` |
| Wait Unit | `Seconds` |

!!! tip "아이템 10개 빠르게 만들기"
    Code 노드를 쓰면 `return Array.from({length:10}, (_,i) => ({ json: { id: i+1 } }))` 한 줄로 생성 가능. 또는 Set(JSON) 에 10개를 직접 입력.

**구성 순서:**

1. Set(JSON) 으로 아이템 10개 생성
2. Split In Batches(Batch Size=3) 연결
3. loop 쪽: Set(processedAt) → Wait(1s) → Split In Batches 의 상단 입력으로 되돌림
4. done 쪽: Set (`totalCount`: `={{ $items().length }}`) 연결
5. 실행 후 총 소요 시간이 약 4초(배치 4바퀴 × 대기 1초) 이상인지 확인

**예시**

> **완성 워크플로우: "API 쿼터 관리 시뮬레이션"**
>
> **실행 흐름:**
>
> ```
> 배치 1 (id 1,2,3) 처리 → 1초 대기
> 배치 2 (id 4,5,6) 처리 → 1초 대기
> 배치 3 (id 7,8,9) 처리 → 1초 대기
> 배치 4 (id 10)    처리 → 1초 대기
> done 브랜치 실행 → totalCount: 10
> ```
>
> **done 브랜치 최종 Output:**
>
> ```json
> [ { "totalCount": 10 } ]
> ```
>
> **실무 응용 포인트:**
>
> - "API 가 초당 3 요청" 같은 제약이 있을 때 이 패턴으로 안전하게 반복 호출 가능
> - Batch Size 와 Wait 시간을 바꾸면 전체 처리 속도 vs 안정성을 조절 가능
> - done 분기는 **전체 배치 완료 시점에 딱 1회** 실행 — 통계·보고·알림에 최적

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

> - **Loop Over Items (별도 노드) 와 Split In Batches 는 어떻게 다를까?**
>     - 최신 n8n 의 `Loop Over Items` 노드는 아이템 1개씩 명시적으로 루프 돌리는 용도, Split In Batches 는 N 개씩 묶는 용도. 같은 데이터로 둘 다 써 보고 워크플로우 가독성이 어떻게 달라지는지 비교해보세요.
> - **Implicit Loop 를 명시적 Loop 로 바꿔야 할 때는 언제일까?**
>     - 아이템 간 순서 보장, 이전 결과를 다음에 쓰기, 에러 한 건 발생해도 계속 진행 같은 요구가 있을 때. 본인 시나리오에서 어느 경우에 필요한지 정리해보세요.
> - **배치 내부에서 에러가 나면 전체가 멈출까, 해당 배치만 멈출까?**
>     - 기본은 전체 멈춤. 노드의 `On Error` 설정을 바꿔가며 동작 차이를 관찰하고, Lab 15(Error Trigger) 와 어떻게 연결되는지 생각해보세요.
> - **루프 바퀴 수 상한을 두려면?**
>     - Split In Batches 의 Options 에 Reset 옵션, IF 노드로 강제 종료, 외부 카운터 등 다양한 방법이 있다. 조건부 종료 패턴을 직접 설계해보세요.
