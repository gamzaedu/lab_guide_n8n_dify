---
title: "Lab 21. OpenAI 노드 기초"
description: "OpenAI 노드로 System/User Message 를 구성하고, 모델·Temperature 옵션과 JSON 모드로 응답을 제어합니다."
tags:
  - n8n
  - 통합실습
  - OpenAI
  - AI
---

# Lab 21. OpenAI 노드 기초

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | OpenAI 노드로 프롬프트를 구성하고 응답을 받아오며, System Message 와 옵션(Temperature / JSON 모드)으로 출력을 제어할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), OpenAI |
| 소요 시간 | 약 60분 |
| 사전 준비 | OpenAI API Key 발급 및 결제 수단 등록, Lab 03·17 완료 |
| 난이도 | ★★★★☆ |

!!! warning "유료 API — 비용 주의"
    OpenAI 는 호출 1회당 토큰 수만큼 비용이 발생합니다. 이번 실습은 `gpt-4o-mini` 같은 저비용 모델로 진행하며, 각 실행은 통상 $0.001 미만. 그래도 누적되면 요금이 쌓이므로 **무한 루프·대량 아이템 호출을 피하고**, 사용 후 OpenAI 대시보드에서 사용량(usage) 을 확인하세요.

!!! note "OpenAI credential 등록 (사전)"
    Credentials → Create Credential → `OpenAI API` 선택 → https://platform.openai.com/api-keys 에서 발급한 `sk-...` 키 붙여넣기 → Save. 이름은 `OpenAI - Personal` 권장.

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
    OpenAI 노드의 `Message a Model` operation 으로 **가장 단순한 프롬프트** 를 보내고 응답을 받아보세요. User Message 로 `"n8n 이 무엇인지 한 문장으로 설명해줘."` 를 보내고, Output 에서 모델이 반환한 텍스트를 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud, OpenAI credential
- **워크플로우 이름:** `Lab 21 - OpenAI 기초`
- **OpenAI 노드 설정:**

| 항목 | 값 |
|------|-----|
| Credential | `OpenAI - Personal` |
| Resource | `Chat` (또는 Text) |
| Operation | `Message a Model` |
| Model | `gpt-4o-mini` |
| Messages → Role | `User` |
| Messages → Content | `n8n 이 무엇인지 한 문장으로 설명해줘.` |

**구성 단계:**

1. Manual Trigger → OpenAI 노드 연결
2. 위 표대로 설정
3. **[Execute Workflow]** 실행
4. Output 에 응답 텍스트가 들어오는지 확인

??? note "예시"

    > **OpenAI Output JSON (Simplify ON 기본 가정)**
    >
    > ```json
    > [
    >   {
    >     "message": {
    >       "role": "assistant",
    >       "content": "n8n 은 노드를 연결해 코드 없이 업무를 자동화할 수 있는 오픈소스 워크플로우 자동화 도구입니다."
    >     }
    >   }
    > ]
    > ```
    >
    > (혹은 모델의 응답 구조가 약간 다르게 `content` 가 최상위 필드로 나올 수 있음. 실제 Output 을 기준으로 참조 경로를 결정.)
    >
    > **확인 포인트:**
    >
    > - 응답 본문은 보통 `message.content` 또는 `content` 필드에 있음
    > - 같은 프롬프트를 여러 번 실행해도 응답이 조금씩 달라짐 (생성 모델 특성)
    > - 노드 Output 하단에 **토큰 사용량** (`usage.total_tokens`) 이 표시될 수 있음 — 비용 추적에 유용

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
    **System Message** 를 3가지로 바꿔가며 같은 User Message 에 대한 응답 차이를 비교 분석하세요. 추가로 **Temperature** 옵션을 `0` / `0.7` / `1.2` 로 바꿔가며 같은 프롬프트의 응답 일관성·창의성 변화를 관찰하세요.

**활용 도구 & 데이터**

- **고정 User Message:** `"워크플로우 자동화의 장점 3가지를 bullet 으로 알려줘."`

- **System Message 3종:**

