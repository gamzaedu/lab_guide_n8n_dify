---
title: "Lab 15. Error Trigger & Try/Catch 패턴"
description: "Error Trigger 워크플로우를 만들어 메인 워크플로우 실패 시 자동 알림을 보내고, 노드별 On Error 옵션으로 try/catch 동작을 구성합니다."
tags:
  - n8n
  - 심화실습
  - Error
  - 모니터링
---

# Lab 15. Error Trigger & Try/Catch 패턴

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Error Trigger 로 메인 워크플로우 실패를 감지하고, 노드별 `On Error` 옵션으로 try/catch 흐름을 설계할 수 있다 |
| 핵심 노드 | Manual Trigger, Code, HTTP Request, Error Trigger |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 04·12 완료 (HTTP / Code) |
| 난이도 | ★★★★☆ |

!!! note "n8n 에서 에러를 다루는 3가지 계층"
    1. **Node-level**: 각 노드의 `Settings > On Error` 옵션 (Stop / Continue / Continue (using error output))
    2. **Workflow-level**: 실패 시 별도 Error 전용 워크플로우 실행 (이번 실습의 주제)
    3. **Global-level**: 모든 워크플로우에 공통으로 연결되는 Error Workflow (Settings > Error Workflow)

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
    일부러 에러를 발생시키는 워크플로우를 만드세요. Code 노드에서 `throw new Error("Intentional failure")` 를 실행하여 실행이 실패 상태로 기록되는지 Executions 탭에서 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud
- **워크플로우 이름:** `Lab 15 - 실패 유발 (main)`
- **구성:**

```
Manual Trigger → Code (throw new Error)
```

- **Code 노드 내용:**

```javascript
throw new Error("Intentional failure for Lab 15");
```

**구성 단계:**

1. Manual Trigger → Code 노드 연결
2. 위 코드 입력
3. **[Execute Workflow]** 실행 → 캔버스의 Code 노드가 **빨간색 에러 아이콘** 으로 표시
4. 좌측 **Executions** 탭에서 이번 실행이 `Error` 상태로 기록된 것 확인

??? note "예시"

    > **실패 실행 로그 (Executions 탭)**
    >
    > | 항목 | 값 |
    > |------|-----|
    > | Started | `2026-04-18 14:52:30` |
    > | Status | **Error** (빨간 원형 아이콘) |
    > | Error Node | `Code` |
    > | Error Message | `Intentional failure for Lab 15` |
    >
    > **확인 포인트:**
    >
    > - 에러 발생 노드는 캔버스에 빨간 테두리로 표시됨
    > - Executions 탭을 열어 실행을 클릭하면 **에러 스택 트레이스** 전체 확인 가능
    > - 이 실패가 **어디에도 알리지 않고** 조용히 기록만 된다 — 이 상태를 바꾸는 것이 다음 단계의 목표

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
    Code 노드의 **`Settings > On Error`** 옵션을 `Continue` / `Continue (using error output)` 로 각각 바꿔가며 동작 차이를 관찰하세요. 특히 `Continue (using error output)` 모드에서 생기는 **에러 전용 Output 포트** 에 Set 노드를 붙여 에러 정보를 받아보세요.

**활용 도구 & 데이터**

- **도구:** Step 1의 워크플로우에서 Code 노드 Settings 수정
- **On Error 옵션 3종 실험:**

| 옵션 | 동작 | 시각적 변화 |
|------|------|-----------|
| `Stop Workflow (default)` | 에러 즉시 실행 전체 실패 | 노드 빨강, 뒤 노드 실행 안됨 |
| `Continue` | 에러 무시, 빈 Output 으로 다음 노드 실행 | 노드 주황, 뒤 노드 실행됨 |
| `Continue (using error output)` | 에러를 **별도 포트(빨강)** 로 출력, 정상 Output 과 분리 | 노드 옆에 에러 포트 추가 |

- **Continue (using error output) 뒤에 연결할 Set 노드:**

| Field | Expression |
|-------|-----------|
| failedNode | `={{ $node["Code"].error.node }}` |
| errorMessage | `={{ $node["Code"].error.message }}` |
| timestamp | `={{ $now }}` |

**변경 방법:**

1. Code 노드 선택 → **Settings** 탭 이동 → **On Error** 드롭다운
2. 각 옵션으로 바꿔가며 실행, 캔버스 반응과 Output 비교
3. `Continue (using error output)` 모드일 때 노드 옆에 생기는 빨간 포트에 Set 노드 연결
4. 에러 정보를 Set 노드 Output 에서 확인

**(선택) 분석 프롬프트:**

```
n8n 노드의 On Error 3가지 옵션(Stop / Continue / Continue with error output)을
"뒤 노드 실행 여부 / 에러 정보 접근 / 사용 시나리오" 세 기준으로 표로 정리해줘.
```

**예시**

