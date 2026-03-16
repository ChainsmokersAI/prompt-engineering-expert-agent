# Knowledge 시스템 가이드

전문가 에이전트의 도메인 지식 체계를 관리하기 위한 Knowledge 시스템 설계 가이드입니다.
Memory(경험 기록)와는 별개로, 재사용 가능한 도메인 지식을 축적합니다.

## 개념

- **Memory**: 작업 이력, 학습 기록, 사용자 선호도 등 **경험** 중심
- **Knowledge**: 도메인 용어, 프레임워크, 베스트 프랙티스 등 **지식** 중심
- 둘은 보완 관계이며, Knowledge는 "무엇을 아는가", Memory는 "무엇을 겪었는가"에 해당

## 디렉토리 구조

```
knowledge/
├── index.md                        # 카테고리 목록만 (엔트리포인트)
├── {category-1}/
│   ├── index.md                    # topic 메타 테이블
│   └── {topic-name}.md             # 개별 topic 파일
├── {category-2}/
│   ├── index.md
│   └── {topic-name}.md
└── ...
```

## index.md 작성 규칙

### 루트 index.md (knowledge/index.md)

카테고리 목록만 나열합니다. 상세 내용은 포함하지 않습니다.

```markdown
# Knowledge Index

| 카테고리 | 설명 | topic 수 |
|----------|------|----------|
| [prompt-engineering](prompt-engineering/) | 프롬프트 설계 기법 | 0 |
| [claude-code](claude-code/) | Claude Code 런타임 | 0 |
```

### 카테고리 index.md (knowledge/{category}/index.md)

해당 카테고리의 topic 메타 정보를 테이블로 관리합니다.

```markdown
# {카테고리명} Knowledge

| topic | 설명 | 최종 업데이트 | 출처 | 신뢰도 |
|-------|------|---------------|------|--------|
| [topic-name](topic-name.md) | 한 줄 요약 | 2026-03-15 | Anthropic Docs | 공식 문서 |
```

## topic 파일 작성 규칙

### 필수 헤더

모든 topic 파일은 다음 헤더로 시작합니다:

```markdown
# {Topic 제목}

- **최종 업데이트**: {YYYY-MM-DD}
- **출처 신뢰도**: {공식 문서 | 저명 인사 | 커뮤니티}
- **출처**: {URL 또는 출처 설명}
```

### 출처 형식

본문 내에서 주장이나 정보를 기술할 때 출처를 함께 명시합니다:

```markdown
- Anthropic 공식 문서에 따르면 ... (출처: https://docs.anthropic.com/...)
- Boris Cherny에 따르면 ... (출처: Boris Cherny, "Building Effective Agents", 2025)
```

### 신뢰도 등급

| 등급 | 기준 | 예시 |
|------|------|------|
| **공식 문서** | 공식 문서, 공식 블로그, 공식 발표 | Anthropic Docs, OpenAI Docs |
| **저명 인사** | 업계/학계 권위자의 저서, 강연, 논문 | Boris Cherny, Lilian Weng |
| **커뮤니티** | 블로그, 포럼, 커뮤니티 토론 | Reddit, Stack Overflow |

### 파일 크기

제한 없음. 단, 하나의 topic이 지나치게 커지면 하위 topic으로 분리를 권장합니다.

## 업데이트 규칙

### 자동 트리거 3조건

다음 3가지가 **모두** 충족되면 knowledge에 즉시 기록합니다:

1. **웹 서칭을 수행했음**: 실제로 웹 검색/조회가 이루어진 경우
2. **기존 knowledge 카테고리에 부합**: 해당 전문가의 knowledge 카테고리와 관련 있는 내용
3. **새로운 내용**: 기존 knowledge에 없거나 업데이트가 필요한 정보

### 업데이트 시점

대화 중 즉시 기록합니다 (대화 마무리까지 기다리지 않음).
웹 서칭 직후, 결과를 정리하여 해당 topic 파일에 반영합니다.

### 업데이트 절차

1. 웹 서칭 수행
2. 3조건 충족 여부 확인
3. 해당 카테고리 index.md의 topic 테이블에 항목 추가/갱신
4. topic 파일 생성 또는 업데이트 (필수 헤더 포함)
5. 루트 index.md의 topic 수 갱신

## 품질 관리

- **출처 필수**: 출처 없는 지식은 기록하지 않습니다
- **최신성 표기**: 모든 topic에 최종 업데이트 날짜를 명시합니다
- **신뢰도 구분**: 공식 문서 > 저명 인사 > 커뮤니티 순으로 우선합니다
- **정기 점검**: knowledge가 누적될수록 오래된 정보(1년 이상)는 재검증 대상으로 표시합니다

## 초기 세팅 절차

새 전문가를 생성할 때:

1. 인터뷰에서 핵심 지식 카테고리 2-4개를 정의
   - **카테고리 명명 규칙**:
     - 이름만으로 포함된 내용을 즉시 파악할 수 있어야 합니다
     - 범용적 이름 금지: `domain-specific` ✗, `general` ✗, `misc` ✗
     - 구체적 이름 사용: `ai-simulation-qa`, `market-research-methods`, `sales-materials` 등 ✓
     - 판단 기준: "이 카테고리명을 처음 보는 사람이 어떤 topic이 들어있을지 예측할 수 있는가?"
2. `knowledge/index.md` 생성 (카테고리 목록)
3. 각 카테고리 폴더 + `index.md` 생성 (빈 topic 테이블)
4. domain-researcher의 Part B (원본 자료)를 기반으로 초기 topic 파일 생성 (topic 파일 작성 규칙 참조, create-expert가 전문가 맥락에 맞게 가공·재구성 가능)

## Memory와의 관계

| 구분 | Memory | Knowledge |
|------|--------|-----------|
| 목적 | 경험 기록 (무엇을 겪었는가) | 도메인 지식 축적 (무엇을 아는가) |
| 내용 | 작업 이력, 실수, 교훈, 선호도 | 개념, 프레임워크, 베스트 프랙티스 |
| 트리거 | 모든 작업 후 | 웹 서칭 + 카테고리 부합 + 새 내용 |
| 크기 제한 | MEMORY.md 200줄 | topic별 제한 없음 |
| 관리 방식 | 오래된 것 아카이브/탈락 | 오래된 것 재검증 |
| 기록 시점 | 작업 완료 후 | 대화 중 즉시 (웹 서칭 직후) |

상세: [memory-system-guide.md](memory-system-guide.md)
