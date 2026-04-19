---
title: "Lab 13. JSON / CSV / Binary 데이터 변환"
description: "Convert to File, Extract From File, Read/Write Binary 노드로 n8n 의 JSON 데이터와 파일(바이너리) 간 변환을 실습합니다."
tags:
  - n8n
  - 심화실습
  - File
  - CSV
---

# Lab 13. JSON / CSV / Binary 데이터 변환

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Convert to File 과 Extract From File 노드로 n8n 의 JSON items 와 CSV/JSON 파일(binary) 간 변환 왕복을 수행할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Code, Convert to File, Extract From File, HTTP Request |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 04·12 완료 (HTTP / Code) |
| 난이도 | ★★★★☆ |

!!! note "n8n 의 두 가지 데이터 채널"
    n8n 아이템은 **`json`** 과 **`binary`** 두 채널을 가집니다.
    
    - `json`: 우리가 지금까지 다룬 객체 데이터
    - `binary`: 파일 바이트 내용 (CSV, 이미지, PDF 등)
    
    파일 노드들은 `json` ↔ `binary` 변환의 **다리** 역할을 합니다.

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
    Set(JSON) 으로 **3개 아이템** 을 만들고, **Convert to File** 노드로 CSV 파일(binary) 로 변환한 뒤 Output 패널에서 "binary" 탭을 확인하여 파일 이름·크기·Content-Type 이 기록되는지 관찰하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 13 - 파일 변환`
- **Set(JSON) 입력:**

```json
[
  { "id": 1, "name": "Alex",  "city": "Seoul" },
  { "id": 2, "name": "Sam",   "city": "Busan" },
  { "id": 3, "name": "Kelly", "city": "Jeju" }
]
```

- **Convert to File 노드 설정:**

| 항목 | 값 |
|------|-----|
| Operation | `Convert to CSV` |
| Put Output File in Field | `data` |
| Options > File Name | `users.csv` |

**구성 단계:**

1. Manual Trigger → Set(JSON) → **Convert to File**
2. Operation 을 `Convert to CSV` 로 설정
3. **[Execute Workflow]** 실행
4. Convert to File 노드 Output 패널의 **Binary** 탭 클릭 → `data` 필드에 파일 정보 확인

??? note "예시"

    > **Convert to File 노드 Output — Binary 탭**
    >
    > | Key | Value |
    > |-----|-------|
    > | fileName | `users.csv` |
    > | fileExtension | `csv` |
    > | mimeType | `text/csv` |
    > | fileSize | `76 B` |
    >
    > **JSON 탭 (참고)**
    >
    > - JSON 탭은 원본 아이템을 그대로 전달하거나 비어 있을 수 있음
    > - 실제 데이터는 **Binary 탭** 에 들어 있음
    >
    > **파일 내용 미리보기 (Binary 탭 하단):**
    >
    > ```csv
    > id,name,city
    > 1,Alex,Seoul
    > 2,Sam,Busan
    > 3,Kelly,Jeju
    > ```
    >
    > **확인 포인트:**
    >
    > - 3개 아이템의 JSON 이 **하나의 CSV 파일(1 binary)** 로 합쳐졌음
    > - CSV 헤더는 json 키에서 자동 추출
    > - 이 파일을 HTTP Request / Google Drive / 이메일 첨부 등으로 바로 보낼 수 있음

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
    Step 1 의 CSV 파일을 **Extract From File** 노드로 다시 읽어 JSON items 로 복원하세요. 그리고 같은 구조로 **JSON 파일** 로 내보낸 뒤 다시 JSON 파일을 읽어 복원하는 왕복 파이프라인을 만들어 두 포맷의 차이를 분석하세요.

**활용 도구 & 데이터**

- **도구:** Step 1 파이프라인 뒤에 Extract From File 연결

**왕복 구조:**

```
Set(JSON) 3개
  → Convert to File (CSV)  ─┐
                             → Extract From File (From CSV) → JSON 3개
  → Convert to File (JSON)  ─┐
                             → Extract From File (From JSON) → JSON 3개
