---
title: "Lab 20. Slack 알림 보내기"
description: "Slack 노드로 채널에 메시지를 보내고, 멘션·마크다운·블록 UI·첨부 포맷까지 단계별로 익힙니다."
tags:
  - n8n
  - 통합실습
  - Slack
  - 알림
---

# Lab 20. Slack 알림 보내기

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Slack 노드로 채널·DM 에 메시지를 보내고, 멘션·포맷·블록(block kit)·첨부 패턴으로 실무형 알림을 구성할 수 있다 |
| 핵심 노드 | Manual Trigger, Edit Fields (Set), Slack |
| 소요 시간 | 약 50분 |
| 사전 준비 | Slack Workspace 관리 권한 또는 App 등록 권한, Slack OAuth2 API credential 등록 |
| 난이도 | ★★★☆☆ |

!!! warning "Slack credential 등록 경로 (요약)"
    Slack 은 Google 과 달리 **Workspace 별 App 등록** 이 필요합니다:
    
    1. `https://api.slack.com/apps` → Create New App → From scratch
    2. OAuth & Permissions → **Bot Token Scopes** 에 `chat:write`, `channels:read`, `users:read` 최소 추가
    3. Install App to Workspace → Bot User OAuth Token (`xoxb-...`) 복사
    4. n8n Credentials → Create → **Slack API** (Access Token 방식) 선택 → Token 붙여넣기
    
    또는 Slack OAuth2 API 선택 시 브라우저 승인 플로우로 자동 진행.

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
    본인 Workspace 에 **테스트용 채널** (`#n8n-lab-test`) 을 만들고, Slack 봇을 그 채널에 초대한 뒤, n8n 에서 Slack 노드로 `"Hello from n8n Lab 20!"` 메시지를 채널에 전송하세요. Slack 에서 실제 메시지가 봇 계정으로 뜨는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud, Slack credential, 본인 Slack Workspace
- **워크플로우 이름:** `Lab 20 - Slack 기초`
- **사전 작업:**
    1. Slack 에서 `#n8n-lab-test` 채널 생성
    2. 채널 상단 이름 클릭 → **Integrations** → **Add apps** → 등록한 봇 추가
- **Slack 노드 설정:**

| 항목 | 값 |
|------|-----|
| Credential | `Slack - Personal` |
| Resource | `Message` |
| Operation | `Send` |
| Send Message To | `Channel` |
| Channel | `#n8n-lab-test` (드롭다운에서 선택) |
| Message Type | `Simple Text Message` |
| Text | `Hello from n8n Lab 20!` |

**구성 단계:**

1. Manual Trigger → Slack 노드 연결
2. 위 표대로 설정 (Channel 드롭다운이 자동 로드되지 않으면 봇을 채널에 초대했는지 재확인)
3. **[Execute Workflow]** 실행
4. Slack 으로 이동해 채널 메시지 확인

??? note "예시"

    > **Slack 채널에서 보이는 메시지**
    >
    > ```
    > [Bot 이름]  오후 2:45
    > Hello from n8n Lab 20!
    > ```
    >
    > **n8n Slack 노드 Output**
    >
    > ```json
    > [
    >   {
    >     "ok": true,
    >     "channel": "C012345ABCD",
    >     "ts": "1729241122.000100",
    >     "message": {
    >       "type": "message",
    >       "text": "Hello from n8n Lab 20!",
    >       "user": "U0BOTID",
    >       "bot_id": "B012ABC"
    >     }
    >   }
    > ]
    > ```
    >
    > **확인 포인트:**
    >
    > - `ok: true` 가 성공 신호
    > - `ts` (timestamp) 는 메시지 고유 ID — 이후 업데이트·삭제·스레드 답글의 키
    > - 봇이 채널에 **초대되지 않았다면** `not_in_channel` 에러 발생 → 채널 설정에서 봇 추가 필수

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
    **Slack 마크다운 포맷** (볼드·이탤릭·코드블록·링크·멘션) 을 활용한 메시지를 작성하세요. 메시지에 Expression (`{{ $now }}` 등) 을 섞어 동적 값이 포함된 알림을 보내고, 순수 텍스트 메시지(Step 1) 와의 시각적 차이를 비교하세요.

