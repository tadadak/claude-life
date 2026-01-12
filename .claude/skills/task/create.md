---
name: task-create
description: 태스크 생성. USE WHEN user says "할일 추가", "태스크 생성", "todo", "{task} 해야해"
---

# Task Create Skill

TaskNotes API를 통해 새 태스크를 생성합니다.

## Trigger

- 할일 추가
- 태스크 생성
- {description} 해야해
- {description} 해야 돼
- todo: {description}
- add task

## Steps

### 1. 태스크 정보 파싱

사용자 입력에서 추출:
- **description**: 태스크 내용
- **project**: 프로젝트명 (언급된 경우)
- **priority**: 우선순위 (긴급, 중요 등 키워드)
- **due**: 마감일 (오늘, 내일, 이번주 등)

### 2. 우선순위 매핑

| 키워드 | priority |
|--------|----------|
| 긴급, 급함, ASAP | highest |
| 중요, 꼭 | high |
| (기본) | medium |
| 나중에, 여유있을때 | low |
| 언젠가 | lowest |

### 3. 마감일 계산

| 키워드 | due 계산 |
|--------|----------|
| 오늘 | $(date +%Y-%m-%d) |
| 내일 | $(date -v+1d +%Y-%m-%d) |
| 이번주 | $(date -v+fri +%Y-%m-%d) |
| 다음주 | $(date -v+1w +%Y-%m-%d) |
| {N}일 후 | $(date -v+{N}d +%Y-%m-%d) |

### 4. 프로젝트 확인

프로젝트명이 언급된 경우:

```bash
# 프로젝트 존재 확인
find ~/Develop/claude-life/10-projects -name "*{project}*.md" -type f
```

존재하면 projects 배열에 추가

### 5. TaskNotes API 호출

```bash
curl -X POST "http://127.0.0.1:8090/api/tasks" \
  -H "Content-Type: application/json" \
  -d '{
    "description": "{description}",
    "status": "todo",
    "priority": "{priority}",
    "due": "{due_date}",
    "projects": ["{project}"],
    "tags": ["{extracted_tags}"]
  }'
```

### 6. 생성 확인

API 응답에서 생성된 태스크 ID 확인

## Output

```markdown
✅ 태스크 생성됨

| 항목 | 값 |
|------|-----|
| 내용 | {description} |
| 우선순위 | {priority} |
| 마감 | {due} |
| 프로젝트 | {project} |

📝 파일: 20-tasks/{task_id}.md
```

## Examples

**입력**: "flutter-app 로그인 버그 수정해야해 긴급"

**처리**:
- description: "로그인 버그 수정"
- project: "flutter-app"
- priority: "highest" (긴급)
- due: null

**입력**: "내일까지 API 문서 작성"

**처리**:
- description: "API 문서 작성"
- priority: "medium"
- due: "2025-01-12"
