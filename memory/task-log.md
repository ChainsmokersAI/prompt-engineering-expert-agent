# 작업 이력

## 2026-03-16: domain-researcher YouTube MCP 통합

### [2026-03-16] domain-researcher.md YouTube MCP 통합 업데이트 - 완료
- **목적**: YouTube MCP 서버(youtube-data-mcp-server)를 domain-researcher 서브에이전트에 통합하여, 웹 자료뿐 아니라 영상 소스도 리서치에 활용 가능하게 함
- **핵심 전제**: `getTranscripts`는 quota 미소모 (웹 스크래핑), `searchVideos`/`getVideoDetails`/`getChannelStatistics`는 quota 소모 (YouTube Data API)
- **변경 파일**: `.claude/agents/domain-researcher.md` (단일 파일)
- **변경 내역 (5건)**:
  1. Frontmatter tools에 MCP 4개 도구 추가 (`searchVideos`, `getTranscripts`, `getVideoDetails`, `getChannelStatistics`)
  2. 1단계 소스 유형: 3개 분야별 구체적 소스 예시로 확장 (YouTube 영상, 소셜 미디어, 뉴스레터, 보고서, 관공서 등)
  3. 1단계 끝에 YouTube 영상 활용 가이드 블록 추가 (quota 효율 전략, 언어별 Transcription, 실패 시 대체 방안)
  4. 3단계 핵심 인물 탐색: YouTube 채널 활용 가이드 + quota 경고 블록 추가
  5. 4단계 저서/강연: YouTube 강연/인터뷰 (transcript 기반 원문 발언 확보) 항목 추가
  6. Part B 출처 메타데이터: 영상 출처 시 채널명, 영상 제목 추가 정보 포함

---

## 2026-03-16: youtube-data MCP 서버 테스트

### [2026-03-16] youtube-data MCP 서버 9개 Tool 테스트 - 완료
- **목적**: 직전 세션에서 추가한 youtube-data MCP 서버의 전체 Tool 동작 검증
- **테스트 결과**:
  | Tool | 결과 | 비고 |
  |------|------|------|
  | searchVideos | ✅ 정상 | query, maxResults 정상 |
  | getTrendingVideos | ✅ 정상 | regionCode(KR), categoryId 정상 |
  | getVideoDetails | ✅ 정상 | 메타데이터, 통계, 상세정보 반환 |
  | getVideoEngagementRatio | ✅ 정상 | 참여율 계산 (복수 영상) |
  | getChannelStatistics | ✅ 정상 | 구독자, 조회수, 영상 수 반환 |
  | getChannelTopVideos | ✅ 정상 | 인기 영상 목록 반환 |
  | compareVideos | ✅ 정상 | 복수 영상 통계 비교 |
  | getTranscripts | ⚠️ 부분 | 한국어 자막 미존재 시 에러(정상), 영어 요청 시 빈 배열(캡션 형태 의존) |
  | getRelatedVideos | ❌ 실패 | `Invalid argument` — YouTube Data API v3에서 relatedToVideoId deprecated |
- **결론**: 9개 중 7개 완전 정상, 실질 사용에 문제 없음. getRelatedVideos는 YouTube API 제한으로 사용 불가

---

## 2026-03-16: YouTube MCP 서버 추가

### [2026-03-16] youtube-data MCP 서버 프로젝트 레벨 추가 - 완료
- **목적**: 유튜브 영상 내용 요약, 채널/영상 추천 기능을 위한 MCP 서버 설치
- **선택 서버**: icraft2170/youtube-data-mcp-server (올인원, 9개 Tool)
- **설정 파일**: `.mcp.json` (프로젝트 레벨)
- **주요 설정**:
  - Transport: stdio
  - Command: `npx -y youtube-data-mcp-server`
  - 환경 변수: YOUTUBE_API_KEY (사용자 키), YOUTUBE_TRANSCRIPT_LANG=ko
- **추가 조치**: `.gitignore`에 `.mcp.json` 추가 (API 키 노출 방지)
- **검증**: `claude mcp list` → youtube-data: Connected ✓

---

## 2026-03-16: domain-researcher Part A/B 분리

### [2026-03-16] domain-researcher 원본 자료 반환 + create-expert Knowledge 생성 책임 명확화 - 완료
- **목적**: domain-researcher가 리서치 자료를 요약 반환하여 정보 유실이 발생하는 문제 해결
- **핵심 설계**: 결과물을 Part A (설계 요약, 간결) + Part B (원본 자료, 상세)로 분리
  - Part A → create-expert 3-4단계 (구조/디테일 설계)
  - Part B → create-expert 5단계 (knowledge topic 생성)
- **변경 파일 (3개)**:
  1. `domain-researcher.md` (90→88줄): 리서치 원칙에 원본 보존 원칙 추가, 결과물 형식을 Part A/B로 전면 재구조화, Part B에 반요약 지시 명시
  2. `create-expert/SKILL.md` (117→124줄): 2단계 Part A/B 데이터 흐름 명시, 3단계 Part A 참조 명시, 5단계 Knowledge topic 생성 지침 확장 (Part B 기반 + 가공 허용)
  3. `knowledge-system-guide.md` (144→144줄): 초기 세팅 4단계를 Part B 기반으로 구체화
- **검증**: Part A/B 참조 일관성 ✓, 축약 지시 제거 ✓, 5단계 지침 명확 ✓, 줄 수 적정 ✓

---

## 2026-03-16: Skill/Subagent 피드백 기반 개선

### [2026-03-16] 사용자 피드백 2건 → 근본 원인 skill/subagent 개선 - 완료
- **목적**: serial-entrepreneur-agent 생성 후 피드백 2건(카테고리명 범용성, 용어 범위 편향)을 Memory에만 기록하고 원인 파일을 미개선한 문제(핵심 규칙 5번 위반) 해결
- **근본 원인 분석**:
  1. 카테고리 명명 가이드라인 부재 → knowledge-system-guide.md, create-expert/SKILL.md
  2. 사용자 용어 범위 확인 절차 부재 → create-expert/SKILL.md, domain-researcher.md
