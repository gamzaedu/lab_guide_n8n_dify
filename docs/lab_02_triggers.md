---
title: "Lab 2. Trigger 노드 비교 — Manual / Schedule / Webhook"
description: "동일한 출력 로직을 세 가지 Trigger 노드로 각각 시작해보며, 실행 방식의 차이와 선택 기준을 익힙니다."
tags:
  - n8n
  - 기초실습
  - Trigger
---

# Lab 2. Trigger 노드 비교 — Manual / Schedule / Webhook

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Manual / Schedule / Webhook 세 가지 트리거를 각각 구성하고, 실행 방식의 차이를 설명할 수 있다 |
| 핵심 노드 | Manual Trigger, Schedule Trigger, Webhook, Edit Fields (Set) |
| 소요 시간 | 약 50분 |
| 사전 준비 | n8n cloud 로그인 완료, Lab 01 학습 완료 권장 |
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
    **Manual Trigger + Set 노드** 로 간단한 워크플로우를 만드세요. Set 노드에서 `message` 필드에 `"Hello, n8n"` 을, `source` 필드에 `"manual"` 을 출력하도록 구성하고 실행하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 02 - Trigger 비교`
- **Set 노드 필드:**

| Field Name | Type | Value |
|------------|------|-------|
| message | String | `Hello, n8n` |
| source | String | `manual` |

**구성 단계:**

1. 새 워크플로우 생성 → 이름을 `Lab 02 - Trigger 비교` 로 지정
2. **Manual Trigger** 추가
3. **Edit Fields (Set)** 노드 연결 후 위 필드 2개 입력
4. **[Execute Workflow]** 실행 → Set 노드 Output 에서 JSON 확인

??? note "예시"

    > **Output JSON**
    >
    > ```json
    > [
    >   {
    >     "message": "Hello, n8n",
    >     "source": "manual"
    >   }
    > ]
    > ```
    >
    > **확인 포인트:** Manual Trigger 는 반드시 **[Execute Workflow]** 버튼을 눌러야만 실행됩니다. 저절로 실행되지 않습니다.

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
    Step 1의 Manual Trigger 를 **Schedule Trigger** 로 교체하여 동일한 Set 출력을 내도록 재구성하세요. Trigger Interval 을 `Every 1 Minute` 으로 설정하고, 1~2분 동안 실제 자동 실행되는 것을 관찰한 뒤 Manual 과의 차이를 정리하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우를 복제(Duplicate)하거나, Trigger 노드만 교체
- **Schedule Trigger 설정:**

| 항목 | 값 |
|------|-----|
| Trigger Interval | `Minutes` |
| Minutes Between Triggers | `1` |

- **Set 노드 `source` Value 를 `schedule` 로 변경** (어떤 트리거에서 왔는지 구분하기 위해)

**변경 방법:**

1. Manual Trigger 를 우클릭 → **Delete**
2. `Schedule Trigger` 노드 검색 후 추가, 위 표대로 설정
3. Set 노드의 `source` 를 `schedule` 로 수정
4. 캔버스 우측 상단의 **[Inactive → Active]** 토글을 켬 (Schedule Trigger 는 워크플로우가 **Active** 상태여야 자동 실행)
5. 좌측 메뉴의 **[Executions]** 탭에서 1~2분 후 자동 실행 기록이 쌓이는지 확인
6. 확인 후 반드시 워크플로우를 다시 **Inactive** 로 토글 (크레딧 소모 방지)

**(선택) 비교 피드백 요청:**

```
Manual Trigger 와 Schedule Trigger 의 실행 방식 차이를,
"언제 실행되는가 / 무엇이 필요한가 / 어떤 상황에 적합한가" 3가지 기준으로 3줄로 정리해줘.
```

**예시**

> **비교 분석표 — Manual vs Schedule**
>
> | 기준 | Manual Trigger | Schedule Trigger |
> |------|---------------|------------------|
> | 실행 시점 | 사람이 **[Execute Workflow]** 누를 때만 | 지정된 주기마다 자동 |
> | 활성화 조건 | 워크플로우가 Inactive 여도 테스트 가능 | 반드시 **Active** 상태여야 자동 실행 |
> | 기록 위치 | 캔버스 우측 Output 패널 | **Executions** 탭에 누적 기록 |
> | 적합한 상황 | 개발·디버깅, 일회성 실행 | 정기 리포트, 주기적 데이터 수집, 알림 |
> | 주의사항 | 없음 | Active 로 켜 두면 **계속 크레딧 소모** — 실습 후 반드시 Inactive |
>
> **분석 결과:**
>
> - Manual 은 "내가 원할 때" 돌리는 용도, Schedule 은 "정해진 시각마다" 돌리는 용도
> - 같은 로직이라도 Trigger 선택만으로 **용도와 배포 방식이 완전히 달라진다**
> - Schedule Trigger 사용 시 Active 토글 관리가 필수이며, Interval 을 너무 짧게 잡으면(예: 매초) 요금·API 쿼터 문제 발생 가능

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
    Trigger 를 **Webhook** 으로 교체하여 동일한 Set 출력을 내도록 구성하고, 브라우저 주소창에 **Test URL** 을 붙여넣어 실제로 외부 호출로부터 워크플로우가 실행되는 것을 확인하세요. 마지막으로 Manual / Schedule / Webhook 3종을 한 줄씩 비교한 본인만의 요약표를 만들어 보세요.

**활용 도구 & 데이터**

- **도구:** Step 2 워크플로우의 Trigger 만 교체
- **Webhook 노드 설정:**

| 항목 | 값 |
|------|-----|
| HTTP Method | `GET` |
| Path | `hello` (자동 생성 UUID 그대로 두어도 됨) |
| Respond | `Immediately` |

- **Set 노드 `source` Value 를 `webhook` 으로 변경**

**구성 순서:**

1. Schedule Trigger 를 삭제하고 **Webhook** 노드 추가
2. Webhook 노드 상단의 **Test URL** 을 복사 (예: `https://<your-n8n>/webhook-test/hello`)
3. 캔버스에서 Webhook 노드를 클릭한 상태로 **[Listen for test event]** 버튼 클릭 (→ "Waiting for webhook call..." 상태)
4. 브라우저 새 탭에서 복사한 **Test URL** 접속
5. n8n 캔버스로 돌아가 워크플로우가 실행되고 Set 노드 Output 이 생성되었는지 확인
6. 비교표를 직접 작성하고 워크플로우 저장

