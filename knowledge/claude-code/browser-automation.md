# Claude Code 브라우저 자동화

> 최종 업데이트: 2026-03-17 | 출처: Claude Code 공식 문서 (2026-03-17 리서치) | 신뢰도: 높음

## 두 가지 방식

### 1. Chrome 확장 (beta, 공식)

- **실행**: `claude --chrome` 또는 세션 내 `/chrome`
- **요구사항**: Chrome/Edge + "Claude in Chrome" 확장 v1.0.36+, Claude Code v2.0.73+
- **특징**: 실시간 시각적 실행, 로그인 상태 공유, 렌더링된 화면을 Claude가 직접 인식
- **적합**: 일회성 탐색, 시각적 판단이 중요한 작업, 인증 필요 서비스

### 2. Playwright MCP

- **설정**: `claude mcp add --transport stdio playwright -- npx -y @playwright/mcp@latest`
- **프로젝트 레벨 (.mcp.json)**:
  ```json
  {
    "mcpServers": {
      "playwright": {
        "type": "stdio",
        "command": "npx",
        "args": ["-y", "@playwright/mcp@latest"]
      }
    }
  }
  ```
- **특징**: 프로그래밍적 제어, DOM 대기(`waitForSelector`) 명시적 제어, headless 실행 가능
- **적합**: 반복 루프 워크플로우, 배치 데이터 수집

## 선택 기준

| 시나리오 | 추천 |
|----------|------|
| 일회성 탐색/추출 | Chrome 확장 |
| 반복 루프 자동화 | Playwright MCP |
| Edge Case 시각적 판단 | Chrome 확장 |
| 동적 로딩 사이트 반복 수집 | Playwright MCP |

## WebFetch 차단 사이트 대응

- WebFetch는 특정 도메인 접근 차단 (예: `*.land.naver.com`)
- Chrome 확장/Playwright MCP 모두 실제 브라우저 세션을 사용하므로 접근 가능
- Claude + Playwright MCP는 하드코딩 스크립트가 아닌 LLM 판단 기반 → Edge Case에도 어느 정도 적응 가능

## 참고

- 네이버 부동산 등 동적 SPA 사이트는 Playwright의 명시적 대기 제어가 유리
- 반복 워크플로우 시 실패 건은 스킵/로깅 방어 로직 포함 권장
