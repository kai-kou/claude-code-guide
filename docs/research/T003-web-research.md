# T003: Web検索による最新情報収集 調査結果

**調査日**: 2026-02-26
**調査担当**: sprint-researcher (Claude Sonnet 4.5)

---

## 1. Claude Code 概要

- **定義**: Anthropicが提供するagentic coding tool
- **対応プラットフォーム**: Terminal CLI (macOS, Linux, WSL, Windows), VS Code拡張, JetBrains, デスクトップアプリ, Web版 (claude.ai/code)
- **対応モデル**: Opus 4.6, Sonnet 4.5, Haiku 4.5

---

## 2. CLAUDE.md ベストプラクティス

- `/init` で初期生成、段階的に改善
- **300行以内、指示は150〜200個に収める**（LLMのルール順守に限界あり）
- 簡潔に保つ:「これを削除するとClaudeが間違いを犯すか？」を基準にする
- `@import` で分割可能: `@README.md`, `@docs/git-instructions.md`
- 3階層: Enterprise → グローバル(`~/.claude/`) → プロジェクト(`.claude/`)

---

## 3. コンテキストウィンドウ管理

> コンテキストウィンドウは管理すべき最も重要なリソース

- 満杯に近づくとパフォーマンス低下、以前の指示を「忘れる」
- `/clear`: タスク間で頻繁に実行
- `/compact <instructions>`: 選択的圧縮（例: `/compact Focus on the API changes`）
- 自動コンパクション: 制限に近づくと自動実行
- CLAUDE.mdでcompact時の保持ルール定義可能

---

## 4. Hooks システム（全14イベント）

### よく使う5つ
1. **PreToolUse**: ツール実行前（検証・ブロック）
2. **PostToolUse**: ツール実行後（自動フォーマット等）
3. **Stop**: 停止時（Slack連携等）
4. **Notification**: 通知時
5. **SessionStart**: セッション開始時

### その他のイベント
- PermissionRequest, UserPromptSubmit, SubagentStop
- PreCompact, SessionEnd, TeammateIdle, TaskCompleted

### Hook種類
- **type: "command"**: bashコマンド実行
- **type: "prompt"**: LLMベースの評価（新機能）

### matcher
- 正確一致: `Write`
- 正規表現: `Edit|Write`
- 全ツール: `*` または空文字

---

## 5. MCP (Model Context Protocol)

### 追加方法
```bash
# リモートHTTP（推奨）
claude mcp add --transport http notion https://mcp.notion.com/mcp
# ローカルstdio
claude mcp add --transport stdio airtable -- npx -y airtable-mcp-server
```

### スコープ
- **local**: 現在のプロジェクトのみ
- **project**: `.mcp.json`（git管理）
- **user**: `~/.claude.json`（全プロジェクト）

### @ メンション参照
```
@github:issue://123 を分析して
@docs:file://api/authentication をレビュー
```

### 人気MCP
- GitHub, Sentry, Notion, PostgreSQL/DBHub, Figma, Linear, Slack

---

## 6. Agent Teams（実験的機能）

- **有効化**: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
- **構成**: リードエージェント + チームメイト

### 表示モード
- **in-process**: 1ターミナル内、Shift+Downで切り替え
- **split panes**: tmux/iTerm2で各チームメイト独立ペイン

### 制限事項
- セッション再開でin-processチームメイト復元不可
- 1セッション1チーム、ネスト不可
- split panesはVS Code統合ターミナルで非対応

---

## 7. カスタムスキル & エージェント

### スキル
- `SKILL.md` に指示記述
- 配置: `~/.claude/skills/` (Personal) / `.claude/skills/` (Project)
- フロントマター: name, description, allowed-tools, model, context等
- 動的コンテキスト注入: `` !`command` `` がスキル送信前に実行
- `/skill-name` で直接呼び出し

### Subagent
- 独自コンテキストウィンドウ、メイン会話と分離
- `.claude/agents/<name>.md` に定義
- tools, model 指定可能

---

## 8. VSCode拡張

- インラインdiff表示
- @メンション（ファイル参照）
- プラン確認
- 複数ペイン並行作業
- Ctrl+Esc: チャット起動

---

## 9. セキュリティ

### サンドボックス
- OSレベル分離、`/sandbox` で有効化

### --dangerously-skip-permissions
- 全許可チェックバイパス
- 推奨: サンドボックス内のみ
- AllowedToolsで制限可能: `--allowedTools "Edit,Bash(npm run lint)"`

---

## 10. 利用制限

### 5時間ローリングウィンドウ
- Pro: 5時間あたり約10〜40プロンプト
- Max 5x / Max 20x: Proの5/20倍

### 確認方法
- `/context` で使用状況確認
- Usage API（プログラマティック）

---

## 11. Plan Mode

### 推奨4フェーズ
1. **探索（Plan Mode）**: ファイル読み取り・質問応答
2. **計画（Plan Mode）**: 実装計画作成、Ctrl+Gで編集
3. **実装（Normal Mode）**: コーディング・テスト
4. **コミット**: 説明的メッセージでCP&PR

---

## 12. 推奨学習順序

1. CLAUDE.md（簡潔に保つ）
2. 基本パーミッション設定
3. コンテキスト管理（`/clear`、`/compact`）
4. Hooks（1〜2個の実用的hookから）
5. MCPサーバー（必要なもののみ）
6. Agent Teams・Subagents（複雑なタスクで）

---

## 調査ソース

### 公式ドキュメント
- [Claude Code 概要](https://code.claude.com/docs/ja/overview)
- [ベストプラクティス](https://code.claude.com/docs/ja/best-practices)
- [Hooksリファレンス](https://code.claude.com/docs/ja/hooks)
- [MCP接続](https://code.claude.com/docs/ja/mcp)
- [Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [スキル拡張](https://code.claude.com/docs/ja/skills)
- [設定](https://code.claude.com/docs/ja/settings)

### コミュニティ
- [CLAUDE.md運用のベストプラクティス](https://zenn.dev/imohuke/articles/claude-code-best-practices-2026)
- [Claude Codeベストプラクティスまとめ](https://speakerdeck.com/minorun365/claude-codebesutopurakuteisumatome)
- [公式ベストプラクティス完全解説](https://note.com/samurai_worker/n/ncf736866aab6)
- [Hooks解説](https://zenn.dev/buddypia/articles/99abea47607225)
- [Hooks解説 DevelopersIO](https://dev.classmethod.jp/articles/claude-code-hooks-basic-usage/)
- [使用量上限との付き合い方](https://zenn.dev/long910/articles/2026-02-21-claude-code-usage-limit)
- [settings.json設定ガイド](https://syu-m-5151.hatenablog.com/entry/2025/06/05/134147)