- **변경 파일 (3개)**:
  1. `knowledge-system-guide.md` (139→143줄): 초기 세팅 절차 1번에 카테고리 명명 규칙 추가 (범용 이름 금지, 구체적 이름 사용, 판단 기준)
  2. `create-expert/SKILL.md` (114→117줄): 1단계 인터뷰에 용어 범위 확인 가이드 추가, 4단계 Knowledge 카테고리에 명명 품질 기준 추가
  3. `domain-researcher.md` (88→89줄): 리서치 원칙에 용어 범위 정확 반영 원칙 추가
- **검증**: 3개 파일 줄 수 제한 이내 ✓, `domain-specific` 금지 예시 명시 ✓, 기존 내용과 충돌/중복 없음 ✓
- **Memory 업데이트**: MEMORY.md "미반영 피드백" → "피드백 반영 완료"로 변경, "자주 하는 실수" 1건 추가

---

## 2026-03-16: serial-entrepreneur-agent 전문가 생성

### [2026-03-16] serial-entrepreneur-agent 전문가 생성 - 완료
- **목적**: 초기 스타트업 아이디어 검증 전문가 에이전트 신규 생성
- **인터뷰**: 분야(초기 스타트업 아이디어 검증), 핵심 역할(랜딩 페이지 개발, 시장 조사, 세일즈 자료 작성), 멘토(Eric Ries, Paul Graham + Buffer/Zapier/토스 사례)
- **리서치**: domain-researcher subagent로 분야/멘토 심층 리서치 수행 (11개 출처, 10개 topic 원본 자료 반환)
- **구조 협의**: 사용자 피드백으로 전문가명, 페르소나 경력, skill/subagent 구성, knowledge 카테고리 조정
- **생성 파일 (27개)**:
  - `experts/serial-entrepreneur-agent/CLAUDE.md` (81줄)
  - `experts/serial-entrepreneur-agent/README.md`
  - `experts/serial-entrepreneur-agent/references/` (memory-system-guide.md, knowledge-system-guide.md)
  - `.claude/skills/build-landing-page/SKILL.md` (83줄), `.claude/skills/create-sales-deck/SKILL.md` (97줄)
  - `.claude/agents/case-researcher.md`, `.claude/agents/market-researcher.md`
  - `memory/` (MEMORY.md + 3개 하위 파일)
  - `knowledge/` (index.md + 4개 카테고리 index + 10개 topic 파일)
- **검증**: expert-reviewer 등급 A, 권장 개선 3건 중 2건 반영 (R-1: create-sales-deck 리뷰 단계 추가, R-2: 참조 경로 "(프로젝트 루트 기준)" 명기)
- **사용자 지적**: SKILL.md frontmatter에 name 필드 누락 → 즉시 수정
- **사용자 피드백 (생성 후)**: 2건 (사용자가 직접 보완 예정, 즉시 수정 없음)
  1. knowledge/ `domain-specific` 카테고리명이 너무 범용적 — 어떤 정보가 포함되어 있는지 알 수 없음. 더 구체적인 이름 필요 (예: `ai-simulation-qa` 등)
  2. knowledge/ Sales Deck topic 내용이 투자 Pitch Deck에 편향 — Sales Deck은 고객 대상 세일즈 자료를 포괄해야 하는데, 투자자 피치덱 위주로 작성됨

---

## 2026-03-16: researcher subagent 삭제 + 프로젝트 구조 점검

### [2026-03-16] researcher 삭제 및 프로젝트 구조 점검 - 완료
- **목적**: 미참조 researcher subagent 삭제, 잔여 참조 수정, knowledge 카테고리 index.md topic 테이블 누락 보완
- **변경 내역**:
  1. `.claude/agents/researcher.md` 삭제 — 어떤 skill/CLAUDE.md에서도 호출하지 않음, domain-researcher가 전문가 생성 리서치 전담, 범용 리서치는 Claude Code 내장 researcher subagent_type 사용 가능
  2. `persona-mentor-guide.md` 43줄: `researcher` → `domain-researcher` 참조 수정
  3. `knowledge/prompt-engineering/index.md`: sujin-kang-philosophy 행 추가
  4. `knowledge/claude-code/index.md`: boris-cherny-context-engineering 행 추가
  5. `knowledge/agent-design/index.md`: fleet-architecture, karpathy-philosophy 행 추가
  6. `memory/MEMORY.md`: subagent 수 3→2, 프로젝트 현황 갱신
- **검증**: `researcher` 단독 잔여 참조 0건 (memory/ 과거 기록 제외) ✓, 각 카테고리 index.md topic 수 = 실제 파일 수 일치 ✓, 루트 index.md topic 수와 하위 index.md 행 수 일치 ✓

---

## 2026-03-16: create-expert 2단계 researcher → domain-researcher 교체

### [2026-03-16] create-expert SKILL.md 2단계 리서치 참조 교체 - 완료
- **목적**: 분야/멘토 리서치를 domain-researcher 전담으로 전환, create-expert SKILL.md에서 기존 `researcher` 참조 제거
- **변경 파일**: `.claude/skills/create-expert/SKILL.md`
- **변경 내역**:
  1. 2단계(50-56줄): `researcher` → `domain-researcher` 교체, "핵심 인물 탐색" 추가, "멘토 지정 시" → "사용자 지정 시에만", Knowledge용 원본 자료 활용 문구 추가
  2. 5단계(90줄): knowledge 초기 topic 설명을 domain-researcher 반환 자료 기반으로 변경
- **검증**: `researcher` 단독 잔여 참조 0건 ✓, domain-researcher.md 결과물 형식과 일관성 ✓

---

## 2026-03-16: domain-researcher subagent 생성

