---
name: task-list
description: 태스크 목록 조회. USE WHEN user says "할일 목록", "태스크 보여줘", "오늘 할일", "뭐 해야해?"
---

# Task List Skill

다양한 조건으로 태스크를 조회합니다.

## Trigger

- 할일 목록
- 태스크 보여줘
- 오늘 할일
- 뭐 해야해?
- {project} 태스크
- 급한 거 뭐 있어?
- 막힌 태스크

## Query Types

### 오늘 할 일

```bash
today=$(date +%Y-%m-%d)
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg today "$today" \
  '[.[] | select(
    (.status == "todo" or .status == "in-progress") and
    (.due == null or .due <= $today)
  )] | sort_by(.priority)'
```

### 프로젝트별

```bash
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg proj "{project}" \
  '[.[] | select(
    .status != "done" and
    (.projects[]? | ascii_downcase | contains($proj | ascii_downcase))
  )]'
```

### 우선순위별

```bash
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq '[.[] | select(.status != "done" and .priority == "highest")]'
```

### 막힌 태스크

```bash
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq '[.[] | select(.status == "blocked")]'
```

### 기한 임박

```bash
week_later=$(date -v+7d +%Y-%m-%d)
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg week "$week_later" \
  '[.[] | select(
    .status != "done" and
    .due != null and
    .due <= $week
  )] | sort_by(.due)'
```

### 최근 완료

```bash
week_ago=$(date -v-7d +%Y-%m-%d)
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg since "$week_ago" \
  '[.[] | select(.status == "done" and .completed >= $since)]'
```

## Output Format

### 기본 목록

```markdown
📋 할일 목록

### 🔴 긴급 (highest)
| 태스크 | 프로젝트 | 마감 |
|--------|----------|------|
| 로그인 버그 수정 | flutter-app | 오늘 |

### 🟠 높음 (high)
| 태스크 | 프로젝트 | 마감 |
|--------|----------|------|
| API 문서 작성 | project-x | 1/15 |

### 🟡 보통 (medium)
| 태스크 | 프로젝트 | 마감 |
|--------|----------|------|
| 테스트 코드 작성 | flutter-app | - |

---
📊 총 12개 | 긴급 1 | 높음 3 | 보통 8
```

### 프로젝트별 목록

```markdown
📋 flutter-app 태스크

| 상태 | 태스크 | 우선순위 | 마감 |
|------|--------|----------|------|
| 🔵 진행중 | UI 리팩토링 | high | - |
| ⚪ 할일 | 테스트 작성 | medium | 1/20 |
| 🔴 막힘 | 배포 설정 | high | - |

---
진행중 1 | 할일 5 | 막힘 1
```

### 오늘 집중

```markdown
🎯 오늘 집중할 태스크

1. 🔴 로그인 버그 수정 (flutter-app) - 마감 오늘!
2. 🟠 API 문서 검토 (project-x)
3. 🟡 코드 리뷰 (team-project)

💡 추천: 1번부터 처리하세요. 마감이 오늘입니다.
```

## Interactive Mode

태스크 선택 후 작업:

```markdown
할일 목록입니다. 어떤 작업을 할까요?

1. 로그인 버그 수정 → 시작하기
2. API 문서 작성 → 상세 보기
3. + 새 태스크 추가
```
