---
title: "Lab 19. Gmail 노드로 메일 송수신"
description: "Gmail 노드로 필터 기반 메일 조회, 본문·첨부 추출, 메일 전송을 실습합니다."
tags:
  - n8n
  - 통합실습
  - Gmail
  - 이메일
---

# Lab 19. Gmail 노드로 메일 송수신

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | Gmail 노드로 검색 쿼리 기반 메일 조회, 메시지 본문·메타데이터 파싱, 메일 전송(첨부 포함) 을 수행할 수 있다 |
| 핵심 노드 | Manual Trigger, Gmail, Edit Fields (Set), Convert to File |
| 소요 시간 | 약 60분 |
| 사전 준비 | Lab 17 (credential 등록 법) 완료, Gmail OAuth2 API credential 등록 필요 |
| 난이도 | ★★★☆☆ |

!!! warning "Gmail credential 등록이 먼저"
    Lab 17 에서 Google Sheets 를 등록했다면, 같은 방식으로 **Gmail OAuth2 API** credential 을 추가 등록하세요 (Scope 이 달라 별도 credential 필요). Credentials 메뉴 → Create Credential → `Gmail OAuth2 API` 검색 → Sign in with Google.

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
    Gmail 노드의 `Get Many Messages` operation 으로 **받은편지함의 최근 메일 5개** 를 불러오세요. Output 에서 각 메시지의 `id`, `subject`, `from`, `snippet` 필드를 확인하여 Gmail API 가 어떤 구조로 데이터를 반환하는지 관찰하세요.

**활용 도구 & 데이터**

- **도구:** n8n cloud, Gmail OAuth2 credential (사전 등록)
- **워크플로우 이름:** `Lab 19 - Gmail 기초`
- **Gmail 노드 설정:**

| 항목 | 값 |
|------|-----|
| Credential | `Gmail - Personal` |
| Resource | `Message` |
| Operation | `Get Many` |
| Return All | OFF |
| Limit | `5` |
| Simplify | ON (기본 — 주요 필드만 정리해서 반환) |

**구성 단계:**

1. Manual Trigger → Gmail 노드 연결
2. 위 표대로 설정
3. **[Execute Workflow]** 실행
4. Output 에 5개 아이템이 들어오는지 확인

??? note "예시"

    > **Gmail Output JSON (5개 중 1개 발췌, Simplify ON)**
    >
    > ```json
    > {
    >   "id": "19283ab...",
    >   "threadId": "19283ab...",
    >   "labelIds": ["INBOX", "UNREAD"],
    >   "subject": "Welcome to the service",
    >   "from": {
    >     "value": [{ "name": "Newsletter", "address": "news@example.com" }]
    >   },
    >   "to": { "value": [{ "address": "your-email@gmail.com" }] },
    >   "date": "Thu, 18 Apr 2025 09:12:34 +0900",
    >   "snippet": "Thanks for joining our platform. Here are a few tips to get started..."
    > }
    > ```
    >
    > **확인 포인트:**
    >
    > - `id` 는 Gmail 고유 메시지 ID — 다음 operation(읽음 처리, 삭제, 라벨링)의 입력으로 사용
    > - `from.value` 가 **배열** — 여러 수신자도 담을 수 있는 구조 (tos/ccs 에서 실감)
    > - `snippet` 은 본문의 짧은 미리보기 — 전체 본문은 Simplify 를 끄거나 `Get Message` 로 개별 조회
    > - `labelIds` 로 읽음/안 읽음, 라벨 분류 가능

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
    Gmail 의 **Search Query(필터)** 를 사용해 특정 조건에 맞는 메일만 조회하세요. 3가지 쿼리(안 읽은 메일 / 특정 발신자 / 첨부파일 있는 메일) 를 각각 실행하며 Gmail 검색 문법을 실감하세요.

**활용 도구 & 데이터**