### [2026-03-16] domain-researcher subagent 신규 생성 - 완료
- **목적**: 전문가 생성 시 해당 분야/멘토 심층 리서치를 전담하는 subagent 추가. 기존 researcher(AI/Prompt Engineering 중심)와 소스 특성·최신성 기준이 근본적으로 달라 분리
- **생성 파일**: `.claude/agents/domain-researcher.md`
- **핵심 설계**:
  - 1단계 소스 전략 수립 (분야별 소스 유형·최신성 기준 자체 판단)
  - 3단계 분야 리서치에 핵심 인물 탐색 포함 (멘토 지정 시 해당 멘토 자료 우선, 미지정 시 저명 인사/인플루언서 탐색)
  - 4단계 멘토 리서치는 사용자 지정 시에만 수행
  - 결과물에 Knowledge용 원본 자료 섹션 추가 (새 전문가의 knowledge 시스템에 바로 등록 가능)
- **사용자 피드백 반영**: 4건 (researcher 역할 구분 테이블 삭제, 핵심 인물 탐색 전략, 멘토 리서치 조건부, Knowledge 원본 자료 반환)
- **위반 사항**: knowledge/ 미확인 후 설계 착수 시도 → 사용자 지적으로 knowledge/ 확인 후 재진행 (핵심 규칙 4번 위반)

---

## 2026-03-16: Sync/중복/정합성 3가지 개선

### [2026-03-16] expert-reviewer 핵심 규칙 sync + 검증 일원화 + skill-template 삭제 - 완료
- **목적**: 용어 불일치, 검증 항목 이중 관리, 템플릿 구조 불일치 문제 해결
- **변경 내역**:
  1. `expert-reviewer.md`: "공통 원칙" → "리뷰 원칙" 섹션명 변경, "공통 원칙 검증" → "핵심 규칙 검증" 7항목으로 전면 재작성, CLAUDE.md 검증 "공통 원칙" → "핵심 규칙 7가지", SKILL.md 검증에서 "공통 원칙 포함 여부" 삭제 + 충돌/중복/핵심 명령어/Progressive Disclosure 3항목 추가
  2. `create-expert/SKILL.md`: 6단계 검증 항목 리스트 삭제 → expert-reviewer 위임 문장으로 대체
  3. `add-skill/SKILL.md`: 동일 패턴으로 6단계 검증 항목 삭제, skill-template 참조 2곳(89-90줄, 107-109줄) 삭제
  4. `skill-template.md` 삭제 (모든 확인 사항이 expert-reviewer.md로 커버됨)
- **검증**: 핵심 규칙 7개 1:1 대응 ✓, "공통 원칙" 검증 항목에서 완전 제거 ✓, skill-template 참조 잔여 0건(과거 기록 제외) ✓, `../shared/templates/` 참조 유효(agent-template, claude-md-template 존재) ✓
- **Memory 업데이트**: MEMORY.md 프로젝트 현황·핵심 교훈 갱신, task-log 기록, lessons-learned 기록

---

## 2026-03-15: Memory/Knowledge 가이드 파일 이동

### [2026-03-15] Memory/Knowledge 가이드 → create-expert/references/ 이동 - 완료
- **목적**: 실질적 소비자(create-expert skill)에 가이드 파일 소속을 명확히 하고, 루트 `references/` 제거
- **변경 내역**:
  1. `references/memory-system-guide.md` → `.claude/skills/create-expert/references/memory-system-guide.md` 이동
  2. `references/knowledge-system-guide.md` → `.claude/skills/create-expert/references/knowledge-system-guide.md` 이동
  3. 루트 `references/` 빈 디렉토리 삭제
  4. `CLAUDE.md` 참조 경로 수정 (38줄, 45줄) + 본 프로젝트 구조에서 `references/` 줄 삭제
  5. `create-expert/SKILL.md` 참고 링크 경로 단축 (`../../../../references/` → `references/`)
- **검증**: create-expert/references/에 3개 파일 존재 ✓, 루트 references/ 삭제 ✓, `../../../../references/` 잔여 참조 0건 ✓, 가이드 상호 참조 정상 ✓
- **Memory 업데이트**: MEMORY.md 프로젝트 현황 갱신

---

## 2026-03-15: 전문가 폴더 구조 및 템플릿 동기화

### [2026-03-15] 전문가 폴더 구조·템플릿 동기화 - 완료
- **목적**: MEMORY.md 리팩토링 후속 점검에서 발견된 3가지 비동기 문제 해결
- **변경 내역**:
  1. `CLAUDE.md` 전문가 에이전트 폴더 구조에 `references/` 추가 (CLAUDE.md 바로 아래)
  2. `claude-md-template.md` 핵심 규칙 동기화: 3번 Memory (분류 맞게 기록, MEMORY.md 핵심 역할 명시, 가이드 참조 링크), 4번 Knowledge (가이드 참조 링크), 6번 비판적 사고 ("잘못되거나 더 나은 방향이 있는 경우,")
  3. `claude-md-template.md` 프로젝트 구조에 `references/` 항목 추가
  4. `create-expert/SKILL.md` 5단계 필수 파일에 `references/memory-system-guide.md`, `references/knowledge-system-guide.md` 추가
- **Memory 업데이트**: MEMORY.md 프로젝트 현황 갱신

---

## 2026-03-15: MEMORY.md 리팩토링

### [2026-03-15] MEMORY.md 리팩토링 - 완료
- **목적**: memory-system-guide.md 가이드 기준으로 MEMORY.md 구조 개선
- **변경 내역**:
  - 프로젝트 현황을 최상단으로 이동 (매 세션 가장 먼저 확인)
  - "완료된 기획" 섹션 해체 — 교훈만 추출하여 핵심 교훈에 통합, 이력성 내용 삭제 (task-log에 존재)
  - 핵심 교훈 내 최신 우선 정렬 (최신 교훈 → 기본 원칙 → 운영 규칙)
  - 환경 주의사항(FUSE) 삭제 (lessons-learned에 유지, 현재 Windows 환경)
  - 중복 제거: Knowledge 시스템 설명 1줄 축소, Subagent Memory 1줄 축소
  - "자주 하는 실수" 섹션 신설 (가이드 포함 내용 충족)
