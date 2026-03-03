# tmux で Claude Code を活用する

**作成日**: 2026-02-27

---

## はじめに

### こんな困りごとありませんか？

- SSH接続が切れてClaude Codeのセッションが消えた
- 長時間のリファクタリング中にMacを閉じたら全部やり直し
- Agent Teamsで複数エージェントを同時に見たいが、ターミナルが1画面しかない

### tmuxで解決できます

tmuxを使えば、セッションが永続化されるため接続が切れても作業が継続します。さらに、画面分割でAgent Teamsの各エージェントを並列表示できます。

| Before | After |
|--------|-------|
| SSH切断 → セッション消失、最初からやり直し | tmuxデタッチ → 再接続でセッション復帰 |
| Agent Teamsの出力がターミナル1画面に混在 | Split Panesで各エージェントが独立ペインに表示 |
| 毎回 `--dangerously-skip-permissions` を手入力 | `ct` コマンドで自動付与 |

### 前提条件

| 項目 | 要件 |
|------|------|
| OS | macOS 13.0+ / Linux（WSL含む） |
| tmux | 3.2以上（`tmux -V` で確認。`brew install tmux` でインストール） |
| Claude Code | インストール済み・認証済み |
| シェル | Bash / Zsh |

### tmux + Claude Code のメリット

tmuxを使う理由を一言でいうと「安心して席を離れられる」ことです。以前、SSH経由で大規模なリファクタリングを走らせていた際にWi-Fiが切れて1時間分の作業が吹き飛んだ経験があります。tmuxを使っていれば、接続が切れてもセッションはサーバー側で生き続けるので、再接続するだけで途中から再開できます。

Agent Teamsで複数エージェントを同時に動かすと、tmuxのSplit Panesが真価を発揮します。リードエージェント、コーダー、テスターがそれぞれ独立したペインに表示され、何が起きているかリアルタイムで把握できます。`ct`コマンドを使えば`--dangerously-skip-permissions`の手入力も不要になります。

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

> **⚠️ 重要な警告: このフラグは「全ての安全装置を解除する」ことを意味します**
>
> このフラグを有効にすると、Claude Codeは**一切の確認なし**で以下を実行できます:
>
> - `rm -rf /` のようなファイルシステム破壊コマンドが即実行される
> - `.env` や `credentials.json` など機密ファイルが無確認で読み書きされる
> - `git push --force` が確認なしで実行され、リモートの変更が上書きされる
> - プロジェクト外のファイル（`~/.ssh/`, `~/.gnupg/` など）にもアクセスされる
>
> **必ずサンドボックスと併用してください。** サンドボックスを有効にすることで、OSレベルでのファイルシステム・ネットワークの隔離が担保され、上記リスクを大幅に軽減できます。

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

Agent Teamsにはいくつか把握しておくべき制限があります。in-processモードのチームメイトは`/resume`でセッションを再開しても復元されません（Split Panesモードなら復元可能）。また、1セッションにつき1チームまでで、チーム内でさらにチームを入れ子にすることはできません。VS Code統合ターミナルではSplit Panesモードが動作しないため、tmuxかiTerm2を使う必要があります。

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

```
┌─────────────┐      ┌────────────────────────┐      ┌─────────────┐
│   ct 起動   │─────▶│  Claude Code 実行中    │─────▶│  完了待ち   │
│ (タスク指示) │      │  (tmuxセッション内)    │      │             │
└─────────────┘      └──────────┬─────────────┘      └─────────────┘
                                │
                     Ctrl-b → d │ デタッチ
                                ▼
                     ┌────────────────────────┐
                     │  バックグラウンド実行   │    ← PCを閉じてもOK
                     │  (セッション継続中)     │    ← Wi-Fi切れてもOK
                     └──────────┬─────────────┘
                                │
                     ct --attach│ 再アタッチ
                                ▼
                     ┌────────────────────────┐
                     │  結果を確認            │
                     │  (途中から再開)        │
                     └────────────────────────┘
```

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

### 5. セッション管理とクリーンアップ

tmuxセッションは明示的に終了しないと残り続けます。複数プロジェクトを並行作業していると、使い終わったセッションが溜まってリソースを消費します。

**セッション上限の目安**:
- 同時にアクティブなセッションは**3〜5個**が実用的な上限
- それ以上になると、どのセッションが何のタスクか把握しにくくなる
- 各セッションのClaude Codeプロセスがメモリを消費するため、マシンスペックに応じて調整

