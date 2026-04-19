---
title: "Lab 17. Credentials 관리 기초"
description: "OAuth2 와 API Key 방식의 차이를 이해하고, Google OAuth2 credential 을 실제 등록하여 재사용 가능한 자격 증명 자산을 만듭니다."
tags:
  - n8n
  - 통합실습
  - Credentials
  - OAuth
---

# Lab 17. Credentials 관리 기초

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | OAuth2 와 API Key 방식 credential 의 차이를 이해하고, Google OAuth2 credential 을 등록하여 여러 노드에서 재사용할 수 있다 |
| 핵심 노드 | Google Sheets (read), Google Drive (list files) 등 Google 계열 |
| 소요 시간 | 약 50분 |
| 사전 준비 | Google 계정 필요, Lab 04 (HTTP) 완료 권장 |
| 난이도 | ★★★☆☆ |

!!! warning "실제 credential 등록 전제 Lab"
    이 Lab 은 본인의 **Google 계정으로 실제 OAuth 인증** 을 수행합니다. n8n cloud 환경에서는 redirect URL 이 자동 설정되므로 별도 GCP 콘솔 설정 없이 시작 가능합니다(self-host 는 추가 설정 필요).

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
    **Google Sheets OAuth2 API** credential 을 n8n 의 Credentials 메뉴에서 등록하세요. 좌측 사이드바의 **Credentials** → **[Create Credential]** 순서로 접근하여, Google 로그인 팝업에서 본인 계정으로 승인을 완료하고 "Account connected" 상태를 확인하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud, 본인 Google 계정
- **등록 단계:**

| 단계 | 동작 |
|------|------|
| 1 | 좌측 사이드바의 **Credentials** 클릭 |
| 2 | 우측 상단 **[Create Credential]** 클릭 |
| 3 | 검색창에 `Google Sheets` 입력 → **Google Sheets OAuth2 API** 선택 |
| 4 | credential 이름을 `Google Sheets - Personal` 로 설정 |
| 5 | **[Sign in with Google]** 버튼 클릭 → 팝업에서 Google 로그인 → 권한 동의 |
| 6 | 팝업 닫히면 n8n 화면에 "Account connected" 초록 표시 확인 |
| 7 | **[Save]** 클릭하여 저장 |

??? note "예시"

    > **등록 완료 상태 — Credentials 목록 화면**
    >
    > | Name | Type | 연결 상태 |
    > |------|------|---------|
    > | Google Sheets - Personal | Google Sheets OAuth2 API | ✅ Connected |
    >
    > **credential 상세 화면:**
    >
    > - Connection: `Connected as your-email@gmail.com`
    > - Client ID / Client Secret: 자동 (n8n cloud 가 관리)
    > - Scope: `https://www.googleapis.com/auth/spreadsheets`
    >
    > **확인 포인트:**
    >
    > - "Account connected" 메시지가 보이면 등록 성공
    > - 연결된 Google 이메일이 자동 표시됨
    > - 이 credential 은 **이제 모든 Google Sheets 노드에서 선택 가능**

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
    **OAuth2 방식** (Google Sheets) 과 **API Key 방식** (OpenAI 또는 다른 API Key 서비스) 을 **각각** 등록해보고, 두 방식의 등록 UX, 만료·갱신, 범위(Scope) 제한, 보안 특성을 비교 분석하세요. (Lab 21 에서 OpenAI 를 쓸 예정이므로 여기서 선등록 가능)

**활용 도구 & 데이터**

- **등록 대상 2종:**

| 유형 | Credential Type | 입력 방식 |
|------|-----------------|---------|
| OAuth2 | Google Sheets OAuth2 API (Step 1 에서 완료) | 브라우저 승인 |
| API Key | OpenAI API (또는 다른 키 기반 서비스) | 키 문자열 붙여넣기 |

- **OpenAI API Key 등록 예:**

