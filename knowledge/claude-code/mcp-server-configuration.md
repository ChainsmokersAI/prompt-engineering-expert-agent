# MCP Server Configuration

> 최종 업데이트: 2026-03-16
> 출처: Claude Code 공식 문서 (https://code.claude.com/docs/en/mcp.md, https://code.claude.com/docs/en/settings.md), GitHub Issue #4976
> 신뢰도: 최고 (공식 문서)

## 개요

Claude Code에서 MCP(Model Context Protocol) 서버 설정은 scope에 따라 다른 파일에 저장됩니다. `.claude/settings.json`은 MCP 설정에 사용되지 않습니다.

## Scope별 설정 파일

| Scope | 파일 위치 | 용도 | 공유 |
|-------|-----------|------|------|
| Project | `.mcp.json` (프로젝트 루트) | 팀 공유용 MCP 서버 | git 커밋 가능 |
| Local (기본값) | `~/.claude.json` (프로젝트 경로별) | 개인용, 해당 프로젝트에서만 | 불가 |
| User | `~/.claude.json` | 개인용, 모든 프로젝트에서 | 불가 |
| Managed | `managed-mcp.json` (시스템 디렉토리) | IT 배포용 (Enterprise) | 조직 관리 |

## 주요 명령어

```bash
# MCP 서버 추가 (project scope)
claude mcp add -s project -e KEY=value -- <name> <command> [args...]

# MCP 서버 목록 및 상태 확인
claude mcp list

# 등록된 MCP 서버 제거
claude mcp remove <name>

# 프로젝트 MCP 승인 초기화
claude mcp reset-project-choices
```

## .mcp.json 구조

```json
{
  "mcpServers": {
    "<server-name>": {
      "type": "stdio",
      "command": "<command>",
      "args": ["<arg1>", "<arg2>"],
      "env": {
        "KEY": "value"
      }
    }
  }
}
```

## 주요 특성

- **환경 변수 확장**: `.mcp.json`에서 `${VAR}` 또는 `${VAR:-default}` 문법 지원
- **보안**: project scope 서버는 첫 사용 시 사용자 승인 필요
- **Transport 유형**: stdio (기본값), sse, http 지원

## 혼동 주의

- `.claude/settings.json`은 MCP 설정에 사용되지 않습니다 (이전 문서 오류로 혼란 발생, GitHub Issue #4976에서 수정됨)
- `claude mcp add` 실행 시 scope에 따라 자동으로 올바른 파일에 기록됩니다
