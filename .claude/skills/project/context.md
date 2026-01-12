---
name: context
description: 프로젝트 컨텍스트 로드. USE WHEN user says "/context [project]", "[project] 현황", "[project] 상태"
---

# Context Skill

특정 프로젝트의 상세 컨텍스트를 로드합니다.

## Trigger

- /context {project_name}
- {project_name} 현황
- {project_name} 상태
- {project_name} 컨텍스트

## Steps

### 1. 프로젝트 노트 찾기

```bash
# 프로젝트명으로 검색
find ~/Develop/claude-life/10-projects -name "*{project_name}*.md" -type f
```

### 2. 프로젝트 노트 읽기

전체 내용 읽기 (컨텍스트 로드이므로 전체 필요)

```bash
cat ~/Develop/claude-life/10-projects/{category}/{project_name}.md
```

### 3. 관련 태스크 검색

```bash
curl -s "http://127.0.0.1:8090/api/tasks" | \
  jq --arg proj "{project_name}" \
  '[.[] | select(.status != "done" and (.projects[]? | ascii_downcase | contains($proj | ascii_downcase)))]'
```

### 4. 최근 로그에서 관련 기록

```bash
# 최근 7일 로그에서 프로젝트 언급 검색
for f in $(ls -t ~/Develop/claude-life/40-logs/daily/*.md | head -7); do
  if grep -q "{project_name}" "$f" 2>/dev/null; then
    echo "=== $(basename "$f" .md) ==="
    grep -A2 -B2 "{project_name}" "$f"
  fi
done
```

### 5. GitHub 이슈 스캔 (선택)

프로젝트에 repo 설정되어 있으면:

```bash
# GitHub CLI 사용
gh issue list --repo {owner}/{repo} --state open --limit 10
```

**여러 GitHub 계정 사용 시:**
PAT 자동 전환 스크립트 설정 (CLAUDE.md 참고)

### 6. 관련 코드 파일 (막힌 점 있을 때)

막힌 점에 키워드가 있으면 관련 파일 검색:

```bash
# 프로젝트 폴더에서 키워드 검색
grep -r "{keyword}" {project_path}/lib --include="*.dart" -l | head -5
grep -r "{keyword}" {project_path}/src --include="*.{js,ts,java,py}" -l | head -5
```

## Output Format

```markdown
📋 {project_name} 컨텍스트

## 기본 정보
| 항목 | 값 |
|------|-----|
| 상태 | {status} |
| 카테고리 | {category} |
| 기술 스택 | {tech} |
| 목표 | {goal} |
| 다음 마일스톤 | {next_milestone} |
| 마지막 작업 | {last_worked} |

## 현재 상태
{current_status_from_note}

## 막힌 점
{blocked_items}

## 관련 태스크
| 태스크 | 상태 | 마감 |
|--------|------|------|
| {task1} | {status} | {due} |

## 최근 기록
- {date1}: {log1}
- {date2}: {log2}

## GitHub 이슈 (열린 것)
| # | 제목 | 라벨 |
|---|------|------|
| #23 | {title} | {labels} |

## 제안
🎯 {suggestion_based_on_context}
```

## Parallel Execution

상세 분석 시 병렬 실행:

```
┌─────────────────┐
│  프로젝트 노트   │
└────────┬────────┘
         │
    ┌────┴────┬────────────┐
    │         │            │
    ▼         ▼            ▼
┌───────┐ ┌───────┐ ┌──────────┐
│Tasks  │ │ Logs  │ │ GitHub   │
│Search │ │Search │ │ Issues   │
└───────┘ └───────┘ └──────────┘
         │
    ┌────┴────┐
    │ 종합    │
    └─────────┘
```
