# 학습 기록

## FUSE 파일시스템 관련

### [2026-03-15] mv 명령으로 파일 손상
- **상황**: `.claude/skills/create-expert/references/`에서 `.claude/skills/shared/references/`로 `mv` 명령 실행
- **문제**: FUSE 마운트된 파일시스템에서 `mv`가 파일을 손상시킴. `find`에는 보이지만 실제로는 읽기/삭제 불가한 유령 파일 생성
- **해결**: 새 디렉토리(`shared2/`)를 만들어 파일을 처음부터 재생성. 사용자가 로컬에서 손상된 디렉토리 삭제 후 이름 변경
- **교훈**: FUSE 마운트 환경에서는 `mv` 대신 `cp` + 원본 삭제를 사용해야 한다. 또는 처음부터 목표 경로에 직접 파일을 생성해야 한다.

### [2026-03-15] rm -rf로 손상 파일 삭제 불가
- **상황**: 손상된 `shared/` 디렉토리를 `rm -rf`로 삭제 시도
- **문제**: "Operation not permitted" 에러. `allow_cowork_file_delete`도 "target is busy"로 실패
- **해결**: 사용자가 로컬 환경에서 직접 삭제
- **교훈**: FUSE 손상 파일은 VM 내부에서 복구 불가. 사용자에게 로컬 작업을 요청해야 한다.

## 설계 판단 관련

### [2026-03-15] E2E vs Task 기반 접근
- **상황**: 최초 설계에서 create-expert를 단일 E2E 워크플로우로 구현
- **문제**: 사용자가 "스킬 하나 추가해줘" 같은 개별 작업도 필요한데, E2E로는 부분 작업 불가
- **해결**: Task 기반으로 전환 (A~F 개별 Task)
- **교훈**: 전문가 에이전트는 Task 단위로 설계해야 한다. 사용자는 항상 전체 워크플로우를 원하지 않는다.

### [2026-03-15] 스킬 분리 시점
- **상황**: Task A~F를 하나의 SKILL.md에 모두 넣었음
- **문제**: "새로 만들기"와 "기존 개선"은 트리거 키워드가 완전히 다름. description에 모두 담으면 1024자 초과 위험 + 트리거 정확도 하락
- **해결**: create-expert (Task A) + evolve-expert (Task B~F) 분리
- **교훈**: 트리거 의도가 다르면 스킬을 분리해야 한다. description의 what + when이 명확히 다른 작업은 별도 스킬이 적합하다.

### [2026-03-15] 공유 리소스 위치 결정
- **상황**: references/, templates/, examples/가 두 스킬 모두에서 필요
- **문제**: 각 스킬에 복제하면 중복, 한쪽에만 두면 다른 쪽 접근 불편
- **해결**: `.claude/skills/shared/` 디렉토리 (SKILL.md 없음 = Claude Code가 스킬로 인식하지 않음)
- **교훈**: SKILL.md가 없는 디렉토리는 스킬로 취급되지 않으므로, 순수 공유 리소스 저장소로 활용 가능하다.

## 검증 관련

### [2026-03-15] CLAUDE.md 핵심 명령어 누락
- **상황**: evolve-expert 스킬을 분리했으나 CLAUDE.md의 핵심 명령어 섹션에 반영 안 됨
- **문제**: 사용자가 `/evolve-expert` 명령의 존재를 모를 수 있음
- **해결**: reviewer 서브에이전트 검증에서 발견 → CLAUDE.md에 추가
- **교훈**: 스킬을 추가/분리할 때는 반드시 CLAUDE.md의 핵심 명령어 섹션도 함께 업데이트해야 한다.

### [2026-03-15] 핵심 규칙 변경 시 템플릿 동기화 필수
- **상황**: CLAUDE.md 공통 원칙을 핵심 규칙으로 교체
- **교훈**: 핵심 규칙(공통 원칙) 변경 시 반드시 claude-md-template.md도 동기화해야 한다. 체크리스트 항목도 함께 업데이트 필요.

### [2026-03-15] 작업 후 Memory 기록 누락
- **상황**: 핵심 규칙 교체 + 템플릿 동기화 작업 완료 후 Memory 업데이트를 수행하지 않음
- **문제**: 사용자가 핵심 규칙 3번 "Memory 참조 및 기록 의무" 위반을 지적
- **교훈**: 모든 작업 완료 후 반드시 Memory를 업데이트해야 한다. 사용자가 요청하지 않아도 자동 수행 필수.

