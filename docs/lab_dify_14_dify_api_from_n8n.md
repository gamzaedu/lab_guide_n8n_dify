---
title: "Lab 14. n8n에서 Dify API 호출하기"
description: "Dify 앱의 API Endpoint와 API Key를 이용해 n8n HTTP Request 노드에서 Dify 앱을 프로그래밍적으로 호출하고 결과를 후속 노드로 연결합니다."
tags:
  - dify
  - 통합실습
  - API
---

# Lab 14. n8n에서 Dify API 호출하기

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Dify 앱의 API Endpoint를 확인하고, n8n HTTP Request 노드로 Dify 앱을 호출해 응답을 후속 워크플로우에서 사용할 수 있다 |
| 핵심 기능 | Dify API (Chat/Completion/Workflow), API Key, n8n HTTP Request, JSON 파싱 |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 07 또는 Lab 05에서 Publish한 Dify 앱 1개, n8n Cloud 로그인, n8n Lab 04 (HTTP Request) 수강 경험 권장 |
| 난이도 | ★★★★☆ |

!!! warning "API Key 보관"
    Dify API Key는 앱별로 발급되며, 유출 시 토큰 사용량이 과금될 수 있습니다. n8n의 **Credentials** 기능으로 저장해 노드에서 평문 노출을 피하세요.

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
    Dify **Workflow 앱(Lab 07 문서 요약기)** 의 API Key를 발급받고, **n8n HTTP Request 노드**로 호출해 문서를 전달 → 요약 결과를 받아오세요. 응답 JSON에서 `text` 필드만 추출해 Edit Fields로 정리합니다.

**활용 도구 & 데이터**

- **도구:** Dify Cloud (호출 대상 앱) + n8n Cloud
- **사용할 Dify 앱:** `Lab 07 - 문서 요약기` (이미 Publish)
- **Dify API 기본 정보:**

| 항목 | 값 |
|------|----|
| Base URL | `https://api.dify.ai/v1` |
| 인증 | `Authorization: Bearer {API_KEY}` |
| Workflow 앱 호출 | `POST /workflows/run` |
| 요청 Body | `{"inputs": {"document": "..."}, "response_mode": "blocking", "user": "n8n-lab14"}` |

**Dify에서 API Key 발급:**

1. Dify `Lab 07 - 문서 요약기` 앱 → 좌측 메뉴 **API Access** 또는 앱 상단 **API Reference**
2. **Create Secret Key** → Key 복사 (화면에서만 보임, 다시 안 보이므로 안전한 곳에 저장)
3. API Reference의 샘플 cURL 확인

**n8n 워크플로우 구성:**

1. n8n → **[+ New Workflow]** → 이름 `Lab 14 - Dify 요약 호출`
2. **Manual Trigger** 추가
3. **Edit Fields (Set)** 노드 추가 → `document` (String) 값에 긴 텍스트 입력
4. **HTTP Request** 노드 추가:

| 설정 | 값 |
|------|-----|
| Method | `POST` |
| URL | `https://api.dify.ai/v1/workflows/run` |
| Authentication | `Generic Credential Type` → `Header Auth` → Name: `Authorization` / Value: `Bearer {API_KEY}` |
| Send Body | `On`, Body Content Type: `JSON` |
| JSON Body | 아래 참고 |

JSON Body:

```json
{
  "inputs": { "document": "{{$json.document}}" },
  "response_mode": "blocking",
  "user": "n8n-lab14"
}
```

5. **Edit Fields** 노드 (후처리) → HTTP 응답에서 `summary` 필드 추출:
    - `summary`: `={{$json.data.outputs.text}}`

