---
name: query
description: Vault 데이터 검색. USE WHEN user asks about projects, tasks, logs, or searches for information.
---

# Query Skill

vault 데이터를 검색하고 테이블로 표시합니다.

## IMPORTANT

**grep으로 frontmatter 스캔. 전체 파일 읽지 않기.**

## Tasks Query

### 모든 태스크

```bash
curl -s "http://127.0.0.1:8090/api/tasks"
```

### 상태별 필터

```bash
curl -s "http://127.0.0.1:8090/api/tasks?status=in-progress"
curl -s "http://127.0.0.1:8090/api/tasks?status=open"
curl -s "http://127.0.0.1:8090/api/tasks?status=blocked"
```

### 프로젝트별 필터

```bash
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq '[.[] | select(.projects[]? | contains("flutter-app"))]'
```

## Projects Query

```bash
for f in ~/Develop/claude-life/10-projects/**/*.md; do
  echo "=== $(basename "$f" .md) ==="
  grep "^status:\|^category:\|^next_milestone:\|^last_worked:" "$f" 2>/dev/null
done
```

**Output as table:**
| Project | Status | Category | Next Milestone | Last Worked |
|---------|--------|----------|----------------|-------------|

## Daily Logs Query

```bash
ls -t ~/Develop/claude-life/40-logs/daily/*.md | head -7 | while read f; do
  echo "=== $(basename "$f" .md) ==="
  grep "^projects_touched:\|^mood:\|^energy:" "$f" 2>/dev/null
done
```

## Areas Query

```bash
ls ~/Develop/claude-life/30-areas/*.md | while read f; do
  echo "=== $(basename "$f" .md) ==="
  grep "^category:\|^tags:" "$f" 2>/dev/null
done
```

## People Query

```bash
for f in ~/Develop/claude-life/50-people/*.md; do
  echo "=== $(basename "$f" .md) ==="
  grep "^role:\|^company:\|^last_contact:" "$f" 2>/dev/null
done
```

## Full-Text Search

특정 키워드로 vault 전체 검색:

```bash
grep -r "키워드" ~/Develop/claude-life/ \
  --include="*.md" \
  -l
```

## Output Format

항상 **마크다운 테이블**로 결과 표시.

```markdown
| Name | Status | Due | Project |
|------|--------|-----|---------|
| Task 1 | open | 2025-01-12 | flutter-app |
| Task 2 | in-progress | - | project-x |
```
