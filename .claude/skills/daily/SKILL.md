---
name: daily
description: 일일 정리. USE WHEN user says "오늘 정리", "일일 정리", "하루 마무리", "end of day"
---

# Daily Review Skill

하루를 정리하고 로그를 작성합니다.

## Trigger

- 오늘 정리
- 일일 정리
- 하루 마무리
- end of day
- 오늘 뭐 했지?

## Steps

### 1. 오늘 완료된 태스크 확인

```bash
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg today "$(date +%Y-%m-%d)" \
  '[.[] | select(.status == "done" and .updated >= $today)]'
```

### 2. 오늘 작업한 프로젝트 파악

```bash
# 오늘 수정된 프로젝트 노트
find ~/Develop/claude-life/10-projects -name "*.md" -mtime 0 -type f
```

### 3. 기존 일일 로그 확인

```bash
today=$(date +%Y-%m-%d)
log_file=~/Develop/claude-life/40-logs/daily/${today}.md

if [ -f "$log_file" ]; then
  cat "$log_file"
fi
```

### 4. 사용자에게 질문

```
오늘 하루 정리할게요.

1. 오늘 기분은 어땠나요? (좋음/보통/별로)
2. 에너지 레벨은요? (높음/보통/낮음)
3. 특별히 기록할 것이 있나요?
```

### 5. 일일 로그 생성/업데이트

파일: `40-logs/daily/{YYYY-MM-DD}.md`

```yaml
---
type: daily
date: {today}
mood: {user_input}
energy: {user_input}
projects_touched:
  - "[[{project1}]]"
  - "[[{project2}]]"
tasks_completed: {count}
tasks_created: {count}
---

# {YYYY-MM-DD} {요일}

## 오늘 한 일

### {project1}
- {completed_task1}
- {completed_task2}

### {project2}
- {completed_task3}

## 배운 것 / 인사이트
{user_input_or_empty}

## 막힌 것 / 이슈
{blocked_tasks_summary}

## 내일 할 일
{tomorrow_tasks}
```

### 6. 프로젝트 노트 업데이트

작업한 프로젝트의 `last_worked` 업데이트:

```bash
# frontmatter의 last_worked 필드 업데이트
sed -i '' "s/^last_worked:.*/last_worked: $(date +%Y-%m-%d)/" \
  ~/Develop/claude-life/10-projects/{category}/{project}.md
```

## Output

```markdown
✅ 오늘 하루 정리 완료!

📊 오늘 요약
- 완료한 태스크: 5개
- 작업한 프로젝트: flutter-app, project-x
- 새로 생성한 태스크: 2개

📝 일일 로그 저장됨:
40-logs/daily/2025-01-11.md

🌙 오늘도 수고했어요!
내일 가장 중요한 일: {top_priority_task}
```
