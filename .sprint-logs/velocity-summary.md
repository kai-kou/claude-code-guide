---
velocity_summary:
  last_updated: "2026-02-27"
  sprint_count: 2
  sprints:
    - id: "SPRINT-001"
      date: "2026-02-26"
      planned_sp: 15
      completed_sp: 15
      completion_rate: 100
    - id: "SPRINT-002"
      date: "2026-02-27"
      planned_sp: 16
      completed_sp: 16
      completion_rate: 100
  aggregated:
    avg_planned_sp: 15.5
    avg_completed_sp: 15.5
    avg_completion_rate: 100
    velocity_trend: "stable"
  po_efficiency:
    - sprint: "SPRINT-001"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 85
    - sprint: "SPRINT-002"
      po_wait_minutes: 5
      autonomous_rate: 100
      session_utilization: 85
  time_efficiency: []
  time_aggregated:
    avg_session_minutes: 0
    avg_execution_minutes: 0
    avg_minutes_per_sp: 0
    efficiency_trend: "N/A"
---

# ベロシティサマリー

> sprint-retro完了時に自動更新される、直近3スプリントのベロシティ集約ファイル。
> sprint-plannerがプランニング時にこのファイルを参照し、直近3スプリントログ全文の読み込みを不要にする。
> **自動更新**: sprint-retro Step 8.10 で更新される。手動編集は不要。

## 直近3スプリント

| # | スプリント | 日付 | 計画SP | 完了SP | 消化率 |
|---|-----------|------|--------|--------|--------|
| 1 | SPRINT-001 | 2026-02-26 | 15 | 15 | 100% |
| 2 | SPRINT-002 | 2026-02-27 | 16 | 16 | 100% |

## 集計

| 指標 | 値 |
|------|-----|
| 平均計画SP | 15.5 |
| 平均完了SP | 15.5 |
| 平均消化率 | 100% |
| ベロシティトレンド | stable |

## PO効率指標トレンド

| # | スプリント | PO待ち時間(分) | 自律実行率 | セッション有効率 |
|---|-----------|---------------|-----------|----------------|
| 1 | SPRINT-001 | 5 | 100% | 85% |
| 2 | SPRINT-002 | 5 | 100% | 85% |

## 時間効率トレンド

| # | スプリント | セッション時間(分) | 実行時間(分) | 分/SP |
|---|-----------|-------------------|-------------|-------|
| — | timing未記録のため計測不可 | — | — | — |

## 時間効率集計

| 指標 | 値 |
|------|-----|
| 平均セッション時間(分) | 0 |
| 平均実行時間(分) | 0 |
| 平均分/SP | 0 |
| 効率トレンド | N/A |