- **도구:** Step 1 노드에서 Filters 섹션 설정
- **실험할 쿼리 3종 (Filters > Search):**

| # | Search 쿼리 | 의미 |
|---|-----------|------|
| A | `is:unread` | 안 읽은 메일만 |
| B | `from:noreply@github.com` | GitHub 자동 메일만 |
| C | `has:attachment newer_than:30d` | 최근 30일 이내 첨부파일 있는 메일 |

!!! tip "Gmail 검색 연산자"
    Gmail 웹 UI 의 검색창과 동일한 문법이 그대로 작동한다:
    
    - `from:` / `to:` / `subject:` — 발신·수신·제목 매치
    - `is:unread` / `is:starred` / `is:important`
    - `has:attachment` — 첨부파일 존재
    - `newer_than:7d` / `older_than:30d` — 기간
    - `label:"내 라벨"` — 라벨 기준
    - AND(공백), OR, `-` (제외) 조합 가능

**변경 방법:**

1. Gmail 노드의 **Filters** 섹션 펼치기 → **Search** 필드에 쿼리 입력
2. 쿼리별로 실행 후 반환된 아이템 수·내용 비교
3. 아무 결과가 없는 쿼리도 일부러 만들어 빈 Output 동작 관찰

**(선택) 분석 프롬프트:**

```
Gmail 검색 연산자 중 자동화 시나리오에 자주 쓸 만한 10가지를
"용도 / 예시" 표로 정리해줘.
```

**예시**

> **쿼리별 결과 비교 (본인 메일 상태에 따라 개수 상이)**
>
> | 쿼리 | 반환 아이템 수 | 특징 |
> |------|--------------|------|
> | `is:unread` | 본인 상황에 따라 0 ~ 수십 개 | UNREAD 라벨 포함 메시지만 |
> | `from:noreply@github.com` | GitHub 사용 여부에 따라 상이 | 특정 발신자 전용 |
> | `has:attachment newer_than:30d` | 첨부 메일 수 | 첨부 처리 자동화의 출발점 |
>
> **분석 결과:**
>
> - Gmail 의 **Search 기능이 곧 n8n 의 Trigger/Filter 언어** 가 된다 → UI 에서 잘 만들어진 쿼리를 그대로 붙여 넣으면 됨
> - "본문 내용에 'invoice'" 같은 쿼리(`"invoice"`) 는 강력하지만 false positive 가 많을 수 있어 `subject:` 와 조합 권장
> - 대량 메일 처리 시엔 **Limit + Label 필터 조합** 으로 스코프를 좁히는 것이 안전
> - 쿼리로 원하는 메일이 안 잡히면 Gmail 웹에서 같은 쿼리 먼저 테스트 → 결과가 나오는지 크로스체크

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
    본인 계정으로 **자기 자신에게 메일을 전송** 하는 워크플로우를 완성하세요. Set 노드에서 동적 내용을 구성하고, `Send Message` operation 으로 HTML 본문 + **CSV 첨부파일** 을 함께 보내세요. 실제 받은편지함에서 메일과 첨부가 정상 표시되는지 확인하세요.

**활용 도구 & 데이터**

- **도구:** Gmail credential, 같은 워크플로우
- **구조:**

```
Manual Trigger
  → Set (JSON, 샘플 데이터 3개)
  → Convert to File (CSV, fileName: daily_report.csv)
  → Gmail (Send Message, with attachment)
```

- **Set (JSON) 데이터:**

```json
[
  { "product": "A", "qty": 3 },
  { "product": "B", "qty": 5 },
  { "product": "C", "qty": 1 }
]
```

- **Gmail Send Message 설정:**

| 항목 | 값 |
|------|-----|
| Resource | `Message` |
| Operation | `Send` |
| To | 본인 이메일 (`your-email@gmail.com`) |
| Subject | `=[Test] 일일 리포트 {{ $now.toFormat("yyyy-MM-dd") }}` |
| Email Type | `HTML` |
| Message | (아래 HTML 입력) |
| Attachments | Input Binary Field = `data` |

