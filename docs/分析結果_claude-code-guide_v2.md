# 分析結果: Claude Code 利用ガイド v2

**分析日**: 2026-03-02
**対象読者**: エンジニア（Claude Code初心者〜中級者）

## 対象ドキュメント

| # | ファイル | 主要トピック | ボリューム |
|---|---------|------------|----------|
| 1 | claude-code-intro.md | 概要・Cursor比較・モデル・インストール | 大 |
| 2 | vscode-guide.md | VSCode拡張・ステータスライン・ダッシュボード | 大 |
| 3 | tmux-guide.md | ctコマンド・Agent Teams・セッション管理 | 中 |
| 4 | slack-integration.md | Hooks全14種・Slack連携・Lint・Push前チェック | 大 |
| 5 | cursor-migration.md | 移行手順・タイムリープ戦略・画像・Plan Mode | 大 |
| 6 | usage-limits-sandbox.md | 権限・サンドボックス・コンテキスト・制限 | 最大 |
| 7 | mcp-servers.md | MCP概要・設定方法・おすすめ6選 | 中 |
| 8 | skills-agents.md | スキル・エージェント・Agent Teams | 中 |

## 要点抽出（対象読者向け）

### 初級（すぐ使い始めるための情報）
- Claude Codeとは何か・Cursorとの違い
- インストール・認証・初回起動
- VSCode拡張のセットアップ・レイアウト
- CLAUDE.mdの概念と3層構造
- 基本コマンド（/init, /model, /plan, /compact, /clear）

### 中級（効率的に使いこなすための情報）
- 権限モード（ask/auto/delegate/plan）
- Allow/Denyリスト・サンドボックス
- コンテキストウィンドウの仕組み・管理方法
- 利用制限の種類・確認方法・最適化
- Hooks（Slack連携・Lint・Push前チェック）
- MCPサーバー（Slack/GitHub/Notion等）

### 上級（高度な活用）
- カスタムスキル・SKILL.mdの書き方
- カスタムエージェント・ツール制限
- Agent Teams・並列タスク実行
- tmux + ct コマンド
- --dangerously-skip-permissions の正しい使い方

## 推奨スライド構成

全13セクション・48コンテンツスライド + セクション区切り

| セクション | 枚数 | テーマ |
|-----------|------|--------|
| 0: 表紙・目次 | 2 | イントロ |
| 1: Claude Codeとは | 3 | 概要・比較・モデル |
| 2: セットアップ | 4 | インストール・VSCode・tmux・/init |
| 3: CLAUDE.md | 5 | 概念・3層・コツ・実例・共存 |
| 4: 権限とセキュリティ | 5 | モード・Allow/Deny・サンドボックス・機密保護 |
| 5: Hooks | 6 | 概念・イベント・Slack・Lint・Push・コンテキスト |
| 6: コンテキスト管理 | 4 | 仕組み・ステータスライン・/compact・ベストプラクティス |
| 7: 利用制限 | 4 | 種類・確認・最適化・対応 |
| 9: MCP | 3 | 概要・設定・おすすめ6選 |
| 10: スキル&エージェント | 4 | スキル・作り方・エージェント・Teams |
| 11: Cursor移行 | 3 | 手順・タイムリープ・画像 |
| 12: 通知 | 2 | macOS通知・Slack連携フロー |
| 13: まとめ | 3 | ロードマップ・ベストプラクティス・参考リンク |
| **合計** | **48** | |
