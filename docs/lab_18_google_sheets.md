---
title: "Lab 18. Google Sheets 연동 (읽기·쓰기·업데이트)"
description: "Google Sheets 노드로 실제 스프레드시트를 읽고, 새 행을 추가하고, 기존 행을 업데이트하는 CRUD 패턴을 실습합니다."
tags:
  - n8n
  - 통합실습
  - GoogleSheets
  - CRUD
---

# Lab 18. Google Sheets 연동 (읽기·쓰기·업데이트)

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Google Sheets 노드로 행 읽기·추가·업데이트를 수행하고, 시트의 데이터를 n8n 워크플로우의 영구 저장소처럼 활용할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Google Sheets |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 17 완료 (Google Sheets OAuth2 credential 등록) |
| 난이도 | ★★★☆☆ |

!!! note "준비할 Google 스프레드시트"
    실습용 시트를 **하나만** 만들어 반복 사용합니다:
    
    - 이름: `n8n-lab-users`
    - Sheet1 에 A1:C1 헤더: `id | name | city`
    - 샘플 3행 입력:
    
    ```
    id | name  | city
    1  | Alex  | Seoul
    2  | Sam   | Busan
    3  | Kelly | Jeju
    ```

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
    `n8n-lab-users` 시트에서 **모든 행 읽기** 를 수행하세요. Google Sheets 노드의 `Get Rows` operation 으로 3개의 행을 아이템으로 불러와 Output 에서 헤더가 키, 셀 값이 밸류로 변환되는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud, Lab 17 에서 등록한 Google Sheets credential
- **워크플로우 이름:** `Lab 18 - Sheets 기초 CRUD`
- **Google Sheets 노드 설정 (Read):**

| 항목 | 값 |
|------|-----|
| Credential | `Google Sheets - Personal` |
| Resource | `Sheet within Document` |
| Operation | `Get Row(s)` |
| Document | 드롭다운에서 `n8n-lab-users` 선택 |
| Sheet | `Sheet1` |
| Options > Range | `A1:C` (기본값 유지 가능) |

**구성 단계:**

1. Manual Trigger → Google Sheets 노드 연결
2. 위 표대로 설정
3. **[Execute Workflow]** 실행
4. Output 에 3개 아이템이 JSON 형태로 들어오는지 확인

??? note "예시"

    > **Google Sheets Output JSON**
    >
    > ```json
    > [
    >   { "id": "1", "name": "Alex",  "city": "Seoul" },
    >   { "id": "2", "name": "Sam",   "city": "Busan" },
    >   { "id": "3", "name": "Kelly", "city": "Jeju" }
    > ]
    > ```
    >
    > **확인 포인트:**
    >
    > - 헤더 행(A1:C1)이 JSON **키** 로 자동 변환
    > - 각 데이터 행이 하나의 **아이템** 이 됨
    > - 모든 값은 **String** 으로 반환 (Sheets 셀 타입과 상관없이) — 필요시 Set 노드에서 Number/Boolean 으로 변환
    > - 빈 셀은 `""` (빈 문자열) 로 표시됨

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
    **Append Row** operation 으로 시트에 **새 행 1개** 를 추가하세요. 추가 후 다시 Get Rows 로 읽어 총 4행이 되었는지 확인하세요. 시트에 직접 가서 눈으로도 새 행이 들어왔는지 교차 검증하세요.

**활용 도구 & 데이터**

- **도구:** Step 1 워크플로우 확장
- **구성:**

```
Manual Trigger
  → Set (id, name, city 값 준비)
  → Google Sheets (Append Row)
  → Google Sheets (Get Rows) ← 검증용
```

- **Append 전 Set 노드:**

| Field | Type | Value |
|-------|------|-------|
| id | String | `4` |
| name | String | `Jordan` |
| city | String | `Incheon` |

- **Append 노드 설정:**

| 항목 | 값 |
|------|-----|
| Resource | `Sheet within Document` |
| Operation | `Append Row` |
| Document / Sheet | 동일 시트 |
| Mapping Column Mode | `Map Each Column Manually` |
| Values to Send | id/name/city 각각 Expression `={{ $json.id }}` 식으로 매핑 |

**관찰 포인트:**

1. Append 는 **맨 아래 빈 행** 을 찾아 값을 씀 (기존 데이터 위치 변경 없음)
2. Column Mode 를 `Auto-map Input Data` 로 바꾸면 입력 JSON 의 키를 자동으로 매칭
3. 실행 후 실제 Google Sheets 를 열어 교차 확인하는 것이 운영 습관상 중요

**(선택) 분석 프롬프트:**

```
Google Sheets 노드의 Append Row 와 Update Row 의 차이를
"대상 행 결정 방식 / 기존 데이터 변경 여부 / 흔한 실수" 세 기준으로 정리해줘.
```

**예시**