**활용 도구 & 데이터**

- **도구:** Step 1 노드의 Text 내용을 변경
- **Slack 마크다운 문법 (mrkdwn):**

| 효과 | 문법 | 예시 |
|------|------|------|
| **볼드** | `*굵게*` | `*중요*` |
| 이탤릭 | `_기울임_` | `_강조_` |
| ~~취소선~~ | `~취소~` | `~기존 값~` |
| 인라인 코드 | `` `code` `` | `` `200 OK` `` |
| 코드블록 | `` ```...``` `` | 여러 줄 코드 |
| 링크 | `<https://url|표시텍스트>` | `<https://n8n.io|n8n 공식>` |
| 사용자 멘션 | `<@USERID>` | `<@U12345>` |
| 채널 멘션 | `<#CHANNELID|channel-name>` | `<#C012345ABCD|n8n-lab-test>` |

- **변경할 Text:**

```
:rocket: *n8n Lab 20 — Markdown 테스트*

이 메시지는 `Lab 20` 워크플로우에서 발송되었습니다.

• 실행 시각: `{{ $now.toFormat("yyyy-MM-dd HH:mm:ss") }}`
• 워크플로우: *{{ $workflow.name }}*
• 상세: <https://n8n.io|n8n 공식 사이트>

>>> _참고:_ 이 메시지는 자동화 파이프라인이 _정상 작동한다는_ 신호입니다.
```

!!! tip "사용자 ID 찾기"
    Slack 에서 본인 프로필 열기 → More (⋮) → **Copy member ID** → `U0XXXXX` 형태. 이 ID 를 멘션에 쓰면 상대방에게 알림이 간다(표시 이름이 아닌 ID 사용).

**관찰 포인트:**

1. Slack 은 일반 마크다운과 약간 다름 — **`*bold*`** (`**bold**` 아님), **`_italic_`** (`*italic*` 아님)
2. `\n` 줄바꿈은 그대로 동작, Slack UI 에서 자동 wrap
3. `<@USER>`, `<#CHAN>` 은 멘션/채널 링크 — 알림과 깔끔한 링크를 동시에
4. 이모지는 `:rocket:` 같은 shortcode 로 입력

**(선택) 분석 프롬프트:**

```
Slack 메시지의 마크다운과 일반 마크다운(예: GitHub) 의 차이를
"볼드 / 이탤릭 / 코드블록 / 링크" 네 기준으로 표로 만들어줘.
```

**예시**

> **Slack 에서 렌더링된 메시지 (시각적 표현, 실제 채널 캡처 대체)**
>
> ```
> 🚀 n8n Lab 20 — Markdown 테스트
>
> 이 메시지는  Lab 20  워크플로우에서 발송되었습니다.
>
>   • 실행 시각:  2025-04-18 14:50:11
>   • 워크플로우: Lab 20 - Slack 기초
>   • 상세: n8n 공식 사이트 (↗)
>
>   ┃ 참고: 이 메시지는 자동화 파이프라인이 정상 작동한다는 신호입니다.
> ```
>
> **Step 1 (순수 텍스트) vs Step 2 (마크다운) 비교표**
>
> | 요소 | Step 1 | Step 2 |
> |------|--------|--------|
> | 가독성 | 단순, 단조 | 구조 명확, 시선 유도 |
> | 정보 밀도 | 낮음 | 높음 (bullet, 인용) |
> | 링크 | 긴 URL 그대로 | 깔끔한 표시 텍스트 |
> | 주의 환기 | 약함 | 이모지·볼드·`>>>` 인용으로 시각 강조 |
>
> **분석 결과:**
>
> - 마크다운 한 줄 추가로 알림의 **신호 대 잡음비** 가 크게 올라감
> - 동적 값(`{{ $now }}`) 을 섞으면 "내용이 살아있는" 알림이 됨
> - 과한 포맷(이모지 과다, 중첩 인용) 은 오히려 피로 — **1 메시지 1 주제** 원칙 권장

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
    **HTTP Request(Lab 04) + Slack** 을 조합하여 실전 알림을 만드세요. GitHub Zen API 를 호출해 랜덤 인용문을 받고, Slack 에 **Block Kit** 형식(헤더 + 본문 + 구분선)으로 포맷된 메시지를 전송하세요.