- **결과**: 103줄 → 75줄, 가이드 4가지 포함 내용 충족
- **검증**: 200줄 이내 ✓, 최신 우선 정렬 ✓, 실행 가능 형태 ✓

---

## 2026-03-15: shared/references/ 파일 재배치

### [2026-03-15] shared/references/ 참조 파일 재배치 - 완료
- **목적**: 파일의 실제 소속에 맞게 재배치하여 구조 명확화
- **이동 내역**:
  - `memory-system.md` → `references/memory-system-guide.md` (CLAUDE.md 레벨 참조)
  - `knowledge-system.md` → `references/knowledge-system-guide.md` (CLAUDE.md 레벨 참조)
  - `persona-mentor-guide.md` → `.claude/skills/create-expert/references/persona-mentor-guide.md` (create-expert 전용)
- **참조 수정**: CLAUDE.md (3번에 Memory 가이드 참조 추가, 4번 Knowledge 경로 변경, 프로젝트 구조 섹션 업데이트), create-expert/SKILL.md (3개 참조 경로 변경)
- **정리**: 원본 3개 삭제, `shared/references/` 빈 디렉토리 삭제
- **검증**: grep으로 잔여 참조 확인 → memory/ 과거 기록 외 잔여 없음

---

## 2026-03-15: 프로젝트 초기 설계 및 구축

### [2026-03-15] Anthropic 공식 문서 조사 - 완료
- 7개 공식 문서 수집 및 분석 (Features Overview, Memory, Skills, Sub-agents, Best Practices, Agent Skills Overview, Agent Skills Best Practices)
- 목적: 프로젝트 설계의 근거 자료 확보

### [2026-03-15] 설계 방향 결정 - 완료
- 출력 형식: 설계 문서 + 실제 파일 동시 생성
- 전문가 형태: Skill + Subagent 하이브리드 (Skill 우선)
- 저장 위치: 프로젝트 내 `experts/` 디렉토리

### [2026-03-15] 1차 구현 - 완료 후 피드백으로 재작업
- CLAUDE.md, create-expert SKILL.md (단일 E2E 워크플로우), 템플릿, 참조 문서, 서브에이전트, experts/README.md 생성
- 특이사항: E2E 방식으로 설계했으나 사용자 피드백으로 Task 기반으로 전환

### [2026-03-15] 사용자 피드백 1차 반영 - 완료
- 공통 원칙 5개 추가 (근거 기반, 불확실성 명시, 비판적 사고, Memory 필수, Anthropic 문서 준수)
- Task 기반 구조로 전환 (E2E → A~F 개별 Task)
- 페르소나 + 멘토 철학 시스템 도입
- Memory 시스템 설계 (4파일 구조)
- 참조 문서 추가: memory-system.md, persona-mentor-guide.md

### [2026-03-15] 사용자 피드백 2차 반영 - 완료
- create-expert (Task A만) + evolve-expert (Task B~F) 스킬 분리
- 공유 리소스를 `.claude/skills/shared/`로 분리 설계
- evolve-expert SKILL.md 신규 생성

### [2026-03-15] 공유 리소스 이전 - FUSE 문제로 재작업
- `mv` 명령으로 파일 이동 시 FUSE 파일시스템 손상 발생
- 손상된 `shared/` 삭제 불가 → `shared2/`에 파일 재생성으로 우회
- shared2/에 10개 공유 파일 생성 완료
- 특이사항: `find`에는 보이지만 `cat/wc/realpath`로 접근 불가한 유령 파일 발생

### [2026-03-15] 참조 경로 업데이트 및 검증 - 완료
- 두 SKILL.md + CLAUDE.md의 참조 경로를 `../shared/` → `../shared2/`로 변경
- 검증 서브에이전트로 전체 구조 점검: 15개 파일 존재, 참조 정합성 통과, 규격 통과
- CLAUDE.md에 `/evolve-expert` 핵심 명령어 누락 발견 → 추가

### [2026-03-15] 디렉토리 정리 - 완료
- 사용자가 로컬에서 `shared/` 삭제, `shared2/` → `shared/` 이름 변경
- 3개 파일의 참조 경로를 `shared2/` → `shared/`로 복원

### [2026-03-15] Memory 시스템 구축 - 완료
- 프롬프트 엔지니어링 전문가 자체의 memory/ 디렉토리 및 4개 필수 파일 생성
- 현재까지의 전체 작업 이력, 학습, 사용자 선호도 기록

