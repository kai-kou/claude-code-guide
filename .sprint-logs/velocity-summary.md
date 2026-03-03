---
velocity_summary:
  last_updated: "2026-03-03"
  sprint_count: 6
  sprints:
    - id: "SPRINT-004"
      date: "2026-02-27"
      planned_sp: 16
      completed_sp: 16
      completion_rate: 100
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
  aggregated:
    avg_planned_sp: 16.3
    avg_completed_sp: 16.3
    avg_completion_rate: 100
    velocity_trend: "increasing"
  po_efficiency:
    - sprint: "SPRINT-004"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 93
    - sprint: "SPRINT-005"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 95
    - sprint: "SPRINT-006"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 52
  time_efficiency:
    - sprint: "SPRINT-004"
      session_minutes: 14
      execution_minutes: 13
      minutes_per_sp: 0.8
    - sprint: "SPRINT-005"
      session_minutes: 209
      execution_minutes: 205
      minutes_per_sp: 14.6
    - sprint: "SPRINT-006"
      session_minutes: 62
      execution_minutes: 32
      minutes_per_sp: 1.7
  time_aggregated:
    avg_session_minutes: 95
    avg_execution_minutes: 83
    avg_minutes_per_sp: 5.7
    efficiency_trend: "variable"
---

# ベロシティサマリー

> sprint-retro完了時に自動更新される、直近3スプリントのベロシティ集約ファイル。
> sprint-plannerがプランニング時にこのファイルを参照し、直近3スプリントログ全文の読み込みを不要にする。
> **自動更新**: sprint-retro Step 8.10 で更新される。手動編集は不要。

## 直近3スプリント

| # | スプリント | 日付 | 計画SP | 完了SP | 消化率 |
|---|-----------|------|--------|--------|--------|
| 1 | SPRINT-004 | 2026-02-27 | 16 | 16 | 100% |
| 2 | SPRINT-005 | 2026-03-02 | 14 | 14 | 100% |
| 3 | SPRINT-006 | 2026-03-03 | 19 | 19 | 100% |

## 集計

| 指標 | 値 |
|------|-----|
| 平均計画SP | 16.3 |
| 平均完了SP | 16.3 |
| 平均消化率 | 100% |
| ベロシティトレンド | increasing |

## PO効率指標トレンド

| # | スプリント | PO待ち時間(分) | 自律実行率 | セッション有効率 |
|---|-----------|---------------|-----------|----------------|
| 1 | SPRINT-004 | 5 | 100% | 93% |
| 2 | SPRINT-005 | 5 | 100% | 95% |
| 3 | SPRINT-006 | 5 | 100% | 52% |

## 時間効率トレンド

| # | スプリント | セッション時間(分) | 実行時間(分) | 分/SP |
|---|-----------|-------------------|-------------|-------|
| 1 | SPRINT-004 | 14 | 13 | 0.8 |
| 2 | SPRINT-005 | 209 | 205 | 14.6 |
| 3 | SPRINT-006 | 62 | 32 | 1.7 |

## 時間効率集計

| 指標 | 値 |
|------|-----|
| 平均セッション時間(分) | 95 |
| 平均実行時間(分) | 83 |
| 平均分/SP | 5.7 |
| 効率トレンド | variable |
