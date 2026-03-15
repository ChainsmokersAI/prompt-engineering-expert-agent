# Boris Cherny의 Context Engineering 철학

**최종 업데이트**: 2026-03-15  
**출처**: Substack(Karolina Zieminski, Push to Prod), Pragmatic Engineer(Gergely Orosz), DEV Community, X(Twitter) 인터뷰 및 강연  
**신뢰도**: 높음 (Anthropic 공식 자료, Boris Cherny 직접 인터뷰)

---

## 1. Context Engineering vs Prompt Engineering의 철학적 차이

### Boris Cherny의 관점

"Context Engineering"은 **단회성 프롬프트 최적화(Prompt Engineering)를 넘어**, 시스템 수준의 **지속적인 정보 구조화 및 재사용**에 초점을 맞춥니다.

**주요 차이점**:

| 관점 | Prompt Engineering | Context Engineering |
|-----|-------------------|-------------------|
| 초점 | 개별 요청에 대한 프롬프트 작성 | 시스템 전체의 컨텍스트 인프라 구축 |
| 시간 범위 | 단회성, 세션 내 | 장기, 다중 세션에 걸친 누적 |
| 주요 결과물 | 효과적인 프롬프트 | 공유 가능한 제도화된 지식(CLAUDE.md) |
| 편집자 | 개인 | 팀 |

**핵심 원칙**: "Pay once, benefit forever"
- 반복되는 피드백 패턴을 발견하면 → 규칙화하여 CLAUDE.md에 기록
- 이후 모든 Claude 세션이 그 규칙을 자동으로 따름
- 일회 투자(writing rule) vs 지속적 비용(checking everything repeatedly) 트레이드오프

---

## 2. CLAUDE.md: 핵심 Context Engineering 도구

### 구조 및 운영 원칙

**What is CLAUDE.md?**
- 프로젝트 루트에 체크인된 파일
- "Personal training data for Claude"로 기능
- 팀 전체가 주당 여러 번 업데이트
- 각 Claude 세션 시작 시 자동으로 읽혀짐

**포함되는 내용**:
1. **Project conventions**: 코딩 스타일, 아키텍처 결정사항
2. **Common mistakes**: Claude가 반복적으로 저지르는 오류와 해결책
3. **Architectural decisions**: 왜 이렇게 했는가에 대한 기록
4. **Best practices**: 팀이 발견한 효과적인 패턴

