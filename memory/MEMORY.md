# Prompt Engineering Expert Memory

## 프로젝트 현황

- **버전**: User Inputs 3체계 시스템 적용 완료 (2026-03-19)
- **구조**: create-expert + add-skill 2개 skill, shared/(템플릿 2개: agent-template, claude-md-template), create-expert/references/(Memory·Knowledge·User Inputs·Persona 가이드 4개), 2개 subagent(domain-researcher opus, expert-reviewer opus), knowledge/ 시스템 (3개 카테고리), user-inputs/ 시스템
- **최근 완료**: User Inputs 3체계 시스템 적용 — 기존 2체계(Memory+Knowledge)를 3체계(+User Inputs)로 확장. 8개 파일 수정/생성, 핵심 규칙 7→8가지, 모든 비교표·검증항목·템플릿 동기화 완료 (2026-03-19)
- **MCP 서버**: `.mcp.json`은 `${VAR}` 환경 변수 확장 사용 (API 키 분리, git 커밋 가능)
  - youtube-data (youtube-data-mcp-server) — 9개 Tool 중 7개 정상, 1개 부분(getTranscripts: 자막 존재 시만), 1개 실패(getRelatedVideos: YouTube API deprecated)
  - naver-search (@isnow890/naver-search-mcp) — 네이버 검색 11종 + DataLab 2종 + 유틸리티 1종. 환경 변수: `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET`
- **domain-researcher MCP 통합**: YouTube 4개 + 네이버 4개(blog, news, web, cafe) 도구 사용. YouTube/네이버 활용 가이드 포함. 네이버는 라이프스타일 분야 리서치에 특히 효과적
- **3체계 시스템**: Memory(경험 기록) + Knowledge(도메인 지식) + User Inputs(사용자 제공 원본 자료). 핵심 규칙 8가지 체제
- **생성된 전문가**: serial-entrepreneur-agent (초기 스타트업 아이디어 검증, 등급 A), zero-to-one-advisor (초기 스타트업 Zero-to-One 전문가, 등급 B→A 수정 완료)
- **피드백 반영 완료 (serial-entrepreneur-agent)**: 2건의 사용자 피드백 → 근본 원인 3개 파일에 재발 방지 가이드 추가 완료 (2026-03-16)
- **zero-to-one-advisor 생성 완료 (2026-03-16)**: 34개 파일. expert-reviewer 검증 후 필수 수정 1건 + 권장 개선 1건 반영 완료
- **zero-to-one-advisor 사용자 피드백 반영 완료 (2026-03-17)**: 3건 피드백 → 5개 파일 수정 (CLAUDE.md Sales Deck 역할 수정, sales-deck SKILL.md Pitch Deck 과도 강조 제거, 4개 skill description 차별화)
- **Part A/B 분리 적용 완료**: domain-researcher 결과물을 Part A (설계 요약) + Part B (원본 자료)로 분리. 서브에이전트 요약에 의한 정보 유실 방지 (2026-03-16)

## 다음 세션 할 일

- (없음)

## 핵심 교훈

### 최신 교훈

- 각 전문가는 별도 프로젝트이므로, subagent가 MCP 도구를 사용하면 해당 프로젝트에 `.mcp.json`이 필요하다. `.mcp.json`의 민감값은 `${VAR}` 환경 변수 확장으로 작성하여 git 커밋 가능하게 관리한다
- Knowledge 규칙 4번의 두 의무(사전 확인 + 사후 기록)는 독립적이다 — 하나만 지키면 되는 것이 아니라 둘 다 매번 지켜야 한다
- 세션 재시작이 필요한 작업 시, 반드시 미완료 작업을 MEMORY.md `## 다음 세션 할 일`에 기록한 후 종료해야 한다 (세션 핸드오프)
- 서브에이전트가 요약하여 반환하면 정보 유실이 발생한다. 원본 자료는 최대한 보존하여 반환하고, 소비자(create-expert)가 맥락에 맞게 가공해야 한다 (Part A/B 분리)
- Knowledge 카테고리명은 내용을 즉시 파악할 수 있도록 구체적으로 명명해야 한다 (`domain-specific` ✗ → `ai-simulation-qa` 등 ✓)
- 사용자의 용어를 자의적으로 해석하지 않는다. Sales Deck ≠ Pitch Deck — 범위를 정확히 확인하고 반영해야 한다
- 교훈의 핵심 원리를 이해하고, 예시에 과도 집중하지 않는다 — 예: "자의적 해석 금지"가 핵심이지, "Sales Deck vs Pitch Deck 구분을 강조하라"가 핵심이 아님
- skill description은 태그/키워드 나열이 아닌, 자연스러운 문장으로 각 skill의 핵심 가치와 트리거 조건을 명시적으로 서술한다
- 참조 파일 재배치: 실질적 소비자가 skill이면 해당 skill 하위 `references/`로 이동해야 한다 (루트 `references/` 삭제됨)
- SKILL.md에서 핵심 내용/원칙을 절차보다 먼저 제시해야 한다 — LLM이 "왜 이렇게 하는지" 모른 채 실행하는 것을 방지
- 인터뷰는 분야/역할/지식/멘토에 집중하고, 스타일/도구는 리서치 후 제안해야 한다
- 구조 제안(큰 틀) → 디테일 제안(내용) 순서로 Human Checkpoint를 분리해야 한다
- 범용 skill에서 독립 skill 분리 시 트리거 의도가 다르면 분리가 유리하다
- 공통 원칙은 개별 SKILL.md가 아닌 CLAUDE.md 핵심 규칙에 배치해야 모든 전문가에 적용된다
- 검증 기준의 Single Source of Truth는 expert-reviewer.md — 개별 skill의 6단계에 검증 항목을 나열하지 않는다
- Subagent 분리 기준: 소스 특성과 최신성 기준이 근본적으로 다르면 범용 subagent에서 전용으로 분리가 유리하다
- 중복 판단 기준: "이 파일이 없으면 knowledge/로 대체 가능한가?" → Yes면 삭제 대상
- Subagent memory 필드 미사용 — 메인 에이전트가 memory를 일원 관리, subagent는 결과 반환에만 집중