- **HTML Message 내용:**

```html
<h2>일일 리포트</h2>
<p>첨부된 CSV 파일에 오늘의 샘플 데이터를 담았습니다.</p>
<ul>
  <li>총 상품 수: 3</li>
  <li>생성 시각: {{ $now.toFormat("yyyy-MM-dd HH:mm") }}</li>
</ul>
<p>이 메일은 n8n Lab 19 워크플로우로 자동 발송되었습니다.</p>
```

!!! tip "첨부파일이 붙으려면"
    Convert to File 의 Output 이 Gmail 노드의 입력으로 들어와야 하며, Gmail 노드의 **Add Option > Attachments > Input Binary Field** 에 `data` 를 지정해야 CSV 가 첨부된다.

**구성 순서:**

1. Set(JSON) → Convert to File (CSV) 로 파일 생성
2. Gmail 노드 연결, 위 설정 입력
3. Subject/Message 에서 `{{ $now }}` 같은 Expression 이 잘 평가되는지 프리뷰로 확인
4. **[Execute Workflow]** 실행 → Output 에 `id`(발송된 메시지 ID) 가 나오면 성공
5. 자신의 Gmail 받은편지함 열어 제목·본문·첨부 모두 정상인지 검증

**예시**

> **완성 워크플로우: "자기 자신에게 일일 CSV 리포트 보내기"**
>
> **받은편지함에서 보이는 메일 (시각 예)**
>
> - 제목: `[Test] 일일 리포트 2025-04-18`
> - 본문: HTML 렌더링된 리포트 (제목, 설명, 리스트)
> - 첨부: `daily_report.csv` (클릭해 다운로드)
>
> **n8n Gmail 노드 Output**
>
> ```json
> [
>   {
>     "id": "19283cd...",
>     "threadId": "19283cd...",
>     "labelIds": ["SENT"]
>   }
> ]
> ```
>
> **구현 포인트:**
>
> - HTML 메일 + 첨부파일 = 실무에서 가장 흔한 **리포트 메일** 패턴
> - Subject/Message 에 Expression 을 쓰면 매일 다른 제목·본문 자동 생성
> - Schedule Trigger(Lab 02) 로 바꾸면 **매일 오전 9시 자동 리포트 메일** 이 됨
> - 보내는 from 주소는 credential 에 연결된 Google 계정 그대로 사용됨

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

> - **Gmail Trigger (Polling) 는 어떻게 쓸까?**
>     - `Gmail Trigger` 노드를 쓰면 지정 쿼리에 맞는 새 메일 도착 시 자동 워크플로우 실행. Schedule + Get Many 조합과 어떤 차이가 있는지 비교해보세요.
> - **메일 본문 파싱 자동화 (예: 영수증 → 금액 추출) 는 어떻게 구성할까?**
>     - Gmail 로 본문을 받은 뒤 Code 노드(정규식) 또는 AI 노드(Lab 21)로 추출. 어느 쪽이 안정적인지 본인 데이터로 실험해보세요.
> - **라벨을 자동으로 붙이거나 뗄 수 있을까?**
>     - `Add Label to Message` / `Remove Label from Message` operation 활용. 자동 분류 시나리오를 설계해보세요.
> - **첨부파일을 Drive 에 저장 후 원본은 보관 처리하려면?**
>     - Gmail 로 첨부 다운로드 → Google Drive 노드로 업로드 → 원본 메일에 라벨링 + Archive. 파이프라인을 직접 그려보세요.
> - **대량 메일 처리 시 Rate Limit 을 어떻게 관리할까?**
>     - Gmail API 쿼터, Split In Batches + Wait 조합, Error Handler 와 결합. Lab 10·11·15 를 통합해 안전한 배치 설계를 해보세요.