| # | System Message |
|---|----------------|
| A | `당신은 간결한 기술 문서 작성자입니다. 설명을 최대한 짧고 명확하게 합니다.` |
| B | `당신은 10년차 자동화 컨설턴트입니다. 실무 ROI 관점으로 답변합니다.` |
| C | `당신은 유머러스한 블로거입니다. 가볍고 친근한 톤으로 답변합니다.` |

- **Temperature 3단계:**

| 단계 | 값 | 기대 |
|------|-----|------|
| 낮음 | `0.0` | 거의 같은 답변 반복 |
| 중간 | `0.7` (기본) | 적당한 다양성 |
| 높음 | `1.2` | 창의적이지만 불안정 |

**변경 방법:**

1. Messages 에 Role = `System` 메시지 하나 추가, User Message 는 동일 유지
2. System 내용을 A → B → C 로 바꿔가며 각 3회씩 실행
3. **Options > Temperature** 를 0 / 0.7 / 1.2 로 바꿔 각 2~3회 실행
4. 응답을 비교 분석

**(선택) 비교 프롬프트 (메타 프롬프트):**

```
아래 세 가지 System Message 에 대한 응답을 받았어.
각 응답의 "톤 / 구조 / 전문성 / 활용 가능성" 네 기준으로 비교 분석해줘.

[A 응답] ...
[B 응답] ...
[C 응답] ...
```

**예시**

> **System Message 별 응답 비교 (발췌)**
>
> | System | 응답 스타일 (예시 bullet 1번째) | 길이·톤 |
> |--------|------------------------------|---------|
> | A. 간결한 기술 문서 | "반복 업무 제거로 작업 시간 단축" | 짧고 건조 |
> | B. 10년차 컨설턴트 | "비용 회수 기간(Payback) 이 평균 6개월 이내로 빠름" | 전문 용어, 근거 중심 |
> | C. 유머러스 블로거 | "커피 마시는 동안 내 대신 일을 해주는 **자동화 요정** ☕" | 친근, 비유 |
>
> **Temperature 변화 관찰**
>
> | Temperature | 3회 실행 답변 유사성 | 표현 다양성 |
> |-------------|-------------------|------------|
> | 0.0 | 거의 동일 | 단조로움 |
> | 0.7 | 일부 유사, 일부 상이 | 자연스러운 변주 |
> | 1.2 | 매번 상이 | 창의적이지만 간혹 부정확/이상함 |
>
> **분석 결과:**
>
> - **System Message** 는 응답의 **성격** 을 결정 — 톤·전문성·구조를 바꾸는 가장 효과적인 레버
> - **User Message** 는 응답의 **내용** — 질문 자체
> - **Temperature** 는 응답의 **다양성** — 결정적 출력이 필요한 파이프라인(예: JSON 파싱) 은 낮게, 창작용은 높게
> - 두 레버를 조합하면 같은 모델로도 **전혀 다른 서비스** 같은 출력이 가능

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
    **JSON 모드** 를 사용해 응답을 **파싱 가능한 구조화 출력** 으로 받으세요. 사용자가 보낸 자유 텍스트 리뷰에서 `summary`, `sentiment`, `score` 세 필드를 추출하는 "리뷰 분석기" 를 만들고, 뒤에 Set 노드로 OpenAI 응답을 파싱해 활용 가능한 형태로 변환하세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우
- **구조:**

```
Manual Trigger
  → Set (review: 자유 텍스트 한 줄)
  → OpenAI (JSON 모드)
  → Set (OpenAI 응답에서 각 필드 추출)
```

- **Set (입력 리뷰):**

| Field | Value |
|-------|-------|
| review | `배송이 빨라서 좋았는데 포장은 살짝 아쉬웠어요. 다시 살 것 같아요.` |

- **OpenAI 노드 설정:**

| 항목 | 값 |
|------|-----|
| Model | `gpt-4o-mini` |
| System Message | (아래) |
| User Message | `={{ $json.review }}` |
| Options > Response Format | `JSON Object` |
| Options > Temperature | `0` |

**System Message:**

