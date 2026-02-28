# ドキュメント分析結果: Claude Code 利用ガイド

**分析日**: 2026-02-28
**対象読者**: Claude Code初心者〜中級者
**推奨スライド数**: 51枚（既存アウトライン準拠）

---

## 分析対象ドキュメント

| # | ドキュメント | 対応セクション | 要点数 |
|---|------------|--------------|--------|
| 1 | claude-code-intro.md | Section 0-2 | 12 |
| 2 | vscode-guide.md | Section 2-2, 8 | 15 |
| 3 | tmux-guide.md | Section 2-3 | 10 |
| 4 | slack-integration.md | Section 5 | 14 |
| 5 | cursor-migration.md | Section 11 | 11 |
| 6 | usage-limits-sandbox.md | Section 4, 6-7 | 16 |
| 7 | mcp-servers.md | Section 9 | 8 |
| 8 | skills-agents.md | Section 10 | 10 |

---

## セクション別要点

### Section 0: 表紙・イントロ（2枚）
- タイトル: Claude Code 利用ガイド & ベストプラクティス
- 目次: 13セクション構成、ストーリーライン提示

### Section 1: Claude Code とは（3枚）
- エージェント型AI（コード補完ではなくタスク委譲）
- Cursorとの比較: GUI vs CLI、エージェント性の違い
- 3モデル: Opus 4.6（最高性能）、Sonnet 4.6（バランス）、Haiku 4.5（軽量）

### Section 2: セットアップ（4枚）
- インストール: Native Installer推奨、認証はClaude.ai（Pro以上）
- VSCode拡張: セカンダリサイドバー配置、Ctrl+Escで起動
- tmux: ctコマンド（claude-tmux.sh）、Agent Teams対応
- /init: CLAUDE.md自動生成、基本コマンド一覧

### Section 3: CLAUDE.md（5枚）
- AIへの永続的な指示書（.cursorrules相当）
- 3層構造: グローバル→ワークスペース→プロジェクト
- 書き方: 300行以内、150〜200指示限界、@importで分割
- Bashルール例: シェル演算子禁止（Allowリスト連携）
- Cursor共存: symlink、setup-coexistence.sh

### Section 4: 権限とセキュリティ（5枚）
- 4つの権限モード: ask/auto/delegate/plan
- Allow/Denyリスト: パターンマッチ、設定例
- サンドボックス: OSレベル隔離、autoAllowBashIfSandboxed
- --dangerously-skip-permissions: ctでの自動付与、サンドボックス併用必須
- 機密ファイル保護: 二段構え（Denyリスト + Hook）

### Section 5: Hooks（6枚）
- イベント駆動スクリプト実行、command型 vs prompt型
- 全14イベント（主要5つ + その他）
- Slack連携: slack-feedback.sh、リアクション→指示変換、段階的待機
- Lint自動チェック: post-edit-lint.sh（MD/Shell/JSON）
- Push前チェック: pre-push-check.sh、exit code 2でブロック
- コンテキスト再注入: compact-reinject.sh

### Section 6: コンテキスト管理（4枚）
- トークン制限、キャッシュ（Write/Read 90%割引）
- ステータスラインの読み方（4行ダッシュボード）
- /compact: 情報圧縮、保持ルール指定
- ベストプラクティス: 部分読み、タスク分割、定期compact

### Section 7: 利用制限（4枚）
- 3種類: 5時間ローリング、7日間、モデル別
- 確認方法: ステータスライン（⚡📅🎵🎹）
- 最適化: キャッシュヒット率向上、Haiku活用
- 制限到達時: モデル切替、セッション分割

### Section 8: ステータスライン（3枚）
- settings.jsonのstatusLine設定
- 4行ダッシュボード: セッション/コンテキスト/制限/メモリ
- 色の意味: 緑=OK、黄=WATCH、赤=SWITCH

### Section 9: MCP（3枚）
- Model Context Protocol: 外部ツール連携の標準
- 設定: claude mcp addコマンド、3スコープ、OAuth 2.0
- おすすめ: Slack, GitHub, Sentry, Notion, Playwright, Context7

### Section 10: スキル & エージェント（4枚）
- スキル: /コマンドで呼べるプロンプト、SKILL.md
- スキル作り方: フロントマター、動的コンテキスト注入
- エージェント: 独立サブプロセス、ツール制限
- Agent Teams: 複数エージェント協調、Split Panes

### Section 11: Cursor移行（3枚）
- .cursorrules → CLAUDE.md変換、setup-coexistence.sh
- タイムリープ戦略: 両方の強みを使い分け
- 画像: Read toolで直接参照、ペーストキャッシュ

### Section 12: 通知（2枚）
- macOS通知: terminal-notifier + ccnotify.py
- Slack連携: リアクション駆動フロー（Section 5と連携）

### Section 13: まとめ（3枚）
- 推奨学習順序: CLAUDE.md→権限→コンテキスト→Hooks→MCP→Teams
- ベストプラクティス10選
- 参考リンク

---

## デザイン方針

- 白背景 + アクセントカラー（ダークブルー #2D5B8A）
- フォント: Hiragino Sans
- サイズ: タイトル36pt / 見出し28pt / 本文20pt / 注釈14pt
- コード例: ダークグレー背景のコードブロック
- 1スライド1主題、情報密度は16:9に最適化
