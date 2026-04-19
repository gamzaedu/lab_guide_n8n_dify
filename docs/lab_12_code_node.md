---
title: "Lab 12. Code 노드로 JavaScript 데이터 처리"
description: "Code 노드에서 JavaScript 로 items 를 순회·필터·변환하며, Expression 만으로는 어려운 로직을 구현합니다."
tags:
  - n8n
  - 심화실습
  - Code
  - JavaScript
---

# Lab 12. Code 노드로 JavaScript 데이터 처리

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Code 노드에서 `items` 배열을 직접 조작하여 필터·매핑·집계하고, 복잡한 데이터 변환 로직을 짧은 JS 코드로 구현할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Code |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 03·08 (Expression) 완료 필수, 기본 JavaScript 문법 이해 권장 |
| 난이도 | ★★★★☆ |

!!! note "Code 노드의 두 가지 실행 모드"
    - **Run Once for All Items**: `items` 배열 전체를 한 번 처리 (집계·필터링 등에 적합)
    - **Run Once for Each Item**: 아이템마다 1회씩 실행 (단순 매핑에 편하지만 느림)
    
    이번 실습은 주로 **Run Once for All Items** 모드를 사용합니다.

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
    Manual Trigger 뒤에 Code 노드를 붙여, `Array.from(...)` 으로 아이템 **5개** 를 생성하세요. 각 아이템은 `id` (1~5), `squared` (id²), `isEven` (id 가 짝수인지) 필드를 가져야 합니다.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 12 - Code 노드`
- **Code 노드 설정:**

| 항목 | 값 |
|------|-----|
| Language | `JavaScript` |
| Mode | `Run Once for All Items` |

**Code 내용:**

```javascript
const result = Array.from({ length: 5 }, (_, i) => {
  const id = i + 1;
  return {
    json: {
      id,
      squared: id * id,
      isEven: id % 2 === 0
    }
  };
});

return result;
```

**구성 단계:**

1. Manual Trigger → **Code** 노드 연결
2. Language: JavaScript, Mode: Run Once for All Items
3. 위 코드를 복사해 입력
4. **[Execute Workflow]** → Output 확인

??? note "예시"

    > **Output JSON (5개 아이템)**
    >
    > ```json
    > [
    >   { "id": 1, "squared": 1,  "isEven": false },
    >   { "id": 2, "squared": 4,  "isEven": true },
    >   { "id": 3, "squared": 9,  "isEven": false },
    >   { "id": 4, "squared": 16, "isEven": true },
    >   { "id": 5, "squared": 25, "isEven": false }
    > ]
    > ```
    >
    > **확인 포인트:**
    >
    > - Code 노드의 반환값은 **반드시 `[{ json: {...} }, ...]` 형태** 여야 함 (n8n 의 아이템 구조)
    > - 간단한 배열(`[1,2,3]`) 을 바로 return 하면 에러
    > - `json` 키 안에 실제 데이터가 들어감

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
    Set(JSON) 노드로 제품 데이터 10개를 만든 뒤, Code 노드에서 `items` 를 입력으로 받아 **(1) 가격 500 이상만 필터링, (2) `total` 필드 추가 (quantity × price), (3) total 내림차순 정렬** 을 한 번에 수행하세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우 또는 Step 1 확장
- **Set(JSON) 입력 데이터:**

```json
[
  { "product": "A", "price": 300, "quantity": 2 },
  { "product": "B", "price": 800, "quantity": 1 },
  { "product": "C", "price": 500, "quantity": 5 },
  { "product": "D", "price": 200, "quantity": 10 },
  { "product": "E", "price": 1200, "quantity": 1 },
  { "product": "F", "price": 450, "quantity": 3 },
  { "product": "G", "price": 900, "quantity": 2 },
  { "product": "H", "price": 600, "quantity": 4 },
  { "product": "I", "price": 150, "quantity": 20 },
  { "product": "J", "price": 1000, "quantity": 2 }
]
```

- **Code 노드 (All Items):**

```javascript
const result = items
  .map(item => ({
    json: {
      ...item.json,
      total: item.json.price * item.json.quantity
    }
  }))
  .filter(item => item.json.price >= 500)
  .sort((a, b) => b.json.total - a.json.total);

return result;
```

**관찰 포인트:**

1. n8n Code 노드에서 `items` 는 `[{ json: {...} }, ...]` 형태 — `.json` 으로 실제 값 접근
2. map → filter → sort 체인으로 **한 노드 안에서 다단계 변환** 가능
3. Set 노드로는 여러 번 연결해야 하는 작업이 Code 한 번에 끝남

**(선택) 프롬프트:**

```
아래 코드를 Set 노드 체인으로 만들면 몇 개 노드가 필요하고,
Code 노드 한 개와 비교해 유지보수 관점에서 어느 쪽이 유리한지 2줄씩 분석해줘.

