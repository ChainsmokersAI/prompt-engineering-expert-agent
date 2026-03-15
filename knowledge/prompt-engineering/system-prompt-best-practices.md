# System Prompt Best Practices

Anthropic 공식 가이드에서 정리한 시스템 프롬프트 작성의 핵심 원칙들입니다.

## 핵심 원칙

### 1. Goldilocks Zone (적정 수준의 구체성)
- **너무 구체한 로직**: 복잡한 if-else 형태의 지시사항 → 유지보수 어려움, 취약함
- **너무 추상적**: 맥락을 가정하는 모호한 지시 → 일관성 부족
- **적정 수준**: 행동을 가이드하면서도 Claude의 판단 여지를 남김

**출처**: [Anthropic - Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

### 2. 명확하고 직설적 표현
- 구체적인 출력 형식 명시
- 수열적 단계가 중요하면 번호 리스트 사용
- 일반인도 이해할 수 있는 수준의 명확성 필요

**예시**: 
```
Less: "Create an analytics dashboard"
More: "Create an analytics dashboard. Include as many relevant features as possible. 
Go beyond the basics to create a fully-featured implementation."
```

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

### 3. 문맥(Context) 추가
- 왜 그 행동이 중요한지 설명하면 Claude가 더 정교한 응답 생성
- 예: "This will be read by a text-to-speech engine, so never use ellipses"

### 4. 짧은 계약(Short Contract) 패턴
좋은 시스템 프롬프트는 짧은 계약처럼 명시적, 범위가 정해진, 검증 가능해야 함:

```
You are: [역할 - 한 줄]
Goal: [성공이 무엇인지]
Constraints: [제약 목록]
If unsure: [명확하지 않으면 명시적으로 물어보기]
```

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

## 특화된 시스템 프롬프트 패턴

### Role Assignment (역할 부여)
한 줄의 역할 정의만으로도 Claude의 동작이 크게 달라짐:

```python
system="You are a helpful coding assistant specializing in Python."
```

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

### Long-Context Prompting (긴 문서 처리)
- **긴 데이터는 상단에 배치**: 프롬프트 상단에 20k+ 토큰 문서 배치
- **쿼리는 하단에**: 쿼리를 끝부분에 배치하면 응답 품질 30% 향상 가능
- **XML 구조화**: `<document>`, `<document_content>`, `<source>` 태그 사용
- **인용 추출**: "먼저 관련 부분을 인용하고, 그 다음 작업 수행"

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

### Output Format 제어
1. **하지 말 것보다는 할 것 명시**
   - ❌ "Do not use markdown in your response"
   - ✅ "Your response should be composed of smoothly flowing prose paragraphs"

2. **XML 포맷 지시자 사용**
   - `<smoothly_flowing_prose_paragraphs>` 태그로 감싸기

3. **프롬프트 스타일 일치**
   - 프롬프트의 문체가 원하는 출력 문체에 영향

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

## 안티패턴

### 1. Context 오염 (Context Rot)
- **문제**: 맥락이 커질수록 주의력 저하 → 모순, 혼동, 분산
- **해결**: 신중한 큐레이션과 just-in-time 검색 설계

**출처**: [Anthropic - Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

### 2. 과도한 예시 적재
- ❌ 모든 엣지 케이스를 프롬프트에 나열
- ✅ 다양하고 대표적인 3-5개 예시 선별

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

### 3. 모호한 제약사항
- ❌ "NEVER use ellipses"
- ✅ "Your response will be read by a text-to-speech engine, so never use ellipses"

## 최근 변화 (2026)

### Prefilled Response 제거
- Claude 4.6부터 마지막 assistant turn의 prefilled response 미지원
- **대안**: Structured Outputs 사용 또는 XML 태그 활용

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

### 더 강력한 모델의 영향
- Claude 4.6은 Sonnet 4.5보다 훨씬 더 자발적
- 이전에 "MUST" 형태의 강한 지시가 필요했다면, 이제는 더 완화된 톤으로도 충분
- 과도한 "must/critical" 지시는 오버트리거 야기

**출처**: [Prompting best practices - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)

## 맥락 효율성 (Context Efficiency)

### Precious Resource로서의 Context
핵심 통찰: "Claude는 이미 충분히 똑똑하다. 문제는 context다."

- 토큰을 가장 신호가 강한 최소 집합으로 구성
- 정보는 많지만 타이트하게 유지

**출처**: [Anthropic - Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

---

**신뢰도**: 공식 Anthropic 문서 (최고)  
**최종 업데이트**: 2026-03-15
