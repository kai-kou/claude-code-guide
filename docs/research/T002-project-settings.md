# T002: プロジェクト設定の調査結果

**調査日**: 2026-02-26
**対象**: `~/dev/` 配下のプロジェクト設定構造

---

## ワークスペース全体構造

```
~/dev/
├── CLAUDE.md              # プロジェクト横断ルール（全プロジェクト共通）
├── CHEATSHEET.md          # コマンドチートシート
├── _management/
│   ├── PROJECT_STATUS.md  # プロジェクト一覧（57件、24 Active）
│   ├── DASHBOARD.md       # AI生成ダッシュボード
│   └── BEST_PRACTICES.md  # 設計書
├── _templates/            # プロジェクトテンプレート
│   ├── project-init.md    # プロジェクト初期化テンプレート
│   ├── milestones.md      # マイルストーンテンプレート
│   └── tasks.md           # タスク管理テンプレート
├── .claude/
│   └── settings.local.json # ワークスペースレベルの権限追加
│
├── 01_active/             # 進行中プロジェクト（24件）
├── 02_completed/          # 完了プロジェクト（24件）
├── 03_on-hold/            # 保留プロジェクト（1件）
├── 04_not-started/        # 未着手プロジェクト（3件）
├── 05_archive/            # アーカイブ（5件）
└── docs/                  # 共通ドキュメント
```

---

## CLAUDE.md の階層構造（3層）

Claude Codeは複数の `CLAUDE.md` を自動的にマージして読み込む:

### レベル1: グローバル `~/.claude/CLAUDE.md`
- ユーザーの全プロジェクト共通ルール
- キャラクター設定、Bash制約、Git操作ルール等

### レベル2: ワークスペース `~/dev/CLAUDE.md`
- プロジェクト横断ルール
- デグレ防止ルール（Regression Prevention）: リスク分類、変更チェック、禁止事項
- コミット規律
- スプリントワークフロー

### レベル3: プロジェクト `~/dev/01_active/{project}/CLAUDE.md`
- プロジェクト固有のルール
- ディレクトリ構造、開発ルール、技術スタック

---

## settings の階層構造

権限設定も3層で管理:

### レベル1: グローバル `~/.claude/settings.json`
- 75+のBashコマンド許可パターン
- MCP tools（Slack）の許可
- Hooks設定
- ステータスライン設定
- サンドボックス設定

### レベル2: ワークスペース `~/dev/.claude/settings.local.json`
- 追加のBash許可（tmux, jq, ps, top等）
- WebFetch許可ドメイン（api.github.com, influxdata, docs.anthropic.com）
- spinnerTipsEnabled: false

### レベル3: プロジェクト `{project}/.claude/settings.local.json`
- プロジェクト固有の追加設定

---

## プロジェクトテンプレート

`/project-new` コマンドで `_templates/` からプロジェクトを自動生成:

### project-init.md
- YAMLフロントマター（name, title, status, priority, created, owner, tags, summary）
- セクション: 概要, ゴール, スコープ（含む/含まない）, 関連リソース, メモ

### milestones.md / tasks.md
- スクラム管理テンプレート
- YAMLフロントマター集計値（total, completed, progress）
- フェーズ別・優先度別のタスク管理テーブル

---

## スラッシュコマンド体系（CHEATSHEET.md）

### プロジェクト操作
| コマンド | 機能 |
|---------|------|
| `/project-new` | 新規プロジェクト作成 |
| `/task-add` | タスク追加 |
| `/task-update` | ステータス変更 |
| `/milestone-update` | マイルストーン更新 |
| `/status-change` | プロジェクトステータス変更 |
| `/project-list-sync` | PROJECT_STATUS.md同期 |

### 分析・レポート
| コマンド | 機能 |
|---------|------|
| `/today` | 今日のタスク一覧 |
| `/dashboard` | ダッシュボード更新 |
| `/analyze` | 進捗分析 |
| `/weekly-review` | 週次振り返り |
| `/inventory` | 全プロジェクト棚卸し |
| `/mtg-report` | MTG進捗報告 |

### Slack分報
| コマンド | 機能 |
|---------|------|
| `/slack-enable` | Slack自動投稿有効化 |
| `/slack-post` | 即時Slack投稿 |

### その他
| コマンド | 機能 |
|---------|------|
| `/doc-review` | 7軸並列レビュー |
| `/pre-push` | Push前チェック |
| `/slides` | スライド作成 |
| `/infographic` | インフォグラフィック生成 |

---

## 日常ワークフロー

```
朝    → /today
作業中 → /task-update（開始・完了）
作業後 → 自動でSlack投稿（/slack-enable 済みなら）
任意  → /slack-post（手動で即時投稿）
MTG前 → /mtg-report
週末  → /weekly-review
月末  → /inventory
```

---

## Cursor共存戦略

`setup-coexistence.sh` によるClaude Code / Cursor共存:

1. **CLAUDE.md**: プロジェクトルールの単一情報源（Source of Truth）
2. **.cursorrules**: CLAUDE.md へのsymlink（Cursorが読む）
3. **.cursor/rules/*.mdc**: Cursor固有ルール（そのまま維持）
4. **.claude/rules/**: Claude固有ルール + .cursor/rules/ のsymlink共有

---

## ガイドへの活用ポイント

### 初心者向けTIPS候補
1. **CLAUDE.mdの3層構造**: グローバル→ワークスペース→プロジェクトの自動マージ
2. **settings.jsonの階層管理**: 権限設定のカスケード
3. **プロジェクトテンプレート**: `/project-new` で標準化されたプロジェクト構造
4. **スラッシュコマンド**: 日常ワークフローの自動化
5. **Cursor共存**: .cursorrules symlink戦略
6. **デグレ防止ルール**: 変更リスク分類の仕組み
