---
title: "Lab 1. 첫 워크플로우 만들기"
description: "Manual Trigger와 Set(Edit Fields) 노드를 연결하여 첫 n8n 워크플로우를 만들고, JSON 데이터를 생성·실행합니다."
tags:
  - n8n
  - 기초실습
  - Set노드
---

# Lab 1. 첫 워크플로우 만들기

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Manual Trigger와 Set 노드를 연결하여 워크플로우를 만들고, 원하는 형태의 JSON 데이터를 생성·실행할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set) |
| 소요 시간 | 약 40분 |
| 사전 준비 | n8n cloud 로그인 완료 |
| 난이도 | ★☆☆☆☆ |

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
    아래 안내를 따라 n8n에서 **Manual Trigger + Edit Fields(Set) 노드**를 연결한 첫 번째 워크플로우를 만드세요. Set 노드에서 필드 3개를 직접 입력하여 JSON 데이터를 생성하고, 실행하여 Output 패널에서 결과를 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **Set 노드에 입력할 필드 (Manual Mapping 모드):**

| Field Name | Type | Value |
|------------|------|-------|
| name | String | `Alex` |
| city | String | `Seoul` |
| role | String | `Student` |

**워크플로우 구성 단계:**

1. **새 워크플로우 생성**
    - n8n 홈 화면에서 **[Create Workflow]** 클릭
    - 좌측 상단의 워크플로우 이름을 `Lab 01 - 첫 워크플로우`로 변경

2. **Manual Trigger 노드 추가**
    - 캔버스 중앙의 **[Add first step]** 클릭
    - 검색창에 `Manual` 입력 → **Manual Trigger** 선택
    - 설정 없이 바로 사용 가능

3. **Edit Fields(Set) 노드 추가 및 연결**
    - Manual Trigger 노드 우측의 **[+]** 버튼 클릭
    - 검색창에 `Edit Fields` 또는 `Set` 입력 → **Edit Fields (Set)** 선택
    - **Mode:** `Manual Mapping` 확인

4. **필드 추가**
    - **[Add Field]** 버튼을 3번 클릭하여 필드 3개 생성
    - 위 표의 Field Name / Type / Value 를 그대로 입력

5. **실행 및 결과 확인**
    - 캔버스 좌측 하단의 **[Execute Workflow]** 버튼 클릭
    - Set 노드를 클릭하여 **Output** 패널에서 JSON 결과 확인

??? note "예시"

    > **입력 설정**
    >
    > | Field Name | Type | Value |
    > |---|---|---|
    > | name | String | `Alex` |
    > | city | String | `Seoul` |
    > | role | String | `Student` |
    >
    > **출력 결과** (Output 패널 — Table / JSON 뷰)
    >
    > ```json
    > [
    >   {
    >     "name": "Alex",
    >     "city": "Seoul",
    >     "role": "Student"
    >   }
    > ]
    > ```
    >
    > **확인 포인트:** Output 패널에 위 JSON과 동일한 구조가 보이면 성공입니다. 필드 3개가 정확한 key/value로 출력되어야 합니다.

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
    Step 1의 Set 노드에서 **Type(자료형)을 3가지 이상 바꿔서** 실행 결과가 어떻게 달라지는지 관찰하세요. 그리고 필드를 하나 추가·삭제해보며 Output 패널의 구조 변화를 비교 분석하세요.

**활용 도구 & 데이터**

- **도구:** Step 1에서 만든 워크플로우 (그대로 재활용)
- **실험할 Type 3종:**

| # | Field Name | Type | Value |
|---|-----------|------|-------|
| A | age | Number | `27` |
| B | isActive | Boolean | `true` |
| C | joinedAt | String | `2025-01-15` |

**변경 방법:**

1. Set 노드를 더블클릭하여 설정 화면 열기
2. **[Add Field]** 로 위 3개 필드를 추가 (Type 드롭다운에서 Number / Boolean / String 선택)
3. **[Execute Workflow]** 실행 후 Output 패널에서 JSON 값의 따옴표 유무 관찰
4. 필드 하나를 **[Remove]** 로 삭제 후 다시 실행
5. Type 을 틀리게 지정하면 (예: Number 에 `abc` 입력) 어떤 에러가 나는지 확인

**(선택) 동료·AI 와 비교:**

```
아래 Set 노드 Output JSON 을 보고, 자료형(String / Number / Boolean)에 따라
표시 방식이 어떻게 달라지는지 3줄 이내로 정리해줘.

[내 Output] ...
```

**예시**

