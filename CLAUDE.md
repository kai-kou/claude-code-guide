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
├── docs/                   # ドキュメント原稿
│   ├── slides-outline.md   # スライド構成
│   ├── vscode-guide.md     # VSCode利用ガイド
│   ├── tmux-guide.md       # tmux活用ガイド
│   ├── slack-integration.md # Slack連携ガイド
│   ├── cursor-migration.md # Cursorからの移行ガイド
│   ├── security-guide.md    # 権限とセキュリティガイド
│   ├── context-management.md # コンテキスト管理ガイド
│   ├── usage-limits.md      # 利用制限ガイド
│   └── research/           # リサーチ成果物
└── assets/                 # 画像・素材
```
