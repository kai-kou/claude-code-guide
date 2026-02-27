---
project:
  name: "claude-code-guide"
  title: "Claude Code 利用ガイド & ベストプラクティス"
  status: active
  priority: medium
  created: "2026-02-26"
  updated: "2026-02-27"
  owner: "kai.ko"
  tags: [claude-code, documentation, best-practices, tips, google-slides]
  summary: "Claude Code初心者向け利用ガイド・ベストプラクティス・TIPSのドキュメント作成（Googleスライドで共有）"
  next_action: "情報収集・構成案作成"
---

# Claude Code 利用ガイド & ベストプラクティス

## 概要

Claude Codeを初めて利用するユーザー向けに、利用ガイド・ベストプラクティス・TIPSを共有するためのドキュメントを作成する。最終成果物はGoogleスライドとして共有する。

## 想定読者

このガイドは、読者の経験レベルに応じて3段階の難易度で構成しています。自分のレベルに合ったドキュメントから読み始めてください。

| レベル | 対象者 | 推奨ドキュメント |
|-------|-------|----------------|
| ⭐ 初心者 | Claude Codeを初めて使う人、VSCodeでの基本利用を学びたい人 | VSCode利用ガイド、利用制限ガイド |
| ⭐⭐ 中級者 | 基本操作に慣れ、Hooks・MCP連携・Plan Modeを活用したい人 | Slack連携ガイド、tmux活用ガイド |
| ⭐⭐⭐ 上級者 | Agent Teams、カスタムスキル作成、チーム導入を検討している人 | Cursorからの移行ガイド |

### 前提知識

- **必須**: ターミナル操作（基本的なシェルコマンド）、VSCodeの基本操作
- **推奨**: Git操作、Node.js/npmの基本知識
- **対象OS**: macOS（一部のKeychainやtmux設定はmacOS固有）

## ゴール

- [ ] Claude Code利用ガイドのドキュメント完成
- [ ] Googleスライド形式での共有資料作成
- [ ] チームメンバーへの共有・フィードバック収集

## スコープ

### 含むもの
- VSCodeでの利用方法（セカンダリパネル、ステータスライン）
- tmux活用（ctコマンド紹介）
- hook活用したSlack連携・通知
- Cursorでできていたことの再現（タイムリープ戦略、画像貼り付け）
- 5時間・週制限の回避方法（ステータス確認方法）
- サンドボックス活用（bypass permissionsの使いどころ）
- コンテキストウインドウの把握・クリアタイミング
- 関連するおすすめ情報

### 含まないもの
- Claude Code以外のAIツールの詳細比較
- API利用方法（SDK等の開発者向け情報）
- 社内固有の業務フローとの統合手順

## 関連リソース

- マイルストーン: [milestones.md](./milestones.md)
- タスク一覧: [tasks.md](./tasks.md)

## メモ

- ローカルのグローバル設定（~/.claude/）、~/dev配下のプロジェクト設定を参照
- 必要に応じてWeb検索で最新情報を収集
- 最終的にGoogleスライド形式で共有