### [2026-03-15] 프로젝트 memory 우선순위
- **상황**: 피드백 기록 시 auto memory(~/.claude/projects/.../memory/)만 업데이트하고 프로젝트 memory(propmt-engineering-expert/memory/)를 건너뜀
- **문제**: 사용자가 "더 중요한 건 프로젝트 메모리"라고 지적
- **교훈**: 프로젝트 memory/가 auto memory보다 우선순위가 높다. 항상 프로젝트 memory/ 먼저 업데이트한 뒤 auto memory를 업데이트해야 한다.

### [2026-03-15] 프로젝트 memory/MEMORY.md는 자동 로딩되지 않음
- **상황**: 대화 시작 시 프로젝트 `memory/MEMORY.md`를 읽지 않고 응답
- **문제**: CLAUDE.md와 auto memory(`~/.claude/projects/.../memory/`)는 자동 로딩되지만, 프로젝트 루트의 `memory/MEMORY.md`는 자동 로딩 대상이 아님
- **해결**: CLAUDE.md 핵심 규칙 3번에 "대화 첫 응답 전 반드시 Read 도구로 읽을 것" 명시적 지시 추가
- **교훈**: 자동 로딩되지 않는 파일은 CLAUDE.md에 명시적 Read 지시를 넣어야 확실히 읽힌다.

### [2026-03-15] 피드백 기록 시 프로젝트 memory 누락 반복
- **상황**: 문체 유지 피드백을 auto memory에만 기록하고 프로젝트 memory/를 또 건너뜀
- **문제**: 이전에도 동일한 실수로 지적받았음에도 반복
- **교훈**: 피드백 기록 순서를 반드시 지킨다 — ① 프로젝트 memory/ 먼저 ② auto memory 그 다음. 이 순서를 절대 뒤바꾸거나 생략하지 않는다.

### [2026-03-15] Git Push 대상 프로젝트의 개인 정보 관리
- **상황**: user-preferences.md에 이름과 이메일이 기록되어 있었음
- **문제**: Git Push 시 개인 정보가 공개 저장소에 노출될 수 있음
- **해결**: 개인 정보 제거 후 전체 파일 검색으로 잔여 개인 정보 없음 확인
- **교훈**: Git Push 대상 프로젝트에서는 파일에 개인 정보(이름, 이메일 등)를 포함하지 않아야 한다. 향후 전문가 생성 시에도 이 원칙을 적용해야 한다.

### [2026-03-15] 리서치 자료의 날짜 정밀도
- **상황**: 4인 철학 리서치(Anthropic, Boris Cherny, Karpathy, 강수진 박사) 결과물에서 날짜 세밀도 부족
- **문제**: AI/LLM 분야는 6개월만 지나도 과거 데이터가 됨. 발행일이 모호하면 자료의 유효성 판단 불가
- **해결**: researcher 서브에이전트에 "날짜 정밀도 원칙" 섹션 추가 (필수 기록 항목, 표기 형식, 경과 시간별 태그)
- **교훈**: 모든 리서치 결과에는 원본 발행일(최소 연-월), 리서치 수행 날짜, 버전 정보를 필수로 기록해야 한다. 특히 AI/LLM 분야에서는 날짜가 자료의 신뢰도에 직접 영향을 미친다.

## 원칙 배치 관련

### [2026-03-15] 공통 원칙은 skill이 아닌 CLAUDE.md에 배치
- **상황**: "자기 진화" 원칙이 create-expert SKILL.md의 6번 원칙으로 위치해 있었음
- **문제**: 특정 skill 안에 "자기 자신을 수정하라"는 지시를 넣는 것은 skill의 책임 범위를 넘음. 또한 create-expert에서만 적용되어 evolve-expert 등 다른 skill에는 미적용
- **해결**: CLAUDE.md 핵심 규칙 5번 "Skill, Subagent 지속적 개선 의무"로 이동
- **교훈**: 모든 전문가/skill에 공통으로 적용되어야 하는 원칙은 개별 SKILL.md가 아닌 CLAUDE.md 핵심 규칙에 배치해야 한다. SKILL.md에는 해당 skill 고유의 원칙만 둔다.

## SKILL.md 작성 관련

