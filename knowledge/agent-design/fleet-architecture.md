# Fleet Architecture: 병렬 Agent 관리

**최종 업데이트**: 2026-03-15  
**출처**: Boris Cherny (Anthropic Claude Code creator), 인터뷰 및 사례  
**신뢰도**: 높음 (Anthropic 사내 실전 적용)

---

## 개념 정의

**Fleet Architecture**는 하나의 강력한 Agent를 운영하는 대신, **다수의 전문화된 경량 Agent를 병렬로 관리**하는 에이전트 설계 패턴입니다.

---

## 핵심 철학

### "One Heavy Brain vs Multiple Light Brains"

**기존 접근**: 하나의 Agent가 모든 컨텍스트를 유지
- 컨텍스트 윈도우 압박
- 인지 부하 집중
- 오류 누적 위험

**Fleet 접근**: 여러 전문화된 Agent를 병렬로 운영
- 각 Agent는 작은, 명확한 컨텍스트만 유지
- 역할별 특화
- 독립적 실패 격리

---

## Boris Cherny의 운영 사례

### 규모

```
동시 실행 세션: 약 10-15개

분포:
├─ Terminal: 5개 (탭화, 번호 지정, OS 알림 설정)
├─ Browser: 5-10개
└─ Mobile: 여러 개 (아침에 시작, 나중에 확인)
```

### 각 세션의 역할

| 세션 유형 | 목적 | Context | 특징 |
|----------|------|---------|------|
| Planning Session | 전략 수립, 요구사항 분석 | 프로덕트 철학, 기존 설계 | 반복 개선 |
| Implementation Session | 코드 작성 | 코딩 스타일, 아키텍처 | 자동 실행 모드 |
| Review Session | PR 리뷰, 검증 | 코드 컨벤션, 테스트 기준 | 피드백 제공 |
| Debug Session | 문제 해결 | 로그, 에러 스택 | 독립적 조사 |
| Testing Session | QA, 성능 측정 | 테스트 케이스, UX 가이드 | 병렬 검증 |

---

## 설계 패턴

### 1. Context Isolation (컨텍스트 격리)

```
전체 프로젝트 컨텍스트 1000 토큰
  ↓
각 Agent가 필요한 부분만: 100-200 토큰
  ↓
더 정확하고 효율적인 작업
```

**구현 원칙**:
- Agent A: 파일 X, Y만 알아야 함
- Agent B: 파일 Z, W만 알아야 함
- 공유 자산: CLAUDE.md (모든 Agent가 읽음)

### 2. Specialization (역할 특화)

각 Agent는 **하나의 책임**에 집중:

```
Agent: "나는 UI 컴포넌트를 작성한다"
├─ 필요한 지식: React 패턴, 스타일 가이드
├─ 불필요한 지식: 백엔드 API 세부사항
└─ 효율성: ↑ (관련 없는 정보로 방해받지 않음)

vs

Agent: "나는 모든 것을 한다"
├─ 필요: 전체 프로젝트 컨텍스트
├─ 부담: 매우 큼
└─ 정확도: ↓ (정보 과부하)
```

### 3. Orchestration (조율)

```
Human Orchestrator
├─ Task 1 → Agent A (Planning)
├─ Task 2 → Agent B (Implementation)
├─ Task 3 → Agent C (Verification)
└─ Task N+1 → Ready to execute

흐름:
Task A 완료 대기 (X)
→ 준비된 Task 실행 (O)
```

**효과**: 총 작업 시간 단축, 병렬성 증가

---

## 구체적 구현 사례

### 사례 1: Frontend 개발

```
Stage 1: Specification Agent
├─ 역할: "이 UI는 어떻게 생겼나?"
├─ Context: 디자인 가이드, 기존 컴포넌트
├─ Output: Detailed component spec
└─ CLAUDE.md: UI 컨벤션

Stage 2: Implementation Agent
├─ 역할: "코드 작성"
├─ Context: Stage 1 output + 코딩 스타일
├─ Output: React component
└─ CLAUDE.md: 코드 스타일

Stage 3: Verification Agent
├─ 역할: "이게 제대로 작동하나?"
├─ Context: 테스트 케이스, UX 기준
├─ Output: 버그 리포트 또는 승인
└─ CLAUDE.md: QA 기준
```

**병렬화**:
- Specification과 이전 단계의 Verification 동시 실행 가능

### 사례 2: API 개발

```
Agent Groups:

Group A (동시)
├─ Spec Agent: "API 스키마 설계"
└─ Doc Agent: "API 문서 작성"

Group B (A 완료 후)
├─ Implementation Agent: "코드 작성"
└─ Test Agent: "테스트 케이스 작성"

Group C (B 완료 후)
├─ Integration Agent: "프론트엔드 통합"
└─ Performance Agent: "성능 측정"
```

---

## 컨텍스트 관리 전략

### CLAUDE.md 공유

모든 Agent가 **동일한 CLAUDE.md**를 읽음:

