# n8n · Dify 업무자동화 실습 가이드

5일 동안 n8n과 Dify를 이용해 직접 업무자동화 워크플로우를 만들어 봅니다.

## 학습 목표

- 반복 업무를 **n8n 워크플로우**로 자동화할 수 있다 (Trigger·HTTP·IF·Code·외부 서비스 연동).
- **Dify**로 챗봇과 지식기반 QA 봇, Agent 앱을 만들 수 있다.
- n8n과 Dify를 **연동**해 실제 업무 시나리오를 해결하는 통합 워크플로우를 설계할 수 있다.
- 노드간의 데이터를 주고받는 방식을 통해 프로그래밍적 (시스템적) 사고방식을 기를 수 있다

## 실습 진행 방식

!!! info "각 Lab은 다음 4단계로 진행됩니다"
    1. **Step 1. 탐색하기** — 가이드를 따라 직접 손으로 만들어 봅니다.
    2. **Step 2. 분석과 성찰** — 결과물을 동료·AI와 비교하며 관찰합니다.
    3. **Step 3. 창조하기** — 나의 상황에 맞게 변형·확장해 봅니다.
    4. **Beyond Level. 연구하기** — (선택) 심화 질문을 스스로 탐구합니다.

완성도보다 시도, 정답보다 관찰을 우선합니다.

## 커리큘럼 맵

### n8n (총 21개 Lab)

| 구분 | 범위 | 핵심 주제 |
|------|------|-----------|
| 기초 | Lab 01–05 | 첫 워크플로우, Trigger, Edit Fields, HTTP, Webhook |
| 중급 — 제어 흐름 & Expression | Lab 06–11 | IF, Switch, Expression, Merge, Loop, Wait |
| 심화 — Code & Error | Lab 12–16 | Code 노드, 파일 변환, Pagination, Error Trigger, Sub-workflow |
| 통합 — 외부 서비스 | Lab 17–21 | Credentials, Google Sheets, Gmail, Slack, OpenAI |

### Dify (총 15개 Lab)

| 구분 | 범위 | 핵심 주제 |
|------|------|-----------|
| 기초 | Lab 01–03 | 첫 챗봇, 프롬프트 템플릿, 변수 입력 폼 |
| 지식베이스(RAG) | Lab 04–06 | 지식베이스, 규정 QA 봇, 검색 최적화 |
| Workflow | Lab 07–10 | 문서 요약, IF/ELSE 분류, Iteration, HTTP Request |
| Chatflow & Agent | Lab 11–13 | 민원 Chatflow, Agent 기본, 내장 도구 |
| 통합·실전 | Lab 14–15 | n8n에서 Dify API 호출, 종합 프로젝트 |

Lab별 상세 내용은 왼쪽 네비게이션에서 원하는 Lab을 선택해 열람하세요.

## 사전 준비

!!! tip "수업 전 준비 사항"
    - [ ] ChatGPT 계정 (유료 플랜 권장)
    - [ ] n8n Cloud 가입 및 로그인
    - [ ] Dify Cloud 가입 및 로그인 + 사용 가능한 LLM 모델(OpenAI 등) 연결
    - [ ] 크롬 등 최신 브라우저

## 시작하기

!!! abstract "첫 실습으로 이동하기"
    왼쪽 네비게이션에서 **Lab 01 첫 워크플로우**부터 순서대로 진행하는 것을 권장합니다. 이미 익숙한 주제가 있다면 해당 Lab은 건너뛰고 필요한 Lab만 골라 진행해도 됩니다.
