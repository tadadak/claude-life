---
name: task-update
description: 태스크 업데이트. USE WHEN user says "태스크 완료", "done", "할일 수정", "우선순위 변경"
---

# Task Update Skill

기존 태스크의 상태, 우선순위, 내용을 업데이트합니다.

## Trigger

- {task} 완료
- {task} done
- 태스크 수정
- 우선순위 변경
- 마감일 변경
- {task} 취소

## Steps

### 1. 태스크 검색

```bash
# 키워드로 태스크 검색
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg keyword "{keyword}" \
  '[.[] | select(.description | ascii_downcase | contains($keyword | ascii_downcase))]'
```

### 2. 대상 확인

검색 결과가 여러 개면 사용자에게 확인:

```markdown
🔍 "{keyword}" 관련 태스크:

1. [todo] 로그인 버그 수정 (flutter-app)
2. [in-progress] 로그인 UI 개선 (flutter-app)

어떤 태스크를 업데이트할까요?
```

### 3. 업데이트 타입 판단

| 키워드 | 업데이트 |
|--------|----------|
| 완료, done, 끝 | status: "done" |
| 시작, start | status: "in-progress" |
| 취소, cancel | status: "cancelled" |
| 보류, hold | status: "blocked" |
| 긴급, urgent | priority: "highest" |
| 내일까지 | due: 내일 날짜 |

### 4. API 호출

```bash
# 태스크 파일 경로 확인
task_file="20-tasks/{task_id}.md"

# frontmatter 업데이트
# status 변경
sed -i '' "s/^status:.*/status: {new_status}/" \
  ~/Develop/claude-life/$task_file

# completed 날짜 추가 (완료 시)
if [ "{new_status}" = "done" ]; then
  sed -i '' "s/^---$/---\ncompleted: $(date +%Y-%m-%d)/" \
    ~/Develop/claude-life/$task_file
fi
```

또는 TaskNotes API (지원되는 경우):

```bash
curl -X PATCH "http://127.0.0.1:8090/api/tasks/{task_id}" \
  -H "Content-Type: application/json" \
  -d '{"status": "{new_status}"}'
```

### 5. 연관 업데이트

태스크 완료 시:
- 프로젝트 노트의 last_worked 업데이트
- 오늘 일일 로그에 기록

## Output

```markdown
✅ 태스크 업데이트됨

| 항목 | 이전 | 현재 |
|------|------|------|
| 상태 | todo | done |
| 완료일 | - | 2025-01-11 |

🎉 오늘 완료한 태스크: 3개
```

## Batch Update

여러 태스크 한번에 완료:

**입력**: "오늘 한 일 다 완료 처리해줘"

```bash
# 오늘 작업한 in-progress 태스크 검색
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq '[.[] | select(.status == "in-progress")]'
```

확인 후 일괄 업데이트