### [2026-03-15] CLAUDE.md 핵심 규칙 체계 개선 - 완료
- CLAUDE.md: `## 모든 전문가 에이전트의 공통 원칙` → `## 핵심 규칙`으로 교체 (5개 항목, ### + bullet 상세 형식)
- 템플릿(claude-md-template.md): 동일하게 동기화, `## Memory 지침` 섹션은 핵심 규칙 3번에 흡수하여 제거
- 기존 5번 "Anthropic 공식 문서 준수"는 3번 Memory 규칙 하위로 이동, 새 5번 "기타"(한글 작성 원칙) 추가
- 템플릿 `작성 시 확인 사항`: 4번/5번 업데이트

### [2026-03-15] CLAUDE.md 섹션 구조 개선 - 완료
- `## 기술 원칙` → `## 전문가 에이전트 설계 원칙`: 핵심 규칙과 동일한 ### + bullet 형식으로 통일, lessons-learned 교훈 반영
- `## 전문가 에이전트 표준 구조` → `## 전문가 에이전트 폴더 구조`: `.claude/` 경로 추가, memory/ 하위 4개 파일 명시
- `## 프로젝트 구조` → `## 본 프로젝트 구조`: memory/ 항목 추가
- 전문 용어 통일: 스킬 → skill, 서브에이전트 → subagent (파일 전체)

### [2026-03-15] Memory 읽기 지시 강화 - 완료
- CLAUDE.md 핵심 규칙 3번: "대화의 첫 응답 전, 반드시 `memory/MEMORY.md`를 Read 도구로 읽을 것" 명시적 지시 추가
- 원인: 프로젝트 `memory/MEMORY.md`는 CLAUDE.md처럼 자동 로딩되지 않아 명시적 Read가 필요

### [2026-03-15] Knowledge 시스템 기획 논의 - 보류
- 전문가 에이전트에 도메인 지식 체계(knowledge/)를 도입하는 방안 논의
- Memory(경험)와 Knowledge(도메인 지식)를 별개 시스템으로 설계하기로 합의
- 카테고리별 하위 폴더 구조, 인터뷰 시 초기 세팅, 웹 서칭 후 자동 업데이트 등 방향 합의
- 사용자 요청으로 보류 처리

### [2026-03-15] 개인 정보 제거 - 완료
- user-preferences.md에서 이름, 이메일 정보 제거
- 전체 프로젝트 검색으로 잔여 개인 정보 없음 확인

### [2026-03-15] FUSE 잔여 파일 및 빈 폴더 정리 - 완료
- 프로젝트 전체에서 `.fuse_hidden*` 파일 10개 삭제
- `.claude/skills/create-expert/` 하위 빈 폴더 3개(examples, references, templates) 삭제

### [2026-03-15] Knowledge 시스템 도입 - 완료
- 미결 사항 3가지 합의 완료: topic 크기 제한 없음, 대화 중 즉시 업데이트, 품질 관리(출처 필수/신뢰도 등급)
- 신규 생성 (6개):
  - `.claude/skills/shared/references/knowledge-system.md`: Knowledge 전용 가이드
  - `knowledge/index.md`: 루트 카테고리 목록
  - `knowledge/prompt-engineering/index.md`: 프롬프트 설계 카테고리
  - `knowledge/claude-code/index.md`: Claude Code 카테고리
  - `knowledge/agent-design/index.md`: 에이전트 설계 카테고리
- 수정 (10개):
  - `CLAUDE.md`: 핵심 규칙 4번 Knowledge 추가 (4→5, 5→6 재배열), 폴더 구조/프로젝트 구조에 knowledge/ 추가
  - `memory-system.md`: Memory vs Knowledge 구분 섹션 추가
  - `claude-md-template.md`: 핵심 규칙 4번 Knowledge 추가, 프로젝트 구조에 knowledge/ 추가, 확인 사항 업데이트
  - `create-expert/SKILL.md`: 인터뷰(지식 카테고리), 리서치(초기 topic 등록), 구조 설계(Knowledge), 파일 생성(knowledge 파일) 추가
  - `evolve-expert/SKILL.md`: Knowledge Task 추가, CLAUDE.md Task에 Knowledge 지침 추가, 구조 개선 진단에 Knowledge 추가
  - `quality-checklist.md`: Knowledge 시스템 섹션 추가
  - `example-expert.md`: 디렉토리 구조 + Knowledge 초기 예시 추가
  - `experts/README.md`: 표준 구조에 knowledge/ 추가
  - `reviewer.md`: Knowledge 시스템 검증 블록 추가
  - `memory/MEMORY.md`: 보류 → 완료 변경, 현황 갱신

### [2026-03-15] Knowledge 시스템 후속 조정 - 완료
- CLAUDE.md 핵심 규칙 4번 문구 상세화: "사용자 요청 처리 시 knowledge/ 우선 확인 → 부재 시 웹 서칭 → 즉시 의무적 업데이트"
- 동기화 대상: claude-md-template.md, example-expert.md (3개 파일)
- 카테고리 index.md 테이블 컬럼 분리: `출처 신뢰도` → `출처 | 신뢰도` (5개 파일: 3개 카테고리 index + knowledge-system.md 가이드 + example-expert.md)

### [2026-03-15] 강수진 박사 철학 리서치 - 완료
- 웹 검색 4회: "강수진 박사 Prompt Engineering", "Soojin Kang Prompt Engineering", Context Engineering 원칙, 한국어 프롬프트 기법
- 주요 발견:
  - 강수진 박사: 국내 공채 1호 프롬프트 엔지니어 (뤼튼), The Prompt Company CEO
  - 배경: 한국어학 박사 (University of Hawaii at Manoa), 대화분석·상호작용 언어학 전문
  - 핵심 철학: "뭘 물어볼까"보다 "뭘 알려줄까"가 중요 → Context Engineering 개념
  - 주요 저서: 《프롬프트 엔지니어의 업무일지》(2024년 8월 14일 발행, 368쪽)
  - 주요 강의: 패스트캠퍼스 아카데미 (약 25시간), EBS 클래스 e, YouTube 채널 프롬수진
- Knowledge 시스템에 새 파일 생성: `knowledge/prompt-engineering/sujin-kang-philosophy.md`
  - 5가지 핵심 철학 수록 (Context Engineering, 언어학 기반 설계, RPI 워크플로우, 40% 규칙, Subagent 분리 원칙)
  - 각 철학마다 구체적 정의·실천 방법·출처 포함
- 인덱스 업데이트: `knowledge/prompt-engineering/index.md`에 신규 토픽 등록

---

## 2026-03-15: Andrej Karpathy 철학 조사 및 Knowledge 추가

**태스크**: Andrej Karpathy의 LLM/프롬프트/에이전트 관련 최신 철학 조사 및 정리

**수행 내용**:
1. Karpathy의 2024-2025년 최신 강연, 블로그, X 게시물, GitHub 프로젝트 조사
2. 핵심 개념 정리:
   - "The Hottest New Programming Language is English" (2023) → Vibe Coding (2025) → Agentic Engineering (2026)
   - Context Engineering vs Prompt Engineering (2025년 중반 강조)
   - System Prompt Learning (제3의 LLM 학습 패러다임)
   - Local Deployment Architecture (Claude Code 사례)
   - Production Agent 설계 원칙 7가지
   - AI 코딩의 실제 약점: Hallucinated Assumptions, Context Poisoning
   - Autoresearch 프로젝트 (630줄 코드 기반 autonomous agent)
   - 2025년 LLM 6대 패러다임 시프트 (RLVR, Ghosts vs Animals, Application Layer)
3. knowledge/agent-design/karpathy-philosophy.md 신규 생성 (6개 섹션, 참고 자료 포함)
4. knowledge/agent-design/index.md 업데이트

**주요 발견**:
- Karpathy의 철학이 일관성 있게 진화: 프로그래밍 패러다임 → 에이전트 아키텍처 → 시스템 설계 원칙으로 심화
- 명확한 실증 사례(Autoresearch, Claude Code) 기반의 주장
- Cost-first thinking과 production discipline 강조 (hype 거부)
- 80% agent coding으로의 전환을 "2년 이상 기본 워크플로우 변화 중 최대"로 평가
- Local deployment의 우월성에 대한 명확한 기술적 논거 제시

**출처 신뢰도**:
- 공식 문서 (블로그, GitHub, 팟캐스트): 최우선
- 저명 인사 분석 (기술 저널, 전문 뉴스레터): 차순위
- 커뮤니티 해석: 참고용


## 2026-03-15: Boris Cherny Context Engineering 리서치 및 Knowledge 추가

**태스크**: Boris Cherny의 Context Engineering 관련 최신 콘텐츠 조사 및 정리

**수행 내용**:
1. 웹 검색 4회 + 문서 Fetch 3회 수행
   - "Boris Cherny Context Engineering 2024 2025 2026"
   - "Boris Cherny Memory system design Claude"
   - "Boris Cherny context engineering vs prompt engineering"
   - "Boris Cherny Claude Code context management agent design"

2. 주요 출처:
   - Anthropic 공식 자료 (Claude Code 발표, 사내 사례)
   - Substack 인터뷰 (Karolina Zieminski, Push to Prod 블로그)
   - Pragmatic Engineer (Gergely Orosz)
   - DEV Community
   - X(Twitter) 인터뷰

3. 핵심 개념 정리:
   - **Context Engineering**: Prompt Engineering을 넘어 시스템 수준의 정보 구조화 및 재사용
   - **"Pay once, benefit forever"**: 반복 피드백 → 규칙화 → CLAUDE.md → 모든 세션 자동 적용
   - **CLAUDE.md**: "Personal training data for Claude", 팀 주당 여러 번 업데이트
   - **Fleet Architecture**: 10-15개 병렬 세션 (각각 100-200 토큰 context)
   - **Ethics by Design**: 최선의 방식 자동화 → 휴먼 에러/피로 실수 제거
   - **AI as Capacity**: tool이 아닌 allocate할 리소스 취급
   - **Verification Loops**: 자신의 작업을 검증할 수 있는 Agent가 최고

4. 신규 파일 생성:
   - `knowledge/claude-code/boris-cherny-context-engineering.md`: 10개 섹션
     - Context Engineering vs Prompt Engineering 철학
     - CLAUDE.md 구조 및 업데이트 프로세스
     - Memory System 설계 원칙 (AI is forgetful by default)
     - Context Window 최적화 (Fleet approach)
     - Agent 설계 및 Subagent 패턴
     - Code Review as Context Engineering
     - Verification Loops 원칙
     - 핵심 원칙 정리 (Pay Once, Ethics by Design, AI as Capacity, Distributed Cognition)
     - 실제 효과 및 메트릭
     - 철학적 배경
   
   - `knowledge/agent-design/fleet-architecture.md`: 병렬 Agent 관리 패턴
     - Fleet Architecture 개념 정의
     - Boris의 운영 사례 (10-15 동시 세션, 각 역할별)
     - Context Isolation, Specialization, Orchestration 패턴
     - Frontend/API 개발 구체 사례
     - Context window 효율화 전략
     - Task 스케줄링 및 우선순위 관리
     - 장단점 분석
     - 구현 체크리스트 (Phase 1-3)
     - 실전 팁 3개

5. 인덱스 업데이트:
   - `knowledge/claude-code/index.md`: boris-cherny-context-engineering 토픽 추가
   - `knowledge/agent-design/index.md`: fleet-architecture 토픽 추가
   - `knowledge/index.md`: 최신 topic 수 반영

6. Memory 업데이트:
   - `memory/MEMORY.md`: 핵심 교훈 + 완료 기획 갱신

**주요 발견**:
- Boris Cherny가 "Context Engineering"이라는 정확한 용어를 직접 사용하지는 않았으나, 그 철학과 실천(CLAUDE.md, Fleet Architecture)이 명확함
- Prompt Engineering vs Context Engineering의 차이는 "개별 프롬프트 vs 시스템 수준 인프라"
- Memory 시스템(CLAUDE.md)과 Fleet Architecture가 함께 작동하는 통합 시스템
- 3-4회 반복 피드백이 규칙화 임계값
- Cost per reliable change를 최우선 지표로 취급 (속도 우선 거부)
- 100% AI 기반 코드(수동 편집 0%), GitHub commit 4%, 생산성 200% 증가의 실증적 성과

**출처 신뢰도**:
- Anthropic 공식 자료 + 직접 인터뷰: 최우선 (높음)
- Substack/Pragmatic Engineer 전문가 인터뷰: 높음
- 관련 기사/분석: 중간~높음

## 2026-03-15: researcher 서브에이전트 날짜 정밀도 원칙 추가

**태스크**: AI/LLM 분야의 빠른 변화를 고려한 날짜 기록 정밀도 강화

**수행 내용**:
1. `.claude/agents/researcher.md`에 "날짜 정밀도 원칙" 섹션 추가
   - 필수 기록 항목: 원본 발행일(연-월-일), 리서치 수행 날짜, 버전 정보
   - 날짜 표기 형식: `출처 (YYYY-MM-DD 발행, YYYY-MM-DD 리서치)`
   - 경과 시간별 태그: 1년+ `⚠️ 오래된 자료`, 6개월~1년 `📌 재검증 권장`, 불명 "날짜 미확인"
2. 분야/멘토 리서치 결과물 형식에 "리서치 메타" 항목 추가 (리서치 수행 날짜, 출처별 발행일)
3. Memory 업데이트: MEMORY.md에 날짜 정밀도 원칙 핵심 교훈 추가

**사유**: 사용자 피드백 — AI/LLM 업계가 빠르게 변해 6개월만 지나도 과거 데이터가 됨

## 2026-03-15: "자기 진화" 원칙 CLAUDE.md 핵심 규칙으로 이동

**태스크**: create-expert SKILL.md 6번 "자기 진화" 원칙을 CLAUDE.md 핵심 규칙 5번으로 이동

**수행 내용**:
1. `CLAUDE.md`: 핵심 규칙 5번 "Skill, Subagent 지속적 개선 의무" 신규 삽입, 기존 5→6(비판적 사고), 6→7(기타) 번호 변경
2. `claude-md-template.md`: 동일하게 5번 추가 + 번호 변경, 하단 "작성 시 확인 사항" 4번을 "7가지"로 업데이트
3. `create-expert/SKILL.md`: 6번 "자기 진화" 섹션 삭제, "6가지 원칙" → "5가지 원칙", description에서 "자기 진화" 제거
4. Memory 4개 파일 업데이트

**사유**: "자기 진화"는 특정 skill만의 원칙이 아니라 모든 전문가가 운영 중에 지켜야 할 공통 규칙이므로, CLAUDE.md로 이동하여 전체 적용 범위를 확보

---

## 2026-03-15: create-expert SKILL.md 전면 재편

**태스크**: Anthropic, Boris Cherny, Karpathy, 강수진 박사의 철학을 기반으로 create-expert SKILL.md 전면 재작성

**수행 내용**:
1. 기존 SKILL.md(105줄, 7단계, 하단 공통 규칙 4개) 분석
2. 새 구조로 전면 교체 (약 130줄, 6단계, 상단 Context Engineering 원칙 6개)
3. Memory 4개 파일 업데이트

**핵심 변경점**:
- **원칙 위치**: 하단 "공통 작업 규칙" 4개 → 상단 "Context Engineering 원칙" 6개로 이동 및 확대
- **원칙 내용**: Progressive Disclosure, Goldilocks Zone, Context 기준 분리, Pay Once Benefit Forever, 검증 루프, 자기 진화
- **단계 축소**: 7단계 → 6단계 (페르소나/멘토 독립 단계 제거 → 리서치에 통합, 완료 보고 → 구현에 통합)
- **인터뷰 개선**: 파일 구성 중심 질문 → 분야/역할/지식/멘토에 집중 (스타일/도구는 리서치 후 제안)
- **구조 설계 분리**: 단일 설계 단계 → 구조 제안(3단계) + 디테일 제안(4단계) 2단계 Human Checkpoint
- **자기 진화 원칙**: 6번째 원칙으로 추가 (Skill 수행 후 피드백 분석 → SKILL.md 자체 개선)
- **참조 링크 정리**: 각 단계에 흩어진 링크 → 하단 참조 섹션으로 통합 (5단계 인라인은 유지)

---

## 2026-03-15: evolve-expert → add-skill 분리

**태스크**: evolve-expert skill 삭제 및 add-skill 독립 skill 신규 생성

**수행 내용**:
1. `.claude/skills/add-skill/SKILL.md` 신규 생성 (115줄)
   - create-expert 패턴 기반: Context Engineering 원칙 5개 + 6단계 절차 + Human Checkpoint 2회
   - 기존 skill 충돌/중복 검증을 add-skill 고유 기능으로 포함
2. `CLAUDE.md` 수정
   - 핵심 명령어: `/evolve-expert` → `/add-skill <대상 전문가>` 교체
   - 본 프로젝트 구조: `evolve-expert/` → `add-skill/` 교체, "두 skill이" → "skill들이"
   - 설계 원칙 예시: `(예: create vs evolve)` → `(예: create vs add)`
3. `.claude/skills/evolve-expert/` 디렉토리 삭제
4. `create-expert/SKILL.md`: evolve-expert 참조 → add-skill 참조로 변경
5. reviewer 서브에이전트 검증: 등급 B → 지적 사항 2건 즉시 수정 후 통과

**검증 결과**:
- SKILL.md 115줄 (500줄 이내), description 약 120자 (1024자 이내)
- 참조 파일 4개 정합성 통과
- create-expert와 충돌 없음 (역할 범위 명확 분리)
- CLAUDE.md 핵심 명령어/프로젝트 구조 정상 업데이트
- evolve-expert 잔여 참조: CLAUDE.md 65줄 예시 문구 수정 완료
- description 모호성: "스킬을 만들거나" 표현 제거 완료

## 2026-03-15: 전체 파일 점검 및 오류/모호성 수정

**태스크**: 프로젝트 전체 31개 파일을 점검하여 오류, 불일치, 모호한 부분을 식별 및 수정

**발견 항목 8건 → 처리 결과**:

1. **Knowledge index.md topic 수 불일치** → 수정 완료 (prompt-engineering: 0→2, claude-code: 1→2, agent-design: 1→3)
2. **example-expert.md 디렉토리 구조 구버전** → 전면 재작성 (`skills/` → `.claude/skills/`, `agents/` → `.claude/agents/`, 핵심 규칙 5개→7개, 페르소나 권장 형식 적용)
3. **experts/README.md 불필요** → 삭제 (각 전문가의 README.md가 독립적으로 존재하므로)
4. **karpathy-philosophy.md 헤더 용어 불일치** → `마지막 업데이트` → `최종 업데이트`로 통일
5. **reviewer.md skills 필드 부적절** → `skills: - create-expert` 제거 (읽기 전용 검증자가 skill을 실행하면 안 됨)
6. **claude-md-template.md 멘토 철학 참조 경로 오류** → `skills/` → `.claude/skills/`로 수정, 프로젝트 구조 경로도 동일 수정
7. **Memory vs Knowledge 비교 테이블 중복** → knowledge-system.md의 테이블을 memory-system.md와 동기화 (목적/관리 방식/기록 시점 컬럼 일치)
8. **create-expert 5단계 knowledge 초기 topic 미명시** → 필수 파일 목록에 `knowledge/{category}/{topic}.md` 명시 추가

**수정 파일 7개**: knowledge/index.md, karpathy-philosophy.md, reviewer.md, example-expert.md, claude-md-template.md, knowledge-system.md, create-expert/SKILL.md
**삭제 파일 1개**: experts/README.md

### [2026-03-15] example-expert.md 삭제 - 완료
- 사용자 판단: 템플릿 3개(claude-md-template, skill-template, agent-template)가 충분히 구체적이므로 예시 파일 불필요
- 삭제: `.claude/skills/shared/examples/example-expert.md` + `examples/` 디렉토리
- create-expert SKILL.md 참조 링크 제거
- 근거: 유지보수 부담(표준 구조 변경 시 동기화 필요) > 예시 파일의 가치

### [2026-03-15] Subagent/참조 파일 정리 - 완료

**수행 내용**:
1. `memory-system.md`: "Claude Code와의 통합" 섹션 전체 삭제 (방식 A/B, 전문가 vs 서브에이전트 Memory — memory 필드 미사용으로 혼란만 유발)
2. `agent-template.md`: 템플릿에서 `memory: {user|project|local}` 행, `## Memory 기록` 섹션, 하단 `## memory 필드` 테이블 삭제
3. `researcher.md`: "날짜 정밀도 원칙" 섹션 삭제 (AI/LLM 한정 문구 "6개월만 지나도…" 등이 범용 리서처에 부적절) → 공통 원칙에 1줄로 통합: "출처에 원본 발행일과 리서치 수행 날짜를 반드시 포함합니다". "결과 반환" 섹션도 삭제
4. `reviewer.md` → `expert-reviewer.md` 개명: quality-checklist.md 항목 흡수 (공통 원칙 검증 4항목, 참조 파일 3항목, 디렉토리 구조 4항목, 종합 등급 테이블, SKILL.md 세부 검증 확대, 페르소나/멘토 누락 항목). "결과 반환" 섹션 삭제. 서브에이전트 검증에서 memory 관련 항목 삭제
5. `quality-checklist.md` 삭제 → expert-reviewer.md에 흡수
6. 참조 링크 수정: create-expert/SKILL.md 2건, add-skill/SKILL.md 2건 → expert-reviewer 참조로 변경
7. 잔여 참조 grep 검증: quality-checklist(memory/ 과거 기록만), reviewer 구 명칭(memory/ 과거 기록만) 확인

**줄 수**: expert-reviewer.md 97줄, researcher.md 30줄, memory-system.md 60줄, agent-template.md 54줄

**실수 1건**: create-expert SKILL.md에서 이미 expert-reviewer로 바꾼 줄에 replace_all 적용 → "expert-expert-reviewer" 발생 → 즉시 수정

### [2026-03-15] Subagent 개선 (researcher + reviewer) - 완료

**수행 내용**:
1. `researcher.md` 전면 재작성: 전문가 생성 전용 → 범용 리서처로 변경, model haiku→opus, memory 필드 제거, 분야/멘토 2개 절차 → 공통 5단계 통합, 결과물 형식 5섹션 통일
2. `reviewer.md` 수정: model sonnet→opus, memory 필드 제거, 서브에이전트 검증 항목 문구 개선, Memory 기록→결과 반환 섹션 교체
3. `agent-template.md` 업데이트: 모델 선택 가이드(haiku/sonnet/(상속) → opus/haiku/inherit), memory 필드 테이블에 (미설정) 행 추가
4. `quality-checklist.md` 업데이트: 서브에이전트 섹션 — 모델 선택/memory 필드 항목 문구 개선
5. `memory-system.md` 업데이트: task-log subagent 호출 기록 형식 추가, 서브에이전트 Memory 관리 방식 A/B 추가, 전문가 vs 서브에이전트 Memory 설명에 "(memory 필드 사용 시)" 단서 추가

**사용자 피드백 반영**:
- researcher.md의 `📌 재검증 권장` (6개월~1년) 태그 제거 — AI 분야 한정 태그이므로 범용 리서처에 부적절

**검증**:
- skill-authoring-guide/subagent-patterns 잔여 참조: memory 파일(과거 기록)에만 존재 — 정상
- researcher.md 53줄, reviewer.md 73줄, description 235자 — 모두 규격 이내
- `.claude/agent-memory/` 디렉토리 미존재 — 정리 불필요

### [2026-03-15] shared/references/ 중복 파일 정리 - 완료
- 삭제 2건:
  - `skill-authoring-guide.md` (73줄): Anthropic 공식 문서 정리물 → `knowledge/claude-code/skill-design-patterns.md`에 더 상세 버전 존재
  - `subagent-patterns.md` (55줄): 공식 문서 + 일반 설계 패턴 → CLAUDE.md 설계 원칙 + `knowledge/agent-design/`에 심화 내용 존재
- 참조 링크 제거 3건:
  - `create-expert/SKILL.md` 하단 참조: 2개 링크 삭제, 품질 체크리스트 링크 추가
  - `add-skill/SKILL.md` 5단계 인라인: skill-authoring-guide 링크 삭제
  - `add-skill/SKILL.md` 하단 참조: 2개 링크 삭제
- 잔여 참조 검증: grep 결과 0건 확인
- 유지 4건: memory-system.md, knowledge-system.md, persona-mentor-guide.md, quality-checklist.md (자체 설계 정보)
- 판단 기준: 외부 참조 정보는 knowledge/가 커버 → 삭제, 자체 설계 정보 → 유지