### 기본 원칙

- 전문가 에이전트는 Task 단위로 설계해야 한다 (E2E 금지)
- 핵심 작업은 skill 우선, subagent는 독립 실행이 필요한 작업에만 (Research, Review 등)
- 독립적으로 분리할 만큼 큰 기능은 별도 skill로 구분하여 구현한다
- SKILL.md 없는 디렉토리는 skill로 인식되지 않으므로 공유 리소스 저장소로 활용 가능
- 스킬 추가/분리 시 반드시 CLAUDE.md 핵심 명령어 섹션도 함께 업데이트해야 한다

### Memory/Knowledge 운영

- Memory 기록 순서: MEMORY.md → 하위 파일들(task-log, lessons-learned, user-preferences) → auto memory
- 프로젝트 memory/가 auto memory보다 항상 우선
- 프로젝트 `memory/MEMORY.md`는 자동 로딩되지 않으므로 대화 첫 응답 전 반드시 Read 필요
- Knowledge 시스템 도입 완료 (3개 카테고리: prompt-engineering, claude-code, agent-design)
- 출처에 원본 발행일과 리서치 수행 날짜를 반드시 포함해야 한다

### Prompt Engineering 핵심 원칙

- **System Prompt**: Goldilocks zone / 명확한 역할 부여 / 맥락 추가 (왜인지 설명)
- **Context Engineering**: "Claude는 이미 똑똑하다. 문제는 context다"
- **4대 기둥**: System prompt + Tools + Data Retrieval (just-in-time) + Long-horizon optimization
- **Progressive Disclosure**: 3단계 정보 로딩 (항상/triggered/on-demand)
- **Skill Design**: SKILL.md는 500줄 이내, supporting files 활용, reference vs task content 구분

## 자주 하는 실수

| 실수 | 해결법 |
|------|--------|
| Memory 기록 누락 (4회 반복) | 모든 작업 완료 후 반드시 Memory 업데이트 — MEMORY.md의 **최근 완료** 필드를 포함하여 task-log, lessons-learned 등 하위 파일도 빠짐없이 |
| 프로젝트 memory 우선순위 역전 (2회 반복) | 기록 순서: 프로젝트 memory/ 먼저 → auto memory 그 다음. 절대 뒤바꾸지 않는다 |
| 스킬 변경 시 CLAUDE.md 핵심 명령어 미동기화 | 스킬 추가/분리/삭제 시 반드시 CLAUDE.md 핵심 명령어 섹션 함께 업데이트 |
| Knowledge index.md topic 수 미갱신 | topic 파일 추가 시 반드시 루트 index.md의 topic 수도 함께 갱신 |
| 사용자 요청 처리 시 knowledge/ 미확인 **(2회 반복)** | 반드시 knowledge/를 먼저 확인한 후 작업을 시작한다. 확인 없이 작업을 시작하지 않는다 |
| 웹 서칭 후 knowledge 업데이트 누락 (1회) | 웹 서칭을 수행했으면, 작업 완료 전 반드시 knowledge topic을 생성/업데이트한다. 3조건(웹 서칭 수행 + 기존 카테고리 부합 + 새로운 내용) 중 하나라도 해당되면 필수 |
| 사용자 피드백 후 skill/subagent 미개선 (1회) | 핵심 규칙 5번: Memory에만 기록하지 않고, 원인이 된 skill/subagent 파일도 반드시 개선한다 |
| 세션 재시작 전 미완료 작업 미기록 (1회) | 세션 핸드오프: MEMORY.md `## 다음 세션 할 일`에 미완료 작업 + 배경 맥락을 기록한 후 종료한다 |
| 기존 전문가 선제 참조 (1회) | experts/ 내 기존 전문가는 사용자가 직접 비교/참조를 요청하지 않는 한 절대 먼저 참조 금지 |
| 전문가 생성 시 MCP 설정 미상속 (1회) | 각 전문가는 별도 프로젝트이므로, subagent가 `mcp__` 도구를 사용하면 해당 프로젝트에 `.mcp.json`이 필요하다. create-expert 5단계에서 확인 |

## 사용자 핵심 선호

- Memory 시스템을 "정말 핵심"으로 간주 → 모든 전문가에 반드시 포함
- Task 기반 접근 선호, 각 전문가는 별도 프로젝트
- 구체적 피드백을 주고받으며 점진적 개선하는 방식
- 근거 기반 + 비판적 사고 + 불확실성 명시를 최우선 원칙으로
- 전문 용어는 원문 표기 (skill, subagent 등), 대/소문자는 문맥에 따라 자유
- 파일 수정 시 기존 문체(~합니다 등) 반드시 유지
- Git Push 대상 프로젝트이므로 개인 정보(이름, 이메일 등) 파일에 포함 금지
- **기존 전문가 참조 금지**: experts/ 내 기존 전문가 에이전트를 사용자가 직접 비교/참조를 요청하지 않는 한 절대 먼저 참조하지 않는다 (정말 중요한 사안)

## 참조

- [task-log.md](task-log.md): 전체 작업 이력
- [lessons-learned.md](lessons-learned.md): 상세 학습 기록
- [user-preferences.md](user-preferences.md): 사용자 선호도
