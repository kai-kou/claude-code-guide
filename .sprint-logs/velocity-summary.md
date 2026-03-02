---
velocity_summary:
  last_updated: "2026-03-02"
  sprint_count: 5
  sprints:
    - id: "SPRINT-003"
      date: "2026-02-27"
      planned_sp: 12
      completed_sp: 12
      completion_rate: 100
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
  aggregated:
    avg_planned_sp: 14.0
    avg_completed_sp: 14.0
    avg_completion_rate: 100
    velocity_trend: "stable"
  po_efficiency:
    - sprint: "SPRINT-003"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 85
    - sprint: "SPRINT-004"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 93
    - sprint: "SPRINT-005"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 95
  time_efficiency:
    - sprint: "SPRINT-004"
      session_minutes: 14
      execution_minutes: 13
      minutes_per_sp: 0.8
    - sprint: "SPRINT-005"
      session_minutes: 209
      execution_minutes: 205
      minutes_per_sp: 14.6
  time_aggregated:
    avg_session_minutes: 112
    avg_execution_minutes: 109
    avg_minutes_per_sp: 7.7
    efficiency_trend: "variable"
---

# ベロシティサマリー

> sprint-retro完了時に自動更新される、直近3スプリントのベロシティ集約ファイル。
> sprint-plannerがプランニング時にこのファイルを参照し、直近3スプリントログ全文の読み込みを不要にする。
> **自動更新**: sprint-retro Step 8.10 で更新される。手動編集は不要。

## 直近3スプリント

| # | スプリント | 日付 | 計画SP | 完了SP | 消化率 |
|---|-----------|------|--------|--------|--------|
| 1 | SPRINT-003 | 2026-02-27 | 12 | 12 | 100% |
| 2 | SPRINT-004 | 2026-02-27 | 16 | 16 | 100% |
| 3 | SPRINT-005 | 2026-03-02 | 14 | 14 | 100% |

## 集計

| 指標 | 値 |
|------|-----|
| 平均計画SP | 14.0 |
| 平均完了SP | 14.0 |
| 平均消化率 | 100% |
| ベロシティトレンド | stable |

## PO効率指標トレンド

| # | スプリント | PO待ち時間(分) | 自律実行率 | セッション有効率 |
|---|-----------|---------------|-----------|----------------|
| 1 | SPRINT-003 | 5 | 100% | 85% |
| 2 | SPRINT-004 | 5 | 100% | 93% |
| 3 | SPRINT-005 | 5 | 100% | 95% |

## 時間効率トレンド

| # | スプリント | セッション時間(分) | 実行時間(分) | 分/SP |
|---|-----------|-------------------|-------------|-------|
| 1 | SPRINT-004 | 14 | 13 | 0.8 |
| 2 | SPRINT-005 | 209 | 205 | 14.6 |

## 時間効率集計

| 指標 | 値 |
|------|-----|
| 平均セッション時間(分) | 112 |
| 平均実行時間(分) | 109 |
| 平均分/SP | 7.7 |
| 効率トレンド | variable |