```
CLAUDE.md 내용:
├─ 프로젝트 전반 컨벤션
├─ 공통 오류 방지 규칙
├─ 팀의 학습된 패턴
└─ 메타 작업: 지속적 개선

효과:
- 각 Agent는 팀의 누적 지식 상속
- 개별 세션의 실수가 다른 Agent를 갱신하지 않음
- 일관성 보장
```

### 세션별 Context Window

```
Agent A의 컨텍스트 구성:
├─ CLAUDE.md (공유): 10-20%
├─ Task-specific: 50-60%
├─ Conversation history: 20-30%
└─ 유휴: <10%

→ 효율적인 토큰 사용
```

---

## 스케줄링 및 할당

### "AI as Capacity"

AI를 tool이 아닌 **compute resource**로 취급:

```
Management Logic:
Task Queue
├─ Task A: 상태 = Ready → Agent 할당
├─ Task B: 상태 = Waiting on Task A → Queue에서 대기
├─ Task C: 상태 = Ready → Agent 할당 (B와 독립)
└─ Task D: 상태 = Ready → Agent 할당

병렬 실행:
A ████████░░░░
C ██░░░░░░░░░░
D ██░░░░░░░░░░
(B는 A 완료 대기, 그 동안 C, D 진행)

결과:
- 일렬 순서 vs 병렬: 총 시간 ↓↓
- 사람의 주의력도 효율적으로 할당
```

### 우선순위 관리

```
High Priority:
├─ 버그 수정 (블로킹 이슈)
├─ 성능 개선 (프로덕트 영향)

Medium Priority:
├─ 새 기능 구현
├─ 리팩토링

Low Priority:
├─ 문서 작성
├─ 테크 채무 상환
```

**Fleet 관점**: 우선순위에 따라 Agent 할당

---

## 이점 및 단점

### 장점

| 이점 | 설명 |
|-----|------|
| **Context efficiency** | 각 Agent의 compact context = 빠르고 정확한 처리 |
| **Parallelization** | 독립 Task 동시 실행 = 총 시간 단축 |
| **Specialization** | 역할별 최적화 = 높은 품질 |
| **Failure isolation** | 한 Agent의 오류가 다른 Agent에 영향 없음 |
| **Scalability** | 복잡도 증가해도 각 Agent는 compact |
| **Institutional learning** | CLAUDE.md 공유 = 팀 전체 학습 |

### 단점 및 주의사항

| 단점 | 완화책 |
|-----|------|
| **조율 복잡도** | 명확한 Task 정의, 의존성 맵핑 |
| **상태 동기화** | CLAUDE.md + 중앙 저장소(git, DB) |
| **오버헤드** | 세션 수 최적화, 작은 Task는 단일 Agent로 |
| **학습 곡선** | 체계적 설계 + 점진적 도입 |

---

## 구현 체크리스트

### Phase 1: 기초 설정

- [ ] CLAUDE.md 생성 및 초기화
- [ ] Task 분류 (어떤 작업들이 병렬화 가능한가?)
- [ ] Agent 역할 정의 (각 Agent의 책임은?)
- [ ] Context isolation 계획 (각 Agent는 무엇을 알아야 하는가?)

### Phase 2: 구현

- [ ] 각 Agent의 인스턴스 생성
- [ ] 공유 자산(CLAUDE.md, git repo) 설정
- [ ] 상태 추적 메커니즘 수립
- [ ] 간단한 Task로 시작 (2-3개 Agent)

### Phase 3: 최적화

- [ ] 성능 모니터링
- [ ] 피드백 루프 수립
- [ ] CLAUDE.md 지속적 업데이트
- [ ] Agent 수 및 역할 조정

---

## 실전 팁

### Tip 1: Session Naming

```
Good:
- "PRD-v2-review-2026-03-15"
- "backend-api-impl"
- "ui-testing-login-flow"

Bad:
- "work"
- "test"
- "claude"
```

→ Session 목적이 명확하면 context 관리 용이

### Tip 2: Handoff Protocol

```
Agent A → Agent B로 Task 이관할 때:

1. 명확한 Output 정의
2. 입력값 검증
3. CLAUDE.md 참조 (공유 규칙)
4. Context 최소화 (필요한 것만 전달)

Example:
"다음은 Agent B가 구현해야 할 API spec이다.
- Endpoint: /api/users/{id}
- Method: GET
- Response schema: {...}
- CLAUDE.md에 따라 에러 처리는 {...}"
```

### Tip 3: Verification Loop

각 Agent는 자신의 출력을 검증:

```
Implementation Agent:
1. 코드 작성
2. Linter 실행
3. 테스트 케이스 실행
4. 문제 있으면 재작업

→ 다음 Agent는 이미 검증된 입력 받음
```

---

## 참고: Boris Cherny의 실적

- **동시 세션**: 10-15개 유지
- **코드 생산성**: 100% AI 기반 (수동 편집 0%)
- **커밋 기여도**: GitHub 전체의 4% (2026년 말 20% 예상)
- **엔지니어 생산성**: 200% 증가

→ Fleet Architecture의 실효성 입증

