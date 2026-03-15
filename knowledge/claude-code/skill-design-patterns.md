# Skill Design Patterns

Claude Code에서 skills(SKILL.md) 작성의 핵심 패턴과 베스트 프랙티스입니다.

## SKILL.md 구조

모든 skill은 두 부분으로 구성:

```yaml
---
name: skill-name
description: What this skill does and when to use it
---

# Your Skill Content
Instructions and guidance...
```

### 필수/권장 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `name` | 예 | 슬래시 명령어 이름 (소문자, 숫자, 하이픈만, 64자 이하) |
| `description` | 권장 | 무엇을 하는지, 언제 사용하는지 명시 (1024자 이하) |
| `disable-model-invocation` | 아니오 | `true`: 사용자만 호출 가능 (e.g., `/deploy`) |
| `user-invocable` | 아니오 | `false`: Claude만 자동으로 호출 (배경 지식용) |
| `allowed-tools` | 아니오 | 허용할 도구 목록 (보안) |
| `context` | 아니오 | `fork`: subagent에서 실행 |
| `agent` | 아니오 | Subagent 타입 지정 (Explore, Plan, general-purpose) |

**출처**: [Claude Code - Extend Claude with skills](https://code.claude.com/docs/en/skills)

## Skill 내용 분류

### 1. Reference Content (참고 자료형)
지식을 제공하고 현재 작업에 적용:
- 컨벤션, 패턴, 스타일 가이드
- 도메인 지식
- 인라인 실행 → Context에 적재됨

```yaml
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

### 2. Task Content (작업형)
단계별 지시사항, 특정 액션:
- 배포, 커밋, 코드 생성
- `disable-model-invocation: true` 권장 (의도적 실행)
- 사용자가 `/skill-name`으로 직접 호출

```yaml
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
---

Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
```

**출처**: [Claude Code - Extend Claude with skills](https://code.claude.com/docs/en/skills)

## Progressive Disclosure (단계적 정보 로딩)

Skill의 핵심 아키텍처. 3단계 로딩:

### Level 1: Metadata (항상 로딩)
- YAML frontmatter의 `name`, `description`
- System prompt에 항상 포함 (검색)
- Context 비용: 매우 낮음 (~100 토큰/skill)

### Level 2: Instructions (triggered 시 로딩)
- SKILL.md의 마크다운 본문
- Claude가 skill을 호출할 때만 로딩
- Context 비용: 중간 (5k 토큰 이하)

### Level 3+: Resources (필요할 때만)
- 참고 문서 (FORMS.md, REFERENCE.md)
- 실행 스크립트 (fill_form.py)
- 데이터 파일, 템플릿
- Context 비용: 거의 없음 (스크립트는 출력만 로딩)

**핵심**: 추가 파일은 filesystem에 남아있고, 필요할 때만 읽힘

**출처**: [Agent Skills - Progressive Disclosure](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

## Supporting Files 패턴

SKILL.md를 집중화하고, 상세 내용은 별도 파일로:

```
my-skill/
├── SKILL.md (500줄 이내, 핵심만)
├── reference.md (상세 API 문서)
├── examples.md (사용 예시)
└── scripts/
    └── helper.py (유틸리티)
```

SKILL.md에서 참조 명시:
```markdown
## Additional resources
- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

**핵심**: 500줄 유지 → 최적의 context 효율

**출처**: [Claude Code - Extend Claude with skills](https://code.claude.com/docs/en/skills)

## Skill 호출 제어

| Frontmatter | 사용자 호출 | Claude 호출 | Context 로딩 |
|-------------|----------|----------|----------|
| (기본) | ✅ | ✅ | 설명은 항상, 내용은 호출 시 |
| `disable-model-invocation: true` | ✅ | ❌ | 설명 제외, 사용자 호출 시만 |
| `user-invocable: false` | ❌ | ✅ | 설명은 항상, 호출 시 내용 로딩 |

**패턴**: 
- Deploy, commit 같은 side-effect → `disable-model-invocation: true`
- 배경 지식 → `user-invocable: false`
- 협력 형 tool → 기본값 유지

**출처**: [Claude Code - Extend Claude with skills](https://code.claude.com/docs/en/skills)

## Skill에서 Subagent 실행

`context: fork`로 격리된 context에서 실행:

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:
1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings
```

**Built-in Agents**:
- `Explore`: 읽기 전용, 빠름 (Haiku)
- `Plan`: 계획 모드 (정보 수집)
- `general-purpose`: 모든 도구 가능

**출처**: [Claude Code - Extend Claude with skills](https://code.claude.com/docs/en/skills)

## 동적 Context 주입

`!`command`` 문법으로 실행 결과를 프롬프트에 삽입:

```yaml
---
name: pr-summary
description: Summarize PR changes
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- Comments: !`gh pr view --comments`
- Files: !`gh pr diff --name-only`

Summarize this PR...
```

**실행 순서**:
1. 모든 `!`command`` 먼저 실행
2. 출력이 placeholder를 대체
3. Claude가 최종 프롬프트 수신

**출처**: [Claude Code - Extend Claude with skills](https://code.claude.com/docs/en/skills)

## 인수 처리 (Arguments)

`$ARGUMENTS` 또는 `$N` (0-based) 치환:

```yaml
---
name: migrate-component
description: Migrate a component framework
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

사용: `/migrate-component SearchBar React Vue`
→ SearchBar를 React에서 Vue로 마이그레이션

**출처**: [Claude Code - Extend Claude with skills](https://code.claude.com/docs/en/skills)

## 보안 고려사항

### Skill 감사 체크리스트
- ✅ SKILL.md의 모든 지시사항 검토
- ✅ 번들된 스크립트의 외부 API 호출 확인
- ✅ 원치 않는 파일 접근, bash 명령어 검사
- ✅ 신뢰할 수 있는 출처에서만 설치

**주의**: Skill은 소프트웨어 설치와 같음. 신뢰할 수 없는 출처는 위험.

**출처**: [Agent Skills - Security](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)

---

**신뢰도**: 공식 Claude Code 문서 (최고)  
**최종 업데이트**: 2026-03-15
