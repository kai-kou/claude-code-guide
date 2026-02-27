# tmux で Claude Code を活用する

**作成日**: 2026-02-27

---

## はじめに

Claude Code はターミナルで動作するAIコーディングアシスタントですが、tmux（ターミナルマルチプレクサ）と組み合わせることで、さらに強力なワークフローを実現できます。

### tmux + Claude Code のメリット

| メリット | 説明 |
|---------|------|
| **セッション永続化** | SSH接続が切れてもセッション継続。長時間タスクも安心 |
| **バックグラウンド実行** | tmuxセッションをデタッチして他の作業が可能 |
| **複数ペイン表示** | Agent Teamsで複数エージェントを並列表示（Split Panes） |
| **セッション管理** | 複数のClaude Codeセッションを名前付きで管理 |
| **自動権限スキップ** | `--dangerously-skip-permissions`を自動付与 |

---

## ctコマンドの導入

### セットアップ

`claude-tmux.sh`スクリプトを`ct`コマンドとして利用可能にします。

**1. スクリプトの配置**

```bash
# ~/.claude/scripts/claude-tmux.sh として保存
# （既に配置済みの場合はスキップ）
```

**2. エイリアス設定**

`~/.zshrc`（または`~/.bashrc`）に以下を追加:

```bash
alias ct='~/.claude/scripts/claude-tmux.sh'
```

**3. 設定の反映**

```bash
source ~/.zshrc
```

---

## ctコマンドの使い方

### 基本起動

```bash
ct
```

- tmuxセッション名: `claude-HHMMSS`（例: `claude-143052`）
- 自動的に`--dangerously-skip-permissions`が付与される
- セッション内でClaude Codeが起動する

### プロンプト付き起動

```bash
ct "タスクの説明をここに書く"
```

起動直後にプロンプトが送信されます。

**例:**
```bash
ct "現在のプロジェクトのREADME.mdを作成してください"
```

### セッション一覧表示

```bash
ct --list
```

起動中のClaude Code tmuxセッション一覧を表示。

**出力例:**
```
claude-143052: 1 windows (created Thu Feb 27 14:30:52 2026)
claude-150234: 1 windows (created Thu Feb 27 15:02:34 2026)
```

### 既存セッションにアタッチ

```bash
ct --attach
```

既存のClaude Code tmuxセッションに再接続します。複数ある場合は選択プロンプトが表示されます。

### セッション終了

```bash
ct --kill
```

すべてのClaude Code tmuxセッションを終了します。

---

## --dangerously-skip-permissions の理解

### 何が省略されるのか

`--dangerously-skip-permissions`フラグは、以下の権限確認を**すべて**スキップします:

- ファイル読み取り・書き込み
- Bashコマンド実行
- MCPツール呼び出し
- プロジェクト外ファイルへのアクセス

### なぜctコマンドで自動付与するのか

**理由:**
1. **tmux環境の特性**: セッションがバックグラウンドで動作するため、対話的な権限確認が困難
2. **ワークフロー効率化**: 開発者が信頼できる環境での作業を前提とする
3. **サンドボックスとの併用**: サンドボックス内で動作させることでリスクを軽減

### 安全な使い方

**推奨構成:**

```json
// ~/.claude/settings.json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  },
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)",
      "Bash(python *)"
      // 信頼できるコマンドのみリスト化
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(sudo *)"
    ]
  }
}
```

**ポイント:**
- サンドボックス有効化でOSレベルの分離を確保
- `permissions.deny`で危険なコマンドを明示的にブロック
- `permissions.allow`で許可するツール・コマンドパターンを定義

---

## Agent Teams との連携

Agent Teamsは複数のエージェントが協調してタスクを遂行する実験的機能です。tmux環境では**Split Panesモード**で威力を発揮します。

### 有効化方法

**1. 環境変数を設定**

