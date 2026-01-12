# Claude-Life

> Obsidian + Claude Code 통합 개인/팀 AI 시스템

프로젝트 관리, 태스크 관리, 지식 축적을 Claude Code와 Obsidian으로 통합 관리하는 시스템입니다.

## Features

- **자연어 우선**: 구조화된 명령 없이 자연어로 입력하면 알아서 분류/저장
- **스마트 트리거**: 버그, 에러 등 키워드 감지 시 자동 분석 제안
- **프로젝트 연동**: 개발 프로젝트의 CLAUDE.md에서 컨텍스트 참조
- **스킬 시스템**: 브리핑, 캡처, 동기화 등 재사용 가능한 스킬

## Quick Start

### 1. 요구사항

- [Obsidian](https://obsidian.md/) v1.10.1+
- [Claude Code](https://claude.ai/claude-code) (Claude Pro 구독)
- [TaskNotes 플러그인](https://github.com/TaskNotes/obsidian-tasknotes)

### 2. Obsidian Vault 설정

```bash
# Obsidian vault 생성 (Obsidian 앱에서)
# 또는 폴더 생성
mkdir ~/Develop/claude-life
cd ~/Develop/claude-life
```

### 3. 이 저장소 클론

```bash
# claude-life vault 폴더에서
git clone https://github.com/tadadak/claude-life.git .

# 또는 필요한 파일만 복사
cp -r claude-life-repo/.claude .
cp claude-life-repo/CLAUDE.md .
cp -r claude-life-repo/docs .
```

### 4. Obsidian 폴더 구조 생성

Obsidian에서 다음 폴더들을 생성하세요:

```
00-inbox/       # 빠른 캡처 (정리 전)
10-projects/    # 프로젝트 노트
20-tasks/       # TaskNotes 태스크 저장
30-areas/       # 기술/관심 영역 지식
40-logs/        # daily, weekly, meetings
50-people/      # 팀원/연락처 (선택)
90-bases/       # 커스텀 뷰 (선택)
```

### 5. TaskNotes 플러그인 설정

Obsidian Settings > Community Plugins > TaskNotes

| 설정 | 값 |
|------|-----|
| Task folder | `20-tasks` |
| HTTP API | Enable |
| Port | `8090` |

### 6. Claude Code 실행

```bash
cd ~/Develop/claude-life
claude
```

```
나: 브리핑
나: /sync  # 프로젝트 동기화
```

## Folder Structure

```
claude-life/
├── .claude/
│   └── skills/          # Claude Code 스킬
│       ├── core/        # briefing, capture, query
│       ├── project/     # context, sync, status
│       ├── review/      # daily, weekly
│       └── task/        # create, update, list
├── CLAUDE.md            # Claude Code 설정
├── docs/                # 설계 문서, 가이드
│
├── 00-inbox/            # 빠른 캡처
├── 10-projects/         # 프로젝트 노트
├── 20-tasks/            # TaskNotes 태스크
├── 30-areas/            # 기술/지식 영역
├── 40-logs/             # 일일/주간 기록
├── 50-people/           # 팀원 (선택)
└── 90-bases/            # 커스텀 뷰 (선택)
```

## Skills

| 스킬 | 트리거 | 용도 |
|------|--------|------|
| briefing | "브리핑", "오늘 뭐 해?" | 현황 요약 |
| capture | 자연어 입력 | 자동 분류 & 저장 |
| query | 검색 질문 | vault 검색 |
| sync | "/sync" | 프로젝트 동기화 |
| context | "/context [이름]" | 프로젝트 컨텍스트 로드 |

## Daily Workflow

### 하루 시작
```
나: 브리핑
```

### 작업 중 기록
```
나: 로그인 API 연동 완료. 토큰 갱신은 내일
나: 버그 발견 - 세션 만료 시 크래시
```

### 하루 끝
```
나: 오늘 정리해줘
```

## Project Integration

개발 프로젝트의 `CLAUDE.md`에 다음을 추가:

```markdown
## Context
이 프로젝트의 작업 기록과 컨텍스트:
- ~/Develop/claude-life/10-projects/{category}/{project}.md

세션 시작 시 위 파일을 확인하고 현재 상태를 파악하세요.
```

## Customization

### 폴더 구조 변경

`CLAUDE.md`의 Folder Structure 섹션 수정

### 스킬 추가/수정

`.claude/skills/` 폴더에 마크다운 파일 추가

### 프로젝트 카테고리

`10-projects/` 아래에 원하는 카테고리 폴더 생성:
- `personal/` - 개인 프로젝트
- `team/` - 팀 프로젝트
- `client/` - 클라이언트 프로젝트

## Documentation

- [설계 문서](docs/plans/design.md)
- [사용자 가이드](docs/guides/user-guide.md)
- [Obsidian 설정 가이드](docs/guides/obsidian-setup.md)

## License

MIT
