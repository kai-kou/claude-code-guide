# T001: ローカル設定の調査結果

**調査日**: 2026-02-26
**対象**: `~/.claude/` 配下の構造・設定

---

## ディレクトリ構造概要

```
~/.claude/
├── CLAUDE.md                    # グローバルユーザールール
├── settings.json                # メイン設定（権限・hooks・sandbox）
├── settings.local.json          # ローカル設定オーバーライド
├── keybindings.json             # キーバインド設定
├── statusline-command.sh        # ステータスライン（4行ダッシュボード）
├── try-stock.md                 # Tryストック
├── history.jsonl                # セッション履歴
├── stats-cache.json             # 統計キャッシュ
├── mcp-needs-auth-cache.json    # MCP認証キャッシュ
│
├── agents/                      # エージェント定義（44ファイル）
├── skills/                      # スキル定義（45ディレクトリ）
├── hooks/                       # フックスクリプト（6ファイル）
├── scripts/                     # ユーティリティスクリプト（3ファイル）
├── ccnotify/                    # 通知システム（SQLite + Python）
├── templates/                   # スクラムテンプレート（8ファイル）
├── rules/                       # 共有ルール（2ファイル）
├── plugins/                     # 公式プラグインマーケットプレイス
├── config/                      # 通知状態管理
│
├── projects/                    # プロジェクト別設定
├── plans/                       # プランファイル
├── cache/                       # 利用制限キャッシュ
├── state/                       # セッション状態（Slack feedback等）
├── session-env/                 # セッション環境
├── file-history/                # ファイル変更履歴
├── shell-snapshots/             # シェルスナップショット
├── todos/                       # Todoリスト
├── ide/                         # IDE連携
├── debug/                       # デバッグログ
├── logs/                        # ログ
├── telemetry/                   # テレメトリ
├── backups/                     # バックアップ
├── downloads/                   # ダウンロード
├── paste-cache/                 # ペーストキャッシュ
├── references/                  # 参照資料
├── tools/                       # ツール
├── usage-data/                  # 利用データ
├── tasks/                       # チームタスク管理
└── teams/                       # チーム管理
```

---

## 1. コア設定ファイル

### CLAUDE.md（グローバルルール）

ユーザーの全プロジェクト共通ルール。主な内容:

- **基本方針**: 日本語応答、ねこキャラクター、フレンドリー対応
- **ショートカット**: `cp` = コミット&push、`ns` = 次のスプリント開始
- **技術環境**: zsh、pyenv、nvm、ghコマンド
- **Bashルール（最重要）**: シェル演算子（`&&`, `||`, `;`, `|`）を一切使用禁止。1コマンド=1Bash呼び出しの徹底
- **Git操作ルール**: ユーザー確認なしのコミット&push禁止
- **完全性保証**: 技術的完成度100% + ユーザー合意の両方が必要
- **プロジェクト管理**: `/Users/kai.ko/dev/` 配下のワークスペース構造・スラッシュコマンド体系

### settings.json

```json
{
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" },
  "permissions": {
    "defaultMode": "delegate",
    "allow": [ /* 75+のBashコマンドパターン, MCP tools, etc. */ ],
    "deny": [ "Bash(rm -rf /)", "Bash(rm -rf /*)", "Bash(sudo *)" ]
  },
  "hooks": { /* 6種類のhookイベント */ },
  "statusLine": { "type": "command", "command": "bash ~/.claude/statusline-command.sh" },
  "language": "日本語",
  "sandbox": { "enabled": true, "autoAllowBashIfSandboxed": true, "allowUnsandboxedCommands": true },
  "autoUpdatesChannel": "stable",
  "skipDangerousModePermissionPrompt": true
}
```

**権限モード**: `delegate`（チーム委譲モード）
**サンドボックス**: 有効化（sandboxed時はBash自動許可）

---

## 2. Hooks システム（6スクリプト）

### イベント → Hook マッピング

| Hook Event | Matcher | スクリプト | 役割 |
|-----------|---------|-----------|------|
| UserPromptSubmit | - | ccnotify.py | プロンプト記録（SQLite） |
| SessionStart | compact | compact-reinject.sh | コンパクション後のコンテキスト再注入 |
| PostToolUse | Edit\|Write | post-edit-lint.sh | 編集後の構文チェック |
| Notification | - | slack-feedback.sh + ccnotify.py | Slack通知 + Mac通知 |
| Stop | - | slack-feedback.sh + ccnotify.py | Slack完了投稿 + フィードバックポーリング |
| PreToolUse | Bash | pre-push-check.sh | git push前チェック |
| PreToolUse | Edit\|Write | protect-sensitive-files.sh | 機密ファイル保護 |

### 各Hookの詳細

#### slack-feedback.sh（Stop/Notification）
- タスク完了時にSlack `#kai-cursor-times` に投稿
- **段階的待機**: 1分→3分→5分の3ラウンドでフィードバック待ち
- リアクション→指示変換: 👍=LGTM/CP, 👀=レビュー, 🔁=やり直し, ❌=破棄, 🚀=テスト, 📝=ドキュメント
- スレッド返信でもフィードバック可能
- tmuxステータスバーに待機状態を表示

#### compact-reinject.sh（SessionStart/compact）
- コンテキストウィンドウ圧縮後に重要情報を再注入
- 再注入内容: プロジェクト名、ブランチ、直近5コミット、スプリント情報、未コミット変更

#### post-edit-lint.sh（PostToolUse/Edit|Write）
- Markdown: YAMLフロントマターの閉じ`---`チェック
- Shell: `bash -n` 構文チェック
- JSON: `jq empty` 構文チェック

