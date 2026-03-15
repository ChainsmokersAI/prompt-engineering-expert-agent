# Andrej Karpathy의 AI Agent & 프롬프트 엔지니어링 철학

**최종 업데이트**: 2026-03-15  
**신뢰도**: 공식 문서(연설/저서), 저명 인사(블로그/X)  
**기반 자료**: 2024-2025년 최신 강연, 블로그, 소셜 미디어, 2026년 3월 Autoresearch 프로젝트

---

## 1. 핵심 철학

### 1.1 "The Hottest New Programming Language is English"
- **출처**: X 게시물 (2023년 1월)
- **의미**: LLM의 능력으로 인해 인간은 더 이상 전통 프로그래밍 언어를 배울 필요 없음
- **진화**: 2025년 "Vibe Coding" → 2026년 초 "Agentic Engineering"으로 진화
- **현재 관점**: "80% agent coding + 20% edits/touchups"로의 전환 (2026년 1월)
  - 이전: 80% 수동 코딩 + 20% agent
  - 현재: 80% agent 코딩 + 20% 수동 편집
  - **평가**: "2년 이상의 기본 코딩 워크플로우 변화 중 가장 큰 변화, 단 몇 주 내에 발생"

### 1.2 Context Engineering vs Prompt Engineering (2025년 중반)
- **출처**: X 게시물 (2025년 중반), "2025 LLM Year in Review"
- **핵심 주장**:
  > "Context engineering is the delicate art and science of filling the context window"
  - "Prompt Engineering"은 단순 작업 설명을 의미하는 것으로 오해됨
  - **진정한 도전**: 전체 context window를 지능적으로 구성하는 것
  
- **구성 요소**:
  1. Task descriptions and explanations
  2. Few-shot examples
  3. RAG (검색 기반 context)
  4. 관련 multimodal 데이터
  5. Tools and API definitions
  6. State and history
  7. Context compacting

- **핵심 통찰**:
  - Context engineering은 "산업용 강도의 LLM 앱"에서 실제 병목
  - 너무 많거나 부적절한 context는 LLM 비용 증가 + 성능 저하
  - "매우 비자명하고 어려운 과학"

### 1.3 System Prompt Learning (제3의 LLM 학습 패러다임)
- **출처**: X 게시물 (2025년 1월), Karpathy의 LLM 튜토리얼 및 논의
- **개념**: 기존 2가지 패러다임의 한계를 넘는 학습 방식
  1. **Pretraining**: 지식 습득 (가중치 변경)
  2. **Finetuning (SL/RL)**: 행동 습관 (가중치 변경)
  3. **System Prompt Learning** (새로운): 문제 해결 전략 (시스템 프롬프트 동적 편집)

- **작동 원리**:
  - LLM이 경험을 통해 자동으로 문제 해결 전략 개발
  - 마치 인간이 "다음 번에 이런 문제가 나오면 이 접근법을 시도할 것"이라고 메모하는 것처럼 작동
  - 사람이 손으로 시스템 프롬프트를 작성하는 것이 아니라 시스템이 자동으로 진화
  
- **영감의 원천**:
  - Claude의 시스템 프롬프트 크기: ~17,000단어
  - 이 정도 크기의 문제 해결 지식이 gradient descent(가중치)가 아닌 **명시적 텍스트**로 인코딩되어야 한다는 직관
  
- **성과 사례**:
  - 실제 구현: 500개 쿼리 후 129개 전략 개발 + 97개 개선

---

## 2. AI Agent 설계 핵심 원칙

### 2.1 Local Deployment > Cloud Orchestration
- **출처**: "2025 LLM Year in Review", Dwarkesh 팟캐스트, Claude Code 관찰
- **주요 통찰**:
  - Cloud 기반 에이전트 (예: ChatGPT API orchestration) = **실수**
  - Claude Code의 "ghost on your computer" 아키텍처 = **올바른 방향**
  
- **이유**:
  1. **"Jagged Capabilities" 시대**: 모델이 일부 분야는 뛰어나지만, 다른 분야는 여전히 취약
  2. **낮은 대기 시간**: 사용자 컴퓨터에서 직접 실행 (클라우드 왕복 제거)
  3. **데이터 프라이버시**: 개인 context, 시크릿, 설정 파일 직접 접근
  4. **장시간 컨텍스트**: 사용자의 전체 작업 세션을 컨텍스트 유지
  
- **Claude Code의 성공**:
  > "Claude Code는 LLM Agent이 어떤 모습인지를 보여주는 첫 번째로 설득력 있는 사례"

