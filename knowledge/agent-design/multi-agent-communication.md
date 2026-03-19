# Multi-Agent Communication (에이전트 간 대화)

> 최종 업데이트: 2026-03-19 | 리서치 수행: 2026-03-19

## 1. Claude Code Agent Teams

**상태**: 실험적 기능 (Claude Code v2.1.32+)
**출처**: https://code.claude.com/docs/en/agent-teams (Anthropic 공식 문서, 신뢰도 최고)

### 핵심 개념

- **Team Lead + Teammates 모델**: 메인 Claude Code 세션이 팀을 생성하고 조율
- **Direct Messaging**: 팀원 간 직접 메시지 (팀 리드를 거치지 않음)
- **Shared Task List**: 모든 에이전트가 볼 수 있는 작업 칸반 보드
- **Mailbox**: 에이전트 간 메시징 시스템
- **Broadcast**: 모든 팀원에게 동시 메시지

### 활성화 방법

```json
// settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### 현재 제약

- Nested teams 불가 (팀 내 팀 X)
- 세션당 1개 팀만 관리
- 세션 재개 시 진행 중 팀원 복구 불가
- 토큰 비용 높음 (각 팀원이 독립 인스턴스)

### Context 모델 (공식 문서 확인)

> "When spawned, a teammate loads the **same project context** as a regular session: CLAUDE.md, MCP servers, and skills. It also receives the **spawn prompt** from the lead."

- **모든 팀원은 동일한 프로젝트 context를 로드** (같은 CLAUDE.md, 같은 MCP, 같은 skills)
- 팀원을 차별화할 수 있는 유일한 수단은 **spawn prompt** (리드가 전달하는 텍스트 지시)
- 팀원에게 별도 CLAUDE.md나 별도 persona를 주입하는 메커니즘은 **문서에 없음**
- 자연어 기반 상호작용 (프로그래밍 API 미제공)

### 통신 방식 상세

| 통신 방식 | 동작 | 비고 |
|---------|------|------|
| Direct Messaging | 특정 팀원에게 메시지 전송 | Mailbox가 자동 배송 |
| Broadcast | 모든 팀원에게 동시 메시지 | 팀 크기에 비례하여 토큰 비용 증가 |
| Shared Task List | 칸반 보드 (Pending → In Progress → Completed) | `~/.claude/tasks/{team-name}/`에 저장, 의존성 자동 관리 |
| Idle Notifications | 팀원 대기 상태 알림 | 작업 완료 후 자동으로 lead에 통보 |

### 유스케이스 적합성 평가 (중요)

**근본적 제약**: 단일 프로젝트 내 협업 모델 ↔ 전문가별 별도 프로젝트 구조 충돌

| 요소 | Agent Teams | 전문가 에이전트 구조 |
|------|-------------|-----------------|
| CLAUDE.md | 팀 전체 공유 (1개) | 전문가마다 별도 |
| memory/ | 프로젝트 공유 | 전문가마다 별도 |
| knowledge/ | 프로젝트 공유 | 전문가마다 별도 |

**결론**: 현 시점에서 Agent Teams로 서로 다른 전문가 에이전트 간 대화를 구현하기는 어려움. Spawn prompt에 전문가 context를 주입하는 우회가 가능하나, 토큰 비용과 구조화 문제 존재

---

## 2. Claude Code Subagent (현재 사용 중)

**상태**: 안정 기능 (기본 활성화)
**출처**: https://code.claude.com/docs/en/sub-agents (Anthropic 공식 문서, 신뢰도 최고)

### Hub-and-Spoke 모델

- 메인 에이전트 → Subagent → 결과 반환
- Subagent 간 직접 통신 불가
- 메인 에이전트가 중개자 역할 필수
- 현재 domain-researcher, expert-reviewer로 활용 중

### 한계

- 에이전트 간 "대화"가 아닌 "위임과 보고" 모델
- Subagent는 단방향 결과 반환만 가능 (멀티턴 대화 불가)

---

## 3. A2A (Agent-to-Agent) Protocol

**상태**: v1.0 정식 릴리스, Production-ready
**출처**: Google 주도, Linux Foundation 관리 (150+ 조직 참여)
- 공식 스펙: https://a2a-protocol.org/latest/specification/ (2026-03-19 리서치)
- GitHub: https://github.com/a2aproject/A2A (22.6k stars)

### 핵심 개념

- **Agent Card**: 에이전트의 기능/스킬/엔드포인트를 기술하는 JSON 메타데이터 (`.well-known/agent.json`)
- **Task**: 상태풀 작업 단위 (Submitted → Working → Completed/Failed)
- JSON-RPC 2.0 over HTTPS, SSE 스트리밍 지원

### MCP와의 관계 (상호보완)

| 프로토콜 | 역할 | 비유 |
|----------|------|------|
| MCP | 에이전트 ↔ 도구/데이터 (수직) | "직원이 도구를 사용" |
| A2A | 에이전트 ↔ 에이전트 (수평) | "직원끼리 대화" |

### Anthropic의 입장

- A2A 직접 개발에는 참여하지 않음
- Google Cloud와 공동 웨비나 (Vertex AI에서 MCP+A2A 통합)
- Agentic AI Foundation (AAIF)에 MCP 기증 (OpenAI, Block과 함께)

---

## 4. Multi-Agent 프레임워크 비교

| 프레임워크 | 개발사 | A2A 대화 방식 | Claude 호환 | Claude Code 통합 | 추천도 |
|-----------|--------|--------------|-----------|-----------------|--------|
| **Claude Code Agent Teams** | Anthropic | Direct messaging + task list | 100% | 100% (네이티브) | 사용자 목적에 최적 |
| **CrewAI** | CrewAI Inc. | Role-based + A2A protocol | O | 별도 통합 필요 | 범용 multi-agent에 최적 |
| **LangGraph** | LangChain | State sharing (supervisor) | O | 별도 통합 필요 | 토큰 효율에 최적 |
| **AutoGen** | Microsoft | Agent conversation | O | 별도 통합 필요 | Human-in-loop에 최적 |
| **Claude Agent SDK** | Anthropic | Subagent 병렬화 | 100% | SDK 레벨 | 프로그래밍 기반 오케스트레이션 |

**참고**: OpenClaw는 multi-agent이 아닌 단일 autonomous agent (multi-agent 대화 지원 안 함)

---

## 5. Anthropic 공식 Multi-Agent 사례

**출처**: https://www.anthropic.com/engineering/multi-agent-research-system (Anthropic Engineering, 신뢰도 최고, 2026-03-19 리서치)

- Lead Agent가 3-5개 Subagent를 병렬 생성
- Task-based communication (구조화된 작업 기반)
- 외부 메모리에 계획 저장하여 연속성 유지
- 복잡 쿼리 연구 시간 최대 90% 단축

---

## 6. 사용자 유스케이스 분석: PE Expert ↔ 전문가 에이전트 대화

**목표**: Prompt Engineering Expert와 생성된 전문가 에이전트 간 대화로 개선점 진단

### 접근법 1: Agent Teams 활용 (제약 있음)

```
[Prompt Engineering Expert (Team Lead)]
    ↕ Direct Messaging
