---
title: "Lab 10. HTTP Request 노드로 외부 API 호출"
description: "HTTP Request 노드로 공공데이터 API를 호출하고, 응답 JSON을 LLM으로 정리해 보고서 형태로 출력합니다."
tags:
  - dify
  - workflow
  - HTTP
---

# Lab 10. HTTP Request 노드로 외부 API 호출

## 개요

| 항목 | 내용 |
|------|------|
| 학습 목표 | HTTP Request 노드로 외부 API를 호출하고, 응답 JSON을 Code 노드로 파싱한 뒤 LLM으로 자연어 보고서를 생성할 수 있다 |
| 핵심 노드 | HTTP Request, Code, LLM, End |
| 소요 시간 | 약 55분 |
| 사전 준비 | Lab 07 완료, 호출 가능한 공개 REST API 1개 (공공데이터포털 API 또는 JSONPlaceholder 등) |
| 난이도 | ★★★☆☆ |

!!! warning "API Key 관리"
    API Key는 노드 설정에서 **Environment Variable** 또는 별도 보안 필드로 저장하세요. 프롬프트/로그에 Key가 그대로 노출되지 않도록 주의합니다.

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
    **HTTP Request 노드**로 공개 API(예: JSONPlaceholder)를 GET 호출해 게시물 목록을 받아온 뒤, **LLM 노드**에서 그 내용을 한국어 보고서로 정리하세요. 연결 구조: Start → HTTP Request → Code → LLM → End.

**활용 도구 & 데이터**

- **도구:** Dify Workflow (앱 이름: `Lab 10 - API 호출 & 요약`)
- **Start 변수:** `topic_keyword` (Text) — 기본값 `id=1`
- **사용 API:** `https://jsonplaceholder.typicode.com/posts` (인증 불필요, 공개 테스트 API)
- **노드 구성:**

| 노드 | 설정 |
|------|------|
| HTTP Request | Method: `GET`, URL: `https://jsonplaceholder.typicode.com/posts?_limit=5`, Headers: `Accept: application/json` |
| Code (Python) | 응답 body를 파싱해 `titles` (Array[String])와 `bodies` (Array[String]) 로 분리 |
| LLM | `titles` / `bodies` 를 받아 한국어 요약 보고서 생성 |
| End | `report = LLM.text` |

**Code 노드 (Python):**

```python
import json

def main(body: str) -> dict:
    data = json.loads(body) if isinstance(body, str) else body
    titles = [item.get("title", "") for item in data]
    bodies = [item.get("body", "") for item in data]
    return { "titles": titles, "bodies": bodies, "count": len(data) }
```

- Input: `body = {{#HTTP.body#}}`
- Output Variables: `titles` (Array[String]), `bodies` (Array[String]), `count` (Number)

**LLM 프롬프트:**

```
[System]
당신은 공공데이터 분석가입니다. 아래 게시글 {{#Code.count#}}건을 한국어로 요약해,
`[게시글 요약 보고서]` 라는 제목 아래 다음 포맷으로 정리하세요.

## 전체 요약 (3줄)
- ...
- ...
- ...

## 개별 게시글
1. [제목] (원문: 영어 그대로)
   - 요약: ...
2. ...

[User]
제목 리스트: {{#Code.titles#}}
본문 리스트: {{#Code.bodies#}}
```

**실행 단계:**

1. Workflow 앱 생성 → HTTP Request 노드 추가
2. URL과 Method, Headers 입력
3. Code 노드 추가 → HTTP의 `body`를 받아 titles/bodies로 파싱
4. LLM 노드에 프롬프트 입력
5. End 노드에 `report = LLM.text` 연결
6. Run → 결과 확인

??? note "예시"

    > **HTTP Request 응답 (원본 일부):**
    >
    > ```json
    > [
    >   {"userId":1,"id":1,"title":"sunt aut facere...","body":"quia et suscipit..."},
    >   {"userId":1,"id":2,"title":"qui est esse","body":"est rerum tempore..."},
    >   ...
    > ]
    > ```
    >
    > **End 노드 `report` 출력:**
    >
    > ```
    > [게시글 요약 보고서]
    >
    > ## 전체 요약 (3줄)
    > - 5건의 테스트 게시글은 모두 userId 1번 사용자가 작성한 더미 데이터이다.
    > - 제목과 본문은 라틴어(lorem ipsum) 스타일의 의미 없는 텍스트로 구성된다.
    > - 이 API는 실제 데이터 테스트가 아닌 구조·연결 테스트용으로 사용된다.
    >
    > ## 개별 게시글
    > 1. [sunt aut facere...]
    >    - 요약: 반복·책임·설명을 논하는 더미 문장
    > 2. [qui est esse]
    >    - 요약: 시간·사건·원인에 관한 더미 문장
    > ... (5건)
    > ```
    >
    > **확인 포인트:**
    >
    > - HTTP 노드 실행 후 body에 배열이 담겼는지
    > - Code 노드에서 `count=5`, `titles` 길이 5인지
    > - LLM이 `count` 변수를 받아 "5건"으로 응답했는지

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
    **(A) Code 노드로 전처리한 버전** 과 **(B) Code 생략 후 LLM에 원본 JSON을 통째로 넘긴 버전** 을 비교하세요. 토큰 사용량·출력 안정성·오류 대응 관점에서 장단점을 정리합니다.

**활용 도구 & 데이터**

- **도구:** Step 1 Workflow (A) + 새 Workflow (B)
- **비교 설계:**

| 버전 | 구조 |
|------|------|
| A (전처리) | HTTP → Code(titles/bodies 파싱) → LLM (clean input) → End |
| B (원본 투입) | HTTP → LLM(body 통째로 전달) → End |