### [2026-03-15] SKILL.md에서 핵심 내용/원칙을 절차보다 먼저 제시
- **상황**: 기존 create-expert SKILL.md는 절차(7단계)가 본문의 대부분을 차지하고, 공통 작업 규칙은 하단에 위치
- **문제**: LLM이 SKILL.md를 읽을 때 절차부터 만나면 "왜 이렇게 하는지"(원칙)를 모른 채 실행함
- **해결**: Context Engineering 원칙 6개를 절차보다 상단에 배치
- **교훈**: SKILL.md 작성 시 핵심 원칙/규칙을 먼저 제시하고, 절차는 그 다음에 배치해야 한다. 원칙이 절차의 판단 기준이 되어야 하므로 순서가 중요하다.

### [2026-03-15] 범용 skill에서 독립 skill 분리 시 독립성 확보
- **상황**: evolve-expert가 6가지 Task(스킬 추가, 서브에이전트 설계, CLAUDE.md 수정, Memory 점검, Knowledge 점검, 구조 개선)를 하나의 skill에 담고 있었음
- **문제**: description 1024자 제한 내에 모든 Task를 설명하기 어렵고, 트리거 정확도가 분산됨
- **해결**: "스킬 추가" 기능을 add-skill로 독립 분리. 나머지 Task는 추후 별도 skill로 분리 예정
- **교훈**: 범용 skill(여러 Task를 하나에 담은 것)은 각 Task가 충분히 독립적이면 별도 skill로 분리하는 것이 트리거 정확도와 description 명확성 면에서 유리하다

### [2026-03-15] 구조 제안과 디테일 제안 분리
- **상황**: 기존 구조 설계 단계에서 폴더 구조 + 각 파일 세부 내용을 한꺼번에 제시
- **문제**: 사용자가 큰 틀과 세부 내용을 동시에 판단해야 하여 피드백이 어려움
- **해결**: 3단계(구조 제안: 폴더/페르소나/Context 배치) → 4단계(디테일 제안: 각 파일 세부 내용) 분리
- **교훈**: Human Checkpoint는 판단 단위별로 분리해야 한다. 큰 틀에 합의한 후 세부 내용을 제안하면 피드백 품질이 높아진다.

### [2026-03-16] 검증 기준은 Single Source of Truth로 관리
- **상황**: create-expert, add-skill 각각의 6단계에 검증 항목 리스트가 있고, expert-reviewer.md에도 검증 항목이 있어 3곳 이중 관리
- **문제**: 검증 항목 변경 시 3곳을 동시에 수정해야 하며, 불일치 발생 위험. 실제로 "공통 원칙 검증" 항목이 CLAUDE.md 핵심 규칙과 sync되지 않았음
- **해결**: 개별 skill의 6단계에서 검증 항목 삭제, "expert-reviewer가 관리" 위임 문장으로 대체
- **교훈**: 검증 기준은 검증 담당자(expert-reviewer)가 유일하게 관리해야 한다. 호출하는 쪽(skill)은 "검증을 수행한다"만 명시하고, 구체적 항목은 열거하지 않는다

### [2026-03-16] 실효성 없는 템플릿은 삭제
- **상황**: skill-template.md가 존재했으나, 실제 생성되는 skill과 구조가 불일치하고, 확인 사항도 expert-reviewer와 중복
- **해결**: skill-template.md 삭제, 모든 참조 제거
- **교훈**: 템플릿의 가치는 "실제 결과물과 구조가 일치"할 때만 있다. 분야마다 구조가 달라지는 파일은 공통 템플릿보다 검증 체크리스트(expert-reviewer)로 관리하는 것이 효과적

### [2026-03-16] 서브에이전트 요약 반환 → 정보 유실 문제
- **상황**: serial-entrepreneur-agent 생성 시 domain-researcher가 리서치 결과를 "1-2문장 설명", "배경 요약" 등으로 축약하여 반환
- **문제**: create-expert가 knowledge topic 파일을 생성할 때 원본 정보가 이미 유실된 상태. 결과적으로 topic 파일의 품질이 저하됨 (예: Sales Deck topic이 Pitch Deck에 편향)
- **해결**: domain-researcher 결과물을 Part A (설계 요약) + Part B (원본 자료)로 분리. Part B에 명시적 반요약 지시 ("요약하지 않습니다"). create-expert 5단계에서 Part B 기반 topic 생성 지침 명확화
- **교훈**: 서브에이전트가 결과를 요약하면 소비자 측에서 복원할 방법이 없다. 원본은 최대한 보존하여 반환하고, 가공·재구성은 소비자의 책임으로 둬야 한다. 특히 "주요 산출물"과 "설계 판단용 요약"은 파트를 분리하여 각각의 깊이를 명시해야 한다