??? note "예시"

    > **n8n 실행 결과**
    >
    > **HTTP Request Response (원본 일부):**
    >
    > ```json
    > {
    >   "task_id": "8a71...",
    >   "workflow_run_id": "b2c0...",
    >   "data": {
    >     "id": "...",
    >     "workflow_id": "...",
    >     "status": "succeeded",
    >     "outputs": {
    >       "text": "- 공무원 AI 교육의 목표는 도구 사용법이 아닌 업무 프로세스 재설계이다.\n- 반복 문서 작성·민원 분류·RAG QA가 핵심 적용 영역이다.\n- 보안과 최종 책임은 공무원에게 남는다는 원칙이 중요하다."
    >     },
    >     "elapsed_time": 3.24,
    >     "total_tokens": 520
    >   }
    > }
    > ```
    >
    > **Edit Fields 노드 출력:**
    >
    > ```json
    > [
    >   {
    >     "summary": "- 공무원 AI 교육의 목표는 도구 사용법이 아닌 업무 프로세스 재설계이다.\n- 반복 문서 작성·민원 분류·RAG QA가 핵심 적용 영역이다.\n- 보안과 최종 책임은 공무원에게 남는다는 원칙이 중요하다."
    >   }
    > ]
    > ```
    >
    > **확인 포인트:**
    >
    > - HTTP Request 결과에 `data.outputs.text` 가 담겼는가
    > - `status: "succeeded"` 인가
    > - Edit Fields에서 요약 텍스트만 깔끔히 추출됐는가

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
    **(A) `response_mode: "blocking"` 동기 호출** 과 **(B) `response_mode: "streaming"` 스트리밍 호출** 두 방식, 그리고 **(C) Chatbot API(`/chat-messages`)** 방식을 비교하세요. 응답 시간·응답 구조·구현 난이도 관점에서 정리합니다.

**활용 도구 & 데이터**

- **도구:** n8n + Dify API
- **비교 설계:**

| 버전 | 엔드포인트 | response_mode |
|------|-----------|---------------|
| A | `/workflows/run` | blocking |
| B | `/workflows/run` | streaming |
| C | `/chat-messages` | blocking (Chatbot 앱 기준) |

**측정 항목:**

| 항목 | 측정 |
|------|------|
| 응답 시간 | n8n의 Execution 타임스탬프 |
| 응답 크기 | body 바이트 수 |
| 파싱 복잡도 | n8n에서 추가 가공이 얼마나 필요한가 |

**비교 방법:**

1. A: Step 1 그대로
2. B: 같은 HTTP Request에서 `response_mode` 만 streaming으로 → n8n은 SSE를 직접 다루기 어려우므로 보통 사용하지 않음(결과 관찰 목적)
3. C: 같은 문서를 Chatbot API로 전달하도록 별도 Chatbot 앱에 대해 호출

**예시**

> **비교 분석표**
>
> | 항목 | A blocking | B streaming | C chat-messages |
> |------|-----------|-------------|-----------------|
> | 응답 시간(체감) | 3~5초 | 첫 청크 1초, 전체 3~5초 | 3~5초 |
> | 응답 크기 | 1 JSON 덩어리 | 다수 SSE 이벤트 | 1 JSON 덩어리 |
> | n8n 파싱 난이도 | 쉬움 (1 JSON) | 어려움 (SSE, n8n 표준 노드 미지원) | 쉬움 |
> | 적합 용도 | 배치·자동화 | 사용자 UI 실시간 표시 | 대화형 챗봇 호출 |
>
> **분석 결과:**
>
> - n8n처럼 **서버 간 자동화에서는 blocking이 표준**
> - streaming은 UI가 있는 프런트엔드에서 이점 — n8n에서는 거의 사용 안 함
> - Chatbot API는 `conversation_id` 를 주고받으며 세션을 유지할 수 있어, **대화 컨텍스트가 필요한 업무**(예: 연속 접수)에 유리
> - Workflow API는 **단발 작업 자동화**, Chat API는 **대화 유지 자동화** — 업무 성격에 맞춰 선택

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
    **n8n → Dify → 외부 서비스(Gmail 또는 Slack)** 체인 워크플로우를 완성하세요. n8n Webhook / Trigger로 시작 → HTTP Request로 Dify 호출 → 응답을 가공해 Gmail/Slack으로 발송까지 자동 흐름을 구현합니다.

