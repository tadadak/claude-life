# Claude-Life: Personal AI System Design

> Obsidian + Claude Code 통합 개인 AI 시스템

---

## 1. 개요

### 1.1 목적

- 개발 프로젝트 관리
- 일상 생산성 (일정, 할 일, 습관)
- 지식 관리 (기술 노트, 인사이트 축적)
- 다른 프로젝트의 Claude Code에서 컨텍스트 참조

### 1.2 핵심 원칙

1. **자연어 우선**: 구조화된 명령 없이도 자연어로 입력하면 알아서 정리
2. **스마트 트리거**: 특정 키워드(버그, 문제, 막힘) 감지 시 자동 분석 제안
3. **병렬 처리**: Subagent로 분석 시간 최소화
4. **컨텍스트 연속성**: 프로젝트 간 작업 기록이 이어짐

---

## 2. 폴더 구조

```
claude-life/
├── .obsidian/              # Obsidian 설정 (자동 관리)
├── TaskNotes/
│   └── Views/              # TaskNotes 기본 뷰 (자동 생성)
│
├── .claude/
│   └── skills/             # Claude Code 스킬
│       ├── core/
│       │   ├── briefing.md
│       │   ├── capture.md
│       │   └── query.md
│       ├── project/
│       │   ├── context.md
│       │   ├── sync.md
│       │   └── status.md
│       ├── task/
│       │   ├── create.md
│       │   ├── update.md
│       │   └── list.md
│       └── review/
│           ├── daily.md
│           └── weekly.md
│
├── CLAUDE.md               # Claude Code 전역 설정
│
├── 00-inbox/               # 빠른 캡처 (정리 전)
├── 10-projects/
│   ├── _index.md           # 프로젝트 대시보드
│   ├── personal/           # 개인 프로젝트
│   └── team/               # 팀 프로젝트
│
├── 20-tasks/               # TaskNotes 태스크 저장 위치
│
├── 30-areas/               # 기술/관심 영역
│
├── 40-logs/
│   ├── daily/              # 일일 기록
│   ├── weekly/             # 주간 회고
│   └── meetings/           # 회의록
│
├── 50-people/              # 팀원/연락처
│
├── 90-bases/               # 커스텀 Bases 뷰 (추후)
│
└── docs/
    ├── plans/              # 설계 문서
    └── guides/             # 사용자 가이드
```

---

## 3. 스킬 시스템

### 3.1 스킬 구조

```
.claude/skills/
├── core/                    # 핵심 스킬
│   ├── briefing.md          # 브리핑 (지금 뭐 해야 해?)
│   ├── capture.md           # 자연어 → 구조화 캡처
│   └── query.md             # vault 검색
│
├── project/                 # 프로젝트 관련
│   ├── context.md           # 프로젝트 컨텍스트 로드
│   ├── sync.md              # ~/Develop/* 스캔 & 동기화
│   └── status.md            # 프로젝트 상태 업데이트
│
├── task/                    # 태스크 관련
│   ├── create.md            # 태스크 생성
│   ├── update.md            # 상태 변경
│   └── list.md              # 태스크 목록
│
└── review/                  # 회고/리뷰
    ├── daily.md             # 일일 정리
    └── weekly.md            # 주간 회고
```

---

## 4. Frontmatter 스키마

### Task (태스크)

```yaml
---
type: task
status: open | in-progress | blocked | done
priority: none | low | normal | high
project: "[[project-name]]"
due: 2025-01-12
scheduled: 2025-01-11
blocked_by: "설명"
tags:
  - bug
created: 2025-01-11
updated: 2025-01-11
---
```

### Project (프로젝트)

```yaml
---
type: project
status: active | paused | completed | archived
category: personal | team
repo: github.com/user/repo
path: ~/Develop/project
tech:
  - flutter
  - dart
has_claude_md: true
goal: "1월 MVP 출시"
next_milestone: "로그인 기능"
last_worked: 2025-01-11
started: 2025-01-01
updated: 2025-01-11
---
```

### Daily Log (일일 기록)

```yaml
---
type: daily
date: 2025-01-11
mood: good | neutral | bad
energy: high | medium | low
projects_touched:
  - "[[project-name]]"
---
```

---

## 5. 기술 스택

### 필수 요구사항

- Obsidian v1.10.1+
- Claude Code (Claude Pro 구독)
- TaskNotes 플러그인

### TaskNotes 설정

| 설정 | 값 |
|------|-----|
| Task folder | `20-tasks` |
| HTTP API | 활성화 |
| Port | `8090` |
| Status options | open, in-progress, blocked, done |

### API 엔드포인트

```
Base URL: http://127.0.0.1:8090/api

GET  /tasks              # 태스크 목록
POST /tasks              # 태스크 생성
PUT  /tasks/{path}       # 태스크 수정
DELETE /tasks/{path}     # 태스크 삭제
GET  /health             # 헬스체크
```
