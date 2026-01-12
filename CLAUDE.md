# CLAUDE.md - Claude-Life AI System

## Overview

이 vault는 개인/팀 AI 시스템입니다. 프로젝트 관리, 태스크 관리, 지식 축적을 담당합니다.

## Folder Structure

```
00-inbox/       # 빠른 캡처 (정리 전)
10-projects/    # 프로젝트 노트 (카테고리별 하위 폴더)
20-tasks/       # TaskNotes 태스크 저장
30-areas/       # 기술/관심 영역 지식
40-logs/        # daily, weekly, meetings
50-people/      # 팀원/연락처
90-bases/       # 커스텀 Bases 뷰
```

## Skills

스킬 파일들은 `.claude/skills/`에 있습니다:

| 스킬 | 트리거 | 용도 |
|------|--------|------|
| briefing | "브리핑", "오늘 뭐 해?" | 현황 요약 |
| capture | 자연어 입력 | 자동 분류 & 저장 |
| query | 검색 질문 | vault 검색 |
| sync | "/sync" | 프로젝트 동기화 |
| context | "/context [이름]" | 프로젝트 컨텍스트 로드 |

## TaskNotes API

```
Base URL: http://127.0.0.1:8090/api

GET  /tasks              # 태스크 목록
POST /tasks              # 태스크 생성
PUT  /tasks/{path}       # 태스크 수정
```

## Smart Triggers

다음 키워드 포함 시 자동 분석 제안:
- 버그, bug, 문제, issue, 막힘, stuck, 에러, error

## Output Format

데이터 조회 시 **마크다운 테이블**로 표시.
frontmatter는 grep으로 빠르게 스캔.

## Project Integration

각 프로젝트의 CLAUDE.md에서 이 vault를 참조합니다.
프로젝트 컨텍스트는 `10-projects/`에서 관리합니다.

### 프로젝트 CLAUDE.md 예시

```markdown
## Context
이 프로젝트의 작업 기록과 컨텍스트:
- ~/Develop/claude-life/10-projects/{category}/{project}.md

세션 시작 시 위 파일을 확인하고 현재 상태를 파악하세요.
```

## GitHub 연동 (선택)

여러 GitHub 계정/조직 사용 시 PAT 전환 스크립트:

```bash
# ~/.env.github 예시
export GITHUB_TOKEN_PERSONAL="ghp_xxx"
export GITHUB_TOKEN_TEAM="ghp_yyy"

# gh-auto 함수
gh-auto() {
  local repo="$2"
  if [[ "$repo" == *"team-org"* ]]; then
    GH_TOKEN=$GITHUB_TOKEN_TEAM gh "$@"
  else
    GH_TOKEN=$GITHUB_TOKEN_PERSONAL gh "$@"
  fi
}
```

## User Guide

상세 사용법: `docs/guides/user-guide.md`
