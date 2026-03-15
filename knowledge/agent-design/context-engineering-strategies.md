# Context Engineering Strategies

Anthropic의 context engineering 전략 및 최적화 기법입니다.

## 핵심 철학

**"Claude는 이미 충분히 똑똑하다. 문제는 context다."**

성공은 모델 능력이 아니라 정보 구조화에 달려있음. 한정된 context window에서 신호가 가장 강한 토큰의 최소 집합을 찾는 것이 목표.

**출처**: [Bojie Li - Context Engineering from Claude](https://01.me/en/2025/12/context-engineering-from-claude/)

## 4가지 주요 기둥

### 1. System Prompt
- 명확한 지시사항, 간단한 언어
- Goldilocks zone: 너무 구체 X, 너무 추상 X
- Role, Goal, Constraints, If unsure 패턴

### 2. Tools (도구)
- 독립적, 겹치지 않음, 목적 특화
- 각 도구가 존재를 정당화할 수 있어야 함
- 명확한 설명, 명시적 파라미터

### 3. Data Retrieval (검색)
- ❌ 모든 데이터를 미리 로딩 (RAG 전통)
- ✅ Just-in-time 검색: 경량 식별자 유지 → 런타임에 동적 로딩
- 인간의 인지와 유사: 필요할 때만 참조

### 4. Long-Horizon Optimization
- Compaction: Context 압축 후 재개
- Structured Note-Taking: Context 외부 지속 메모리
- Sub-Agent Architectures: 전문화된 에이전트에 위임

**출처**: [Anthropic - Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

## Context Rot 방지

### 문제
Context가 커질수록 품질 저하:
- Poisoning: 오래된/잘못된 정보가 계속 로딩
- Distraction: 관련 없는 정보 혼란
- Confusion: 모순되는 정보
- Contradictions: 일관성 부족

### 해결
- **신중한 큐레이션**: 불필요한 정보 제거
- **Just-in-Time Retrieval**: 필요한 것만 로딩
- **Progressive Disclosure**: 단계적 정보 제공

**출처**: [Anthropic - Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

## Just-in-Time Context Loading

### 전통적 RAG (문제점)
```
User Query
↓
Search Vector DB
↓
Load All Matching Documents
↓
Send to Claude
↓
(Context 낭비: 많은 불필요 정보)
```

### 개선된 패턴 (권장)
```
User Query
↓
Tools for Discovery (검색 도구)
↓
Claude reads what matters (필요한 것만)
↓
Dynamically navigates information space
↓
(Context 효율: 신호만 로드)
```

**구현**:
- Tools 제공: 사용자가 데이터 공간 탐색
- 데이터 dump X
- Claude가 필요한 정보 능동적으로 찾기

**출처**: [Bojie Li - Context Engineering from Claude](https://01.me/en/2025/12/context-engineering-from-claude/)

## Tool Design 원칙

### 1. Self-Contained (독립적)
- 각 tool이 standalone으로 작동
- 다른 tool 결과에 의존 X

### 2. Non-Overlapping (겹치지 않음)
- Tool 기능 중복 제거
- 명확한 책임 분리

### 3. Purpose-Specific (목적 특화)
- 각 tool이 구체적 목적 담당
- 일반적 catch-all tool 금지

### 원칙
**"Every tool must justify its existence"**

**출처**: [Anthropic - Writing Tools for Agents](https://www.anthropic.com/engineering/writing-tools-for-agents)

## Long-Horizon 최적화

### 1. Compaction
```
Long Session → Approaching Context Limit
↓
Summarize Conversation
↓
Create New Context with Summary
↓
Continue Work
↓
(성능 저하 최소화)
```

### 2. Structured Note-Taking
- Context 외부에 persistent memory 유지
- 에이전트가 학습 누적
- 다음 세션에서 참고

### 3. Sub-Agent Architectures
```
Complex Task
↓
Delegate to Specialized Sub-Agents
↓
Each runs with focused context
↓
Return Summarized Results
↓
(Main context 유지)
```

**출처**: [Anthropic - Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

## Progressive Disclosure 패턴

Information을 3단계로 로딩:

### Stage 1: 항상 로딩 (Minimal)
- System prompt
- Skill descriptions
- Task metadata

### Stage 2: Triggered (Conditional)
- Skill full content (호출 시)
- Relevant documentation
- Task-specific guides

### Stage 3: On-Demand (Lazy)
- Reference materials
- Examples
- Scripts (출력만 로딩)

**효과**: Context 효율 극대화, 필요 시에만 상세 정보 로드

**출처**: [Agent Skills - Progressive Disclosure](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

## 맥락 의식 (Context Awareness)

### Claude 4.6 기능
Claude가 자신의 남은 context window를 인식:
- 토큰 budget 추적
- 필요 시 메모리 저장
- Context 압박 시 자동 정보 오프로드

### 활용
```
System Prompt에 추가:
"Your context window will be automatically compacted 
as it approaches its limit, allowing you to continue 
working indefinitely. Save progress to memory before 
approaching the limit. Do not stop early due to 
token concerns."
```

**효과**: 장시간 에이전트 작업 지원

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

## MCP (Model Context Protocol)

### 목적
- External data/tools 연결
- Context engineering의 확장
- Just-in-time data retrieval 구현

### Just-in-Time 패턴
```
Agent needs external data
↓
MCP Tool Call
↓
Fetch from external system
↓
Only loaded result enters context
↓
(원본 데이터 pool은 context 외부)
```

**출처**: [Anthropic - Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

## 실전 예시

### Case 1: 코드 마이그레이션 (Spotify)
- 전통: 전체 코드베이스 로드
- 개선: Discovery tools (Grep, Glob) → 필요한 파일만 읽음
- 결과: 엔지니어링 시간 90% 단축

**출처**: [Anthropic - Context Engineering Blog](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

### Case 2: 장시간 작업
- Memory 체크: 시작 전 이전 학습 읽음
- Work: 작업 수행
- Update: 완료 후 메모리에 새 학습 저장
- Result: 매 세션마다 에이전트가 더 강해짐

**출처**: [Claude Code - Persistent Memory](https://code.claude.com/docs/en/sub-agents)

---

**신뢰도**: 공식 Anthropic 엔지니어링 문서 (최고)  
**최종 업데이트**: 2026-03-15
