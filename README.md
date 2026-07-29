# schedule-management

> 컴퓨팅 사고 기반 일정 관리 — 하루 일정을 실행 단위로 쪼개고 효율적인 순서로 배치해, 매일 아침 Slack으로 보고합니다.

커스텀 서버도, 직접 짠 코드도 없습니다. Claude Code의 스케줄(cron) 루틴이 매일 지정한 시각에 캘린더를 읽어 하루 계획을 만들고 Slack으로 보냅니다.

## 동작 방식

매일 아침, 루틴은 네 단계를 거칩니다.

1. **읽기** — Google Calendar에서 오늘 일정을 조회합니다.
2. **쪼개기** — 고정 일정과 유동 태스크를 판별하고, 유동 태스크를 실행 단위로 분해합니다.
3. **배치** — 정해둔 기준(`priorityRule`)에 따라 하루 순서를 정합니다.
4. **보고** — 완성된 하루 계획을 Slack으로 보냅니다.

### 리포트 예시

```
[오늘 하루 계획]

09:00-09:30 팀 주간 회의 (고정)
  1) 3분기 실적 데이터 정리 (45분)
  2) 보고서 초안 작성 (60분)
  
10:30-11:00 코드 리뷰 PR #482 (고정)
  3) PR #485 리뷰 (20분)
```

- **고정 일정** — 캘린더의 실제 시각을 그대로 표시합니다.
- **유동 태스크** — 순서와 예상 소요시간만 표시합니다.

## 시작하기

### 1. MCP 커넥터 연결

claude.ai에서 **Google Calendar**와 **Slack** 커넥터를 연결합니다. → https://claude.ai/customize/connectors

### 2. `config.json` 준비

`config.example.json`을 `config.json`으로 복사한 뒤 본인 값을 채웁니다. `config.json`은 `.gitignore`에 등록되어 있어 커밋되지 않습니다.

| 키 | 설명 |
|----|------|
| `calendarId` | 읽어올 캘린더 (보통 `"primary"`) |
| `slackChannel` | 리포트를 받을 채널 |
| `timezone` | 시간대 |
| `reportTime` / `reportDays` | 리포트 시각 / 요일 |
| `workingHours` | 태스크를 배치할 시간대. 이 밖의 시간은 비워둡니다 |
| `priorityRule` | 배치 기준. 자연어 문장으로 적습니다. 예: `"마감이 급한 일을 먼저 배치한다"` |
| `onOverflow` | 하루에 다 담기지 않는 태스크의 처리 방식 (기본 `"push"`) |

### 3. 루틴 생성 요청

Claude Code에서 이 저장소를 열고 `/schedule` 스킬로 요청합니다. `docs/specs/decompose.md`, `docs/specs/arrange.md`와 방금 채운 `config.json`을 함께 보여주며 "이 스펙대로 매일 아침 실행되는 스케줄 루틴을 만들어줘"라고 요청하면 됩니다.

### 4. 매일 아침 리포트 수신

이후로는 `config.json`에 적어둔 시각마다 루틴이 알아서 실행되어 하루 계획을 Slack으로 보냅니다. 배치 결과가 마음에 들지 않으면 `priorityRule` 문장만 고치면 됩니다.

## 더 읽기

- [decompose](docs/specs/decompose.md) — 고정 일정과 유동 태스크를 판별하고 실행 단위로 쪼개는 로직
- [arrange](docs/specs/arrange.md) — 쪼갠 단위를 배치하고 리포트로 만드는 로직