> **비교 분석표 — Type 에 따른 출력 차이**
>
> | Type | 입력값 | Output JSON | 특징 |
> |------|-------|------------|------|
> | String | `27` | `"age": "27"` | 따옴표로 감싸져 문자열로 저장됨 |
> | Number | `27` | `"age": 27` | 따옴표 없이 숫자로 저장, 산술 연산 가능 |
> | Boolean | `true` | `"isActive": true` | 따옴표 없이 `true`/`false`만 허용 |
> | String | `true` | `"isActive": "true"` | 문자 "true"로 저장, 조건 판단 시 의미 달라짐 |
>
> **분석 결과:**
>
> - Set 노드에서 **Type 을 정확히 지정해야** 뒤 노드에서 원하는 방식으로 활용할 수 있다 (예: Number 여야 IF 노드에서 `>` 비교 가능)
> - 필드를 추가하면 Output JSON 의 key 가 늘어나고, 삭제하면 해당 key 가 사라진다 — **Set 노드의 출력은 입력 필드 구성과 1:1 대응**
> - Type 불일치 에러는 노드 실행 전 빨간 경고로 표시되므로, 실행 전에 체크하는 습관이 중요

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
    Step 1~2의 경험을 바탕으로, **본인을 소개하는 "데이터 카드" 워크플로우**를 완성하세요. Set 노드 하나에 **최소 5개 필드**를 서로 다른 Type(String/Number/Boolean 최소 1개씩 포함)으로 구성하고, 실행해 본인만의 프로필 JSON 을 출력하세요.

**활용 도구 & 데이터**

- **도구:** Step 1~2에서 만든 워크플로우 (또는 새로 생성)
- **설계 포인트:**

| 항목 | 설명 |
|------|------|
| 주제 | 자기 자신 / 가상의 캐릭터 / 좋아하는 책·영화 등 자유 |
| 필드 개수 | 최소 5개 |
| Type 조합 | String / Number / Boolean 각 1개 이상 포함 |
| 워크플로우 이름 | 용도에 맞게 변경 후 **[Save]** |

**설계 순서:**

1. 종이나 메모에 필드 목록을 먼저 작성
2. 각 필드의 Type 을 미리 결정 (String / Number / Boolean)
3. Set 노드에서 **[Add Field]** 로 하나씩 추가
4. **[Execute Workflow]** 실행 후 Output JSON 이 의도한 구조인지 확인
5. 워크플로우 이름을 `Lab 01 - 내 프로필 카드` 처럼 변경하고 **[Save]**

**예시**

> **완성 워크플로우: "내 프로필 카드"**
>
> | Field Name | Type | Value |
> |-----------|------|-------|
> | name | String | `Alex` |
> | age | Number | `27` |
> | city | String | `Seoul` |
> | favoriteLanguage | String | `Python` |
> | yearsOfExperience | Number | `3` |
> | isLearningN8n | Boolean | `true` |
>
> **Output JSON:**
>
> ```json
> [
>   {
>     "name": "Alex",
>     "age": 27,
>     "city": "Seoul",
>     "favoriteLanguage": "Python",
>     "yearsOfExperience": 3,
>     "isLearningN8n": true
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

> - **Set 노드의 Mode 를 `Manual Mapping` 대신 `JSON` 으로 바꾸면 어떻게 될까?**
>     - JSON 모드에서는 전체 오브젝트를 한 번에 `{"name":"Alex",...}` 형태로 작성할 수 있다. Manual Mapping 과 어떤 장단이 있는지 직접 입력해 비교해보세요.
> - **Value 칸에 `={{ $now }}` 처럼 `=` 와 `{{ }}` 를 붙이면 어떤 차이가 있을까?**
>     - 고정값 대신 현재 시각 같은 동적 값이 출력된다. `$now`, `$today`, `$runIndex` 등 n8n 내장 변수를 Value 에 넣어 실행해보세요.
> - **Set 노드 하나로 Output 에 여러 개의 아이템(배열)을 만들 수 있을까?**
>     - Set 노드는 기본적으로 입력 아이템 개수만큼 출력한다. Manual Trigger 대신 Code 노드나 Split Out 노드로 여러 아이템을 만든 뒤 Set 노드를 연결하면 어떤 결과가 나오는지 실험해보세요.
> - **`Keep Other Fields` 옵션을 켜고 끄면 Output 이 어떻게 달라질까?**
>     - 이 옵션은 입력으로 들어온 다른 필드를 유지할지 여부를 결정한다. Manual Trigger 뒤라서 큰 차이가 없지만, 노드를 체인으로 연결하면 중요해진다. 앞 노드 출력이 있는 상황을 만들어 비교해보세요.
