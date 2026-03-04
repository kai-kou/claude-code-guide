---
title: カスタムスキル & エージェント ガイド
section: 10
created: 2026-02-27
updated: 2026-02-27
target: Claude Code中級者〜上級者
topics:
  - カスタムスキル
  - SKILL.md
  - カスタムエージェント
  - Agent Teams
  - サブエージェント
---

# カスタムスキル & エージェント ガイド

Claude Codeを使い込んでいくと、「毎回同じ手順でPRレビューしている」「デプロイ前のチェックリストを毎回伝えている」といった繰り返しが出てきます。スキルはそうした定型ワークフローを`/コマンド名`で一発呼び出しできるようにする仕組みで、エージェントは特定の専門タスク（コードレビュー、技術調査など）を独立したAIサブプロセスに任せる仕組みです。自分のチームでは`/review`（PRレビュー）と`/sprint-start`（スプリント開始）を最初に作り、それだけで日常の定型作業が大幅に減りました。

---

## 1. スキル（カスタムスラッシュコマンド）

### スキルとは

スキルは、`/コマンド名` で呼び出せる再利用可能なプロンプトです。例えば `/review-pr` で毎回同じ品質基準でコードレビューを実行したり、`/deploy-staging` でデプロイ手順を自動化したりできます。

スキルは [Agent Skills](https://agentskills.io) オープン標準に準拠しており、SKILL.mdファイルで定義します。

### 配置場所

| 配置場所 | パス | 適用範囲 |
|---------|------|---------|
| **Personal** | `~/.claude/skills/<名前>/SKILL.md` | 全プロジェクト共通 |
| **Project** | `.claude/skills/<名前>/SKILL.md` | 現プロジェクトのみ |

### SKILL.md の書き方

スキルの定義ファイルは、YAMLフロントマター + Markdown本文で構成されます。

**シンプルな例**

```markdown
---
name: review-pr
description: PRのコード変更をレビューする。「PRレビュー」「コードレビュー」と言われた場合にも使用。
user-invocable: true
---

現在のブランチのPRをレビューしてください。

以下の観点でチェック:
1. バグの可能性
2. セキュリティリスク
3. パフォーマンスへの影響
4. テストの充足度
```

**高度な例（フォークコンテキスト + エージェント指定）**

```markdown
---
name: sprint-start
description: スプリントを開始する
user-invocable: true
context: fork
agent: sprint-master
---

スプリントのプランニングを開始してください。
$ARGUMENTS
```

### フロントマターの主要フィールド

| フィールド | 説明 | デフォルト |
|-----------|------|----------|
| `name` | スキル名。省略時はディレクトリ名を使用 | ディレクトリ名 |
| `description` | スキルの説明。Claudeが自動呼び出しの判断に使用 | 本文の最初の段落 |
| `user-invocable` | ユーザーが `/` メニューから呼び出せるか | `true` |
| `disable-model-invocation` | Claudeの自動呼び出しを禁止するか | `false` |
| `allowed-tools` | スキル実行中に許可するツール | 全ツール |
| `model` | 実行時のモデル（`sonnet`, `opus`, `haiku`） | 継承 |
| `context` | `fork` で独立コンテキストで実行 | なし |
| `agent` | `context: fork` 時に使うエージェント種別 | なし |

### 呼び出し制御

スキルの呼び出し方法は2つあります。

| 設定 | ユーザーから呼び出し | Claudeが自動呼び出し |
|------|-------------------|-------------------|
| デフォルト | `/スキル名` で可能 | descriptionに基づいて可能 |
| `disable-model-invocation: true` | `/スキル名` で可能 | 不可 |
| `user-invocable: false` | 不可（メニューに表示されない） | 可能 |

**ポイント**: `description` にはClaude がいつこのスキルを使うべきか判断できる説明を書きます。「〇〇と言われた場合にも使用」のような自然言語パターンも効果的です。

### 動的コンテキスト注入

`` !`コマンド` `` 記法を使うと、スキル呼び出し時にコマンドの実行結果がプロンプトに注入されます。

```markdown
---
name: pr-summary
description: PRの変更を要約する
---

## 現在のPR情報
- diff: !`gh pr diff`
- コメント: !`gh pr view --comments`

上記の情報をもとにPRの変更を要約してください。
```

これはClaude が実行するのではなく、スキル送信前に前処理として実行されます。

### 引数の受け取り

`$ARGUMENTS` でスキル呼び出し時の引数を受け取れます。

```markdown
---
name: explain
description: 指定されたファイルを解説する
argument-hint: [ファイルパス]
---

$ARGUMENTS のコードを解説してください。
```

使用例: `/explain src/auth/login.ts`

### サポートファイル（references/）

SKILL.mdが長くなる場合は、補足情報を別ファイルに分離します。

```
my-skill/
├── SKILL.md           # メイン指示（500行以内推奨）
├── reference.md       # 詳細なAPIドキュメント
├── examples.md        # 使用例
└── scripts/
    └── helper.py      # ヘルパースクリプト
```

SKILL.md本文からリンクすることで、Claudeが必要時のみ読み込みます。

```markdown
## 詳細情報
- APIの詳細は [reference.md](reference.md) を参照
- 使用例は [examples.md](examples.md) を参照
```

---

## 2. カスタムエージェント（サブエージェント）

### エージェントとは

エージェントは、特定のタスクに特化した独立したAIサブプロセスです。各エージェントは独自のコンテキストウィンドウ、システムプロンプト、ツール権限を持ちます。

スキルが「再利用可能なプロンプト」なのに対し、エージェントは「専門家としての役割定義」です。

### 配置場所

| 配置場所 | パス |
|---------|------|
| **Project** | `.claude/agents/<名前>.md` |
| **Personal** | `~/.claude/agents/<名前>.md` |

### エージェント定義ファイルの書き方

```markdown
---
name: code-reviewer
description: コードレビューの専門家。コード変更後に積極的に使用。
tools: Read, Glob, Grep
model: sonnet
maxTurns: 15
---

あなたはシニアコードレビュアーです。
以下の観点でコードの品質を確認してください。

1. バグの可能性
2. セキュリティリスク
3. 命名規則の一貫性
4. パフォーマンスへの影響
```

### フロントマターの主要フィールド

| フィールド | 説明 |
|-----------|------|
| `name` | エージェント名（一意の識別子） |
| `description` | 役割の説明。Claudeがデリゲートの判断に使用 |
| `tools` | 使用可能ツールのホワイトリスト（省略時は全ツール継承） |
| `disallowedTools` | 使用禁止ツールのブラックリスト |
| `model` | 使用モデル（`sonnet`, `opus`, `haiku`, `inherit`） |
| `maxTurns` | 最大実行ターン数 |
| `permissionMode` | 権限モード（`default`, `plan`, `acceptEdits` 等） |
| `memory` | 永続メモリスコープ（`user`, `project`, `local`） |
| `isolation` | `worktree` で一時的なgit worktreeで実行 |
| `skills` | 起動時に注入するスキル一覧 |
| `mcpServers` | 使用可能なMCPサーバー |

### 組み込みサブエージェント

Claude Codeには以下の組み込みエージェントがあります。

| エージェント | モデル | ツール | 用途 |
|------------|--------|--------|------|
| **Explore** | Haiku（高速） | 読み取り専用 | ファイル探索・コードベース調査 |
| **Plan** | 継承 | 読み取り専用 | Plan Mode時のリサーチ |
| **general-purpose** | 継承 | 全ツール | 複雑なマルチステップタスク |

### ツール制限のベストプラクティス

エージェントには最小権限の原則を適用します。

```yaml
# 調査専用エージェント — 読み取りのみ
tools: Read, Glob, Grep, WebSearch, WebFetch

# コーディングエージェント — 全ツール
tools: Read, Glob, Grep, Bash, Edit, Write

# レビューエージェント — 読み取り + 限定Bash
tools: Read, Glob, Grep, Bash
```

---

## 3. スキル vs エージェント の使い分け

スキルとエージェントは似ているようで役割が違います。スキルは「再利用可能なプロンプト」で、メインセッション内で実行されます（`context: fork`で分離も可能）。`/review`や`/deploy`のようにユーザーが`/コマンド名`で呼び出すか、descriptionに基づいてClaudeが自動的に呼び出します。デプロイ手順やレビュー基準のような定型ワークフローに向いています。

エージェントは「専門家としての役割定義」で、常に独立したコンテキストウィンドウで実行されます。Task toolを通じてメインエージェントからデリゲートされ、技術調査、テスト実行、コードレビューのような専門サブタスクに適しています。

迷ったら、まずスキルとして作ってみてください。独立したコンテキストが必要だと感じたら、そのときエージェントに切り替えれば十分です。

---

## 4. Agent Teams（実験的機能）

### Agent Teamsとは

Agent Teamsは、複数のClaude Codeインスタンスが協調してタスクを並列実行する機能です。一人のTeam Leadが複数のTeammateを指揮し、共有タスクリストとメッセージングで連携します。

### 有効化

settings.jsonに環境変数を設定します。

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### アーキテクチャ

| コンポーネント | 役割 |
|-------------|------|
| **Team Lead** | チームを作成し、タスクを割り振り、進捗を管理 |
| **Teammates** | 独立したコンテキストで並列に作業を実行 |
| **Task List** | チーム共有のタスク管理（作成・割り当て・完了） |
| **Mailbox** | チームメイト間の直接メッセージング |

### サブエージェントとの違い

サブエージェントは「結果だけ返してくれればいい」タスクに向いています。独自のコンテキストウィンドウで作業し、終わったら結果をメインエージェントに報告して終了します。コストも低めです。

Agent Teamsはもっと協調的です。チームメイト同士が直接メッセージをやり取りでき、共有タスクリストで自己調整します。フロントエンドとバックエンドを並列開発したり、複数の仮説で並列デバッグしたりする場面で威力を発揮しますが、チームメイト数に比例してコストが増えます。

### 表示モード

| モード | 説明 | 要件 |
|--------|------|------|
| `in-process` | メインターミナル内で動作。Shift+Downで切り替え | 追加設定不要 |
| `split panes` | tmux/iTerm2で各チームメイトが独立ペインに表示 | tmux or iTerm2 |

tmuxセッション内で実行すると自動的にsplit panesモードになります。

### ユースケース

- 並列コードレビュー（セキュリティ・パフォーマンス・テストカバレッジを各担当）
- 独立したモジュールの並列実装
- 複数の仮説による並列デバッグ調査
- フロントエンド・バックエンド・テストの同時開発

### 既知の制限

- **セッション再開不可**: `/resume` でin-processのチームメイトは復元されない
- **1チームのみ**: 1セッションにつき1チームまで
- **ネスト不可**: チームメイトはさらにチームを作れない
- **タスク遅延**: チームメイトがタスク完了をマークし忘れることがある
- **split panes制限**: VS Code統合ターミナル、Windows Terminal、Ghosttyでは非対応

---

## 5. 実践例

### 例1: PRレビュースキル

```markdown
---
name: review
description: PRのコードレビューを実行する
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash
---

!`gh pr diff` の変更を以下の観点でレビューしてください。

1. バグ・ロジックエラー
2. セキュリティリスク（OWASP Top 10）
3. テストカバレッジ
4. 命名・コードスタイルの一貫性

問題がなければ「LGTM」、問題があれば具体的な修正案を提示してください。
```

### 例2: 調査専用エージェント

```markdown
---
name: researcher
description: 技術調査を担当する。WebSearchとドキュメント分析に特化。
tools: Read, Glob, Grep, WebSearch, WebFetch
model: sonnet
maxTurns: 15
---

あなたは技術調査の専門家です。
コードを書くことは絶対にしないでください。
調査結果は「要約 → 推奨事項 → リスク」の構造で報告してください。
```

### 例3: Agent Teams でのスプリント運用

```markdown
---
name: sprint-master
description: スプリント全体のオーケストレーション
tools: Read, Glob, Grep, Bash, Edit, Write, Task, TeamCreate, TeamDelete, SendMessage
model: sonnet
maxTurns: 30
memory: user
---

スクラムマスターとして、チームメイトにタスクを分配し、進捗を管理してください。
```

Team Leadがsprint-masterとして起動し、sprint-coder、sprint-tester、sprint-documenterなどのチームメイトにタスクを割り振る運用が可能です。

---

## 6. 管理方法

### /agents コマンド

Claude Code内で `/agents` を実行すると、スキルとエージェントの作成・編集・削除をインタラクティブに行えます。

### コンテキスト予算

スキルが多すぎると、説明文がコンテキストウィンドウの予算（コンテキストウィンドウの2%、最大16,000文字）を超える場合があります。不要なスキルは `disable-model-invocation: true` にするか削除して、予算を節約してください。

---

## 参考リンク

- [スキル公式ドキュメント](https://code.claude.com/docs/en/skills)
- [サブエージェント公式ドキュメント](https://code.claude.com/docs/en/sub-agents)
- [Agent Teams 公式ドキュメント](https://code.claude.com/docs/en/agent-teams)
- [Agent Skills オープン標準](https://agentskills.io)
