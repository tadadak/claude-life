---
name: sync
description: 프로젝트 동기화. USE WHEN user says "/sync", "프로젝트 동기화", "sync projects"
---

# Sync Skill

프로젝트 폴더를 스캔해서 등록/업데이트합니다.

## Trigger

- /sync
- 프로젝트 동기화
- sync projects
- 프로젝트 등록

## Steps

### 1. 프로젝트 폴더 스캔

```bash
find ~/Develop/projects -maxdepth 1 -type d | while read dir; do
  if [ -d "$dir/.git" ] || [ -f "$dir/package.json" ] || [ -f "$dir/pubspec.yaml" ] || [ -f "$dir/pom.xml" ] || [ -f "$dir/build.gradle" ]; then
    echo "$dir"
  fi
done
```

### 2. 프로젝트 정보 추출

각 프로젝트에서:

**Git Remote URL**:
```bash
git -C "$project_path" config --get remote.origin.url 2>/dev/null
```

**최근 작업 브랜치 확인 (로컬 + 원격)**:
```bash
# 로컬 브랜치 (최근 커밋순)
git -C "$project_path" for-each-ref --sort=-committerdate \
  --format='%(committerdate:short) %(refname:short)' refs/heads/ | head -5

# 현재 브랜치와 최근 커밋
git -C "$project_path" branch --show-current
git -C "$project_path" log --oneline -3

# 원격 브랜치 (GitHub API)
# gh api repos/{owner}/{repo}/branches --jq '.[] | .name'
```

**IMPORTANT**: 원격만 보면 안 됨! 로컬에서 작업 중인 브랜치가 가장 최신일 수 있음.

**Tech Stack 감지**:
| 파일 | Tech |
|------|------|
| pubspec.yaml | flutter, dart |
| package.json | node, javascript/typescript |
| pom.xml | java, maven |
| build.gradle | java/kotlin, gradle |
| requirements.txt | python |
| Cargo.toml | rust |
| go.mod | go |

**CLAUDE.md 존재 여부**:
```bash
[ -f "$project_path/CLAUDE.md" ] && echo "true" || echo "false"
```

**실제 마지막 작업일 (중요!)**:
```bash
# last_worked는 오늘 날짜가 아니라 실제 마지막 커밋 날짜!
git -C "$project_path" log -1 --format="%cs" 2>/dev/null
# 결과 예: 2025-12-30
```

**휴면 상태 자동 감지**:
```bash
last_commit=$(git -C "$project_path" log -1 --format="%cs" 2>/dev/null)
cutoff=$(date -v-60d +%Y-%m-%d)  # 60일 전
if [[ "$last_commit" < "$cutoff" ]]; then
  status="dormant"  # 60일 이상 비활동 → 휴면
else
  status="active"
fi
```

### 3. 카테고리 결정

```bash
# Git remote URL 기반으로 카테고리 결정
# 예: organization별 또는 프로젝트 유형별로 분류
if [[ "$remote_url" == *"your-org"* ]]; then
  category="work"
else
  category="personal"  # default
fi
```

### 4. 프로젝트 노트 생성/업데이트

경로: `10-projects/{category}/{project_name}.md`

```yaml
---
type: project
status: active
category: {category}
repo: {remote_url}
path: {project_path}
tech:
  - {detected_tech}
has_claude_md: {true/false}
goal: ""
next_milestone: ""
last_worked: {today}
started: {today}
updated: {today}
---

# {project_name}

## 현재 상태

*첫 동기화 - 상태를 입력해주세요*

## 최근 작업 기록

- {today}: 프로젝트 등록됨

## 막힌 점

*없음*

## 다음 할 일

*설정 필요*
```

### 5. CLAUDE.md 생성 제안

CLAUDE.md 없는 프로젝트 발견 시:

```
다음 프로젝트에 CLAUDE.md가 없습니다:
- ~/Develop/projects/app-name
- ~/Develop/projects/script-name

CLAUDE.md를 생성할까요? (y/n)
```

생성 시 템플릿:

```markdown
# Project: {project_name}

## Context
이 프로젝트의 작업 기록과 컨텍스트:
- ~/Develop/claude-life/10-projects/{category}/{project_name}.md

세션 시작 시 위 파일을 확인하고 현재 상태를 파악하세요.

## 작업 기록
작업 완료 시 claude-life vault에 기록:
- 오늘 한 일
- 막힌 점
- 다음 할 일

## Tech Stack
{detected_tech_list}

## Commands
- Build: {detected_build_command}
- Test: {detected_test_command}
- Run: {detected_run_command}
```

### 6. 인덱스 업데이트

`10-projects/_index.md` 업데이트:
- 프로젝트 목록 테이블 갱신
- 통계 업데이트

## Output

```markdown
✅ 프로젝트 동기화 완료

📁 발견된 프로젝트: 8개
- 신규 등록: 2개
- 업데이트: 5개
- 변경 없음: 1개

⚠️ CLAUDE.md 없음: 3개
- projects/app-name
- projects/script-name
- projects/util-name

CLAUDE.md 생성할까요?
```