`~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**2. ctコマンドで起動**

```bash
ct
```

環境変数が自動的に適用されます。

### Split Panesモードの活用

Agent Teamsを起動すると、tmuxが自動的にペインを分割し、各エージェントが独立したペインに表示されます。

**表示例:**

```
┌────────────────────────────────────────┐
│ リードエージェント（メイン）            │
│ タスク全体の調整・指示                 │
├────────────────────────────────────────┤
│ チームメイト1      │ チームメイト2      │
│ コード実装         │ テスト実行         │
└────────────────────────────────────────┘
```

### tmuxでの操作

| キー操作 | 動作 |
|---------|------|
| `Ctrl-b` + `o` | ペイン間の移動 |
| `Ctrl-b` + `↑/↓/←/→` | 指定方向のペインへ移動 |
| `Ctrl-b` + `z` | 現在のペインをフルスクリーン化（トグル） |
| `Ctrl-b` + `d` | セッションをデタッチ（バックグラウンド化） |

### Agent Teamsの構成例

**スプリント管理チーム:**

```
リードエージェント: sprint-master
  ├─ sprint-planner（計画策定）
  ├─ sprint-coder（実装）
  └─ sprint-documenter（ドキュメント作成）
```

各エージェントが独立したペインで動作し、リアルタイムで進捗が確認できます。

---

## 制限事項と注意点

### Agent Teams の制限

| 項目 | 制限内容 |
|------|---------|
| **セッション再開** | in-processモードのチームメイトは復元不可（Split Panesは復元可能） |
| **ネスト** | 1セッション1チームのみ。チーム内でさらにチームを作成不可 |
| **VS Code統合ターミナル** | Split Panesモード非対応（tmux/iTerm2のみ） |

### tmux環境での注意点

**1. サンドボックス設定の確認**

`--dangerously-skip-permissions`を使う場合、サンドボックスが有効であることを確認してください。

```bash
# 確認方法
cat ~/.claude/settings.json | grep -A3 "sandbox"
```

**2. セッション数の管理**

tmuxセッションが増えすぎるとメモリを消費します。定期的に`ct --list`で確認し、不要なセッションは`ct --kill`で終了してください。

**3. コンテキストウィンドウの管理**

長時間のセッションではコンテキストウィンドウが満杯になる可能性があります。適宜`/compact`や`/clear`を実行してください。

---

## TIPS

### 1. セッション名をカスタマイズ

`claude-tmux.sh`を編集してセッション名のプレフィックスを変更できます:

```bash
# デフォルト: claude-HHMMSS
# カスタマイズ例: myproject-HHMMSS
SESSION_NAME="myproject-$(date +%H%M%S)"
```

### 2. ステータスバーでフィードバック待機を表示

`slack-feedback.sh` hookと連携すると、tmuxステータスバーにフィードバック待機状態が表示されます。

**設定例（~/.tmux.conf）:**

```bash
set -g status-right '#[fg=yellow]#(cat ~/.claude/state/feedback-status 2>/dev/null)#[default] %H:%M'
```

### 3. デタッチ・アタッチのワークフロー

```bash
# 1. タスクを開始
ct "大規模リファクタリングを実施"

# 2. セッションをデタッチ（Ctrl-b → d）

# 3. 他の作業を実施...

# 4. 進捗確認のため再アタッチ
ct --attach
```

### 4. 複数プロジェクトの並行作業

```bash
# プロジェクトA
cd ~/projects/project-a
ct "機能Xの実装"

# セッションをデタッチ（Ctrl-b → d）

# プロジェクトB
cd ~/projects/project-b
ct "バグ修正"
```

`ct --list`で両セッションが確認でき、`tmux attach -t claude-143052`で個別にアタッチ可能です。

---

## まとめ

| ポイント | 説明 |
|---------|------|
| **ctコマンド** | tmux + Claude Codeの統合ラッパー |
| **自動権限スキップ** | `--dangerously-skip-permissions`でスムーズな実行 |
| **Agent Teams** | Split Panesで複数エージェントを並列表示 |
| **セッション管理** | 複数プロジェクトの並行作業が可能 |
| **安全性** | サンドボックス + deny リストで保護 |

tmuxとClaude Codeを組み合わせることで、長時間タスク・複数プロジェクト・複数エージェントのワークフローが劇的に効率化されます。まずは`ct`コマンドを試して、徐々にAgent Teamsやカスタマイズに挑戦してください。
