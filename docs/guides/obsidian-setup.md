# Obsidian 설정 가이드

## 1. Obsidian 설치

1. [obsidian.md](https://obsidian.md/) 에서 다운로드
2. 설치 및 실행

## 2. Vault 생성

1. Obsidian 실행
2. "Create new vault" 선택
3. Vault 이름: `claude-life` (또는 원하는 이름)
4. 위치: `~/Develop/claude-life` (또는 원하는 경로)

## 3. 폴더 구조 생성

Obsidian 좌측 파일 탐색기에서 우클릭 → "New folder"로 생성:

```
00-inbox/
10-projects/
20-tasks/
30-areas/
40-logs/
    daily/
    weekly/
    meetings/
50-people/      (선택)
90-bases/       (선택)
```

## 4. TaskNotes 플러그인 설치

1. Settings (⚙️) → Community plugins
2. "Browse" 클릭
3. "TaskNotes" 검색
4. Install → Enable

### TaskNotes 설정

Settings → Community plugins → TaskNotes → Options

| 설정 | 값 |
|------|-----|
| Task folder | `20-tasks` |
| Default status | `open` |
| Status options | `open, in-progress, blocked, done` |
| Enable HTTP API | ✅ On |
| HTTP Port | `8090` |

### API 확인

터미널에서:
```bash
curl http://127.0.0.1:8090/api/health
```

`{"status":"ok"}` 응답이 오면 성공

## 5. Claude-Life 파일 복사

### Git으로 클론

```bash
cd ~/Develop/claude-life
git clone https://github.com/tadadak/claude-life.git temp
mv temp/.claude .
mv temp/CLAUDE.md .
mv temp/docs .
rm -rf temp
```

### 또는 수동 복사

GitHub에서 다음 파일/폴더 다운로드:
- `.claude/` 폴더 전체
- `CLAUDE.md`
- `docs/` 폴더

## 6. 기본 설정 완료 확인

폴더 구조:
```
claude-life/
├── .obsidian/           ← Obsidian 자동 생성
├── .claude/
│   └── skills/
├── CLAUDE.md
├── docs/
├── 00-inbox/
├── 10-projects/
├── 20-tasks/
├── 30-areas/
├── 40-logs/
├── 50-people/
└── 90-bases/
```

## 7. Claude Code 실행

```bash
cd ~/Develop/claude-life
claude
```

처음 실행 시:
```
나: /sync
```

→ 기존 개발 프로젝트들 자동 스캔 & 등록

## 추가 설정 (선택)

### Obsidian 테마

Settings → Appearance → Theme
- 추천: Minimal, Things, Atom

### 유용한 플러그인

| 플러그인 | 용도 |
|----------|------|
| Calendar | 일일 노트 캘린더 뷰 |
| Dataview | 고급 쿼리 & 뷰 |
| Templater | 노트 템플릿 자동화 |
| Quick Switcher++ | 빠른 파일 전환 |

### 단축키 설정

Settings → Hotkeys

| 기능 | 추천 단축키 |
|------|-------------|
| Quick Switcher | `Cmd+O` |
| New note in folder | `Cmd+Shift+N` |
| Toggle sidebar | `Cmd+\` |

## 문제 해결

### TaskNotes API 응답 없음

1. Obsidian 재시작
2. TaskNotes 플러그인 비활성화 → 활성화
3. HTTP API 설정 확인

### Claude Code에서 vault 못 찾음

```bash
# CLAUDE.md 경로 확인
ls ~/Develop/claude-life/CLAUDE.md
```

### 태스크가 보이지 않음

1. `20-tasks/` 폴더 확인
2. TaskNotes Views 새로고침: `Cmd+R`
