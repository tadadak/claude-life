---
name: project-status
description: 프로젝트 상태 업데이트. USE WHEN user says "/project-status", "프로젝트 상태", "상태 업데이트"
---

# Project Status Skill

프로젝트 노트의 상태를 업데이트합니다.

## Trigger

- "{project} 상태 업데이트"
- "{project} 마일스톤 완료"
- "{project} 목표 설정"
- "프로젝트 상태 업데이트" (일괄)
- "/project-status"
- "프로젝트 일시정지"
- "프로젝트 완료"

## Mode

### 단일 프로젝트 모드
특정 프로젝트 지정 시 해당 프로젝트만 업데이트

### 일괄 업데이트 모드 (권장)
프로젝트 미지정 시 → 최근 git 활동 기반 자동 감지

## Steps (일괄 모드)

### 1. 최근 활동 프로젝트 자동 감지

```bash
# 등록된 프로젝트 목록에서 path 추출
grep -h "^path:" ~/Develop/claude-life/10-projects/**/*.md 2>/dev/null | \
  sed 's/path: //' | while read project_path; do
    if [ -d "$project_path/.git" ]; then
      last_commit=$(git -C "$project_path" log -1 --format="%cs" 2>/dev/null)
      project_name=$(basename "$project_path")
      echo "$last_commit $project_name $project_path"
    fi
  done | sort -r | head -10
```

### 2. 각 프로젝트 정보 수집

```bash
# 프로젝트별 실행
project_path="$1"
project_name=$(basename "$project_path")

# 마지막 커밋 날짜
last_worked=$(git -C "$project_path" log -1 --format="%cs" 2>/dev/null)

# 현재 브랜치
current_branch=$(git -C "$project_path" branch --show-current 2>/dev/null)

# 최근 커밋 메시지 (컨텍스트용)
recent_commits=$(git -C "$project_path" log --oneline -3 2>/dev/null)
```

### 3. 프로젝트 노트 업데이트

```bash
project_file="~/Develop/claude-life/10-projects/{category}/{project}.md"

# last_worked 업데이트 (실제 git 커밋 날짜)
sed -i '' "s/^last_worked:.*/last_worked: $last_worked/" "$project_file"

# updated 날짜 (동기화 시점)
sed -i '' "s/^updated:.*/updated: $(date +%Y-%m-%d)/" "$project_file"
```

### 4. _index.md 동기화

프로젝트 업데이트 후 `10-projects/_index.md` 테이블도 갱신

## Actions (단일 모드)

### 상태 변경

```bash
# status 변경: active, paused, completed, dormant, archived
sed -i '' "s/^status:.*/status: {new_status}/" \
  ~/Develop/claude-life/10-projects/{category}/{project}.md
```

### 마일스톤 업데이트

```bash
# next_milestone 변경
sed -i '' "s/^next_milestone:.*/next_milestone: \"{new_milestone}\"/" \
  ~/Develop/claude-life/10-projects/{category}/{project}.md
```

### 목표 설정

```bash
# goal 변경
sed -i '' "s/^goal:.*/goal: \"{new_goal}\"/" \
  ~/Develop/claude-life/10-projects/{category}/{project}.md
```

## Output (일괄 모드)

```markdown
📊 프로젝트 상태 업데이트

| 프로젝트 | Last Worked | 변경사항 |
|----------|-------------|----------|
| project-a | 2026-01-15 | last_worked 갱신 |
| project-b | 2026-01-14 | last_worked 갱신 |

✅ {n}개 프로젝트 업데이트 완료
```

## Output (단일 모드)

```markdown
✅ {project} 상태 업데이트됨

| 항목 | 이전 | 현재 |
|------|------|------|
| status | active | paused |
| next_milestone | 로그인 | 테스트 작성 |
```

## Notes

- `last_worked`는 git 커밋 날짜 기준 (오늘 날짜 아님!)
- 60일 이상 비활동 시 자동으로 dormant 상태 제안
- 마일스톤 변경 시 사용자 입력 필요
