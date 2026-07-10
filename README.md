# schedule-management

컴퓨팅 사고 기반 일정 관리 — 캘린더 일정을 실행 단위로 쪼개고 효율적인 순서로 배치해서 매일 아침 Slack으로 보고합니다.

## 어떻게 동작하나요

커스텀 서버나 코드가 없습니다. Claude Code의 스케줄(cron) 루틴이 매일 지정 시각에:

1. Google Calendar에서 오늘 일정을 읽고 (read-only)
2. 고정 일정 vs 유동 태스크를 판별하고, 유동 태스크는 실행 단위로 분해하고
3. 본인이 정한 기준(`priorityRule`)에 따라 하루 순서를 배치해서
4. Slack으로 보고합니다

로직 상세는 [docs/specs/decompose.md](docs/specs/decompose.md), [docs/specs/arrange.md](docs/specs/arrange.md) 참고. 배경은 [docs/ideas/daily-decompose-report.md](docs/ideas/daily-decompose-report.md).

## 나만의 루틴 만들기

1. claude.ai에서 Google Calendar, Slack MCP 커넥터를 연결하세요 (https://claude.ai/customize/connectors)
2. `config.example.json`을 `config.json`으로 복사하고 본인 값을 채우세요 (커밋하지 마세요 — 이미 `.gitignore`에 포함됨)
3. Claude Code에서 이 저장소를 열고, `docs/specs/decompose.md`·`docs/specs/arrange.md`와 본인의 `config.json`을 보여주면서 "이 스펙대로 매일 아침 실행되는 스케줄 루틴을 만들어줘"라고 요청하세요 (`/schedule` 스킬 사용)
4. Claude가 본인 캘린더/Slack 채널/배치 기준을 반영한 루틴을 만들어줍니다

## config.json 필드

- `calendarId`: 읽을 캘린더 (보통 `"primary"`)
- `slackChannel`: 리포트 받을 채널
- `timezone`: 시간대
- `reportTime` / `reportDays`: 리포트 시각/요일
- `priorityRule`: 본인이 원하는 배치 기준 (자연어 문장 — 예: "마감이 급한 일을 먼저 배치한다")
- `onOverflow`: 하루에 다 못 들어가는 태스크 처리 방식 (`"push"` 기본)