#### pre-push-check.sh（PreToolUse/Bash）
- `git push` コマンドを検出した場合のみ発火
- 未コミット変更、プッシュ対象コミット、高リスクファイル変更を表示
- 警告あり → ユーザー確認（ask）、なし → コンテキスト追加のみ

#### protect-sensitive-files.sh（PreToolUse/Edit|Write）
- ブロック対象: `.env*`, `credentials.json`, `secrets.json`, SSH鍵, GPG鍵
- `exit 2` でブロック

---

## 3. Scripts

### claude-tmux.sh（`ct` コマンド）
- tmux内でClaude Codeを自動起動するラッパー
- `--dangerously-skip-permissions` をデフォルト付与
- サブコマンド: `ct` (起動), `ct <prompt>`, `ct --attach`, `ct --list`, `ct --kill`
- tmuxセッション名: `claude-HHMMSS`
- Agent Teams対応（Split Panesモード）

### limit-tracker-cache.sh
- Anthropic OAuth Usage API から利用制限情報を取得
- macOS Keychainから OAuthトークンを取得（`security find-generic-password`）
- キャッシュファイル: `~/.claude/cache/limit-tracker.json`
- ロック機構で多重実行防止

### setup-coexistence.sh
- Claude Code / Cursor共存セットアップ
- CLAUDE.md雛形生成 → `.cursorrules` へのsymlink
- `.cursor/rules/*.mdc` → `.claude/rules/` へのsymlink共有

---

## 4. ステータスライン（4行ダッシュボード）

`statusline-command.sh` による4行リアルタイム表示:

```
Line 1: [プロジェクト名] [Git:ブランチ*] | [モデル名] | [コンテキストバー XX% OK/WATCH/SWITCH] | [$コスト] | [Xm Xs] | [+追加/-削除]
Line 2: 📊 🪟 [現在トークン]/[最大] (W:[キャッシュ書込] R:[キャッシュ読取]) | In:[累計入力] Out:[累計出力] | 残XX%
Line 3: ⚡ [5h制限%] [リセット時間] | 📅 [週間制限%] [リセット] | 🎵 [Sonnet%] | 🎹 [Opus%] | 🔥 [制限影響量]
Line 4: 🧠 CC:[CLI消費メモリ] | App:[デスクトップアプリ] | VSCode:[VSCode]
```

**注目機能**:
- コンテキスト使用率に応じた色分け（緑<60%<黄<80%<赤）
- 利用制限のリアルタイムモニタリング（5h/7d/Sonnet/Opus個別）
- メモリ使用量監視

---

## 5. CCNotify システム

- **ccnotify.py**: Python製のプロンプト追跡・通知システム
- SQLiteで全プロンプト・完了時刻を記録
- イベント対応: UserPromptSubmit → DB記録、Stop → 完了通知、Notification → 各種通知
- terminal-notifier経由でmacOS通知
- 通知クリックでVSCodeを開く

---

## 6. エージェント・スキル体系

### エージェント（44ファイル、~/.claude/agents/）
- **スプリント管理**: sprint-master, sprint-coder, sprint-documenter, sprint-researcher, sprint-mentor, sprint-reviewer, sprint-tester, sprint-refactorer, sprint-planner, sprint-retro, sprint-review-runner
- **ドキュメントレビュー**: doc-review + 7軸（execution, humanize, integrate, logic-mece, perspective, readability, revision-plan, risk, strategy）
- **要件定義**: reqdef + 6軸（composer, context, hearing, humanize, integrator, risk, scope, stakeholder）
- **プッシュ前レビュー**: pre-push-review + 5軸（code-quality, dependency-check, documentation, git-hygiene, integrate-and-fix, security-check）
- **プロジェクト分析**: project-analyzer + scan, dashboard, report
- **その他**: po-assistant, sp-estimator, times-agent

### スキル（45ディレクトリ、~/.claude/skills/）
スラッシュコマンドとして利用。主なカテゴリ:
- スプリント運用: sprint-init, sprint-start, sprint-end, sprint-status, sprint-dashboard, sprint-master
- タスク管理: task-add, task-update, today
- プロジェクト管理: project-new, project-list-sync, status-change, dashboard, analyze, inventory
- ドキュメント: doc-review, requirement-definition, slides, infographic
- レビュー: pre-push, pre-push-review, regression-guard, permission-audit
- 報告: mtg-report, weekly-review, slack-post, times-agent

---

## 7. テンプレート・ルール

### テンプレート（~/.claude/templates/）
- kpt-history.md, meta-retrospective.md, product-backlog.md
- sprint-backlog.md, sprint-log.md, test-strategy.md
- try-stock.md, velocity-summary.md

### ルール（~/.claude/rules/）
- **regression-prevention.mdc**: デグレ防止ルール
- **story-point-guide.mdc**: SP見積もりガイド（4軸評価）

---

## 8. プラグインシステム

`~/.claude/plugins/marketplaces/claude-plugins-official/` に公式プラグイン群:
- **外部サービス**: asana, context7, firebase, github, gitlab, greptile, laravel-boost, linear, playwright, serena, slack, stripe, supabase
- **内蔵**: agent-sdk-dev, claude-code-setup, claude-md-management, code-review, code-simplifier, commit-commands, explanatory-output-style, example-plugin

---

## ガイドへの活用ポイント

### 初心者向けTIPS候補
1. **settings.json の権限管理**: allow/denyリストで毎回の確認を省略できる
2. **Hooksの活用**: Slack連携、Lint自動チェック、機密ファイル保護
3. **ctコマンド**: tmux + --dangerously-skip-permissions で快適運用
4. **ステータスライン**: コンテキスト使用率・利用制限のリアルタイム確認
5. **コンテキスト再注入**: コンパクション後も重要情報を失わない仕組み
6. **Cursor共存**: .cursorrules symlink戦略
7. **リアクション駆動フィードバック**: Slackからリモート指示