**예시**

> **완성 워크플로우: "Trigger 3종 비교 실험"**
>
> 최종적으로 3번 실험을 마친 뒤, 본인이 정리한 비교표 예시:
>
> | 기준 | Manual Trigger | Schedule Trigger | Webhook Trigger |
> |------|---------------|------------------|-----------------|
> | 실행 방식 | 내가 UI 에서 직접 실행 | 정해진 주기마다 자동 | 외부에서 URL 로 호출 |
> | 대표 사용 예 | 개발·테스트, 수동 조작 | 매일 09시 리포트, 1시간마다 데이터 수집 | 폼 제출, 외부 서비스 이벤트, 챗봇 |
> | 실행에 필요한 것 | 사람 | 시간(주기) | HTTP 요청 |
> | 활성화 조건 | 특별한 설정 불필요 | Active 상태 필수 | Active 상태 필수 (Production URL 기준) |
> | 응답 여부 | 응답 개념 없음 | 응답 개념 없음 | 호출자에게 응답 반환 가능 |
>
> **Webhook 실행 직후 Set 노드 Output:**
>
> ```json
> [
>   {
>     "message": "Hello, n8n",
>     "source": "webhook"
>   }
> ]
> ```
>
> **확인 포인트:** Webhook 은 브라우저 주소창에 URL 을 입력한 순간 즉시 워크플로우가 실행됩니다. Executions 탭에서 해당 실행 기록이 남아 있는지 확인하세요.

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

> - **Webhook 의 Test URL 과 Production URL 은 어떤 차이가 있을까?**
>     - Test URL 은 **[Listen for test event]** 를 누른 상태에서만 1회 동작하고, Production URL 은 워크플로우가 **Active** 일 때 언제든 동작한다. 두 URL 을 각각 호출해 차이를 체감해보세요.
> - **Schedule Trigger 에서 Cron 표현식을 쓸 수 있을까?**
>     - Interval 을 `Custom (Cron)` 으로 바꾸면 `0 9 * * 1-5` (평일 매일 9시) 같은 표현식을 쓸 수 있다. 직접 작성해 Executions 기록을 비교해보세요.
> - **Chat Message Trigger / Form Trigger / Email Trigger 는 어떤 상황에 쓸까?**
>     - Manual/Schedule/Webhook 외의 특수 트리거들이 있다. 각각의 설정 화면을 열어 "어떤 입력을 기대하는지"를 읽어보고, 실무 시나리오를 1개씩 떠올려보세요.
> - **하나의 워크플로우에 Trigger 를 2개 이상 둘 수 있을까?**
>     - Manual + Webhook 을 동시에 둔 뒤 각각 실행해보세요. 동일한 로직을 여러 경로로 트리거하고 싶을 때 어떻게 설계하면 좋을지 고민해보세요.
