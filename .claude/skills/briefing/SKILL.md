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

### 4. 프로젝트 현황 (최근 활동 기준)

```bash
# active 프로젝트 중 실제 최근 작업한 것만 필터링
for f in $(grep -l "status: active" ~/Develop/claude-life/10-projects/**/*.md 2>/dev/null); do
  project_path=$(grep "^path:" "$f" | sed 's/path: //' | sed 's/~/$HOME/')
  if [ -d "$project_path" ]; then
    # 로컬 git에서 실제 마지막 커밋 날짜 확인
    last_commit=$(git -C "$project_path" log -1 --format="%ci" 2>/dev/null | cut -d' ' -f1)
    echo "$last_commit $f"
  fi
done | sort -r
```

**IMPORTANT - 추천 기준:**
- 기본: 최근 60일 이내 커밋이 있는 프로젝트만 추천
- 오래된 프로젝트(60일+)는 "휴면 프로젝트" 섹션에 별도 표시
- last_worked 대신 **실제 git 커밋 날짜** 기준으로 판단
- 사용자가 명시적으로 요청하지 않는 한 오래된 프로젝트는 추천하지 않음

### 5. 휴면 프로젝트 감지

```bash
# 60일 이상 커밋 없는 active 프로젝트 경고
cutoff=$(date -v-60d +%Y-%m-%d)  # macOS
# cutoff=$(date -d "60 days ago" +%Y-%m-%d)  # Linux
```

## Output Format

```markdown
📋 현재 상황 ({date} {day})

🔥 긴급 (오늘/내일 마감)
- [ ] {task} ({project}) - {due}

📌 진행중 (최근 60일 활동)
- [ ] {task} ({project}) - 마지막 커밋: {last_commit}

💡 최근 기록
- {date}: {note}

😴 휴면 프로젝트 (60일+ 비활동, status는 active)
- {project}: 마지막 커밋 {days_ago}일 전

🎯 추천: {suggestion}
```

## Notes

- 태스크 없으면 "오늘은 여유롭네요!" 메시지
- 막힌 것 있으면 해결 제안
- 오래된 in-progress 태스크 경고 (3일+)
- **60일 이상 비활동 프로젝트는 추천 대상에서 제외**
- 휴면 프로젝트는 별도 섹션에 표시 (정리 또는 status 변경 제안)
