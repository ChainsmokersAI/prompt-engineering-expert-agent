# 강수진 박사의 Prompt Engineering 철학 및 방법론

## 인물 개요

**이름**: 강수진 (Sujin Kang, Ph.D.)  
**배경**: 한국어학 박사 (University of Hawaii at Manoa)  
**전문 분야**: 대화분석(Conversation Analysis), 상호작용 언어학(Interactional Linguistics)  
**주요 경력**: 
- 국내 공채 1호 프롬프트 엔지니어 (뤼튼테크놀로지스)
- The Prompt Company (더 프롬프트 컴퍼니) CEO
- 다수의 교육 강의 및 저서 활동

**핵심 철학**: "인간의 언어 원리 기반 프롬프트 설계"

---

## 핵심 철학 (5가지)

### 1. Context Engineering이 Prompt Engineering을 초월한다
**핵심 문장**: "뭘 물어볼까"보다 "뭘 알려줄까"가 더 중요하다

**정의**: "필요한 정보와 도구를, 올바른 형식과 시점에, LLM이 과업을 성취할 수 있게 시스템적으로 제공하는 설계 및 구축의 전문 역량"

**의의**: 
- AI 에이전트 설계의 중심을 "질문의 정확성"에서 "맥락 제공의 완성도"로 이동
- 프롬프트 엔지니어링의 진화 단계를 나타냄
- 단순 프롬프트 최적화를 넘어 시스템 아키텍처 설계로 확장