| 필드 | 값 |
|------|-----|
| Name | `OpenAI - Personal` |
| API Key | `sk-...` (https://platform.openai.com/api-keys 에서 발급) |
| Organization ID | (선택, 조직 계정이면 입력) |

!!! warning "API Key 다루기 주의"
    API Key 는 **발급 시 단 한 번만 전체값 확인 가능** 합니다. 반드시 안전한 곳(패스워드 매니저 등) 에 백업하세요. n8n 에 저장된 뒤에는 마스킹되어 재확인 불가.

**(선택) 비교 프롬프트:**

```
OAuth2 와 API Key 방식의 API 인증을
"등록 UX / 만료·갱신 / 범위 제한 / 유출 시 대응 / 감사 로그" 5가지 기준으로 표로 만들어줘.
```

**예시**

> **OAuth2 vs API Key 비교표**
>
> | 기준 | OAuth2 (Google) | API Key (OpenAI) |
> |------|----------------|-----------------|
> | 등록 UX | 브라우저 팝업 승인 | 키 문자열 복사·붙여넣기 |
> | 만료·갱신 | Access Token 자동 갱신 (Refresh Token) | 키 수동 회전 |
> | 범위 제한 | Scope 단위 세밀 제한 (Sheets vs Gmail 등) | 통상 전체 권한 |
> | 유출 시 대응 | Google 계정에서 앱 권한 회수 | 키 1개 취소 필요 |
> | 감사 로그 | Google 활동 로그 지원 | 서비스별 상이 |
> | 설정 난이도 | 처음엔 복잡 (브라우저 동의) | 키 입력만 하면 끝 |
>
> **분석 결과:**
>
> - **OAuth2** 는 "권한의 최소화" 와 "유저 중심 제어" 가 강점 → 사용자 데이터에 접근할 때 표준
> - **API Key** 는 "설정 단순" · "서버 간 통신" 에 적합 → AI 서비스·결제 API 등에 흔함
> - n8n 은 두 방식 모두 **암호화해서 DB 저장** — 재사용 가능, 평문 노출 없음
> - 실무에선 서비스별 지원 방식이 정해져 있어 선택권은 없음 — 두 방식 다 능숙해야 함

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
    등록한 Google Sheets credential 을 **실제로 노드에 연결** 해 최소 연결 테스트를 수행하세요. 임의의 Google 스프레드시트(비어 있어도 OK) 를 하나 만들어 n8n 의 Google Sheets 노드에서 `Get Rows` operation 을 설정하고, 노드 설정 화면의 드롭다운에서 **스프레드시트 목록·시트 목록** 이 정상 로드되는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** Google Drive 에 테스트용 빈 스프레드시트 1개 생성, n8n cloud
- **워크플로우 이름:** `Lab 17 - Credential 연결 테스트`
- **Google Sheets 노드 설정:**

| 항목 | 값 |
|------|-----|
| Credential | `Google Sheets - Personal` (Step 1 에서 등록) |
| Resource | `Sheet within Document` |
| Operation | `Get Row(s)` |
| Document | 드롭다운에서 테스트 스프레드시트 선택 |
| Sheet | 드롭다운에서 `Sheet1` (또는 기본 시트) 선택 |

**구성 순서:**

1. Google Drive 에서 **테스트용 빈 스프레드시트** 생성 (이름 예: `n8n-test-sheet`)
2. 시트 A1 셀에 `id`, B1 셀에 `name` 헤더 정도만 입력해 저장
3. n8n 새 워크플로우에서 Manual Trigger → Google Sheets 노드 연결
4. Credential 선택 → Document·Sheet 드롭다운에서 방금 만든 스프레드시트가 **자동으로 검색** 되는지 확인
5. **[Execute Workflow]** 실행 → 에러 없이 Output (비었거나 헤더만) 이 돌아오면 성공

**예시**

> **연결 테스트 성공 지표**
>
> - Google Sheets 노드의 Credential 드롭다운에서 `Google Sheets - Personal` 선택 가능
> - Document 드롭다운 클릭 시 본인 Drive 의 스프레드시트 목록이 자동 로드
> - Sheet 드롭다운에서 선택한 Document 의 시트 목록이 자동 로드
> - 실행 결과 Output 에 헤더 또는 빈 결과가 표시되고 에러 없음
>
> **실무 적용 포인트**
>
> - 하나의 credential 을 등록해두면 **모든 Google Sheets / Drive / Gmail / Calendar 노드** 에서 재사용 가능
> - 팀 환경에선 `Google - Personal`, `Google - WorkspaceA`, `Google - WorkspaceB` 처럼 **계정별 분리** 하는 네이밍이 유용
> - 같은 방식으로 Lab 18·19·20 에서 쓸 credential 을 미리 등록하면 이후 실습이 빨라짐
>
> **Credentials 메뉴의 "Test" 버튼 활용**
>
> - 각 credential 상세 화면에서 **[Test]** 버튼 클릭 시 실제 API 호출로 유효성 검증
> - 실패하면 토큰 만료·권한 부족 원인이 즉시 드러남

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

> - **OAuth Scope 을 제한하면 어떻게 될까?**
>     - Google Drive Read-Only vs Full Access 처럼 같은 서비스에도 여러 scope 이 있다. n8n 의 `Generic OAuth2` credential 로 scope 을 수동 조정해보세요.
> - **credential 을 워크플로우와 분리해 관리하는 이점은?**
>     - 한 credential 을 여러 워크플로우가 공유하므로 **키 회전 시 한 번만 수정** 하면 모든 워크플로우에 반영. 반대로 워크플로우 공유 시 credential 이 함께 노출되지 않게 하는 것도 중요.
> - **HTTP Request 노드에서 Predefined Credential 을 사용하는 장점은?**
>     - "공식 노드가 없는 서비스" 도 HTTP Request + Predefined Credential 조합으로 인증 재사용 가능. 예: Notion API 를 HTTP Request 로 부르면서 Notion credential 을 재활용.
> - **Self-host 환경에서 Google OAuth 를 쓰려면 무엇이 추가로 필요할까?**
>     - GCP 콘솔에서 OAuth Consent Screen 설정, Client ID/Secret 발급, redirect URI 에 `https://your-n8n/rest/oauth2-credential/callback` 등록. 각 단계의 목적을 조사해보세요.
> - **credential 유출을 탐지·대응하는 가장 빠른 방법은?**
>     - 서비스별 사용 로그·알림 정책(Google 보안 알림, OpenAI usage 대시보드), n8n 의 Executions 감사 로그 등을 조합. 자체 체크리스트를 만들어보세요.