### [2026-03-16] Knowledge 카테고리명은 구체적으로 명명
- **상황**: serial-entrepreneur-agent의 knowledge 카테고리 중 `domain-specific`이라는 이름 사용
- **문제**: 사용자 피드백 — 이름만 보고 어떤 정보가 포함되어 있는지 전혀 파악할 수 없음
- **교훈**: Knowledge 카테고리명은 내용을 즉시 파악할 수 있도록 구체적으로 명명해야 한다. `domain-specific` ✗ → `ai-simulation-qa`, `market-research-methods` 등 ✓. 범용적인 이름은 카테고리의 탐색성과 유지보수성을 모두 저하시킨다

### [2026-03-16] 사용자 용어를 자의적으로 해석하지 않는다 (Sales Deck ≠ Pitch Deck)
- **상황**: serial-entrepreneur-agent에서 사용자가 "Sales Deck" 작성 skill을 요청
- **문제**: Sales Deck knowledge topic을 투자자 대상 Pitch Deck 위주로 작성. Sales Deck은 고객 대상 세일즈 자료(제품 소개서, 제안서, 데모 자료 등)를 포괄하는 개념인데, 투자 피칭에 편향
- **교훈**: 사용자가 사용한 용어의 범위를 자의적으로 해석하지 않는다. 유사하지만 다른 개념(Sales Deck vs Pitch Deck)은 명확히 구분해야 하며, 불확실하면 사용자에게 확인해야 한다

### [2026-03-16] 사용자 요청 처리 전 knowledge/ 확인 필수
- **상황**: domain-researcher subagent 설계 시 knowledge/ 확인 없이 바로 초안 작성에 착수하려 함
- **문제**: 사용자가 핵심 규칙 4번 "Knowledge 참조 및 기록 의무" 위반을 지적. subagent 설계에 관련된 knowledge(context-engineering-strategies, skill-design-patterns)가 존재했으나 참고하지 않음
- **교훈**: 어떤 작업이든 시작 전에 반드시 knowledge/에서 관련 자료를 확인해야 한다. 특히 설계/구현 작업에서는 기존 지식 기반 위에 설계해야 일관성이 유지된다

### [2026-03-16] Subagent 분리 판단 기준: 소스 특성과 최신성
- **상황**: 범용 researcher가 AI/Prompt Engineering과 재테크/커리어 등 전혀 다른 분야를 모두 담당
- **문제**: 소스(arxiv vs 네이버 뉴스), 최신성 기준(6개월 vs 시대 불문)이 근본적으로 다름
- **해결**: domain-researcher를 별도 subagent로 분리, 소스 전략 수립 단계에서 분야별 자체 판단
- **교훈**: subagent 분리 여부는 "절차가 같은가"보다 "입력 소스와 판단 기준이 근본적으로 다른가"로 판단해야 한다

## 정합성/동기화 관련

### [2026-03-15] Knowledge index.md topic 수는 수동 관리 필요
- **상황**: 3개 카테고리의 index.md topic 수가 모두 실제 파일 수와 불일치
- **문제**: topic 파일을 추가할 때 루트 index.md의 topic 수 갱신을 빠뜨림
- **교훈**: Knowledge topic 추가 시 반드시 루트 index.md의 topic 수도 함께 갱신해야 한다. 자동화되지 않으므로 체크리스트에 포함할 것

### [2026-03-15] 예시/템플릿은 표준 구조 변경 시 즉시 동기화
- **상황**: 전문가 표준 구조가 `.claude/skills/`, `.claude/agents/`로 변경되었으나, example-expert.md와 claude-md-template.md는 구버전(`skills/`, `agents/`) 유지
- **문제**: 예시를 참고하여 생성하면 잘못된 경로 구조가 됨
- **교훈**: 표준 구조 변경 시 예시 파일과 템플릿 파일을 즉시 동기화해야 한다

### [2026-03-15] reviewer subagent에 skills 필드를 부여하면 안 됨
- **상황**: reviewer.md에 `skills: - create-expert`가 설정되어 있었음
- **문제**: skills 필드는 해당 skill을 "실행"할 수 있게 상속하는 것이므로, 읽기 전용 검증자에게 부여하면 의도치 않게 skill 실행이 가능해짐
- **교훈**: subagent의 skills 필드는 해당 subagent가 skill을 직접 실행해야 할 때만 부여한다. 검증/리뷰 목적의 subagent는 Read/Grep/Glob만으로 충분하다