**운영 철학**: Ethics-by-Design
- 최선의 방식을 자동화 → 피로나 인적 오류로 인한 실패 감소
- 신뢰하지 않되, 계측하라 (Don't trust; instrument)

### 업데이트 프로세스

```
Claude 오류 발생 
  ↓
Code Review(human이 태그: @.claude)
  ↓
이 규칙을 CLAUDE.md에 추가해서 다시는 반복되지 않도록 하자
  ↓
CLAUDE.md 업데이트 (다음 세션부터 적용)
  ↓
메타 작업: 프로세스 자체가 개선됨
```

**타겟**: 3-4번 반복되는 피드백이 있으면 규칙화

---

## 3. Memory System 설계 원칙

### "AI is forgetful by default" 가정

Claude(AI 일반)는 기본적으로 각 세션에서 새로 시작합니다. 따라서:

**외부 메모리 필수** → CLAUDE.md, 제도화된 문서로 정보 영속화

### 팀 수준 Pattern

```
Individual Learning → Institutional Memory → System Design
  ↓
개발자가 Claude 오류 발견
  ↓
CLAUDE.md 규칙으로 변환
  ↓
모든 Claude 인스턴스가 자동으로 따름
  ↓
팀 전체의 학습 누적
```

**효과**:
- Collective learning: 한 사람의 학습이 팀 전체에 즉시 적용
- Institutional memory: 개인 개발자가 떠나도 지식 유지
- Leverage: 반복 학습 비용 최소화

---

## 4. Context Window 최적화 전략

### 병렬 처리 아키텍처 (Fleet Approach)

**핵심 통찰**: "하나의 무거운 뇌보다 여러 가벼운 뇌"

**Boris Cherny의 운영 방식**:
- **동시 실행**: 약 10-15개의 Claude 세션 유지
  - Terminal 5개(탭, 번호 지정, OS 알림 설정)
  - Browser 5-10개
  - Mobile 세션들(아침에 시작, 나중에 확인)

**컨텍스트 관리 철학**:

```
AI = 계산 능력을 할당하는 리소스
  ↓
  다중 세션 병렬 운영
  ↓
  전문화(specialization) 가능
  ↓
  각 작업에 최적의 context 크기 할당
```

**장점**:
1. **Context 효율성**: 각 세션이 "필요한 것만" 로드
2. **Specialization**: 하나의 세션은 PR 검토, 다른 하나는 구현 등으로 전문화
3. **Cognitive load distribution**: 주의력 자원을 지능적으로 할당
4. **Queue management**: 준비 완료된 작업부터 처리(이전 완료 대기 불필요)

**Context 보호**:
- 각 세션은 독립적인 워커
- CLAUDE.md를 공유하며 전문성 유지
- Sub-agent로 더 세분화 가능

### 모델 선택: 품질 우선

**Boris의 기준**: "Cost per reliable change" (신뢰할 수 있는 변화당 비용)

```
느리지만 정확한 답변 << 빠르지만 틀린 답변

Reasoning을 포함한 완전한 솔루션 > 빠른 iteration의 부분 솔루션
```

**실제 선택**: Opus 4.5 with extended thinking
- 더 느림
- 훨씬 정확
- 재작업 비용 최소화

---

## 5. Agent 설계에서의 Context 관리

### Subagent의 역할: Context Protection

```
목적: 특정 작업에 필요한 컨텍스트만 격리
  ↓
한 Agent가 모든 것을 알 필요 없음
  ↓
각 Agent가 자신의 CLAUDE.md 규칙 + 필요한 정보만 로드
  ↓
전체 시스템의 인지 부하 감소
```

**Subagent 설계 패턴**:

| Agent | 책임 | 필요한 Context |
|-------|------|-----------|
| Specification Agent | 요구사항 정의(PRD 작성) | 프로덕트 철학, 기존 시스템 지식 |
| Implementation Agent | 코드 작성 | 코딩 스타일, 아키텍처 결정 |
| Verification Agent | QA, 테스트 | 테스트 케이스, UX 가이드 |

**효과**: 
- 각 Agent의 context window를 효율적으로 활용
- 전문화를 통해 정확도 향상
- 협력 가능(한 Agent의 출력이 다음 Agent의 입력)

### Structured Planning (Pre-execution)

**패턴**:

```
1. Planning Mode: 전략 수립 및 반복
   - "이 작업을 어떻게 할까?"
   - Claude와 함께 계획 개선
   - 명시적 제약 설정

2. Implementation Mode: Auto-accept 활성화
   - "계획대로 실행해"
   - 이미 제약이 정의됨 → 원치 않는 변경 방지
   - 빠른 실행
```

**이점**: 
- Specification precision (명확한 의도 사전 정의)
- Execution confidence (지정된 틀 내에서만 동작)

---

## 6. Code Review as Context Engineering

### Meta-work: 프로세스 개선

**Boris의 관점**: Code review는 단순히 "버그 찾기"가 아닙니다.

```
Traditional Code Review:
  버그 발견 → 피드백 → 수정 → 완료

Context Engineering Perspective:
  버그 발견 → 피드백 → CLAUDE.md 규칙 추가 → 시스템 개선 → 완료
```

**GitHub Action 활용**:
- PR에 @.claude 태그 추가
- 새로 발견된 패턴을 자동으로 CLAUDE.md에 제안
- 팀의 지속적 학습 루프 자동화

---

## 7. 검증 루프 (Verification Loops)

### Claude의 자체 검증 능력

**원칙**: "최고의 Agent는 자신의 작업을 검증할 수 있는 Agent"

**예시**: UI 개발
```
Claude:  HTML/CSS 생성
  ↓
Browser 열기 → 실제 렌더링 확인
  ↓
결과가 의도와 다르면? → 자동 수정 및 재시도
  ↓
반복: 정확한 결과까지 자체 반복
```

**Context 관점의 의미**:
- Claude가 결과 검증 도구(Browser)에 접근 가능 → 피드백 루프 단축
- 외부 검증 대기 필요 없음
- Self-correction 능력이 신뢰도 증가

---

## 8. 핵심 원칙 정리

### "Pay Once, Benefit Forever"

```
규칙 수립 비용(1회) << 반복 검증 비용 (매 세션마다)
```

### "Ethics by Design"

최고의 관행을 **자동화하여 실패 가능성 제거**:
- 휴먼 에러 방지
- 피로에 의한 실수 제거
- 일관성 보장

### "AI as Capacity to Schedule"

AI를 "사용하는 도구"가 아닌 "할당하는 리소스"로 관점 전환:
- 병렬 처리
- Specialized allocation
- Queue management
- Context efficiency

### "Distributed Cognition"

복잡한 문제를 여러 전문화된 Agent로 분산:
- 각 Agent의 context는 작고 명확
- 협력을 통해 전체 문제 해결
- 유지보수 용이

---

## 9. 실제 효과 및 메트릭

**Anthropic 내부 사례 (2025-2026)**:

- **코드 생성**: 100% AI 기반(Boris Cherny, 2025년 11월 이후 수동 편집 0)
- **Commit 기여도**: GitHub 전체 commit의 4% (2026년 말 20% 예상)
- **생산성**: 엔지니어당 200% 증가
- **선택 모델**: Opus 4.5 + thinking(품질 우선 철학)

---

## 10. 기반한 철학적 배경

### 복합성 관리
- 한 사람이 모든 컨텍스트를 기억할 수 없음
- 제도화(institutionalization)를 통한 확장성

### 인지 부하 최소화
- Task-specific context 설계
- 병렬 처리를 통한 부하 분산

### 신뢰와 검증 (Trust but Verify)
- "AI를 믿되, 계측하라"
- 자동 검증 루프 구축
- 제도적 안전장치

### 팀 학습 문화
- 개인 학습 → 팀 차원의 제도화 → 시스템 설계
- Meta-work(프로세스 개선)를 1급 시민으로 대우

---

## 참고: 관련 개념들

### CLAUDE.md와의 관계
- Memory 시스템(CLAUDE.md)은 Context Engineering의 핵심 도구
- 지속성(persistence)과 공유성(shareability)을 제공

### Prompt Engineering과의 보완
- Prompt Engineering: "이번 요청에 어떻게 말할까?"
- Context Engineering: "시스템 전체가 어떻게 작동하게 할까?"
- 실제로는 **함께 작동**: 좋은 context + 명확한 prompt = 최고 품질

### Agent Design과의 통합
- Context Engineering의 최종 형태: Specialized sub-agents
- 각 agent의 context는 그 agent의 responsibility 범위로 제한
- Fleet approach를 통한 scalability

