---
category: discussion
created_at: '2026-02-23T11:42:25.794325'
id: 20260223114225
tags:
- claude-code
- token-optimization
- settings
- til
title: Claude Code 토큰 최적화 — 8가지 적용 기법
updated_at: '2026-02-23T11:42:25.794325'
---

## 배경

YouTube 팁 영상 + "Claude Code 완전 가이드 70가지 팁" PDF(ykdojo)를 바탕으로  
`~/.claude` 설정에 직접 적용한 토큰 절약 기법 정리.

---

## 적용 완료

### 1. opusplan 모델
`settings.json`에 `"model": "opusplan"` 설정.  
계획·판단은 opus, 코드 작성·편집은 sonnet으로 자동 전환.

### 2. ENABLE_TOOL_SEARCH 환경변수
`"ENABLE_TOOL_SEARCH": "true"` → MCP 도구 설명 지연 로딩.  
매 턴 시스템 프롬프트에 모든 도구 설명이 포함되지 않아 컨텍스트 절약.

### 3. rules 파일 슬리밍
매 세션 로드되는 파일이므로 줄이면 매 턴 토큰 절약 효과.  
- `git-rules.md`: 157줄 → 55줄  
- `worklog-rules.md`: 116줄 → 52줄  
규칙 자체는 유지, 예시/설명 압축.

### 4. `!` prefix + /compact /clear 습관화
- 단순 셸 명령은 `!` prefix로 직접 실행 → Claude 컨텍스트 소비 0
- 작업 단위 마다 `/reflection` → `/clear` 로 context rot 방지

### 5. 공식 문서 우선 원칙
`.claudeignore`를 비공식 기능인지 모르고 적용했다가 삭제.  
교훈: 기능 적용 전 공식 문서 확인 필수, 불확실하면 "공식 확인 안 됨" 명시.

### 6. /dev 스킬 규모별 분기
불필요한 팀 스폰 방지:
- **Solo** (1-3파일): 메인 에이전트 직접 구현
- **Duo** (4-10파일): Dev 1명 스폰, 메인이 Lead 겸임
- **Team** (10+파일): 풀 팀 (Lead=opus, Dev/QA=sonnet)

### 7. /reflection 스킬 → HANDOFF.md → /clear 체인
세션 끝에 `MEMORY.md` 갱신 + `HANDOFF.md` 생성 후 `/clear`.  
다음 세션이 전 세션 컨텍스트 없이 HANDOFF만 읽어 이어받음.

### 8. 워크로그 소요 시간 → JSONL durationMs 합산
벽시계 시간(사용자 대기 포함) 대신  
`~/.claude/projects/<경로>/*.jsonl`의 `durationMs` 필드 합산 → 실제 Claude 작업 시간만 측정.  
헬퍼: `~/.claude/scripts/duration.py <snapshot_ts> <project_cwd>`

---

## 의도적으로 적용 안 한 것

| 항목 | 이유 |
|------|------|
| `.claudeignore` | 비공식. `.gitignore` 연동이 기본 동작 |
| Extended Thinking budget 조절 | Opus 4.6에서 0(비활성)만 작동, effort level이 자동 처리 |
| agents/commands 파일 슬리밍 | 팀 스폰·스킬 호출 시에만 로드 → 효과 낮음 |

---

## 참고
- 파일 제외 공식 방법: `.gitignore` 자동 연동 + `permissions.deny`
- `ignorePatterns`는 deprecated → `permissions.deny`로 대체
- `permissions.deny` 한계: 직접 Read만 차단, Bash grep 등 우회 가능