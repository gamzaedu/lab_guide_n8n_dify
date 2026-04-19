---
title: "Lab 11. Wait 노드로 지연·재개 제어"
description: "Wait 노드의 3가지 Resume 모드(Time Interval / At Specific Time / On Webhook Call)를 비교하며 워크플로우 일시 중지를 실습합니다."
tags:
  - n8n
  - 중급실습
  - Wait
  - 제어흐름
---

# Lab 11. Wait 노드로 지연·재개 제어

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Wait 노드의 Resume 모드 3종을 상황에 맞게 선택하고, 워크플로우 중간 대기·재개 패턴을 구성할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Wait, Webhook |
| 소요 시간 | 약 50분 |
| 사전 준비 | Lab 05 (Webhook) · Lab 10 (Batches) 완료 권장 |
| 난이도 | ★★★☆☆ |

!!! note "Wait 노드의 3가지 Resume 모드"
    - **After Time Interval**: N초/분/시간 대기 후 자동 재개 (가장 기본)
    - **At Specific Time**: 지정된 특정 일시까지 대기
    - **On Webhook Call**: 외부 Webhook 호출이 올 때까지 무기한 대기 (승인·확인 플로우)

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
    Manual Trigger → Set(시작 시각) → **Wait (5초)** → Set(종료 시각) 구조의 워크플로우를 만들어, 두 Set 노드의 `processedAt` 차이가 약 5초인지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 11 - Wait 기초`
- **Wait 노드 설정:**

| 항목 | 값 |
|------|-----|
| Resume | `After Time Interval` |
| Wait Amount | `5` |
| Wait Unit | `Seconds` |

- **시작/종료 Set 노드:**

| 위치 | Field | Expression |
|------|-------|-----------|
| 시작 | startedAt | `={{ $now.toFormat("HH:mm:ss") }}` |
| 종료 | endedAt | `={{ $now.toFormat("HH:mm:ss") }}` |

**구성 단계:**

1. Manual Trigger → Set(startedAt) → Wait(5s) → Set(endedAt) 연결
2. 두 번째 Set 노드의 `Include Other Input Fields` = ON
3. **[Execute Workflow]** 실행 → 진행 표시가 Wait 노드에서 5초 멈추는지 시각적으로 확인
4. Output 에서 `startedAt` 과 `endedAt` 의 초 차이를 확인

??? note "예시"

    > **최종 Output**
    >
    > ```json
    > [
    >   {
    >     "startedAt": "14:40:01",
    >     "endedAt":   "14:40:06"
    >   }
    > ]
    > ```
    >
    > **확인 포인트:** `endedAt` - `startedAt` = 약 5초. 캔버스에서도 Wait 노드가 실행 중일 때 회색 스피너가 표시되다가 5초 후 다음 노드로 흘러감.

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
    Wait 노드의 Resume 모드를 **At Specific Time** 으로 바꿔 "지금 시각 + 1분 이후" 로 설정하고 실행하세요. 그리고 Resume 모드를 **On Webhook Call** 로 다시 바꿔, 외부에서 브라우저로 Resume URL 을 호출했을 때 워크플로우가 이어지는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에서 Wait 노드 설정만 변경
- **모드 A — At Specific Time:**

| 항목 | 값 |
|------|-----|
| Resume | `At Specific Time` |
| Date and Time | 현재 시각 + 1분 (예: `2026-04-18T15:30:00`) |

- **모드 B — On Webhook Call:**

| 항목 | 값 |
|------|-----|
| Resume | `On Webhook Call` |
| HTTP Method | `GET` |
| (실행 후 노드에 표시되는 `Resume URL` 을 브라우저로 호출) |

**관찰 포인트:**

1. At Specific Time 은 "일회성 알람 시계" — 시각 계산이 현재보다 앞서면 즉시 실행됨
2. On Webhook Call 은 "무기한 대기" — 외부 이벤트가 올 때까지 실행이 멈춤 (승인 플로우에 유용)
3. 두 모드 모두 Wait 노드는 **Output 에 "이전 데이터" 그대로** 전달 (대기는 사이드이펙트 없음)

**(선택) 비교 프롬프트:**

```
Wait 노드의 Resume 모드 3종을
"재개 조건 / 무기한 가능 여부 / 대표 사용 예" 기준으로 표로 정리해줘.
```

**예시**

> **Resume 모드 비교표**
>
> | 모드 | 재개 조건 | 무기한 대기 | 대표 사용 예 |
> |------|---------|-----------|------------|
> | After Time Interval | N초/분/시간 경과 | 불가 (수 시간 상한 있음) | API 쿼터 회피, 일정 간격 폴링 |
> | At Specific Time | 지정된 일시 도달 | 미래라면 그 시점까지 | "오후 6시에 리포트 발송", 예약 메시지 |
> | On Webhook Call | 외부 Webhook 호출 도착 | 가능 (외부 이벤트까지) | 승인 플로우, 사용자 확인, 결제 완료 대기 |
>
> **분석 결과:**
>
> - **After Time Interval** 은 자동화에서 가장 자주 쓰임 — 외부 의존 없이 동작 보장
> - **At Specific Time** 은 "워크플로우 내부 스케줄러" — Schedule Trigger 가 "반복" 이라면 이쪽은 "단일 예약"
> - **On Webhook Call** 은 **휴먼-인-더-루프** 의 핵심 — Slack 승인 요청 후 버튼 클릭 Webhook 으로 이어가는 UX 가능
> - 실행이 Wait 에서 멈춘 동안에도 n8n 은 **Execution 을 유지** — Executions 탭에서 진행 중 상태로 표시

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
    **On Webhook Call 모드** 로 "승인 대기" 패턴을 구현하세요. Webhook Trigger 로 시작 → Set(요청 정보 기록) → Wait(On Webhook Call) → Set(승인 처리 완료) 구조로, Resume URL 을 브라우저에서 호출해야만 워크플로우가 완료되는 휴먼 승인 플로우를 만드세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우
- **구조:**

```
Webhook (시작 - "/request")
  → Set (approvalId, requestedAt)
  → Wait (Resume: On Webhook Call)     ← Resume URL 이 여기서 생성됨
  → Set (approvedAt, status: "approved")