**활용 도구 & 데이터**

- **도구:** 새 워크플로우
- **구조:**

```
Manual Trigger
  → HTTP Request (GET https://api.github.com/zen)
  → Slack (Send Message, Message Type: "Blocks")
```

- **Slack 노드 설정 (Blocks):**

| 항목 | 값 |
|------|-----|
| Message Type | `Blocks` |
| Blocks | (아래 JSON 입력) |
| (Send Message To / Channel 은 Step 1 과 동일) |

**Blocks JSON (Expression 모드로 입력):**

```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": ":sparkles: 오늘의 한마디",
        "emoji": true
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "> *{{ $json.data }}*"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": "Powered by _n8n Lab 20_ · {{ $now.toFormat(\"yyyy-MM-dd HH:mm\") }}"
        }
      ]
    }
  ]
}
```

!!! tip "Block Kit 미리보기 툴"
    `https://app.slack.com/block-kit-builder` 에서 JSON 을 붙여 Slack 렌더링을 실시간 확인. 복잡한 블록 구성 시 필수 도구.

**구성 순서:**

1. HTTP Request 노드: `GET https://api.github.com/zen` (Lab 04 와 동일)
2. Slack 노드: Message Type = `Blocks`, Blocks Input 방식 = `Define Below` / JSON
3. 위 JSON 을 Expression 으로 입력 (노드의 Block 입력 칸에서 `{{ }}` 가 평가됨)
4. 실행 후 Slack 에서 헤더 · 인용구 · 구분선 · 컨텍스트 4 블록이 순서대로 보이는지 확인

**예시**

> **완성 워크플로우: "하루 한 줄 Slack 알림"**
>
> **Slack 에서 실제로 보이는 메시지 (Block Kit 렌더링)**
>
> ```
> ┌───────────────────────────────┐
> │ ✨ 오늘의 한마디              │  ← Header 블록 (큰 글씨)
> ├───────────────────────────────┤
> │ > Design for failure.         │  ← Section 블록 (인용 스타일)
> ├───────────────────────────────┤  ← Divider
> │ Powered by _n8n Lab 20_       │  ← Context 블록 (작은 회색 글씨)
> │ · 2025-04-18 14:55             │
> └───────────────────────────────┘
> ```
>
> **구현 포인트:**
>
> - Block Kit 은 **"단순 텍스트" vs "구조화된 카드"** 의 차이 — 복잡한 리포트·대시보드 알림에 필수
> - HTTP Request 의 Output `$json.data` 가 Section 본문에 자동 주입됨
> - Schedule Trigger(Lab 02) 로 바꾸면 **매일 아침 9시 자동 인용문 알림** 이 됨
> - 실무 응용: 서비스 상태 요약, 배포 알림, 매출 리포트, 에러 발생 알림 등

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

> - **스레드 답글은 어떻게 보낼까?**
>     - Slack 노드의 `Thread TS` 옵션에 앞서 받은 `ts` 를 넣으면 해당 메시지의 스레드 답글로 달린다. 연속 알림을 스레드로 묶는 패턴을 시도해보세요.
> - **Block Kit 의 `actions` 블록으로 버튼을 추가하면 어떻게 될까?**
>     - 버튼 클릭은 Slack 이 지정된 URL 로 Webhook 을 보내는 방식. n8n Webhook(Lab 05) 과 조합하면 Slack 버튼 → n8n 워크플로우 실행 플로우 가능.
> - **DM(direct message) 으로 보내려면?**
>     - Send Message To = `User` 로 바꾸고 User ID 지정. 채널이 아닌 1:1 알림을 언제 써야 할지 판단 기준을 만들어보세요.
> - **ephemeral message (받는 사람에게만 보이는 메시지) 는 어떻게 보낼까?**
>     - 공식 노드에서 직접 지원하지 않을 수 있어 HTTP Request + `chat.postEphemeral` API 로 구현. 필요 상황을 가정해보세요.
> - **Slack Trigger 와 결합해 "메시지 → n8n → 응답" 양방향 봇을 만들려면?**
>     - Slack Trigger 노드 + Event Subscription 설정. 에코 봇부터 시작해 간단한 Q&A 봇까지 진화 경로를 그려보세요.
