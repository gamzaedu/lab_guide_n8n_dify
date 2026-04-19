---
title: "Lab 7. Switch 노드로 다중 분기"
description: "Switch 노드로 3개 이상의 경로를 한 번에 라우팅하고, 규칙 우선순위와 Fallback 동작을 익힙니다."
tags:
  - n8n
  - 중급실습
  - Switch
  - 제어흐름
---

# Lab 7. Switch 노드로 다중 분기

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Switch 노드로 3개 이상의 경로를 한 번에 라우팅하고, 규칙 우선순위·Fallback 동작을 이해할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Switch |
| 소요 시간 | 약 50분 |
| 사전 준비 | Lab 06 (IF 노드) 완료 권장 |
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
    `category` 필드(`"A"` / `"B"` / `"C"`) 에 따라 **3개의 경로** 로 나뉘는 Switch 노드를 구성하세요. 각 경로 뒤에 Set 노드를 붙여 `label` 필드에 `"Alpha"` / `"Beta"` / `"Gamma"` 를 각각 설정하고, category 값을 바꿔가며 실행 경로가 전환되는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 07 - Switch 다중 분기`
- **Set 노드 (입력 데이터):**

| Field | Type | Value |
|-------|------|-------|
| category | String | `A` |

- **Switch 노드 설정:**

| 항목 | 값 |
|------|-----|
| Mode | `Rules` |
| Routing Rules | 3개 |

- **Routing Rules (Rule마다 Output 이름 지정 가능):**

| # | Value 1 | Operation | Value 2 | Output Name |
|---|---------|-----------|---------|-------------|
| 1 | `={{ $json.category }}` | `Equals` | `A` | `A` |
| 2 | `={{ $json.category }}` | `Equals` | `B` | `B` |
| 3 | `={{ $json.category }}` | `Equals` | `C` | `C` |

**구성 단계:**

1. Manual Trigger → Set (category=`A`) 구성
2. **Switch** 노드 추가, Mode: `Rules`
3. **[Add Routing Rule]** 3번 클릭하여 위 표대로 3개 규칙 입력
4. Switch 노드의 각 출력 포트(A/B/C)에 Set 노드를 1개씩 연결
    - A 출력 → Set(`label: Alpha`)
    - B 출력 → Set(`label: Beta`)
    - C 출력 → Set(`label: Gamma`)
5. **[Execute Workflow]** 실행 → A 경로로 흘러가는지 확인
6. 앞의 Set 노드에서 `category` 를 `B`, `C` 로 변경하며 재실행

??? note "예시"

    > **category = `A` 일 때**
    >
    > - Switch 노드의 A 포트에 1 item 출력, B/C 포트는 비어 있음
    > - 최종 Output: `[ { "category": "A", "label": "Alpha" } ]`
    >
    > **category = `B` 일 때**
    >
    > - Switch 노드의 B 포트에 1 item
    > - 최종 Output: `[ { "category": "B", "label": "Beta" } ]`
    >
    > **확인 포인트:** Switch 노드는 **규칙 수만큼 출력 포트** 를 자동으로 만들어줍니다. IF 노드처럼 True/False 두 개가 아니라 N 개로 확장된 개념입니다.

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
    Switch 노드의 **Fallback Output** 옵션을 켜고, category 값이 `A`/`B`/`C` 가 아닌 경우(예: `"D"`) 어디로 흐르는지 관찰하세요. 추가로 **규칙을 중복 매칭** 되게 (예: Rule 1: `length > 0`, Rule 2: `equals "A"`) 설정하여 "규칙 순서가 결과에 영향을 주는지" 실험하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우
- **실험 1 — Fallback:**

| 항목 | 설정 |
|------|------|
| Switch 노드 Options | **Fallback Output**: `Extra Output` 선택 |
| 입력 category | `D` (어떤 규칙에도 매치 안 됨) |

- **실험 2 — 중복 매칭 규칙:**

| # | Operation | Value 2 | Output Name |
|---|-----------|---------|-------------|
| 1 | String Exists | (자동) | `any` |
| 2 | Equals | `A` | `onlyA` |

**변경 방법:**

1. Switch 노드 하단의 **[Add Option]** → `Fallback Output` → `Extra Output` 선택
2. 새로 생긴 Fallback 포트에 Set 노드 연결 (`label: unknown`)
3. category 를 `D` 로 바꿔 실행 → Fallback 경로 확인
4. Switch 의 Mode 하위 **Options > Send Data to All Matching Outputs** 토글을 켜고 끄며 중복 매칭 동작 비교

**(선택) 프롬프트:**

```
n8n Switch 노드에서 "Send Data to All Matching Outputs" 옵션 ON/OFF 차이를
데이터가 하나 있고 규칙 여러 개가 동시에 매치될 때 어떻게 달라지는지 2줄로 설명해줘.
```

**예시**

> **실험 1 — Fallback 경로 테스트**
>
> | category | 매치되는 Rule | 흐름 | 최종 Output |
> |----------|--------------|------|-------------|
> | A | Rule 1 | A 포트 → Alpha Set | `label: "Alpha"` |
> | D | (없음) | **Fallback 포트** → unknown Set | `label: "unknown"` |
>
> **실험 2 — 중복 매칭 vs 첫 매치만**
>
> | Send Data to All Matching Outputs | 결과 |
> |-----------------------------------|------|
> | OFF (기본) | Rule 1(any) 에 매치되는 순간 거기로만 흐름 → onlyA 포트는 비어 있음 |
> | ON | Rule 1, Rule 2 둘 다 매치되면 **양쪽 포트에 각각 복사** → 동일 아이템이 두 경로로 흐름 |
>
> **분석 결과:**
>
> - **Fallback** 은 "예상 못한 값" 을 안전하게 처리하는 장치 — 항상 켜 두는 것이 운영 안정성 측면에서 유리
> - **규칙 순서** 는 "첫 매치만" 모드에서 결정적 — 위에 있는 규칙이 먼저 평가되므로 더 좁은 조건을 위로 배치해야 의도대로 흐름
> - **모든 매치 모드** 는 동일 아이템을 여러 경로로 복제 — fan-out 이 필요한 케이스에서만 의도적으로 사용

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
    본인이 정한 주제로 **4개 이상의 분기 + Fallback** 을 가진 Switch 워크플로우를 완성하세요. 각 분기마다 서로 다른 Set 노드 결과물이 나오도록 구성하고, 의도적으로 매치되지 않는 케이스를 하나 이상 테스트하세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우 또는 Step 1~2 확장
- **설계 포인트:**

| 항목 | 설명 |
|------|------|
| 분기 기준 필드 | 1개 (예: `level`, `status`, `priority`) |
| 규칙 개수 | 4개 이상 |
| Fallback | 필수 |
| 각 분기별 출력 | 서로 다른 `label` 또는 `action` 필드 |

**예시**

> **완성 워크플로우: "HTTP 상태코드 분류기"**
>
> **입력:** `{ statusCode: 200 }`
>
> **Switch 규칙 (5개 + Fallback):**
>
> | # | Operation | Value 2 | Output Name | 이후 Set |
> |---|-----------|---------|-------------|---------|
> | 1 | Equals | `200` | `ok` | `action: "기록"` |
> | 2 | Equals | `301` | `redirect` | `action: "새 URL 따라가기"` |
> | 3 | Equals | `404` | `notFound` | `action: "폴백 데이터 사용"` |
> | 4 | Equals | `500` | `serverError` | `action: "관리자 알림"` |
> | Fallback | - | - | `other` | `action: "로그 남기기"` |
>
> **여러 입력별 결과:**
>
> | statusCode | 경로 | action |
> |-----------|------|--------|
> | 200 | ok | 기록 |
> | 301 | redirect | 새 URL 따라가기 |
> | 404 | notFound | 폴백 데이터 사용 |
> | 500 | serverError | 관리자 알림 |
> | 418 | Fallback | 로그 남기기 |

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

> - **Switch 의 Mode 에 `Expression` 이 있는데 `Rules` 와 어떻게 다를까?**
>     - Expression 모드는 `={{ $json.category === "A" ? 0 : 1 }}` 처럼 **출력 포트 인덱스를 직접 반환** 한다. Rules 가 "UI 로 조건 관리", Expression 이 "코드로 관리" 라는 관점에서 비교해보세요.
> - **IF 중첩 vs Switch — 어느 기준에서 전환해야 할까?**
>     - "분기 수 ≥ 3" 이 기본 전환점. 유지보수성, 가독성, 성능 관점에서 자신만의 기준을 정리해보세요.
> - **Switch 뒤에 여러 경로를 다시 하나로 합치려면?**
>     - Lab 09 에서 배울 **Merge 노드** 가 그 역할. 각 분기에서 다른 전처리를 하고 마지막에 공통 알림 노드로 합치는 패턴을 상상해보세요.
> - **Fallback 경로에 "에러 발생" 로직을 넣는 게 좋을까, "무시" 가 좋을까?**
>     - 실무에서는 어떤 미매치 케이스가 "버그의 신호" 인지, "정상의 일부" 인지 결정하는 것이 중요하다. 본인이 만든 Fallback 을 두 관점에서 점검해보세요.
