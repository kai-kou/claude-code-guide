# claude-code-guide

## プロジェクト概要

Claude Codeを初めて利用するユーザー向けの利用ガイド・ベストプラクティス・TIPSを共有するドキュメントプロジェクト。最終成果物はGoogleスライド形式で共有する。

### 主要トピック
- VSCodeでの利用方法（セカンダリパネル、ステータスライン）
- tmux活用（ctコマンド）
- hook活用したSlack連携・通知
- Cursorからの移行（タイムリープ戦略、画像貼り付け）
- 利用制限の把握と回避（5時間/週制限、ステータス確認）
- サンドボックス活用（bypass permissions）
- コンテキストウインドウの管理

## 開発ルール

- 情報源: ローカル設定（~/.claude/）、~/dev配下のプロジェクト設定、Web検索
- ドキュメントはMarkdownで作成し、最終的にGoogleスライドに変換
- 画像・スクリーンショットは `assets/` ディレクトリに格納
- 最新の公式ドキュメントを参照し、バージョンに注意すること

## ディレクトリ構造

```
claude-code-guide/
├── README.md               # プロジェクト概要
├── CLAUDE.md               # プロジェクトルール
├── .gitignore              # Git除外設定
├── scrum/                  # スクラム管理
│   ├── tasks.md            # タスク管理
│   ├── milestones.md       # マイルストーン管理
│   ├── product-backlog.md  # プロダクトバックログ
│   ├── team-roster.md      # チームロスター
│   ├── kpt-history.md      # KPT履歴
│   └── try-stock.md        # Tryストック
├── .sprint-logs/           # スプリントログ
│   ├── SPRINT-XXX.md       # 各スプリント記録
│   ├── sprint-backlog.md   # スプリントバックログ
│   └── velocity-summary.md # ベロシティサマリー
├── persona/                # ペルソナ（Slack分報投稿用）
├── docs/
│   └── slides/              # スライド成果物・元ドキュメント
│       ├── claude-code-guide.pptx  # PPTX成果物
│       ├── slide_outline.md        # スライド構成（最終版・52枚）
│       ├── claude-code-intro.md    # Section 1: Claude Code とは
│       ├── vscode-guide.md         # Section 2: セットアップ
│       ├── tmux-guide.md           # Section 2: tmux活用
│       ├── security-guide.md       # Section 4: 権限とセキュリティ
│       ├── slack-integration.md    # Section 5: Hooks・Slack連携
│       ├── context-management.md   # Section 6: コンテキスト管理
│       ├── usage-limits.md         # Section 7: 利用制限
│       ├── mcp-servers.md          # Section 8: MCP
│       ├── skills-agents.md        # Section 9: Skills・Agents
│       └── cursor-migration.md     # Section 10: Cursorからの移行
└── assets/                 # 画像・素材
```