> **On Error 비교표**
>
> | 옵션 | 뒤 노드 실행 | 에러 정보 | 사용 시나리오 |
> |------|-----------|---------|-------------|
> | Stop Workflow | ❌ | Executions 탭에서만 | 에러를 반드시 막아야 하는 critical path |
> | Continue | ✅ (빈 데이터) | 없음 | "있으면 좋고 없어도 OK" 같은 비필수 호출 |
> | Continue (using error output) | ✅ (에러 포트로) | **캔버스에서 직접 처리 가능** | try/catch 패턴, 에러 로깅·알림 |
>
> **`Continue (using error output)` Set 노드 Output**
>
> ```json
> [
>   {
>     "failedNode": "Code",
>     "errorMessage": "Intentional failure for Lab 15",
>     "timestamp": "2026-04-18T14:55:02.000+09:00"
>   }
> ]
> ```
>
> **분석 결과:**
>
> - **try/catch 의 catch 에 해당하는 패턴** 이 `Continue (using error output)` — 가장 유연
> - 한 노드에만 적용되므로 **"특정 실패만 허용"** 하고 싶을 때 적합
> - 전체 워크플로우 단위의 실패 감지는 별도 **Error Trigger** 워크플로우 필요 (Step 3)

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
    별도의 **Error 전용 워크플로우** 를 만들어 Step 1 의 메인 워크플로우에 연결하세요. 메인이 실패하면 Error Trigger 가 자동으로 시작되어 에러 정보를 Set 노드로 기록하도록 구성하세요.

**활용 도구 & 데이터**

- **도구:** 두 개의 워크플로우
- **1) Error 전용 워크플로우 (새로 생성, 이름: `Lab 15 - Error Handler`):**

```
Error Trigger  →  Set (에러 정보 기록)
```

- **Set 노드 필드 (Error Trigger 가 제공하는 변수 활용):**

| Field | Expression |
|-------|-----------|
| workflowName | `={{ $json.workflow.name }}` |
| executionUrl | `={{ $json.execution.url }}` |
| errorMessage | `={{ $json.execution.error.message }}` |
| errorNode | `={{ $json.execution.lastNodeExecuted }}` |
| timestamp | `={{ $now }}` |

- **2) 메인 워크플로우 (Step 1 의 `Lab 15 - 실패 유발`) Settings 수정:**

| 항목 | 값 |
|------|-----|
| Settings > Error Workflow | `Lab 15 - Error Handler` 선택 |

**구성 순서:**

1. 새 워크플로우 생성 → 이름 `Lab 15 - Error Handler`
2. **Error Trigger** 노드 추가 (검색: `Error`)
3. 뒤에 Set 노드 연결, 위 표대로 필드 입력
4. Error Handler 워크플로우 **Active 로 토글**
5. 메인 워크플로우로 돌아가 **Settings** 아이콘 클릭 → **Error Workflow** 드롭다운에서 `Lab 15 - Error Handler` 선택 → 저장
6. 메인 워크플로우를 **Active** 로 토글 (Error Workflow 연결은 Active 실행에서만 동작)
7. 메인 워크플로우 실행 → 실패 발생 → Error Handler 워크플로우의 Executions 탭 확인

**예시**

> **완성 워크플로우: "메인 실패 → 자동 에러 로깅"**
>
> **Error Handler 의 Set 노드 Output (메인 실패 후 자동 기록됨)**
>
> ```json
> [
>   {
>     "workflowName": "Lab 15 - 실패 유발 (main)",
>     "executionUrl": "https://<your-n8n>/workflow/XXX/executions/YYY",
>     "errorMessage": "Intentional failure for Lab 15",
>     "errorNode": "Code",
>     "timestamp": "2026-04-18T14:58:17.432+09:00"
>   }
> ]
> ```
>
> **구현 포인트:**
>
> - 메인과 Error Handler 가 **완전히 분리된 워크플로우** 로 관리됨 → 여러 메인이 같은 Handler 공유 가능
> - `$json.execution.url` 은 **Executions 탭 직링크** — Slack 알림에 그대로 넣으면 "한 번의 클릭으로 에러 조사" 가능
> - 실무에선 이 Set 노드 뒤에 Slack(Lab 20) / Email / PagerDuty 등을 연결해 온콜 알림 구성
> - **주의:** Error Workflow 는 **Active 워크플로우** 의 실패만 감지. 수동 Execute 실행은 트리거되지 않음.

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

> - **Retry on Fail 옵션은 언제 유효하고 언제 무의미할까?**
>     - 노드 Settings 의 Retry On Fail 옵션은 일시적 네트워크 오류에 유용, 코드 버그에는 무의미. 시도 횟수·간격 설정을 바꿔가며 동작 차이를 관찰해보세요.
> - **전역 Error Workflow 를 두면 모든 워크플로우가 자동 연결될까?**
>     - Settings > Default Error Workflow 설정으로 전역 기본값 지정 가능. 프로젝트 규모가 커질 때 유지보수 이점을 고민해보세요.
> - **에러 발생 후 "원래 작업을 이어서" 하려면?**
>     - Error Handler 에서 Execute Workflow 로 메인을 **재실행** 하는 패턴이 가능하지만 무한루프 위험. 방지책을 설계해보세요.
> - **IF 노드로 에러 메시지를 분류해 다른 경로로 보내려면?**
>     - Error Handler 안에 IF 노드 → 메시지에 "timeout" 포함되면 A 경로, "auth" 포함되면 B 경로. 실무에서 자주 쓰는 패턴이니 직접 구성해보세요.
> - **Continue (using error output) 과 Error Workflow 는 중복 실행될까?**
>     - 노드 단위에서 에러가 처리되면(`catch`) 워크플로우 전체는 성공으로 간주되어 Error Workflow 가 트리거되지 않는다. 실험으로 확인해보세요.