**출처**: [AI 에이전트 설계의 핵심: 컨텍스트 엔지니어링 3가지 원칙](https://www.aiground.co.kr/ai-agent-context-engineering/), 강수진 박사의 프롬프트·컨텍스트 엔지니어링 아카데미 (패스트캠퍼스)

---

### 2. 언어학적 기초를 통한 AI-Human Interaction 설계
**핵심 문장**: "AI는 사용자의 질문에 기계어가 아닌 인간의 언어로 답한다. 같은 내용이라도 어떻게 표현해야 인간의 언어에 더 가까울지를 고민해야 한다"

**방법론**:
- 대화분석의 원리를 프롬프트 설계에 적용
- 상호작용 언어학을 통한 AI 응답 최적화
- 인간의 언어 사용 패턴을 기반으로 한 프롬프트 구조화

**한국어 프롬프트의 특수성 인식**:
- 한국어의 맥락의존성, 높임말/낮춤말 체계 활용
- 조사, 어미 선택에 따른 의미 변화
- 비명시적 정보 전달 방식의 최적화

**출처**: 프롬프트 엔지니어's Work Journal, 클래스 e - 강수진의 AI 시대 질문의 기술

---

### 3. RPI 워크플로우: 단계별 Context 분리
**구성**: Research → Plan → Implement

**설계 원칙**:

| 단계 | 목적 | Context 역할 |
|------|------|------------|
| **Research** | 필요한 정보 수집 및 분석 | 조사 전문가 에이전트 역할 |
| **Plan** | 수집된 정보 기반 계획 수립 | 기획자 에이전트 역할 |
| **Implement** | 계획 실행 | 실행 에이전트 역할 |

**핵심 이점**:
- 각 단계 결과물을 파일로 저장하여 컨텍스트 초기화 가능
- 중간 결과 검토 및 휴먼 체크포인트 확보
- 재사용성 및 투명성 증대
- 복잡한 프로젝트에서 에러 전파 방지

**출처**: [AI 에이전트 설계의 핵심: 컨텍스트 엔지니어링 3가지 원칙](https://www.aiground.co.kr/ai-agent-context-engineering/)

---

### 4. Context Window의 40% 규칙 (Smart Zone vs Dumb Zone)
**원칙**: 컨텍스트 윈도우의 40%를 초과하면 성능 저하 발생

**개념**:
- **Smart Zone**: 컨텍스트 윈도우의 0~40% 구간 (최적 성능)
- **Dumb Zone**: 컨텍스트 윈도우의 40% 초과 구간 (성능 저하)

**실천 방법**:
- 정확성(Accuracy): 필요한 정보만 선별
- 완결성(Completeness): 누락되지 않도록 주의
- 적절한 양(Right Amount): 용량 초과 피하기
- 대화 흐름(Conversation Flow): 논리적 순서 유지

**출처**: [AI 에이전트 설계의 핵심: 컨텍스트 엔지니어링 3가지 원칙](https://www.aiground.co.kr/ai-agent-context-engineering/)

---

### 5. Subagent는 역할이 아닌 Context 기준으로 분리
**핵심 주장**: 에이전트를 역할 기반으로 분리하는 것은 오류

**올바른 분리 기준**:
- **Context 기준**: 서로 다른 정보와 도구 조합이 필요한 작업
- **예시**: 조사 에이전트는 핵심 정보만 압축하여 전달 → 다음 에이전트가 활용

**설계 원칙**:
- 각 subagent는 명확한 context window를 가짐
- 불필요한 정보 전파 차단
- 다음 에이전트의 집중도 향상

**출처**: [AI 에이전트 설계의 핵심: 컨텍스트 엔지니어링 3가지 원칙](https://www.aiground.co.kr/ai-agent-context-engineering/)

---

## 방법론: Context Engineering 설계 프레임워크

### 3가지 핵심 질문

| 질문 | 초점 | 실천 방법 |
|------|------|---------|
| **무엇을 줄까?** | Context Content | 정확성·완결성·적절한 양·대화 흐름 점검 |
| **언제 줄까?** | Context Timing | RPI 워크플로우로 단계 분리 |
| **어떻게 나눌까?** | Context Architecture | 역할 아닌 context 기준 분리 |

### 실무 적용 모델

**프롬프트 개발 단계** (《프롬프트 엔지니어의 업무일지》 기반):
1. 프롬프트 기획: 목적·대상·성공 기준 정의
2. 프롬프트 제작: 구체적 프롬프트 작성
3. 테스트: 다양한 시나리오에서 검증
4. 평가: 정량적·정성적 평가 기준 적용
5. 연구: 실패 케이스 분석 및 개선

**출처**: 《프롬프트 엔지니어의 업무일지》(강수진, 2024년 8월 14일 발행, 368쪽)

---

## 대표 저서 및 강의

### 주요 저서

**《프롬프트 엔지니어의 업무일지》(2024년 8월 14일 발행)**
- 출판사: 책과나무
- 쪽수: 368쪽
- 내용: 국내 1호 프롬프트 엔지니어가 직접 작성한 실무 경험 및 노하우
- 특징: 실제 사례 중심, 50개 이상의 프롬프트 제작법 수록

**출처**: 
- [교보문고](https://product.kyobobook.co.kr/detail/S000213991301)
- [알라딘](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=344387842)
- [트레바리](https://m.trevari.co.kr/product/7224399178199724032)

### 주요 강의

**1. 강수진 박사의 프롬프트·컨텍스트 엔지니어링 아카데미 (패스트캠퍼스)**
- 기간: 약 25시간
- 구성: 10개 대주제, 약 60개 강의 클립
- 내용: 프롬프트 엔지니어링, 에이전트 설계, 멀티 에이전트, 컨텍스트 엔지니어링

**2. 프롬프트 엔지니어링 A to Z (패스트캠퍼스 온라인)**
- 대상: 국내 1호 프롬프트 엔지니어의 실무 노하우 수강자
- 특징: 50개 이상의 즉시 활용 가능한 프롬프트 제작법

**3. AI 시대 질문의 기술 (클래스 e, EBS)**
- 제목: 강수진의 AI 시대 질문의 기술, 프롬프트 엔지니어링
- 플랫폼: EBS 클래스 e

**4. YouTube 채널 - 프롬수진**
- 내용: LLM 기초, 프롬프트 전략
- 특징: 실무 기반 콘텐츠

**출처**: 
- [패스트캠퍼스 - 컨텍스트 엔지니어링 아카데미](https://fastcampus.co.kr/data_online_ksjprompt)
- [패스트캠퍼스 - 프롬프트 엔지니어링 A to Z](https://fastcampus.co.kr/data_online_no1prompt)
- [클래스 e](https://classe.ebs.co.kr/classe/detail/450136/40009039)

---

## 전문 에이전트 적용 제안

### 1. Context Engineering First 원칙
**에이전트 설계 시 적용**:
- 프롬프트 작성 전에 필요한 정보/도구 목록부터 정의
- "뭘 줄까" → "어떻게 구조화할까" → "프롬프트 작성"의 순서
- Memory 시스템 설계 시에도 Context Engineering 원칙 적용

### 2. RPI 워크플로우 기반 Subagent 설계
**적용 방안**:
- Create-expert: Research(리서치) → Plan(설계 기획) → Implement(에이전트 구축)
- Evolve-expert: 기존 에이전트 분석(Research) → 개선 계획(Plan) → 개선 구현(Implement)
- 각 단계별 산출물(파일)을 명확히 정의하여 context 분리

### 3. 40% 규칙 기반 Memory/Knowledge 시스템
**적용 방안**:
- MEMORY.md 최대 200줄 유지 (40% 규칙 준용)
- Knowledge 시스템에서 필요한 정보만 선별해 에이전트에 제공
- Context Window 초과 시 자동으로 참조 파일 분리

### 4. 한국어 프롬프트 최적화
**적용 방안**:
- 한국 사용자 대상 에이전트: 한국어의 맥락의존성 반영
- 조사·어미 선택의 미묘한 차이를 프롬프트에 명시
- 문화적 뉘앙스를 고려한 응답 포맷 설계

### 5. 언어학적 피드백 루프
**적용 방안**:
- 에이전트의 출력이 "인간의 언어"인지 "기계어 같은 언어"인지 검증
- 사용자 피드백을 대화분석 관점에서 분석
- 개선 사항을 언어학적 근거와 함께 기록

---

## 주의사항 및 한계

### 1. 한계
- **웹 접근 한계**: 강수진 박사의 공식 논문이나 상세 연구 자료는 공개되지 않은 상태
- **정확하지 않을 수 있음**: 유튜브, 강의, 인터뷰 등 공개된 자료는 미세한 변화가 있을 수 있음
- **Context Engineering 체계화 수준**: 아직 학계에서 표준화되지 않은 개념으로, 강수진 박사의 해석이 주요 참조

### 2. 해석상 주의
- Context Engineering의 3가지 원칙(무엇/언제/어떻게)은 강수진 박사의 주요 저서·강의 기반 해석
- 공식 문서가 아니므로 해석에 차이가 있을 수 있음

### 3. 한국어 프롬프트의 문맥성
- 한국어 특성상 문맥(context)이 매우 중요하므로 에이전트 설계 시 주의 필요
- 같은 프롬프트도 상황·청중·시점에 따라 효과가 크게 달라짐

---

## 참고 자료 링크

- [LinkedIn - Sujin Kang Ph.D.](https://www.linkedin.com/in/sujin-kang-ph-d-662444278/)
- [프롬프트 엔지니어's Work Journal (슬래시페이지)](https://slashpage.com/sujin-prompt-engineer?lang=ko)
- [AI 에이전트 설계의 핵심: 컨텍스트 엔지니어링 3가지 원칙](https://www.aiground.co.kr/ai-agent-context-engineering/)
- [패스트캠퍼스 - 컨텍스트 엔지니어링 아카데미](https://fastcampus.co.kr/data_online_ksjprompt)
- [책과나무 출판사](http://www.booknamu.com/contents/book_subject_view.html?page=&show_list=&book_cate=&idx=1233)

---

**최종 업데이트**: 2026-03-15  
**신뢰도**: 높음 (공식 저서, 강의, LinkedIn 프로필 기반)  
**작성자**: Claude Haiku 4.5 (Prompt Engineering Expert Agent)