> **Append 전 시트 (3행) → 실행 → Append 후 시트 (4행)**
>
> ```
> id | name   | city
> 1  | Alex   | Seoul
> 2  | Sam    | Busan
> 3  | Kelly  | Jeju
> 4  | Jordan | Incheon   ← 새로 추가됨
> ```
>
> **Get Rows 재실행 Output**
>
> ```json
> [
>   { "id": "1", "name": "Alex",   "city": "Seoul" },
>   { "id": "2", "name": "Sam",    "city": "Busan" },
>   { "id": "3", "name": "Kelly",  "city": "Jeju" },
>   { "id": "4", "name": "Jordan", "city": "Incheon" }
> ]
> ```
>
> **분석 결과:**
>
> - **Append 는 "로그 쓰기"** 에 최적 — 이전 데이터 보존하며 끝에 추가
> - Column Mapping 을 **자동(Auto-map)** 으로 두면 Set 노드와 시트 헤더 이름만 맞추면 끝
> - Append 후 결과 행 번호는 **Output 에서 `row_number`** 로 받을 수 있음 (정확한 피드백 필요 시)

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
    **Update Row** operation 으로 `id=2` (Sam) 의 `city` 를 `Jeonju` 로 바꾸고, 이어서 **Append or Update** 로 "id 있으면 업데이트, 없으면 추가" 패턴(upsert) 을 실험하세요. 변경 후 Get Rows 로 전체를 다시 읽어 결과가 의도대로 반영되었는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** 이어서 같은 시트 사용
- **① Update Row 설정:**

| 항목 | 값 |
|------|-----|
| Operation | `Update Row` |
| Column to Match On | `id` |
| Value of Column to Match On | `2` |
| Columns to Send > city | `Jeonju` (나머지는 건드리지 않음) |

- **② Append or Update Row (Upsert) 설정:**

| 항목 | 값 |
|------|-----|
| Operation | `Append or Update Row` |
| Column to Match On | `id` |
| 입력 아이템 1 | `{ id: 2, name: "Sam", city: "Daegu" }` → 기존 업데이트 |
| 입력 아이템 2 | `{ id: 5, name: "Morgan", city: "Suwon" }` → 신규 추가 |

!!! tip "매칭 컬럼 값은 기존 데이터와 정확히 일치해야 함"
    Google Sheets 는 매칭을 **문자열 equals** 로 수행. `id=2` 와 `id="2"` 는 동일하게 처리되지만, 공백·대소문자·trailing 문자에 민감하니 주의.

**구성 순서:**

1. Update Row 노드로 id=2 의 city 변경 실행 → 실제 시트에서 변화 확인
2. Set(JSON) 으로 2개 아이템 (기존 수정 1개 + 신규 1개) 생성
3. Append or Update 노드에서 Column to Match = `id` 로 설정, 실행
4. Get Rows 로 전체 재확인 → 5행 (update + append) 이 되었는지 검증

**예시**

> **완성 워크플로우: "Sheets CRUD 종합"**
>
> **② Append or Update 실행 후 최종 시트**
>
> ```
> id | name   | city
> 1  | Alex   | Seoul
> 2  | Sam    | Daegu    ← Update (Jeonju에서 다시 Daegu 로 변경)
> 3  | Kelly  | Jeju
> 4  | Jordan | Incheon
> 5  | Morgan | Suwon    ← Append (신규)
> ```
>
> **최종 Get Rows Output**
>
> ```json
> [
>   { "id": "1", "name": "Alex",   "city": "Seoul" },
>   { "id": "2", "name": "Sam",    "city": "Daegu" },
>   { "id": "3", "name": "Kelly",  "city": "Jeju" },
>   { "id": "4", "name": "Jordan", "city": "Incheon" },
>   { "id": "5", "name": "Morgan", "city": "Suwon" }
> ]
> ```
>
> **구현 포인트:**
>
> - **Upsert 패턴** 은 주기적 동기화(API → Sheets)에서 핵심 — 중복 방지 + 변경 반영을 한 번에
> - **Update Row** 는 부분 업데이트 가능 (나머지 컬럼은 건드리지 않음)
> - Google Sheets 는 DB 처럼 쓰기엔 느리지만, 읽기·공유·시각화가 편해 **"작은 규모 영구 저장소"** 로 유용
> - 이 패턴 + HTTP Request (Lab 04) + Schedule Trigger (Lab 02) 로 "매일 API 수집 → 시트 업데이트" 자동화 구축 가능

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

> - **Delete Row operation 은 어떻게 작동할까?**
>     - 단순히 셀 값을 지우는 것과 행 자체 제거는 다르다. `Clear Cell Values` vs `Delete Row` 의 차이를 직접 실험해보세요.
> - **동시 업데이트(같은 시트를 2 워크플로우가 동시에 쓰기) 시 충돌은?**
>     - Google Sheets API 는 동시성 제어가 약함. Lock 역할을 n8n 수준에서 어떻게 설계할지 고민해보세요.
> - **여러 시트(Sheet1, Sheet2) 를 동시에 다루려면?**
>     - 각 노드마다 Sheet 만 다르게 지정하면 OK. 한 워크플로우에서 여러 시트에 쓰는 시나리오를 상상해보세요.
> - **시트 데이터를 서버에 영구 캐시하지 않고 "쿼리" 하려면?**
>     - n8n 공식 노드에는 SQL 같은 쿼리 기능이 없다. Code 노드에서 `filter` 로 처리하거나, Notion/Supabase 같은 진짜 DB 로 옮기는 것이 대안.
> - **대량 데이터(수천 행) 를 쓸 때는 어떤 방식이 가장 빠를까?**
>     - Append Row 를 루프로 호출하는 것보다 한 번에 여러 행을 넣는 batch 모드가 유리. Sheets API 쿼터 정책도 함께 조사해보세요.