[Expert Agent A (Teammate)] ↔ [Expert Agent B (Teammate)]
    ↕ Shared Task List
[진단 결과 → 구조 개선안]
```

- **장점**: Claude Code 네이티브, 직접 메시징, 공유 작업 목록
- **근본 제약**: 모든 팀원이 동일한 프로젝트 context를 로드 — 전문가별 별도 CLAUDE.md/memory/knowledge 불가
- **우회 가능성**: spawn prompt에 전문가 context 주입, 또는 Wrapper 프로젝트에서 팀원이 하위 프로젝트 파일을 Read로 로드 (미검증)
- **리스크**: 실험적 기능, 토큰 비용 높음

### 접근법 2: Subagent 확장 (현재 방식 강화)

```
[Prompt Engineering Expert (Main)]
    ├→ [Expert Agent A (Subagent)] → 진단 결과 반환
    ├→ [Expert Agent B (Subagent)] → 진단 결과 반환
    └→ Main이 결과 종합하여 개선안 도출
```

- **장점**: 이미 검증된 패턴, 안정적
- **한계**: 에이전트 간 직접 대화 불가, 메인이 모든 중개

### 접근법 3: 외부 프레임워크 (CrewAI/LangGraph)

- **장점**: 풍부한 A2A 기능, 유연한 오케스트레이션
- **한계**: Claude Code 런타임 외부에서 구동, 별도 설정/학습 필요, 기존 전문가 구조와 통합 복잡

### 권장안 (2026-03-19 기준)

1순위: **Subagent 확장** — 안정적, 검증된 패턴. 대화는 불가하지만 현실적으로 가장 실용적
2순위: **Agent Teams** — 직접 메시징 가능하나, 단일 프로젝트 context 제약으로 전문가별 개별 context 반영 어려움. 향후 기능 확장 시 재평가
3순위: **외부 프레임워크** — 풍부한 기능이지만 기존 구조와의 통합 비용이 높음

---

## 출처 및 신뢰도

| 출처 | 유형 | 신뢰도 |
|------|------|--------|
| Anthropic Agent Teams 공식 문서 | 공식 문서 | 최고 |
| Anthropic Sub-agents 공식 문서 | 공식 문서 | 최고 |
| Anthropic Multi-Agent Research System 블로그 | 공식 블로그 | 최고 |
| A2A Protocol 공식 스펙 (a2a-protocol.org) | 공식 스펙 | 최고 |
| A2A GitHub (22.6k stars) | 오픈소스 | 높음 |
| CrewAI / LangGraph / AutoGen 공식 문서 | 공식 문서 | 높음 |
