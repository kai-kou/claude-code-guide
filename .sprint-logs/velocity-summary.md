---
velocity_summary:
  last_updated: "2026-03-03"
  sprint_count: 7
  sprints:
    - id: "SPRINT-005"
      date: "2026-03-02"
      planned_sp: 14
      completed_sp: 14
      completion_rate: 100
    - id: "SPRINT-006"
      date: "2026-03-03"
      planned_sp: 19
      completed_sp: 19
      completion_rate: 100
    - id: "SPRINT-007"
      date: "2026-03-03"
      planned_sp: 8
      completed_sp: 8
      completion_rate: 100
  aggregated:
    avg_planned_sp: 13.7
    avg_completed_sp: 13.7
    avg_completion_rate: 100
    velocity_trend: "variable"
  po_efficiency:
    - sprint: "SPRINT-005"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 95
    - sprint: "SPRINT-006"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 52
    - sprint: "SPRINT-007"
      po_wait_minutes: 2
      autonomous_rate: 100
      session_utilization: 95
  time_efficiency:
    - sprint: "SPRINT-005"
      session_minutes: 209
      execution_minutes: 205
      minutes_per_sp: 14.6
    - sprint: "SPRINT-006"
      session_minutes: 62
      execution_minutes: 32
      minutes_per_sp: 1.7
    - sprint: "SPRINT-007"
      session_minutes: 7
      execution_minutes: 4
      minutes_per_sp: 0.5
  time_aggregated:
    avg_session_minutes: 93
    avg_execution_minutes: 80
    avg_minutes_per_sp: 5.6
    efficiency_trend: "variable"
---

# ベロシティサマリー

> sprint-retro完了時に自動更新される、直近3スプリントのベロシティ集約ファイル。
> sprint-plannerがプランニング時にこのファイルを参照し、直近3スプリントログ全文の読み込みを不要にする。
> **自動更新**: sprint-retro Step 8.10 で更新される。手動編集は不要。

## 直近3スプリント

| # | スプリント | 日付 | 計画SP | 完了SP | 消化率 |
|---|-----------|------|--------|--------|--------|
| 1 | SPRINT-005 | 2026-03-02 | 14 | 14 | 100% |
| 2 | SPRINT-006 | 2026-03-03 | 19 | 19 | 100% |
| 3 | SPRINT-007 | 2026-03-03 | 8 | 8 | 100% |

## 集計

| 指標 | 値 |
|------|-----|
| 平均計画SP | 13.7 |
| 平均完了SP | 13.7 |
| 平均消化率 | 100% |
| ベロシティトレンド | variable |

## PO効率指標トレンド

| # | スプリント | PO待ち時間(分) | 自律実行率 | セッション有効率 |
|---|-----------|---------------|-----------|----------------|
| 1 | SPRINT-005 | 5 | 100% | 95% |
| 2 | SPRINT-006 | 5 | 100% | 52% |
| 3 | SPRINT-007 | 2 | 100% | 95% |

## 時間効率トレンド

| # | スプリント | セッション時間(分) | 実行時間(分) | 分/SP |
|---|-----------|-------------------|-------------|-------|
| 1 | SPRINT-005 | 209 | 205 | 14.6 |
| 2 | SPRINT-006 | 62 | 32 | 1.7 |
| 3 | SPRINT-007 | 7 | 4 | 0.5 |

## 時間効率集計

| 指標 | 値 |
|------|-----|
| 平均セッション時間(分) | 93 |
| 平均実行時間(分) | 80 |
| 平均分/SP | 5.6 |
| 効率トレンド | variable |