**不要なセッション一括削除**:
```bash
# 全セッション一覧を確認
tmux ls

# 特定セッションを終了
tmux kill-session -t claude-143052

# claude-プレフィックスの全セッションを終了
tmux ls -F '#{session_name}' | grep '^claude-' | xargs -I{} tmux kill-session -t {}
```

**自動クリーンアップスクリプト（~/.claude/hooks/cleanup-tmux-sessions.sh）**:
```bash
#!/bin/bash
# 24時間以上アイドル状態のClaude Codeセッションを自動終了
set -euo pipefail

MAX_IDLE_SECONDS=86400  # 24時間

tmux ls -F '#{session_name} #{session_activity}' 2>/dev/null | while read -r NAME ACTIVITY; do
  case "$NAME" in
    claude-*)
      IDLE=$(( $(date +%s) - ACTIVITY ))
      if [ "$IDLE" -gt "$MAX_IDLE_SECONDS" ]; then
        tmux kill-session -t "$NAME"
        echo "Cleaned up idle session: $NAME (idle ${IDLE}s)"
      fi
      ;;
  esac
done
```

**ポイント**:
- タスク完了後は `tmux kill-session` で明示的にセッションを終了する習慣をつける
- `ct --list` で定期的にセッション一覧を確認する
- 長期間放置されたセッションはクリーンアップスクリプトで自動回収

---

## トラブルシューティング

### ctコマンドが見つからない（command not found）

`ct`を実行して`command not found`と表示される場合、エイリアス設定が反映されていません。`~/.zshrc`に`alias ct='~/.claude/scripts/claude-tmux.sh'`が記述されているか確認し、`source ~/.zshrc`で再読み込みしてください。スクリプト自体に実行権限がない場合もあるので、`chmod +x ~/.claude/scripts/claude-tmux.sh`も実行しておきましょう。

### セッションにアタッチできない

`ct --attach`で「no sessions」と表示される場合、セッションがすでに終了している可能性があります。`tmux ls`で全セッション一覧を確認してください。`claude-`プレフィックスのセッションが見当たらなければ、`ct`で新しく起動する必要があります。「sessions should be nested with care」というエラーが出る場合は、すでにtmuxセッション内にいるため、先に`Ctrl-b d`でデタッチしてから再アタッチしてください。

### Agent TeamsでSplit Panesにならない

Agent Teamsを起動してもペインが分割されない場合、まず環境変数`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`が`1`に設定されているか確認します。settings.jsonの`env`セクションに記述したのに反映されない場合、Claude Codeを再起動してください。また、VS Code統合ターミナルではSplit Panesモードは動作しないため、外部のターミナルアプリでtmuxを起動する必要があります。

### tmuxのキーバインドが効かない

`Ctrl-b`を押しても何も反応しない場合、tmuxのプレフィックスキーがカスタマイズされている可能性があります。`~/.tmux.conf`でプレフィックスキーの設定を確認してください（`set -g prefix C-a`のように変更されていることがあります）。また、tmuxが古いバージョンだと一部の機能が動作しないため、`tmux -V`で3.2以上であることを確認してください。

### セッション内でClaude Codeが起動しない

`ct`でtmuxセッションは作成されるがClaude Codeが起動しない場合、`claude`コマンドにPATHが通っていないことが原因です。tmuxはログインシェルとは異なる環境で起動するため、`~/.zshrc`でnvmやnodeのPATH設定がされていても読み込まれないケースがあります。`~/.tmux.conf`に`set -g default-shell /bin/zsh`を追加し、tmuxを再起動すると解消することが多いです。

---

## まとめ

### 参考リンク

- [Claude Code公式 - CLI使用方法](https://docs.anthropic.com/en/docs/claude-code/cli-usage)
- [Agent Teams公式ドキュメント](https://docs.anthropic.com/en/docs/claude-code/agent-teams)
- [tmux公式Wiki](https://github.com/tmux/tmux/wiki)

---

## まとめ

tmux + Claude Codeの組み合わせが真価を発揮するのは「席を離れたい」ときです。`ct`コマンドでセッションを立ち上げ、長時間のリファクタリングを任せてデタッチし、昼食から戻ったらアタッチして結果を確認する。この流れに慣れると、PCの前に張り付いてAIの出力を見守る必要がなくなります。

Agent TeamsのSplit Panesモードを使えば、リードエージェント・コーダー・テスターがそれぞれ独立したペインで動作するのをリアルタイムで眺められます。`--dangerously-skip-permissions`はサンドボックスとDenyリストの併用を前提に使ってください。まずは`ct`コマンドの基本操作から試して、慣れてきたらAgent TeamsやセッションのカスタマイズにStep Upしていくのがおすすめです。