```

- **Wait 노드 설정:**

| 항목 | 값 |
|------|-----|
| Resume | `On Webhook Call` |
| HTTP Method | `GET` |
| Response | `Immediately` |

**구성 순서:**

1. Webhook 노드(시작): Method=GET, Path=`request`, Respond=Immediately
2. Set 노드: `approvalId` = `={{ $now.toMillis() }}`, `requestedAt` = `={{ $now }}`
3. Wait 노드: Resume=On Webhook Call → 실행하면 **Resume URL** 이 노드 옵션에 표시됨
4. 마지막 Set 노드: `approvedAt`, `status: "approved"`, `Include Other Input Fields` = ON
5. **시작 Webhook** 을 **[Listen for test event]** 후 브라우저로 호출 → 워크플로우가 Wait 에서 멈춤
6. Wait 노드가 노출한 **Resume URL** 을 브라우저 새 탭에서 호출 → 마지막 Set 노드까지 흐름
7. 각 Set 노드 Output 에서 `requestedAt` ~ `approvedAt` 시간 차가 실제 대기 시간과 일치하는지 확인

**예시**

> **완성 워크플로우: "휴먼 승인 대기 플로우"**
>
> **시나리오:**
>
> 1. 외부 요청 → `/request` 호출 → 워크플로우 시작
> 2. `approvalId` 기록 → Wait 에서 멈춤
> 3. 관리자가 Resume URL 을 받아 확인 후 호출
> 4. 마지막 Set 에서 승인 완료 기록
>
> **최종 Output JSON**
>
> ```json
> [
>   {
>     "approvalId": "1729240201234",
>     "requestedAt": "2026-04-18T14:50:01.234+09:00",
>     "approvedAt":  "2026-04-18T14:52:47.091+09:00",
>     "status": "approved"
>   }
> ]
> ```
>
> **구현 포인트:**
>
> - Wait (On Webhook Call) 은 **무기한** 대기 — 사람의 의사결정을 워크플로우에 끼워 넣는 핵심 도구
> - Resume URL 은 실행마다 새로 발급 → 외부 시스템(Slack 메시지, 이메일 링크)에 넣어 전달하면 원격 승인 가능
> - 실무에서는 Lab 20(Slack) 과 결합하여 "Slack 메시지 버튼 클릭 = Resume URL 호출" 로 자연스러운 UX 완성

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

> - **Wait 노드의 최대 대기 시간은 얼마일까?**
>     - After Time Interval 모드에서 너무 긴 시간을 입력하면 어떻게 되는지, At Specific Time 으로 아예 먼 미래를 찍으면 어떻게 되는지 테스트해보세요. (cloud 환경의 상한 확인)
> - **Wait 중인 Execution 을 중간에 취소하려면?**
>     - Executions 탭에서 진행 중인 실행을 수동으로 멈출 수 있다. 실행 흐름이 어떻게 기록되는지 관찰해보세요.
> - **승인 플로우에 "거절" 경로를 추가하려면 어떻게 구성할까?**
>     - Wait 뒤에 두 개의 경로가 아닌, Webhook Resume URL 에 `?result=approved`/`?result=rejected` 쿼리를 붙여 IF 로 분기하는 패턴이 있다. 직접 구성해보세요.
> - **Wait 노드가 유지하는 데이터 스냅샷은 언제까지 유지될까?**
>     - 자체 호스팅 vs Cloud, 서버 재시작 등의 상황에서 보존 정책이 다를 수 있다. 긴 승인 플로우 설계 전 반드시 확인해야 하는 포인트를 정리해보세요.
