---
title: "Lab 8. Expression 문법 마스터"
description: "n8n Expression 의 핵심 변수($json, $node, $now 등)와 Luxon 날짜·배열·문자열 메서드를 심화 실습합니다."
tags:
  - n8n
  - 중급실습
  - Expression
  - 문법
---

# Lab 8. Expression 문법 마스터

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | `$json`, `$node["이름"].json`, `$now`, `$items()` 등 n8n 내장 변수와 Luxon 날짜·문자열·배열 메서드를 Expression 으로 활용할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set) × 여러 개 |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 03 (Expression 기초) 완료 필수 |
| 난이도 | ★★★☆☆ |

!!! tip "Expression 입력 방법 복습"
    Value 칸을 클릭하면 오른쪽에 **Expression 편집기**가 뜹니다. 또는 값 앞에 `=` 를 붙이면 Expression 모드. 입력 아래 실시간 프리뷰에서 평가 결과가 바로 보입니다.

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
    Manual Trigger 뒤에 Set 노드 하나를 두고, **내장 변수 5종 이상** (`$now`, `$today`, `$workflow.name`, `$runIndex`, `$executionId`) 을 각각 필드로 만들어 Output 에서 실제 값이 어떻게 표시되는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 08 - Expression 마스터`
- **Set 노드 필드 (모두 Expression 모드, `=` 접두사):**

| Field | Type | Expression |
|-------|------|-----------|
| now | String | `={{ $now }}` |
| today | String | `={{ $today }}` |
| workflowName | String | `={{ $workflow.name }}` |
| runIndex | Number | `={{ $runIndex }}` |
| executionId | String | `={{ $executionId }}` |

**구성 단계:**

1. Manual Trigger → Set 노드 연결
2. 위 5개 필드를 Expression 모드로 입력 (`=` 붙이기 잊지 말 것)
3. Set 노드 설정 화면에서 각 Value 칸 아래의 **실시간 프리뷰** 로 미리 값 확인
4. **[Execute Workflow]** 실행 → Output 패널에서 실제 값 기록

??? note "예시"

    > **Output JSON (값은 실행 환경·시점에 따라 다름)**
    >
    > ```json
    > [
    >   {
    >     "now": "2025-04-18T14:32:05.123+09:00",
    >     "today": "2025-04-18T00:00:00.000+09:00",
    >     "workflowName": "Lab 08 - Expression 마스터",
    >     "runIndex": 0,
    >     "executionId": "12345"
    >   }
    > ]
    > ```
    >
    > **확인 포인트:**
    >
    > - `$now` 는 "지금 이 순간", `$today` 는 "오늘 자정" — Luxon DateTime 객체로 반환
    > - `$workflow.name` 은 워크플로우 이름을 문자열로 반환
    > - `$runIndex` 는 동일 실행 내의 반복 인덱스 (0 부터 시작)
    > - `$executionId` 는 이번 실행의 고유 ID (디버깅·로그에 유용)

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
    Luxon 날짜 메서드와 문자열·배열 메서드를 **각각 3개 이상** 사용하여 Set 노드 하나에 여러 필드를 만드세요. `$now.toFormat(...)`, `$now.plus({ days: 7 })`, `"abc".toUpperCase()`, `[1,2,3].length` 같은 패턴을 직접 실험하며 Expression 의 표현력을 측정하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에 필드 추가 (또는 새 Set 노드)
- **추가 필드:**

| Field | Type | Expression | 설명 |
|-------|------|-----------|------|
| nowFormatted | String | `={{ $now.toFormat('yyyy-MM-dd HH:mm') }}` | Luxon 포맷 |
| nextWeek | String | `={{ $now.plus({ days: 7 }).toFormat('yyyy-MM-dd') }}` | 7일 후 |
| dayOfWeek | String | `={{ $now.weekdayLong }}` | 요일 (영어) |
| upperName | String | `={{ "alex".toUpperCase() }}` | 대문자 |
| greeting | String | `={{ "Hello, World".split(", ").join(" / ") }}` | split + join |
| numbers | String | `={{ [10, 20, 30].reduce((a, b) => a + b, 0) }}` | 합계 |
| firstChar | String | `={{ "n8n".charAt(0) }}` | 첫 글자 |

**관찰 포인트:**

1. Luxon 의 DateTime 메서드는 `$now.plus(...)`, `$now.minus(...)`, `$now.toFormat(...)` 등 체이닝 가능
2. JavaScript 문자열/배열 표준 메서드 대부분 사용 가능 (`map`, `filter`, `reduce`, `split`, `join`)
3. Expression 안에서 함수(`.reduce((a,b)=>a+b)`)도 작성 가능하지만, 복잡해지면 Code 노드가 낫다

**(선택) 비교 프롬프트:**

```
아래 Expression 결과를 보고, n8n 에서 Luxon 을 JS Date 객체 대신 쓰는 이유를
"포맷팅 / 날짜 산술 / 타임존" 세 관점에서 설명해줘.