**측정 항목:**

| 항목 | 측정 |
|------|------|
| 입력 토큰 수 | LLM에 들어간 토큰 (Run Log) |
| 출력 안정성 | 포맷 지시를 얼마나 지키는가 |
| 오류 대응 | 응답이 비거나 에러일 때 어떻게 되는가 |

**비교 방법:**

1. A/B 두 Workflow 각 3회 실행
2. Run Log에서 LLM 입력 토큰 평균 계산
3. 일부러 `_limit=0` 으로 빈 응답 → 두 버전 반응 비교
4. 잘못된 URL(404)로 에러 → 두 버전 반응 비교

**예시**

> **비교 분석표**
>
> | 항목 | A (전처리) | B (원본 투입) |
> |------|-----------|--------------|
> | 평균 입력 토큰 | 420 | 780 (약 1.9배) |
> | 포맷 지시 준수율 | 3/3 | 2/3 (1회는 JSON을 그대로 복사) |
> | 빈 응답 | "0건" 으로 정상 안내 | LLM이 "응답을 이해하지 못함" |
> | 404 에러 | Code에서 명시적 실패, LLM 호출 생략 가능 | LLM이 HTML 에러페이지를 그대로 요약하려 함 |
>
> **분석 결과:**
>
> - Code 노드로 **필요한 필드만 추출**하면 토큰 절반 수준, 포맷 안정성 상승
> - 빈 응답·에러 시 **Code가 방어선 역할** — LLM 호출 전에 검증·분기 처리 가능
> - 실무 원칙: **"HTTP → Code(파싱·검증) → LLM"** 이 외부 API 호출의 표준 패턴
> - 단, 응답 구조가 자주 바뀌거나 탐색용이라면 원본을 넘기는 것이 빠르게 실험하기 편함

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
    본인 업무와 관련된 **공공데이터포털 API** 또는 **공개 API** 1개를 선정해 "외부 데이터 → 보고서" Workflow를 완성하세요. Start 변수로 검색 파라미터를 받고, HTTP 응답을 Code로 정제한 뒤, LLM이 한국어 보고서를 생성합니다.

**활용 도구 & 데이터**

- **도구:** Dify Workflow + 실제 외부 API
- **완성 체크리스트:**

| 항목 | 기준 |
|------|------|
| 실제 API 사용 | 공공데이터 API 또는 공개 API |
| Start 변수 | 최소 1개 검색 파라미터 |
| HTTP 노드 | URL에 변수 삽입(동적 쿼리) |
| Code 노드 | 응답 파싱 + 빈 응답 처리 |
| LLM | 한국어 보고서 포맷 |
| 에러 대응 | Code에서 유효성 검증 후 LLM으로 분기 |

**완성 순서:**

1. 관심 API 선정 + API Key 발급(필요 시)
2. Postman 등으로 먼저 수동 호출 → 응답 구조 이해
3. Dify에서 HTTP 노드 URL에 `{{#Start.keyword#}}` 삽입
4. Code 노드 작성 → 응답 필드 추출 + 빈 응답 플래그
5. LLM 프롬프트에 보고서 포맷 정의
6. Run 테스트 → 빈 응답·오류 응답도 함께 테스트
7. Publish

**예시**

> **완성 Workflow: "도서관 정보 나루 API로 베스트셀러 보고서"**
>
> (사용 API: 정보나루 `https://www.nl.go.kr/NL/contents/api/bestseller.do` — 공개 사례, 실제 사용 시 해당 기관 안내에 따라 Key 발급)
>
> **Start 변수:**
>
> | 변수 | 값 예시 |
> |------|--------|
> | genre | 소설, 자기계발, 역사 |
> | period | 최근 1개월 / 3개월 / 6개월 |
>
> **HTTP 노드 URL (예):** `https://api.example.go.kr/bestseller?genre={{#Start.genre#}}&period={{#Start.period#}}&key=ENV_API_KEY`
>
> **Code 출력:** `books: [{title, author, rank}]`, `count`
>
> **LLM 보고서 출력 샘플:**
>
> ```
> [도서 베스트셀러 보고서] (소설, 최근 3개월)
>
> ## 요약
> - 총 10종, 1~3위는 국내 작가 작품 우세
> - 장르적으로 미스터리·가족 드라마 혼재
>
> ## TOP 5
> 1. 『책제목A』 / 저자A — ...
> 2. 『책제목B』 / 저자B — ...
> ... (5건)
>
> ## 인사이트
> - ○○ 소재 작품이 3편 포함, 사회 트렌드 반영으로 해석 가능
> - 시립도서관 신간 구비 시 참고 가치 있음
> ```

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

> - **POST + Body(JSON)가 필요한 API는 어떻게 호출할까?**
>     - HTTP 노드에서 Method POST, Body에 JSON 문자열 입력. 실제 공공데이터 중 POST 방식 API를 골라 호출해보세요.
> - **Pagination이 있는 API는 Iteration과 어떻게 결합할까?**
>     - 1페이지 호출 → 총 페이지 수 확인 → Iteration으로 나머지 페이지 호출. 전체 데이터 수집 Workflow를 설계해보세요.
> - **응답이 XML인 API는 어떻게 처리할까?**
>     - Code 노드에서 `xml.etree.ElementTree` 같은 파서를 써서 JSON으로 변환한 뒤 LLM에 전달해보세요.
> - **호출 빈도 제한(Rate Limit)이 있는 API는 어떻게 안정적으로 다룰까?**
>     - Iteration의 Concurrency 1로 낮추고, 에러 시 재시도 로직(Code 노드)으로 설계해보세요.