```

- **Extract From File (CSV 읽기) 설정:**

| 항목 | 값 |
|------|-----|
| Operation | `Extract From CSV` |
| Input Binary Field | `data` |
| Options > Header Row | `true` |

- **Extract From File (JSON 읽기) 설정:**

| 항목 | 값 |
|------|-----|
| Operation | `Extract From JSON` |
| Input Binary Field | `data` |

**관찰 포인트:**

1. CSV 를 거치면 **모든 값이 문자열** 로 돌아온다 (`id: "1"`, 숫자 아님)
2. JSON 을 거치면 **원본 타입이 보존** 된다 (`id: 1`)
3. CSV 는 표로 보기 쉽고, JSON 은 중첩 구조 보존에 유리

**(선택) 분석 프롬프트:**

```
CSV 왕복과 JSON 왕복 후 복원된 데이터 타입을 비교하여
"타입 보존 / 중첩 지원 / 용량 / 호환성" 4가지 기준으로 표를 만들어줘.
```

**예시**

> **CSV 왕복 후 Output (타입 소실)**
>
> ```json
> [
>   { "id": "1", "name": "Alex",  "city": "Seoul" },
>   { "id": "2", "name": "Sam",   "city": "Busan" },
>   { "id": "3", "name": "Kelly", "city": "Jeju" }
> ]
> ```
>
> (`id` 가 Number 에서 **String 으로 변환됨**)
>
> **JSON 왕복 후 Output (타입 보존)**
>
> ```json
> [
>   { "id": 1, "name": "Alex",  "city": "Seoul" },
>   { "id": 2, "name": "Sam",   "city": "Busan" },
>   { "id": 3, "name": "Kelly", "city": "Jeju" }
> ]
> ```
>
> **CSV vs JSON 비교표**
>
> | 기준 | CSV | JSON |
> |------|-----|------|
> | 타입 보존 | 모두 String (복원 시) | Number/Boolean/null 보존 |
> | 중첩 구조 | 불가 (flat 구조만) | 완전 지원 |
> | 용량 | 작음 (쉼표만) | 키명 반복으로 더 큼 |
> | 사람 가독성 | 표로 보기 좋음 | 구조 보기 좋음 |
> | 대표 사용처 | 엑셀, Google Sheets, 업로드 양식 | API 주고받기, 구성 파일 |
>
> **분석 결과:**
>
> - CSV 로 내보낸 후 다시 들여오면 **반드시 타입 복원 로직 필요** (Set 노드에서 Number 변환)
> - 중첩 오브젝트를 CSV 로 그대로 내보내면 `[object Object]` 가 찍힘 → 사전에 평탄화(flatten) 필요
> - API 간 데이터 전달은 JSON 이 기본, 사람이 볼 리포트·대량 업로드는 CSV 가 편함

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
    외부 API 에서 받은 JSON 을 가공해 **CSV 리포트 파일** 을 만들어 HTTP Response 로 다운로드할 수 있게 하거나 로컬 다운로드 테스트까지 이어가세요. `https://jsonplaceholder.typicode.com/users` 를 호출 → 필요한 필드만 추출 → CSV 변환 → 파일명에 오늘 날짜 포함.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우
- **구조:**

```
Manual Trigger
  → HTTP Request (GET https://jsonplaceholder.typicode.com/users)
  → Set (name, email, city, company.name 만 추출)
  → Convert to File (CSV, fileName: users_YYYY-MM-DD.csv)
```

- **Set 노드 필드 (Expression, Include Other Input Fields = OFF):**

| Field | Type | Expression |
|-------|------|-----------|
| name | String | `={{ $json.name }}` |
| email | String | `={{ $json.email }}` |
| city | String | `={{ $json.address.city }}` |
| company | String | `={{ $json.company.name }}` |

- **Convert to File 설정:**

| 항목 | 값 |
|------|-----|
| Operation | `Convert to CSV` |
| Put Output File in Field | `data` |
| Options > File Name | `=users_{{ $now.toFormat("yyyy-MM-dd") }}.csv` |

**구성 순서:**

1. HTTP Request 로 사용자 10명 받아오기
2. Set 노드로 4개 필드만 추출 (Implicit Loop 로 10회 실행됨)
3. Convert to File 이 10개 아이템을 한 CSV 파일로 합침
4. Output Binary 탭에서 파일명·내용 확인 → 다운로드 아이콘 클릭하여 실제 로컬 저장

**예시**

> **완성 워크플로우: "사용자 일일 CSV 리포트 생성기"**
>
> **파일명:** `users_2025-04-18.csv`
>
> **파일 내용 (발췌):**
>
> ```csv
> name,email,city,company
> Leanne Graham,Sincere@april.biz,Gwenborough,Romaguera-Crona
> Ervin Howell,Shanna@melissa.tv,Wisokyburgh,Deckow-Crist
> Clementine Bauch,Nathan@yesenia.net,McKenziehaven,Romaguera-Jacobson
> Patricia Lebsack,Julianne.OConner@kory.org,South Elvis,Robel-Corkery
> ... (총 10줄)
> ```
>
> **구현 포인트:**
>
> - 파일명에 **`{{ $now }}` 표현식** 이 들어가 매일 다른 파일명 자동 생성
> - 같은 파이프라인을 Schedule Trigger(Lab 02) 로 바꾸면 **매일 자동 리포트** 가 됨
> - Convert to File 뒤에 **Google Drive 업로드 노드(Lab 18 범위 확장)** 를 붙이면 완전 자동화
> - 이 패턴이 실무 자동화의 **가장 흔한 빌딩 블록** 중 하나

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

> - **Excel(.xlsx) 파일은 어떻게 다룰까?**
>     - Convert to File 의 Operation 에서 `Convert to XLSX` 선택 가능. 여러 시트를 가진 XLSX 를 만들려면 어떤 Options 가 필요한지 탐구해보세요.
> - **PDF 생성은 n8n 에서 직접 가능할까?**
>     - 기본 노드로는 제한적. Convert to File 에는 PDF 옵션이 없고, HTML 을 PDF 로 만드는 외부 API(예: DocRaptor, 자체 호스팅 puppeteer) 를 HTTP Request 로 호출하는 패턴을 조사해보세요.
> - **이미지 파일을 받아 Set 노드로 메타데이터만 추출하려면?**
>     - Extract From File 의 Extract Image Metadata 옵션, Code 노드의 `item.binary.data.fileSize` 등으로 접근. 이미지 몇 장을 테스트해보세요.
> - **바이너리 필드가 여러 개인 아이템(예: 첨부 3개)을 어떻게 만들고 다룰까?**
>     - `item.binary = { data: {...}, data2: {...} }` 처럼 키를 여러 개 둘 수 있다. 실제로 만들어 다음 노드가 어느 키를 기본으로 보는지 확인해보세요.
> - **압축(zip) 파일을 만들거나 풀 수 있을까?**
>     - Compression 노드가 있다. zip 생성·해제의 양방향 동작을 테스트해보세요.