[코드] ...
```

**예시**

> **Code 노드 Output**
>
> ```json
> [
>   { "product": "E", "price": 1200, "quantity": 1, "total": 1200 },
>   { "product": "C", "price": 500,  "quantity": 5, "total": 2500 },
>   { "product": "J", "price": 1000, "quantity": 2, "total": 2000 },
>   { "product": "G", "price": 900,  "quantity": 2, "total": 1800 },
>   { "product": "H", "price": 600,  "quantity": 4, "total": 2400 },
>   { "product": "B", "price": 800,  "quantity": 1, "total": 800 }
> ]
> ```
>
> 정렬 기준(total 내림차순) 대로 정렬하면:
>
> ```json
> [
>   { "product": "C", "total": 2500 },
>   { "product": "H", "total": 2400 },
>   { "product": "J", "total": 2000 },
>   { "product": "G", "total": 1800 },
>   { "product": "E", "total": 1200 },
>   { "product": "B", "total": 800 }
> ]
> ```
>
> **분석 결과:**
>
> - Code 1 개 = Set(total 추가) + IF(filter) + 정렬 불가(Set로는 불가) → **Set 만으로는 구현 불가능한 로직**
> - `items` 를 직접 조작 가능 → JS 의 `map/filter/reduce/sort` 표준 메서드 전부 사용 가능
> - 복잡한 로직은 Code 가 유리하지만, **노드 이름만 봐서는 어떤 로직인지 알 수 없다** 는 단점 → 노드 이름 네이밍이 중요

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
    Step 2 의 데이터를 다시 사용하여, Code 노드로 **집계 리포트** 를 한 아이템으로 반환하세요. 반환 아이템은 `totalItems`(개수), `totalRevenue`(모든 price×quantity 합), `avgPrice`(평균가격, 소수점 2자리), `topProduct`(최고 매출 상품명) 네 필드를 포함해야 합니다.

**활용 도구 & 데이터**

- **도구:** Step 2 의 데이터를 입력으로 사용 (Code 노드 1개 추가 또는 교체)
- **Code 내용 힌트:**

```javascript
const data = items.map(i => i.json);

const totalItems = data.length;
const totalRevenue = data.reduce((sum, d) => sum + d.price * d.quantity, 0);
const avgPrice = Number(
  (data.reduce((s, d) => s + d.price, 0) / data.length).toFixed(2)
);

const topProduct = data
  .map(d => ({ ...d, total: d.price * d.quantity }))
  .sort((a, b) => b.total - a.total)[0].product;

return [{
  json: { totalItems, totalRevenue, avgPrice, topProduct }
}];
```

**구성 순서:**

1. Set(JSON) 10개 → Code 노드 연결
2. 위 코드 입력
3. 실행 후 Output 이 **1개 아이템** 으로 축약되는지 확인
4. 집계 결과가 손계산과 일치하는지 검증

**예시**

> **완성 워크플로우: "상품 매출 집계 리포트"**
>
> **Output JSON (1개 아이템)**
>
> ```json
> [
>   {
>     "totalItems": 10,
>     "totalRevenue": 14300,
>     "avgPrice": 610.00,
>     "topProduct": "C"
>   }
> ]
> ```
>
> **구현 포인트:**
>
> - **여러 아이템 → 1개 아이템 집계** 는 Code 노드의 전형적 용도
> - `reduce` 로 합계, `sort()[0]` 로 최고값, `toFixed(2)` 로 소수점 정리
> - 리턴 배열의 길이가 **1** 이기 때문에 뒤 노드는 1번만 실행됨 (Implicit Loop 끊김)
> - 이 패턴 + Slack(Lab 20) 을 결합하면 "일일 매출 리포트 알림" 자동화 가능

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

> - **Code 노드의 `Run Once for Each Item` 모드는 언제 써야 할까?**
>     - 각 아이템이 서로 독립적인 변환일 때, 또는 한 아이템에 오류가 나도 다른 아이템을 계속 처리하고 싶을 때 유리하다. 같은 작업을 두 모드로 만들어 차이를 체감해보세요.
> - **Code 노드에서 `$input.all()`, `$input.first()`, `$input.item` 은 각각 무엇을 반환할까?**
>     - n8n 0.x 와 1.x 에서 권장 API 가 조금 다르다. 공식 문서와 실제 반환값을 비교하여 상황별 추천 API 를 정리해보세요.
> - **Code 노드에서 npm 모듈을 쓸 수 있을까?**
>     - Cloud 에서는 제한적, Self-host 에서는 `NODE_FUNCTION_ALLOW_EXTERNAL` 환경변수 설정으로 허용 가능. 실습 중 원하는 라이브러리가 있다면 어떻게 접근할 수 있을지 탐구해보세요.
> - **Code 노드 vs AI Agent 노드 — 복잡한 데이터 변환을 무엇으로 할까?**
>     - 확정적 로직은 Code, 모호하고 자연어 기반 변환은 AI. 본인이 다룰 실무 시나리오에서 경계를 어디에 그을지 고민해보세요.
> - **`throw new Error(...)` 로 에러를 일부러 발생시키면 워크플로우는 어떻게 반응할까?**
>     - 전체 실행이 실패 처리되고 Executions 탭에 빨간 마커. Lab 15(Error Trigger) 와 연결해 에러 알림 플로우를 만들 수 있다. 직접 시도해보세요.