[Output] ...
```

**예시**

> **Output JSON (부분)**
>
> ```json
> {
>   "nowFormatted": "2025-04-18 14:32",
>   "nextWeek": "2025-04-25",
>   "dayOfWeek": "Friday",
>   "upperName": "ALEX",
>   "greeting": "Hello / World",
>   "numbers": 60,
>   "firstChar": "n"
> }
> ```
>
> **Expression 표현력 요약**
>
> | 카테고리 | 대표 메서드 | 활용 예 |
> |---------|-----------|--------|
> | Luxon 날짜 | `.toFormat()`, `.plus()`, `.minus()`, `.weekdayLong`, `.startOf("month")` | 리포트 파일명, 마감일 계산, 요일별 분기 |
> | 문자열 | `.toUpperCase()`, `.split()`, `.replace()`, `.includes()`, `.trim()` | 입력 정제, 키 추출, 대소문자 통일 |
> | 배열 | `.length`, `.map()`, `.filter()`, `.reduce()`, `.join()` | 합계, 요소 변환, 개수 기반 분기 |
> | 숫자 | `.toFixed()`, `Math.round()`, `Math.max()` | 소수점 정리, 가격 계산 |
>
> **분석 결과:**
>
> - Expression 은 **"Set 노드 한 줄로 해결"** 되는 가벼운 로직에 최적
> - 2줄 이상 복잡한 로직(조건문, 루프, try/catch) 은 **Code 노드(Lab 12)** 가 더 적합
> - Luxon 의 타임존·날짜 산술은 JS `Date` 보다 월등히 편함 → 실무에서는 Luxon 을 기본 툴로 삼을 것

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
    **두 개 이상의 Set 노드가 직렬로 연결된** 파이프라인에서 `$node["이름"].json` 을 사용해 **중간 노드의 값을 건너뛰어 첫 번째 노드의 값을 직접 참조** 하세요. 그리고 같은 파이프라인에서 Luxon 으로 "실행 시각에 7일을 더한 보고서 파일명" (예: `report_2025-04-25.csv`) 을 생성하세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우 (또는 Step 2 확장)
- **파이프라인 구조:**

```
Manual Trigger
  → Set(name="Alex")
  → Set(city="Seoul")
  → Set(최종 필드 조합)
```

- **최종 Set 노드 필드:**

| Field | Expression |
|-------|-----------|
| nameFromFirstNode | `={{ $node["Edit Fields"].json.name }}` (첫 번째 Set 노드명) |
| cityFromPrev | `={{ $json.city }}` |
| reportFile | `={{ "report_" + $now.plus({ days: 7 }).toFormat("yyyy-MM-dd") + ".csv" }}` |
| summary | `={{ $node["Edit Fields"].json.name }} lives in {{ $json.city }}` |

!!! tip "노드 이름 주의"
    `$node["이름"]` 의 "이름" 은 **캔버스에 보이는 노드 표시 이름** 입니다. 기본값은 `Edit Fields`, `Edit Fields1` 같은 형식이므로 정확히 입력해야 합니다. 노드를 우클릭해 이름을 변경하면 찾기 쉬워집니다.

**구성 순서:**

1. 첫 번째 Set 노드를 우클릭 → **Rename** 으로 `UserName` 으로 변경
2. 두 번째 Set 노드를 `UserCity` 로 변경
3. 세 번째 Set 노드의 Expression 에서 `$node["UserName"].json.name`, `$node["UserCity"].json.city` 참조
4. **[Execute Workflow]** 실행, Output 에서 `$node` 참조가 정상 동작하는지 확인

**예시**

> **완성 워크플로우: "실행 시각 기반 리포트 파일명 생성기"**
>
> **Output JSON**
>
> ```json
> [
>   {
>     "city": "Seoul",
>     "nameFromFirstNode": "Alex",
>     "cityFromPrev": "Seoul",
>     "reportFile": "report_2025-04-25.csv",
>     "summary": "Alex lives in Seoul"
>   }
> ]
> ```
>
> **구현 포인트 정리**
>
> - `$json` → **직전 노드** 의 출력 (가장 자주 쓰임)
> - `$node["NodeName"].json` → **임의의 앞선 노드** 의 출력을 건너뛰기 접근
> - `$now.plus({ days: 7 }).toFormat("yyyy-MM-dd")` → 날짜 산술 + 포맷팅 한 번에
> - 문자열 연결은 `+` 또는 템플릿 리터럴 `\`...\${}...\`` 모두 가능

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

> - **`$items()` 와 `$json` 은 어떻게 다를까?**
>     - `$json` 은 현재 아이템 1개, `$items()` 는 들어온 모든 아이템의 배열. 여러 아이템이 들어오는 상황에서 각각 어떻게 동작하는지 Set 노드로 실험해보세요.
> - **Luxon 의 `setZone("Asia/Seoul")` 로 타임존을 바꾸면 Output 은 어떻게 달라질까?**
>     - `$now.setZone("UTC").toFormat(...)` vs `$now.setZone("Asia/Seoul").toFormat(...)` 을 비교해보세요. 해외 API 연동 시 타임존 버그의 원인을 체감할 수 있습니다.
> - **Expression 에서 `try/catch` 나 `if` 문을 쓸 수 있을까?**
>     - 삼항 연산자(`? :`) 와 optional chaining(`?.`)으로 대부분 해결 가능하지만, 본격적인 로직은 Code 노드가 정답. 경계선을 직접 체감해보세요.
> - **`$workflow.id`, `$workflow.active` 같은 메타 정보는 언제 유용할까?**
>     - 로그 남기기, 실행 환경 식별, 조건부 동작에 활용 가능. Set 노드로 출력해 보고 "어떤 상황에 쓸 수 있을지" 3개 이상 적어보세요.
> - **Expression 에러 (빨간 밑줄) 가 떴을 때 가장 자주 보이는 원인은?**
>     - `$json.undefinedField.x` 같은 null 접근, 오타, `=` 누락 등. 일부러 에러를 만들어 프리뷰 메시지를 관찰해보세요.
