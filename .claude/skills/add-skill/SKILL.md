---
name: add-skill
description: 기존 전문가 에이전트에 새로운 skill을 추가합니다. 대상 전문가의 구조와 기존 스킬을 분석하고, 충돌/중복 검증을 거쳐 SKILL.md를 작성합니다. 전문가에 새 기능을 추가하거나, 기존 전문가를 확장하고 싶을 때 사용합니다. 일반 프로젝트에 스킬을 추가할 때도 사용 가능합니다.
---

# 기존 전문가에 새 Skill 추가

기존 전문가 에이전트의 구조를 분석하고, 충돌/중복 없이 새 skill을 추가합니다.
새 전문가를 처음부터 만들려면 `/create-expert`를 사용하세요.

## Context Engineering 원칙

모든 설계 판단의 근거가 되는 5가지 원칙입니다.

### 1. Progressive Disclosure (Anthropic)
- 새 skill도 3단계 로딩을 따릅니다: 메타데이터(description) → 본문(SKILL.md) → 참조 파일(references/)
- 기존 전문가의 정보 배치 패턴과 일관되게 설계합니다

### 2. Goldilocks Zone (Anthropic)
- SKILL.md 500줄 이내, description 1024자 이내를 준수합니다
- 초과 시 참조 파일로 분리합니다

### 3. Context 기준 분리 (강수진 박사)
- 새 skill의 context가 기존 skill과 겹치지 않도록 설계합니다
- description과 트리거 키워드가 기존 skill과 중복되면 안 됩니다

### 4. Pay Once, Benefit Forever (Boris Cherny)
- 반복 사용될 워크플로우를 skill로 정규화합니다
- 한 번 잘 만든 skill은 모든 세션에서 자동으로 적용됩니다

### 5. 검증 루프 필수 (Karpathy)
- expert-reviewer 서브에이전트로 충돌/품질 검증을 수행합니다
- 기존 skill과의 중복/충돌 부재를 반드시 확인합니다

## 추가 절차

### 1단계: 요건 확인

사용자에게 다음을 확인합니다:

- **대상**: 어떤 전문가에 skill을 추가하는지 (experts/ 하위 또는 일반 프로젝트)
- **기능**: 추가할 기능이 무엇인지
- **트리거 시나리오**: 사용자가 어떤 상황에서 이 skill을 사용할지

$ARGUMENTS가 있으면 대상 전문가명으로 사용하되, 기능과 트리거 시나리오는 반드시 확인합니다.
기술적 판단(skill name, 구조 등)은 2단계 분석 후 제안합니다.

### 2단계: 기존 구조 분석

대상 전문가의 현재 구조를 철저히 파악합니다:

- **CLAUDE.md 읽기**: 페르소나, 핵심 규칙, 핵심 명령어 확인
- **기존 skill 전수 조사**: 각 skill의 name, description, 워크플로우 확인
- **기존 subagent 확인**: 활용 가능한 subagent 파악
- **충돌/중복 예비 판단**: 새 skill의 예상 description과 기존 skill의 description, 트리거 키워드 대조
- 필요 시 해당 분야 knowledge/ 참조

### 3단계: Skill 설계 제안 (Human Checkpoint)

분석 결과를 바탕으로 **큰 틀**을 사용자에게 제시합니다:

- **skill name**: 명명 규칙(소문자, 하이픈)에 맞는 이름
- **description 초안**: what + when + 트리거 키워드
- **충돌 분석 결과**: 기존 skill과의 중복/충돌 여부 및 근거
- **워크플로우 개요**: 주요 단계 요약
- **참조 파일 필요 여부**: 새로 생성할 참조 파일 또는 기존 shared/ 활용 여부
- **CLAUDE.md 변경 사항**: 핵심 명령어 섹션에 추가할 내용

사용자 확인 후 다음 단계로 진행합니다.

### 4단계: Skill 디테일 제안 (Human Checkpoint)

3단계에서 합의된 설계 위에 **세부 내용**을 제시합니다:

- **SKILL.md 전문 초안**: frontmatter + 본문 전체
- **참조 파일 내용**: 필요 시
- **CLAUDE.md 수정 diff**: 핵심 명령어 섹션의 변경 전/후

사용자 확인 후 다음 단계로 진행합니다.

### 5단계: 구현

합의된 내용을 실제 파일로 생성합니다:

- `.claude/skills/{skill-name}/SKILL.md` 생성
- 필요 시 `references/` 하위 파일 생성
- 대상 전문가의 CLAUDE.md 핵심 명령어 업데이트

구현 완료 후 사용자에게 결과를 보고합니다:
1. 생성된 skill 요약 (name, description, 주요 기능)
2. 생성/수정된 파일 목록
3. 사용법 예시

### 6단계: 검증

expert-reviewer 서브에이전트로 전체 품질 검증을 수행합니다.
검증 기준과 체크리스트는 expert-reviewer가 관리하며, 결과를 사용자에게 보고합니다.
