---
name: create-expert
description: 새로운 분야별 전문가 에이전트를 처음부터 생성합니다. Context Engineering 원칙(Progressive Disclosure, Goldilocks Zone, Context 기준 분리, Pay Once Benefit Forever, 검증 루프)에 따라 페르소나, 멘토 철학, Memory, Knowledge, Skill, Subagent를 포함한 완전한 전문가 패키지를 설계하고 구축합니다. 사용자가 새로운 전문가를 만들고 싶다고 하거나, 특정 분야의 에이전트가 필요하다고 할 때 사용합니다.
---

# 새 전문가 에이전트 생성

새로운 분야의 전문가 에이전트를 처음부터 설계하고 구축합니다.
기존 전문가에 skill을 추가하려면 `/add-skill`을 사용하세요.

## Context Engineering 원칙

모든 설계 판단의 근거가 되는 5가지 원칙입니다.

### 1. Progressive Disclosure (Anthropic)
- 정보를 3단계로 로딩합니다: 상시(CLAUDE.md) → 트리거(SKILL.md) → 온디맨드(참조 파일)
- 상시 컨텍스트에는 핵심만, 상세 내용은 참조 파일로 분리합니다

### 2. Goldilocks Zone (Anthropic)
- Context는 너무 적으면 hallucination, 너무 많으면 성능 저하를 유발합니다
- 적정 수준의 정보만 제공합니다: SKILL.md 500줄 이내, description 1024자 이내

### 3. Context 기준 분리 (강수진 박사)
- "뭘 물어볼까"보다 "뭘 알려줄까"가 중요합니다
- 에이전트에게 필요한 context를 기준으로 파일을 분리합니다 (역할 vs 절차 vs 도메인 지식)

### 4. Pay Once, Benefit Forever (Boris Cherny)
- 반복 피드백은 규칙화하여 CLAUDE.md에 반영합니다
- 한 번 정리한 context는 모든 세션에서 자동으로 적용됩니다

### 5. 검증 루프 필수 (Karpathy)
- 자신의 작업을 검증할 수 있는 Agent가 최고입니다
- 모든 생성 결과에 expert-reviewer 검증 단계를 포함합니다

## 생성 절차

### 1단계: 사용자 인터뷰

사용자에게 다음을 질문하여 전문가의 범위를 정의합니다:

- **분야**: 전문가가 다루는 영역 (예: AI, 재테크, 커리어)
- **핵심 역할**: 2-3가지 주요 수행 작업
- **도메인 지식**: 이 전문가가 축적해야 할 핵심 지식 분야 2-4개
- **멘토**: 영감을 줄 인물 (예: AI의 Geoffrey Hinton, 투자의 워런 버핏)

$ARGUMENTS가 있으면 분야명으로 사용하되, 나머지 항목은 반드시 인터뷰합니다.

> **인터뷰에서 묻지 않는 것**: 선호 스타일, 필요 도구, 파일 구성 등은 리서치(2단계) 결과를 바탕으로 제안합니다. 사용자에게 기술적 판단을 전가하지 않습니다.

### 2단계: 리서치

domain-researcher 서브에이전트로 해당 분야의 지식과 멘토 철학을 수집합니다:

- **분야 리서치**: 핵심 개념, 워크플로우, 프롬프트 패턴, 주의사항, 핵심 인물 탐색
- **멘토 리서치**: 멘토의 철학, 저서, 주요 주장 (사용자 지정 시에만)
- 리서치 결과 중 재사용 가치가 높은 내용은 knowledge 초기 topic으로 등록합니다
- Knowledge용 원본 자료를 함께 반환받아 새 전문가의 knowledge 시스템 초기 데이터로 활용합니다

### 3단계: 에이전트 프로젝트 구조 제안

리서치 결과를 바탕으로 **큰 틀**을 사용자에게 제시합니다:

- **폴더 구조**: 어떤 파일들이 생성되는지
- **페르소나**: 경력, 전문 영역, 성향, 커뮤니케이션 스타일
- **멘토 철학 요약**: 멘토의 핵심 원칙 3-5개
- **Context 배치 전략**: 어떤 내용이 CLAUDE.md(상시) / SKILL.md(트리거) / 참조 파일(온디맨드)에 각각 배치되는지

사용자 확인 후 다음 단계로 진행합니다.

### 4단계: 에이전트 프로젝트 디테일 제안

3단계에서 합의된 구조 위에 **각 파일의 세부 내용**을 제시합니다:

- CLAUDE.md 핵심 규칙 목록
- SKILL.md 원칙 및 워크플로우 초안
- Memory 초기 구조
- Knowledge 카테고리 및 초기 topic 계획
- Subagent 역할 및 도구 목록 (필요 시)

사용자 확인 후 다음 단계로 진행합니다.

### 5단계: 구현

`experts/{expert-name}/`에 파일을 생성합니다.

**필수 파일**:
- `CLAUDE.md`: 페르소나, 멘토 철학, 핵심 규칙
- `references/memory-system-guide.md`, `references/knowledge-system-guide.md`
- `.claude/skills/{skill-name}/SKILL.md`: 메인 skill
- `memory/MEMORY.md`, `memory/task-log.md`, `memory/lessons-learned.md`, `memory/user-preferences.md`
- `knowledge/index.md`, `knowledge/{category}/index.md`, `knowledge/{category}/{topic}.md` (domain-researcher가 반환한 Knowledge용 원본 자료를 기반으로 초기 topic 생성)
- `README.md`: 설치 가이드

**필요 시**:
- `.claude/skills/{skill-name}/references/`: 참조 문서
- `.claude/agents/`: subagent 정의

생성 시 참고:
- 템플릿: [../shared/templates/](../shared/templates/)
- Memory 가이드: [references/memory-system-guide.md](references/memory-system-guide.md)
- Knowledge 가이드: [references/knowledge-system-guide.md](references/knowledge-system-guide.md)
- 페르소나/멘토 가이드: [references/persona-mentor-guide.md](references/persona-mentor-guide.md)

구현 완료 후 사용자에게 생성 결과를 보고합니다:
1. 생성된 전문가 요약 (페르소나, 멘토, 핵심 기능)
2. 포함된 파일 목록
3. 설치 방법 및 사용법 예시

### 6단계: 검증

expert-reviewer 서브에이전트로 전체 품질 검증을 수행합니다.
검증 기준과 체크리스트는 expert-reviewer가 관리하며, 결과를 사용자에게 보고합니다.

## 참조

