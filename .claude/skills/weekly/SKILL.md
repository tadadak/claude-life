---
name: weekly
description: 주간 회고. USE WHEN user says "주간 회고", "weekly review", "이번주 정리"
---

# Weekly Review Skill

한 주를 회고하고 다음 주를 계획합니다.

## Trigger

- 주간 회고
- weekly review
- 이번주 정리
- 이번주 뭐 했지?

## Steps

### 1. 이번 주 일일 로그 수집

```bash
# 최근 7일 로그
ls -t ~/Develop/claude-life/40-logs/daily/*.md | head -7
```

각 로그에서:
- mood, energy 패턴
- projects_touched
- tasks_completed 합계

### 2. 이번 주 완료된 태스크

```bash
week_ago=$(date -v-7d +%Y-%m-%d)
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg since "$week_ago" \
  '[.[] | select(.status == "done" and .updated >= $since)]'
```

### 3. 프로젝트별 진척

```bash
for f in ~/Develop/claude-life/10-projects/**/*.md; do
  name=$(basename "$f" .md)
  status=$(grep "^status:" "$f" | cut -d: -f2 | xargs)
  last=$(grep "^last_worked:" "$f" | cut -d: -f2 | xargs)
  echo "$name | $status | $last"
done
```

### 4. 막힌 태스크 분석

```bash
curl -s "http://127.0.0.1:8090/api/tasks?status=blocked"
```

### 5. 사용자에게 질문 (AI가 어려운 질문)

```
이번 주를 돌아볼게요.

1. 이번 주 가장 큰 성과는 뭐였나요?
2. 이번 주 아쉬웠던 점이 있다면?
3. 시간 배분은 잘 됐나요? (예: 회의가 너무 많았다)
4. 다음 주 가장 중요한 목표는?
```

### 6. 주간 로그 생성

파일: `40-logs/weekly/{YYYY}-W{WW}.md`

```yaml
---
type: weekly
week: {YYYY}-W{WW}
date_range: {start_date} ~ {end_date}
projects_touched:
  - "[[{project1}]]"
  - "[[{project2}]]"
tasks_completed: {total_count}
tasks_created: {total_count}
mood_avg: {average}
energy_avg: {average}
---

# {YYYY} Week {WW}

## 이번 주 요약

### 완료한 것들
| 프로젝트 | 완료 태스크 |
|---------|-------------|
| flutter-app | 5개 |
| project-x | 3개 |

### 주요 성과
- {user_input_achievement}

### 프로젝트 현황
| 프로젝트 | 상태 | 진척 |
|---------|------|------|
| flutter-app | active | 로그인 기능 완료 |
| project-x | active | API 설계 중 |

## 회고

### 잘한 것
{user_input}

### 개선할 것
{user_input}

### 배운 것
{extracted_from_daily_logs}

## 막힌 것들
{blocked_tasks_list}

## 다음 주 계획

### 목표
{user_input_goal}

### 우선순위 태스크
1. {task1}
2. {task2}
3. {task3}

## 패턴 분석

### 기분/에너지
- 평균 기분: {mood_avg}
- 평균 에너지: {energy_avg}
- 패턴: {detected_pattern}

### 시간 배분
- 가장 많이 작업한 프로젝트: {top_project}
- 작업 일수: {work_days}
```

## Output

```markdown
✅ 주간 회고 완료!

📊 이번 주 요약 (W02)
- 완료한 태스크: 15개
- 작업한 프로젝트: 4개
- 평균 기분: 좋음
- 평균 에너지: 보통

🏆 이번 주 성과
- flutter-app 로그인 기능 완료
- project-x API 설계 80% 진행

⚠️ 주의 필요
- blocked 태스크 2개 (3일+ 방치)
- project-y 1주일 미작업

📝 주간 로그 저장됨:
40-logs/weekly/2025-W02.md

🎯 다음 주 포커스: {next_week_goal}
```

## AI Hard Questions

주간 회고에서 AI가 던지는 질문들:

- "이번 주 가장 많은 시간을 쓴 프로젝트가 가장 중요한 프로젝트인가요?"
- "blocked된 태스크가 3개 있는데, 이게 계속 방치되는 이유가 있나요?"
- "지난주 목표했던 OOO는 어떻게 됐나요?"
- "에너지가 낮았던 날들에 공통점이 있나요?"