```
당신은 리뷰 분석기입니다. 사용자가 보낸 리뷰 텍스트를 분석하여
반드시 아래 JSON 스키마로 응답합니다. 추가 설명이나 문장 없이 JSON 만 반환하세요.

{
  "summary": "리뷰를 한 문장으로 요약",
  "sentiment": "positive | neutral | negative 중 하나",
  "score": 1~5 사이의 정수 (5가 가장 긍정적)
}
```

- **후속 Set 노드 (파싱):**

| Field | Type | Expression |
|-------|------|-----------|
| summary | String | `={{ JSON.parse($json.message.content).summary }}` |
| sentiment | String | `={{ JSON.parse($json.message.content).sentiment }}` |
| score | Number | `={{ JSON.parse($json.message.content).score }}` |

!!! tip "응답 필드 경로는 실제 Output 기준으로"
    n8n 의 OpenAI 노드 Output 구조가 버전에 따라 다를 수 있음. 실제 Output 을 보고 `$json.message.content` 가 아니면 경로를 맞춰 조정하세요 (예: `$json.content`, `$json.choices[0].message.content` 등).

**구성 순서:**

1. Set(review) → OpenAI(JSON 모드, Temperature 0) → Set(파싱) 연결
2. System Message 를 그대로 입력 (JSON 스키마 포함)
3. 실행 → OpenAI 응답이 올바른 JSON 문자열인지 확인
4. 후속 Set 노드가 summary/sentiment/score 로 분리해주는지 확인

**예시**

> **완성 워크플로우: "리뷰 자동 분석기"**
>
> **OpenAI Output (message.content)**
>
> ```json
> {
>   "summary": "배송은 만족스럽지만 포장 품질은 아쉬움. 재구매 의향 있음.",
>   "sentiment": "positive",
>   "score": 4
> }
> ```
>
> **후속 Set 노드 최종 Output**
>
> ```json
> [
>   {
>     "summary": "배송은 만족스럽지만 포장 품질은 아쉬움. 재구매 의향 있음.",
>     "sentiment": "positive",
>     "score": 4
>   }
> ]
> ```
>
> **구현 포인트:**
>
> - **JSON 모드 + Temperature 0** = 구조화된, 결정적인 출력 → 뒤 노드가 안전하게 파싱
> - System Message 안에 **JSON 스키마를 예시로** 넣으면 모델이 형식 준수 확률이 크게 올라감
> - 이 패턴이 실무 AI 자동화의 핵심: "자유 텍스트 → 구조화 JSON → 시트/DB/알림"
> - Lab 18(Sheets), Lab 20(Slack) 과 조합하면 "리뷰 자동 분류 → 구글시트 기록 → 부정 리뷰는 Slack 알림" 완성형 워크플로우 구축 가능

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

> - **n8n 의 AI Agent 노드와 OpenAI 노드는 어떻게 다를까?**
>     - AI Agent 는 Tool 호출·Memory·Chain 을 내장한 상위 추상화. 동일 프롬프트를 두 방식으로 처리해 복잡도·비용·가시성을 비교해보세요.
> - **`max_tokens` 와 `top_p` 옵션은 어떤 상황에 의미 있을까?**
>     - max_tokens 는 비용·길이 제한, top_p 는 Temperature 의 대안. 한쪽씩 극단적으로 바꾸면 어떤 변화가 생기는지 관찰해보세요.
> - **Few-shot 프롬프트 기법을 System/User Message 에 녹이려면?**
>     - System 에 "예시 입력 → 예시 출력" 쌍을 3개 넣고 User 로 실제 케이스 보내기. 모델의 포맷 준수율이 얼마나 오르는지 확인해보세요.
> - **실패·이상 응답을 자동 감지하려면?**
>     - JSON 파싱 에러를 try/catch 로 잡고 재시도, 또는 IF 노드로 score 가 없는 케이스 분기. Lab 15(Error) 와 결합해 robust 한 AI 파이프라인을 설계해보세요.
> - **비용 관리를 n8n 안에서 하려면?**
>     - Output 의 `usage.total_tokens` 를 Set 노드로 추출해 누적 기록(Sheets 에 Append) → 월별 대시보드. 과다 사용 시 Slack 알림도 추가해보세요.
