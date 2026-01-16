---
name: project-status
description: 프로젝트 상태 업데이트. USE WHEN user updates project status, milestone, or goal.
---

# Project Status Skill

프로젝트 노트의 상태를 업데이트합니다.

## Trigger

- "{project} 상태 업데이트"
- "{project} 마일스톤 완료"
- "{project} 목표 설정"
- "프로젝트 일시정지"
- "프로젝트 완료"

## Actions

### 상태 변경

```bash
# status 변경: active, paused, completed, archived
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

### last_worked 업데이트

```bash
sed -i '' "s/^last_worked:.*/last_worked: $(date +%Y-%m-%d)/" \
  ~/Develop/claude-life/10-projects/{category}/{project}.md
```

## Output

```markdown
✅ {project} 상태 업데이트됨

| 항목 | 이전 | 현재 |
|------|------|------|
| status | active | paused |
| next_milestone | 로그인 | 테스트 작성 |
```
