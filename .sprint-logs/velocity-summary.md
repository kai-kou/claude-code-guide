---
velocity_summary:
  last_updated: "2026-03-06"
  sprint_count: 10
  sprints:
    - id: "SPRINT-008"
      date: "2026-03-03"
      planned_sp: 9
      completed_sp: 9
      completion_rate: 100
    - id: "SPRINT-009"
      date: "2026-03-03"
      planned_sp: 13
      completed_sp: 13
      completion_rate: 100
    - id: "SPRINT-010"
      date: "2026-03-06"
      planned_sp: 8
      completed_sp: 8
      completion_rate: 100
  aggregated:
    avg_planned_sp: 10.0
    avg_completed_sp: 10.0
    avg_completion_rate: 100
    velocity_trend: "stable"
  po_efficiency:
    - sprint: "SPRINT-008"
      po_wait_minutes: 2
      autonomous_rate: 100
      session_utilization: 95
    - sprint: "SPRINT-009"
      po_wait_minutes: 0
      autonomous_rate: 100
      session_utilization: 90
    - sprint: "SPRINT-010"
      po_wait_minutes: 3
      autonomous_rate: 100
      session_utilization: 90
  time_efficiency:
    - sprint: "SPRINT-008"
      session_minutes: 11
      execution_minutes: 8
      minutes_per_sp: 0.9
    - sprint: "SPRINT-009"
      session_minutes: 0
      execution_minutes: 0
      minutes_per_sp: 0
    - sprint: "SPRINT-010"
      session_minutes: 166
      execution_minutes: 155
      minutes_per_sp: 19.4
  time_aggregated:
    avg_session_minutes: 59
    avg_execution_minutes: 54
    avg_minutes_per_sp: 6.8
    efficiency_trend: "variable"
---

# ベロシティサマリー

> sprint-retro完了時に自動更新される、直近3スプリントのベロシティ集約ファイル。
> sprint-plannerがプランニング時にこのファイルを参照し、直近3スプリントログ全文の読み込みを不要にする。
> **自動更新**: sprint-retro Step 8.10 で更新される。手動編集は不要。

## 直近3スプリント

| # | スプリント | 日付 | 計画SP | 完了SP | 消化率 |
|---|-----------|------|--------|--------|--------|
| 1 | SPRINT-008 | 2026-03-03 | 9 | 9 | 100% |
| 2 | SPRINT-009 | 2026-03-03 | 13 | 13 | 100% |
| 3 | SPRINT-010 | 2026-03-06 | 8 | 8 | 100% |

## 集計

| 指標 | 値 |
|------|-----|
| 平均計画SP | 10.0 |
| 平均完了SP | 10.0 |
| 平均消化率 | 100% |
| ベロシティトレンド | stable |

## PO効率指標トレンド

| # | スプリント | PO待ち時間(分) | 自律実行率 | セッション有効率 |
|---|-----------|---------------|-----------|----------------|
| 1 | SPRINT-008 | 2 | 100% | 95% |
| 2 | SPRINT-009 | 0 | 100% | 90% |
| 3 | SPRINT-010 | 3 | 100% | 90% |

## 時間効率トレンド

| # | スプリント | セッション時間(分) | 実行時間(分) | 分/SP |
|---|-----------|-------------------|-------------|-------|
| 1 | SPRINT-008 | 11 | 8 | 0.9 |
| 2 | SPRINT-009 | - | - | 未記録 |
| 3 | SPRINT-010 | 166 | 155 | 19.4 |

> 注: SPRINT-009はtiming記録未実施。SPRINT-010はセッション跨ぎ（コンテキスト上限到達）を含むため分/SPが高い。

## 時間効率集計

| 指標 | 値 |
|------|-----|
| 平均セッション時間(分) | 59 |
| 平均実行時間(分) | 54 |
| 平均分/SP | 6.8 |
| 効率トレンド | variable |
