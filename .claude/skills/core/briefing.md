---
name: briefing
description: 현황 브리핑. USE WHEN user says "브리핑", "오늘 뭐 해?", "what should I do", "현황", "할 일"
---

# Briefing Skill

현재 상황과 오늘 할 일을 요약해서 보여줍니다.

## Trigger Keywords

- 브리핑
- 오늘 뭐 해?
- 오늘 할 일
- what should I do
- 현황
- 지금 뭐 해야 해?

## Steps

### 1. 긴급 태스크 확인

```bash
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq '[.[] | select(.status != "done" and .due != null)] | sort_by(.due)'
```

오늘/내일 마감인 것 필터링.

### 2. 진행중 태스크

```bash
curl -s "http://127.0.0.1:8090/api/tasks?status=in-progress"
```

### 3. 최근 기록 확인

```bash
ls -t ~/Develop/claude-life/40-logs/daily/*.md | head -3
```

최근 3일 로그에서 막힌 점, 중요 기록 추출.

### 4. 프로젝트 현황

```bash
grep -l "status: active" ~/Develop/claude-life/10-projects/**/*.md 2>/dev/null
```

활성 프로젝트 수 확인.

## Output Format

```markdown
📋 현재 상황 ({date} {day})

🔥 긴급 (오늘/내일 마감)
- [ ] {task} ({project}) - {due}

📌 진행중
- [ ] {task} ({project})

💡 최근 기록
- {date}: {note}

🎯 추천: {suggestion}
```

## Notes

- 태스크 없으면 "오늘은 여유롭네요!" 메시지
- 막힌 것 있으면 해결 제안
- 오래된 in-progress 태스크 경고 (3일+)