### 2.2 "AGI Agents는 10년 과제"
- **출처**: Dwarkesh 팟캐스트 (2025년 10월), Karpathy의 일관된 주장
- **실태 평가**:
  - "The Decade of Agents" ≠ "Year of Agents"
  - 현재 모델은 "매우 jagged한 능력"을 지님
  - 복잡하고 고유한 작업(예: 내 프로젝트의 "intellectually intense" 코드)에서는 여전히 "많은 인지적 결함"
  
- **병목 분석**:
  1. Boilerplate 코드/반복 작업에는 뛰어남 (훈련 데이터 풍부)
  2. 도메인 특화 및 복잡한 추론에는 약함
  3. Context 관리의 어려움
  4. 자가 수정 능력 부족

### 2.3 Production Agent 설계 원칙 (Podcast 재해석)
- **출처**: "Everyone Misread Andrej's Podcast" (Nate's Newsletter 재정리)
- **핵심 원칙** (7가지):
  1. **Memory Architecture > Raw Model Capability**
     - 모델 능력보다 에이전트가 어떻게 기억을 유지하는가가 중요
  2. **Architecture Determines Reliability**
     - 시스템 설계가 신뢰성 보장
  3. **Test Outcomes, Not Steps**
     - 에이전트가 취한 행동이 아닌 최종 결과 측정
  4. **Model Economics First**
     - API 호출 비용 먼저 이해한 후 코드 작성
  5. **Start with Expensive Problems**
     - 경제적 가치가 큰 고비용 문제부터 집중
  6. **Follow Cost, Ignore Hype**
     - 경제학에 따라 우선순위 결정, 마케팅 신화 무시
  7. **Use Proven Patterns**
     - 검증된 아키텍처 패턴 재사용

---

## 3. AI 코딩 시대의 실제 교훈

### 3.1 AI의 약점: "Hallucinated Assumptions"
- **출처**: Claude Code 필드 노트 (2026년 1월)
- **AI가 만드는 오류 유형**:
  1. **개념적 오류** (구문 오류 아님)
  2. **환각된 가정들**: 무언가를 그냥 가정하고 진행
  3. **잘못된 단순화**: 실제 문제보다 더 간단한 버전을 풂
  4. **숨겨진 모순**: 불일치를 감지하지 못함
  5. **트레이드오프 미숙지**: 다양한 선택지의 장단점을 제시하지 못함
  6. **비판적 거절 불가**: 명확히 잘못된 요청도 따름

- **Context Poisoning 개념**:
  - 대화 초반 하나의 오류 (가정/할루시네이션)가 모든 후속 단계에 영향
  - AI는 오류를 "진실"로 취급하고 계속 구축
  
- **Karpathy의 결론**:
  > "Important code는 반드시 hawk처럼 지켜봐야 한다"

### 3.2 Hallucination 재해석: "Bug가 아니라 Feature"
- **출처**: X 게시물 (2024년 12월), Karpathy 저작
- **관점 전환**:
  - LLM은 본질적으로 "dream machine" (꿈을 꾸는 기계)
  - "Hallucination"은 LLM의 **가장 큰 특징 (장점)**
  - 문제는 LLM 자체가 아니라, 이를 사용하는 **제품 레이어**가 미티게이션을 위해 설계되어야 함
  - 예: RAG, 검증 루프, multi-step 추론

- **핵심 메시지**:
  - 완벽한 모델을 기대하지 말고, 한계를 이해하고 그에 맞게 시스템을 구축할 것

### 3.3 Developer Experience Paradox
- **출처**: Claude Code 필드 노트 (2026년 1월)
- **역설**:
  - 프로그래밍이 **더 재미있어짐** (반복 작업 제거)
  - 동시에 **손으로 코딩 능력이 위축됨**
  - 가능한 결과: 엔지니어 시장 **양극화**
    - "Craftsmen": 코드 품질 중심 (손으로 코드 작성)
    - "Architects": 비즈니스 결과 최적화 중심 (LLM orchestration)

- **2026년 전망**:
  > "Slopacolypse" 경고: 저급 AI 생성 콘텐츠 폭증
  - AI-amplified 능력 차이로 상위/평균 엔지니어 생산성 격차 **급격히 증가**

---

## 4. Autoresearch 프로젝트: Agent 설계의 실현 사례

### 4.1 프로젝트 개요 (2026년 3월)
- **출처**: GitHub karpathy/autoresearch, MarkTechPost, The New Stack
- **규모**: 630줄 Python 코드
- **목표**: ML 실험을 autonomous AI agent가 통제
- **성능**:
  - 단일 GPU에서 밤새 80-100개 실험 실행
  - 시간당 약 12개 실험 (대략 5분/실험)
  - 사람이 며칠 걸릴 작업을 1박에 완료

### 4.2 아키텍처 설계 철학
- **출처**: Karpathy Autoresearch GitHub 문서, Context Studios 분석
- **핵심**: Markdown과 Python의 분리
  1. **Humans edit**: `program.md` (Markdown 파일)
     - 연구 지시사항 (what to search for)
     - 제약 조건 (what must not change)
     - 종료 기준 (when to stop)
  2. **AI edits**: `program.py` (단일 Python 파일)
     - 모델 아키텍처
     - Optimizer
     - 훈련 루프
  
- **설계 제약**: ~630줄 한계
  - **이유**: 최신 LLM의 context window에 전체 코드가 맞아야 오류 최소화
  - Token efficiency를 에이전트 신뢰성으로 전환

### 4.3 실제 성과
- **출처**: Karpathy, Shopify CEO Tobi Lütke 언급
- **사례**: Autoresearch 사용 시 더 작은 모델이 더 큰 모델을 능가
  - 기존 수동 설정 큰 모델 < AI 최적화된 작은 모델
  - **개선율**: 19% validation score 향상

---

## 5. 2025년 LLM 6대 패러다임 시프트

### Karpathy가 정의한 핵심 변화
- **출처**: "2025 LLM Year in Review" (karpathy.bearblog.dev)

#### 시프트 1: RLVR (Reinforcement Learning from Verifiable Rewards)
- SFT+RLHF에서 **objective reward 함수**로 전환
- 수학/코드 같은 검증 가능한 도메인에서 자동으로 추론 전략 개발

#### 시프트 2: "Ghosts vs Animals"
- LLM은 인간과 완전히 다른 지능 형태
- Jagged capabilities: 검증 가능한 영역에서 뛰어남, 속임수/트릭에 취약
- Benchmark 신뢰성 재평가 필요

#### 시프트 3: Application Layer의 부상 (Cursor, Claude Code)
- 범용 LLM 위의 **distinct software tier** 등장
- Context orchestration, multi-call 관리, 도메인 특화 UI/UX, autonomy control
- 범용 모델 + 특화 앱의 계층 구분 명확화

#### 추가: "Vibe Coding" → "Agentic Engineering"
- Vibe Coding (초기, 2025년 2월): 코드를 완전히 무시하고 LLM에 전적 신뢰
- Agentic Engineering (2026년 초): 감시 + 조율을 강조하는 방향으로 진화

---

## 6. Prompt 작성 원칙 (암시적)

Karpathy는 명시적인 "프롬프트 레시피"를 제시하지 않지만, 행동에서 도출되는 원칙:

### 6.1 Context 최적화
- Context window 전체를 전략적으로 활용
- 불필요한 정보는 제거 ("compacting")
- 최신 history와 state를 명시적 포함

### 6.2 Task Clarity
- 모호한 요청이 아닌 명확한 목표/제약 정의
- Markdown 형식으로 human-readable 구조화 (Autoresearch 예)

### 6.3 Oversight와 Verification
- AI 완전 자동화가 아닌 인간의 check point 포함
- "program.md"로 인간이 명확하게 지시 가능
- 결과 검증 루프 필수

### 6.4 Cost-First Thinking
- 모든 LLM 시스템은 API 호출 경제학부터 설계
- Token 효율성이 능력과 동등하게 중요

---

## 참고 자료

| 자료 | URL | 신뢰도 |
|------|-----|--------|
| 2025 LLM Year in Review | https://karpathy.bearblog.dev/year-in-review-2025/ | 공식 블로그 |
| Context Engineering Tweet | https://x.com/karpathy/status/1937902205765607626 | 저자 직접 |
| Autoresearch GitHub | https://github.com/karpathy/autoresearch | 공식 오픈소스 |
| Dwarkesh Podcast | https://www.dwarkesh.com/p/andrej-karpathy | 공식 인터뷰 |
| Claude Code Field Notes | https://dev.to/jasonguo/karpathys-claude-code-field-notes | 저자 분석 |
| Podcast 재해석 (Nate) | https://natesnewsletter.substack.com/p/everyone-misread-andrejs-podcastheres | 전문가 해석 |
| System Prompt Learning | https://x.com/karpathy/status/1921368644069765486 | 저자 직접 |

