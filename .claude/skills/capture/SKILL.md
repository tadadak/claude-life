---
name: capture
description: 자연어 입력을 분류하고 적절한 위치에 저장. 스마트 트리거로 자동 분석 제안.
---

# Capture Skill

자연어 입력을 자동으로 분류해서 저장합니다.

## Smart Triggers

다음 키워드 포함 시 **자동 분석 제안**:
- 버그, bug
- 문제, issue
- 막힘, stuck
- 에러, error
- 안됨, 안돼

스마트 트리거 시:
1. 먼저 태스크/기록 생성
2. "관련 정보 분석할까요?" 또는 자동 분석
3. 병렬 subagent로 Git 이슈, 코드, vault 검색

## Classification Rules

| 입력 패턴 | 분류 | 저장 위치 |
|-----------|------|-----------|
| "OOO 해야해", "OOO 해줘" | Task | 20-tasks/ (TaskNotes API) |
| "OOO 완료", "OOO 끝" | Task Update | status → done |
| "OOO 막혔어", "OOO blocked" | Task Update | status → blocked |
| "아이디어: OOO" | Idea | 00-inbox/ |
| "배운 것: OOO", "TIL: OOO" | Knowledge | 30-areas/{topic}.md |
| "인박스: OOO", "나중에: OOO" | Quick Capture | 00-inbox/ |
| 프로젝트 관련 기록 | Daily Log | 40-logs/daily/{date}.md |

## Task Creation via API

```bash
curl -X POST "http://127.0.0.1:8090/api/tasks" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "{extracted_title}",
    "status": "open",
    "priority": "{detected_priority}",
    "projects": ["[[{project_name}]]"],
    "due": "{extracted_date}",
    "contexts": ["{detected_context}"]
  }'
```

## Date Extraction

| 입력 | 변환 |
|------|------|
| 오늘 | {today} |
| 내일 | {tomorrow} |
| 모레 | {day_after_tomorrow} |
| 이번주 | {this_friday} |
| 다음주 | {next_monday} |

## Priority Detection

| 키워드 | Priority |
|--------|----------|
| 급함, 긴급, ASAP, 중요 | high |
| (default) | normal |
| 나중에, 여유있게 | low |

## Project Detection

1. 현재 작업 폴더 기반 추론
2. 명시적 언급 ("flutter-app에서", "project-x의")
3. 최근 작업 프로젝트

## Parallel Analysis (Smart Trigger)

스마트 트리거 시 병렬 실행:

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Task Create │  │ Git Scanner │  │ Vault Search│
└─────────────┘  └─────────────┘  └─────────────┘
      │                │                │
      └────────────────┴────────────────┘
                       ↓
              ┌─────────────────┐
              │ Suggestion Maker│
              └─────────────────┘
```

## Example Interactions

**Input**: "flutter 앱에서 세션 토큰 버그 발견, 내일까지 고쳐야해"

**Process**:
1. 키워드 "버그" 감지 → 스마트 트리거
2. 태스크 생성: title="세션 토큰 버그 수정", project=flutter-app, due=내일
3. 분석 제안 또는 자동 실행

**Output**:
```
✅ 태스크 생성: 세션 토큰 버그 수정 (flutter-app, 내일까지)
🔍 관련 정보 분석 중...

💡 분석 완료:
- 관련 이슈: #23 Session expires too fast
- 관련 파일: lib/auth/session_manager.dart
- 과거 기록: 12/20 비슷한 이슈 해결함
```