**활용 도구 & 데이터**

- **도구:** n8n + Dify + (Gmail 또는 Slack)
- **완성 구조:**

```
[Trigger: Webhook 또는 Schedule]
    ↓
[Edit Fields: 입력 데이터 준비]
    ↓
[HTTP Request: Dify 호출 → 요약/분석]
    ↓
[Edit Fields: 응답에서 필요한 필드 추출]
    ↓
[Gmail 또는 Slack: 결과 전송]
```

- **체크리스트:**

| 항목 | 기준 |
|------|------|
| Trigger | Webhook 또는 주기 스케줄 |
| Dify API | Credentials에 Key 등록해 Header Auth |
| 에러 처리 | Dify 응답 실패 시 분기 (IF 노드) |
| 전송 채널 | Gmail/Slack 중 택1 |
| 테스트 | 실제 1건 전송 성공 확인 |

**완성 순서:**

1. 시나리오 선정 (예: 매일 오전 9시 정책 관련 RSS를 크롤링 → Dify로 3줄 요약 → 팀 Slack에 전송)
2. n8n Credentials에 Dify API Key 등록 (Header Auth)
3. Trigger 노드 구성
4. HTTP Request 노드: Dify Workflow/Chat API 호출
5. IF 노드로 `status == "succeeded"` 체크
6. Gmail/Slack 노드에 최종 메시지 포맷 작성
7. 테스트 실행 → 수신 채널에서 실제 도착 확인

**예시**

> **완성 워크플로우: "매일 오전 9시 정책 요약 → 팀 Slack"**
>
> **노드 흐름:**
>
> ```
> Schedule Trigger (매일 09:00)
>   → HTTP Request (RSS 또는 공공데이터 API로 기사 리스트)
>   → Code (기사 본문 3건 추려 문자열 결합)
>   → HTTP Request (Dify Workflow 요약 호출)
>   → IF (data.status === 'succeeded')
>     ✅ → Slack (아래 포맷으로 전송)
>     ❌ → Slack (#ops 채널에 에러 알림)
> ```
>
> **Slack 메시지 포맷:**
>
> ```
> 📰 오늘의 정책 요약 (2026-04-19)
>
> - 요약 1: ...
> - 요약 2: ...
> - 요약 3: ...
>
> 🔗 원문 링크: https://...
> 🤖 By Dify + n8n
> ```
>
> **확인 포인트:** 매일 정해진 시간에 Slack 채널에 메시지가 자동 도착하고, 실패 시 별도 채널로 알림이 가는 구조.

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

> - **Dify API를 n8n Sub-workflow로 추상화하면 어떤 이점이 있을까?**
>     - "Dify 호출" 전용 Sub-workflow를 만들어 여러 워크플로우가 재사용하게 하는 구조를 설계해보세요. n8n Lab 16(Sub-workflow)과 연결됩니다.
> - **Dify API 호출 실패율이 1% 정도 발생한다면 어떤 재시도 전략이 적절할까?**
>     - n8n의 Retry 기능과 IF 분기를 조합해 지수 백오프 재시도 로직을 만들어보세요.
> - **Dify Chat API의 `conversation_id` 를 n8n에서 어떻게 유지·갱신할까?**
>     - 첫 호출에서 받은 `conversation_id` 를 Google Sheets나 DB에 저장하고 이후 호출에 재사용하는 세션 유지 구조를 설계해보세요.
> - **Dify 앱이 업데이트될 때 n8n 쪽은 어떤 영향을 받을까?**
>     - Dify 앱의 입력 변수명을 바꾸면 n8n HTTP Body도 바뀌어야 합니다. 배포·테스트 프로세스를 어떻게 만들면 안전할지 정리해보세요.
