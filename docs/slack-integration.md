# Claude Code Hooks活用ガイド: Slack連携・通知システム

**最終更新日**: 2026-02-27
**対象読者**: Claude Code初心者〜中級者

---

## はじめに

### こんな困りごとありませんか？

- Claude Codeが作業完了したのに気づかず、画面を見に行くまで待ちぼうけ
- 長時間タスクの進捗をPCの前で監視し続けている
- ファイル編集後のLintエラーを見落として後から手戻りが発生

### Hooks + Slack連携で解決できます

Hooksを設定すると、タスク完了時にSlackへ自動通知、Slackのリアクションで次の指示を送信できます。PCから離れていてもスマホで進捗を把握し、遠隔操作できます。

| Before | After |
|--------|-------|
| 作業完了を目視で確認、PCの前で待機 | Slack通知で完了を把握、スマホからリアクションで次指示 |
| Lint/構文エラーに後で気づいて手戻り | 編集直後に自動チェック、エラーは即座にフィードバック |
| git push前に手動でチェックリストを確認 | push検出時に自動で未コミット変更・高リスクファイルを警告 |

### 前提条件

| 項目 | 要件 |
|------|------|
| OS | macOS 13.0+（Keychain使用のため。Linux/WSLは代替手段あり） |
| Claude Code | インストール済み・認証済み |
| jq | 1.6以上（`jq --version` で確認。`brew install jq` でインストール） |
| Slack | ワークスペースへのアクセス権。Bot Token（Slack App作成が必要） |
| オプション | terminal-notifier（macOS通知用。`brew install terminal-notifier`） |

Claude Codeの**Hooks**は、特定のイベントに応じてスクリプトを自動実行する仕組みです。「タスクが完了したらSlackに通知」「ファイルを編集したらLintを自動実行」「git push前に未コミット変更を警告」といった自動化を、settings.jsonにスクリプトを登録するだけで実現できます。

Hooksを入れる前は、タスク完了のたびにSlackを手動で開いて報告し、push前にチェックリストを目視確認していました。Hooksを導入してからはその手間がゼロになり、何より「確認し忘れ」がなくなったのが一番大きな変化です。

---

## 1. Hooksの基本概念

### 1-1. イベント駆動モデル

Hooksは、Claude Codeの実行ライフサイクル中に発生する**イベント**に応じてスクリプトを実行します。

```
UserPromptSubmit → PreToolUse → [Tool実行] → PostToolUse → Stop → Notification
```

各イベントでカスタムスクリプトを設定できます。

### 1-2. settings.jsonでの設定

Hooksは `~/.claude/settings.json` または `.claude/settings.json` で定義します。

```json
{
  "hooks": {
    "Stop": {
      "type": "command",
      "command": "bash ~/.claude/hooks/slack-feedback.sh"
    },
    "PostToolUse": {
      "type": "command",
      "command": "bash ~/.claude/hooks/post-edit-lint.sh",
      "matcher": "Edit|Write"
    }
  }
}
```

**設定項目**:
- `type`: `"command"` (bashスクリプト実行) または `"prompt"` (LLMベースの評価)
- `command`: 実行するコマンド（絶対パス推奨）
- `matcher`: ツール名のフィルタ（正規表現対応）
  - 完全一致: `"Write"`
  - 正規表現: `"Edit|Write"`
  - 全ツール: `"*"` または省略

### 1-3. command型 vs prompt型

Hooksには2種類あります。**command型**はbashスクリプトを実行する方式で、処理が高速で外部サービスとの連携に向いています。Slack投稿やLintチェックのような定型処理はこちらを使います。**prompt型**はLLMに自然言語で判断を委ねる方式で、「認証コードに影響する変更ならセキュリティレビューのリマインダーを追加する」のように、文脈によって動作を変えたい場面で使います。ただしLLM呼び出しが入るぶん、command型より応答が遅くなります。

**prompt型の例**（実験的機能）:
```json
{
  "hooks": {
    "PreToolUse": {
      "type": "prompt",
      "prompt": "If this edit affects authentication code, add a security review reminder.",
      "matcher": "Edit"
    }
  }
}
```

---

## 2. 全14イベント一覧と使い分け

### 2-1. よく使う5つのイベント

| イベント | タイミング | 主な用途 |
|---------|----------|---------|
| **PreToolUse** | ツール実行前 | 検証・ブロック・事前チェック |
| **PostToolUse** | ツール実行後 | フォーマット・Lint・後処理 |
| **Stop** | 停止時 | 完了通知・Slack投稿・クリーンアップ |
| **Notification** | 通知発生時 | macOS通知・外部連携 |
| **SessionStart** | セッション開始時 | コンテキスト初期化・再注入 |

### 2-2. その他のイベント

上記5つ以外にも、UserPromptSubmit（プロンプト送信時の記録・分析）、PermissionRequest（カスタム許可ロジック）、SubagentStop（サブエージェント結果処理）、PreCompact（コンテキスト圧縮前の情報保存）、SessionEnd（ログ保存・統計収集）、TeammateIdle（チームメイトのタイムアウト処理）、TaskCompleted（タスク記録・レポート）といったイベントがあります。最初から全部使う必要はなく、実際に困りごとが出てきたタイミングで追加していく進め方がおすすめです。

---

## 3. 実例①: Slack連携（slack-feedback.sh）

### 3-1. 概要

タスク完了時にSlackへ自動投稿し、絵文字リアクションで次の指示を受け取る仕組み。リモート環境からでもClaude Codeを操作できます。

**実行フロー**:
```
Stop イベント → Slack投稿 → フィードバック待機（1分→3分→5分） → リアクション取得 → 指示変換
```

### 3-2. settings.json設定

```json
{
  "hooks": {
    "Stop": {
      "type": "command",
      "command": "bash ~/.claude/hooks/slack-feedback.sh"
    },
    "Notification": {
      "type": "command",
      "command": "bash ~/.claude/hooks/slack-feedback.sh"
    }
  }
}
```

### 3-3. リアクション→指示変換マッピング

| 絵文字 | 意味 | 変換後の指示 |
|-------|------|------------|
| 👍 | LGTM / CP | コミット&プッシュを実行 |
| 👀 | レビュー依頼 | コードレビューを開始 |
| 🔁 | やり直し | タスクをやり直す |
| ❌ | 破棄 | 変更を破棄 |
| 🚀 | テスト実行 | テストを実行 |
| 📝 | ドキュメント作成 | ドキュメントを更新 |

スレッド返信でもフィードバック可能です。

### 3-4. 段階的待機の仕組み

フィードバック待機は3ラウンドで段階的にタイムアウトを延長します。

```bash
# ラウンド1: 1分待機
sleep 60
check_feedback

# ラウンド2: 3分待機
sleep 180
check_feedback

# ラウンド3: 5分待機
sleep 300
check_feedback
```

この間、tmuxステータスバーに待機状態が表示されます。

### 3-5. スクリプト構造の概要

```bash
#!/bin/bash
# ~/.claude/hooks/slack-feedback.sh

# 1. セッション情報を取得
PROJECT_NAME=$(basename $(pwd))
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

# 2. Keychainからトークンを取得（平文での環境変数保存は非推奨）
SLACK_TOKEN=$(security find-generic-password -a "claude-code" -s "slack-bot-token" -w 2>/dev/null)
if [ -z "$SLACK_TOKEN" ]; then
  echo "❌ Slack Token未設定。Keychainに登録してください。" >&2
  exit 1
fi

# 3. Slack投稿
MESSAGE="[${PROJECT_NAME}] タスク完了 (${TIMESTAMP})\nフィードバックをお待ちしています..."
RESPONSE=$(curl -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer ${SLACK_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "{\"channel\":\"#kai-cursor-times\",\"text\":\"${MESSAGE}\"}")

# 4. メッセージIDを取得
MESSAGE_ID=$(echo $RESPONSE | jq -r '.ts')

# 5. フィードバック待機（3ラウンド）
wait_for_feedback $MESSAGE_ID

# 6. リアクションを指示に変換
FEEDBACK=$(get_reactions $MESSAGE_ID)
convert_to_instruction $FEEDBACK
```

### 3-6. 必要な設定

**Slack App設定**:
1. Slack App作成（https://api.slack.com/apps）
2. Bot Token Scopes: `chat:write`, `reactions:read`, `channels:history`
3. トークンをmacOS Keychainに保存（下記参照）

**トークンの安全な管理（macOS Keychain）**:

> **警告**: `export SLACK_TOKEN=xoxb-...` のように環境変数に平文でトークンを保存するのは**非推奨**です。`.zshrc` が流出した場合、トークンも漏洩します。macOS標準のKeychainを使いましょう。

**Step 1: Keychainにトークンを登録**

```bash
security add-generic-password \
  -a "claude-code" \
  -s "slack-bot-token" \
  -w "xoxb-your-token-here" \
  -T "" \
  ~/Library/Keychains/login.keychain-db
```

**Step 2: スクリプトからKeychainのトークンを取得**

```bash
SLACK_TOKEN=$(security find-generic-password \
  -a "claude-code" \
  -s "slack-bot-token" \
  -w 2>/dev/null)

if [ -z "$SLACK_TOKEN" ]; then
  echo "❌ Slack Tokenが見つかりません。Keychainに登録してください。" >&2
  exit 1
fi
```

**MCP経由の場合**（トークン管理が不要）:
```bash
claude mcp add --transport http slack https://mcp.slack.com
```

> MCP経由ならOAuth認証でトークン管理を自動化できるため、Keychain設定は不要です。

---

## 4. 実例②: Lint自動チェック（post-edit-lint.sh）

### 4-1. 概要

ファイル編集後に構文チェックを自動実行し、エラーを即座に検出します。

**対応フォーマット**:
- Markdown: YAMLフロントマターの閉じ`---`チェック
- Shell: `bash -n` 構文チェック
- JSON: `jq empty` 構文チェック

### 4-2. settings.json設定

```json
{
  "hooks": {
    "PostToolUse": {
      "type": "command",
      "command": "bash ~/.claude/hooks/post-edit-lint.sh",
      "matcher": "Edit|Write"
    }
  }
}
```

### 4-3. スクリプト構造

```bash
#!/bin/bash
# ~/.claude/hooks/post-edit-lint.sh

# ツール情報を取得
TOOL_NAME=$1
FILE_PATH=$2

# ファイル拡張子で処理分岐
case "${FILE_PATH##*.}" in
  md)
    # YAMLフロントマターチェック
    if head -n 1 "$FILE_PATH" | grep -q "^---$"; then
      if ! sed -n '2,/^---$/p' "$FILE_PATH" | grep -q "^---$"; then
        echo "❌ YAMLフロントマターが閉じられていません: $FILE_PATH"
        exit 1
      fi
    fi
    ;;
  sh|bash)
    # Shell構文チェック
    if ! bash -n "$FILE_PATH" 2>/dev/null; then
      echo "❌ Shell構文エラー: $FILE_PATH"
      bash -n "$FILE_PATH"
      exit 1
    fi
    ;;
  json)
    # JSON構文チェック
    if ! jq empty "$FILE_PATH" 2>/dev/null; then
      echo "❌ JSON構文エラー: $FILE_PATH"
      jq empty "$FILE_PATH"
      exit 1
    fi
    ;;
esac

echo "✅ Lint OK: $FILE_PATH"
```

### 4-4. エラー時の動作

構文エラーが検出されると、Hookが`exit 1`を返し、Claude Codeのコンテキストにエラー情報が追加されます。Claude Codeは自動的に修正を試みます。

---

## 5. 実例③: Push前チェック（pre-push-check.sh）

### 5-1. 概要

`git push` コマンド検出時に、未コミット変更や高リスクファイルをチェックし、警告を表示します。

### 5-2. settings.json設定

```json
{
  "hooks": {
    "PreToolUse": {
      "type": "command",
      "command": "bash ~/.claude/hooks/pre-push-check.sh",
      "matcher": "Bash"
    }
  }
}
```

### 5-3. スクリプト構造

```bash
#!/bin/bash
# ~/.claude/hooks/pre-push-check.sh

BASH_COMMAND=$1

# git pushコマンドのみ対象
if [[ ! "$BASH_COMMAND" =~ git.*push ]]; then
  exit 0
fi

# 未コミット変更チェック
UNCOMMITTED=$(git status --short)
if [ -n "$UNCOMMITTED" ]; then
  echo "⚠️ 未コミット変更があります:"
  echo "$UNCOMMITTED"
fi

# プッシュ対象コミット取得
COMMITS=$(git log origin/$(git branch --show-current)..HEAD --oneline)
if [ -n "$COMMITS" ]; then
  echo "📤 プッシュ対象コミット:"
  echo "$COMMITS"
fi

# 高リスクファイル変更チェック
HIGH_RISK=$(git diff origin/$(git branch --show-current)..HEAD --name-only | grep -E '\.(env|secret|key)$')
if [ -n "$HIGH_RISK" ]; then
  echo "🚨 高リスクファイルの変更を検出:"
  echo "$HIGH_RISK"
  echo "本当にプッシュしますか？ (y/n)"
  exit 2  # ユーザー確認を要求
fi

echo "✅ Push前チェック完了"
```

### 5-4. exit codeの意味

| exit code | 意味 | Claude Codeの動作 |
|-----------|------|------------------|
| 0 | 正常 | そのまま実行 |
| 1 | エラー | コンテキストにエラー追加 |
| 2 | ブロック | 実行を中断、ユーザー確認を要求 |

---

## 6. 実例④: コンテキスト再注入（compact-reinject.sh）

### 6-1. 概要

コンテキストウィンドウの圧縮（compaction）後、重要な情報を再注入して情報ロスを防ぎます。

### 6-2. settings.json設定

```json
{
  "hooks": {
    "SessionStart": {
      "type": "command",
      "command": "bash ~/.claude/hooks/compact-reinject.sh",
      "matcher": "compact"
    }
  }
}
```

### 6-3. 再注入する情報

```bash
#!/bin/bash
# ~/.claude/hooks/compact-reinject.sh

# 1. プロジェクト名
PROJECT_NAME=$(basename $(pwd))
echo "プロジェクト: $PROJECT_NAME"

# 2. 現在のブランチ
BRANCH=$(git branch --show-current)
echo "ブランチ: $BRANCH"

# 3. 直近5コミット
echo "直近のコミット:"
git log -5 --oneline

# 4. スプリント情報（存在する場合）
if [ -f ".sprint-logs/sprint-backlog.md" ]; then
  echo "スプリント情報:"
  head -n 20 .sprint-logs/sprint-backlog.md
fi

# 5. 未コミット変更
UNCOMMITTED=$(git status --short)
if [ -n "$UNCOMMITTED" ]; then
  echo "未コミット変更:"
  echo "$UNCOMMITTED"
fi
```

### 6-4. compaction時の注意点

- `/compact` コマンド実行時に自動圧縮が行われる
- 圧縮後は以前の会話履歴が要約される
- 重要な情報はCLAUDE.mdに「圧縮時の保持ルール」として記述可能

```markdown
# CLAUDE.md

## コンパクション時の保持ルール

以下の情報は/compact後も必ず保持してください:
- 現在のタスクID・ステータス
- 未解決のエラーメッセージ
- プロジェクトのディレクトリ構造
```

---

## 7. 機密ファイル保護（Slack連携環境での注意点）

Slack連携で `SLACK_TOKEN` を扱う場合、機密情報の保護が特に重要です。Section 3-6で紹介したmacOS Keychainでのトークン管理に加えて、**settings.jsonのDenyリスト**と**Hookスクリプト**による二段構えの保護を設定しましょう。

> 機密ファイル保護の詳細な設定方法（settings.json Denyリスト、protect-sensitive-files.shスクリプトの実装例、ブロック対象パターン一覧）については、[権限とセキュリティガイド Section 4](./security-guide.md#4-機密ファイル保護) を参照してください。

**Slack連携で特に注意すべき機密ファイル:**

- `.env`（SLACK_TOKENを環境変数で管理する場合 — 非推奨）
- `~/.claude/hooks/slack-feedback.sh`（トークンをハードコードしていないか確認）
- Slack App設定のBot Token（Keychain管理を推奨）

---

## 8. macOS通知システム（ccnotify.py + terminal-notifier）

### 8-1. 概要

タスク完了時にmacOSネイティブ通知を表示し、通知クリックでVSCodeを開きます。

### 8-2. 必要なツール

```bash
# terminal-notifierインストール
brew install terminal-notifier

# Python依存パッケージ
pip install sqlite3
```

### 8-3. ccnotify.py の構造

```python
#!/usr/bin/env python3
# ~/.claude/ccnotify/ccnotify.py

import sqlite3
import subprocess
from datetime import datetime

DB_PATH = "~/.claude/ccnotify/prompts.db"

def record_prompt(prompt_text):
    """プロンプトをDBに記録"""
    conn = sqlite3.connect(DB_PATH)
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO prompts (text, timestamp) VALUES (?, ?)",
        (prompt_text, datetime.now().isoformat())
    )
    conn.commit()
    conn.close()

def send_notification(title, message, project_path):
    """macOS通知を送信"""
    subprocess.run([
        "terminal-notifier",
        "-title", title,
        "-message", message,
        "-open", f"vscode://file/{project_path}",
        "-sound", "default"
    ])

# Hook経由で呼び出される
if __name__ == "__main__":
    import sys
    event_type = sys.argv[1]

    if event_type == "UserPromptSubmit":
        record_prompt(sys.argv[2])
    elif event_type == "Stop":
        send_notification(
            "Claude Code",
            "タスクが完了しました",
            os.getcwd()
        )
```

### 8-4. settings.json設定

```json
{
  "hooks": {
    "UserPromptSubmit": {
      "type": "command",
      "command": "python3 ~/.claude/ccnotify/ccnotify.py UserPromptSubmit"
    },
    "Stop": {
      "type": "command",
      "command": "python3 ~/.claude/ccnotify/ccnotify.py Stop"
    }
  }
}
```

---

## 9. 自作Hookの作り方TIPS

### 9-1. 基本テンプレート

```bash
#!/bin/bash
# ~/.claude/hooks/my-hook.sh

# 1. イベント情報を取得
TOOL_NAME=$1
TOOL_ARG=$2
PROJECT_PATH=$(pwd)

# 2. 処理を実行
echo "Hook実行中: $TOOL_NAME on $TOOL_ARG"

# 3. 必要に応じて外部サービスと連携
# curl -X POST ...

# 4. 結果を返す
exit 0  # 0: 成功, 1: エラー, 2: ブロック
```

### 9-2. デバッグ方法

Hook実行ログは `~/.claude/debug/` に記録されます。

```bash
# ログ確認
tail -f ~/.claude/debug/hooks.log
```

デバッグ用にstderrへ出力:
```bash
echo "Debug: TOOL_NAME=$TOOL_NAME" >&2
```

### 9-3. 実行権限の付与

```bash
chmod +x ~/.claude/hooks/my-hook.sh
```

### 9-4. Hookのテスト

```bash
# 直接実行してテスト
bash ~/.claude/hooks/my-hook.sh "Edit" "/path/to/file.md"
```

### 9-5. 再帰（無限ループ）防止

HookがClaude Codeのツール呼び出しをトリガーし、そのツール呼び出しが再びHookをトリガーする...という無限ループが発生するケースがあります。たとえば、PostToolUseのEditイベントでLintを実行し、そのLint結果に基づいてClaude Codeが再度Editを行うと、ループが始まります。

**再帰防止フラグの実装例**:

```bash
#!/bin/bash
# post-edit-lint-safe.sh — 再帰防止フラグ付きLintチェック
set -euo pipefail

LOCK_FILE="/tmp/claude-hook-lint.lock"

# 再帰検出: ロックファイルが存在すればスキップ
if [ -f "$LOCK_FILE" ]; then
  exit 0
fi

# ロックファイルを作成（最大60秒で自動削除）
touch "$LOCK_FILE"
trap 'rm -f "$LOCK_FILE"' EXIT

# --- ここにLint処理を記述 ---
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

if [ -z "$FILE_PATH" ]; then
  exit 0
fi

case "$FILE_PATH" in
  *.sh) shellcheck "$FILE_PATH" 2>&1 || true ;;
  *.json) jq empty "$FILE_PATH" 2>&1 || true ;;
  *.yaml|*.yml) yamllint "$FILE_PATH" 2>&1 || true ;;
esac
```

**ポイント**:
- `/tmp/claude-hook-lint.lock` の存在で再帰を検出し、即座に`exit 0`でスキップ
- `trap` でスクリプト終了時にロックファイルを必ず削除
- 万が一trapが効かなくても、60秒のttlで自動クリーンアップ（cronやtmpwatchを活用）

**環境変数による再帰防止（別のアプローチ）**:

```bash
#!/bin/bash
# 環境変数で再帰を検出するパターン
if [ "${CLAUDE_HOOK_RUNNING:-}" = "1" ]; then
  exit 0
fi
export CLAUDE_HOOK_RUNNING=1

# --- 処理本体 ---
```

> **注意**: 環境変数方式は、Hookが子プロセスとして実行される場合に親プロセスの環境変数を引き継がない場合があります。ロックファイル方式の方が確実です。

### 9-6. よくある問題と対処法

| 問題 | 原因 | 対処法 |
|------|------|--------|
| Hookが実行されない | 実行権限がない | `chmod +x` で権限付与 |
| パスが通らない | 相対パスを使用 | 絶対パス (`~/.claude/...`) を使用 |
| 環境変数が取れない | シェル環境が異なる | スクリプト内で `source ~/.zshrc` |
| タイムアウト | 処理が長すぎる | バックグラウンド実行 (`&`) を検討 |
| 無限ループ | Hook→ツール呼出→Hook | ロックファイルで再帰防止（Section 9-5参照） |

---

## 10. トラブルシューティング

### Hookが実行されない

settings.jsonにHookを設定したのに動作しない場合、まずスクリプトに実行権限があるか確認してください（`chmod +x ~/.claude/hooks/スクリプト名.sh`）。次に、スクリプトのパスが絶対パスになっているか確認します。`~/`は展開されますが、環境変数（`$HOME`など）はsettings.json内では展開されません。それでも動かない場合、Claude Codeを再起動してsettings.jsonを再読み込みさせてください。

### Slack通知が届かない

slack-feedback.shは動作しているのにSlackに投稿されない場合、以下を順番に確認します。Keychainからトークンが取得できているか（`security find-generic-password -a "claude-code" -s "slack-bot-token" -w`をターミナルで実行）。Bot Tokenのスコープに`chat:write`が含まれているか。Botが投稿先チャンネルに招待されているか（チャンネル設定の「インテグレーション」タブで確認）。トークンが正しいのにAPIエラーが返る場合、Slack Appの設定画面でトークンを再生成してみてください。

### Keychainのパスワード入力を毎回求められる

macOS Keychainにトークンを保存したのに、スクリプト実行のたびにパスワードダイアログが表示される場合、Keychainアクセス.appで該当エントリの「アクセス制御」を確認します。「すべてのアプリケーションにこの項目へのアクセスを許可」にチェックが入っていれば、ダイアログなしでアクセスできます。セキュリティを重視する場合は`/usr/bin/security`を許可リストに追加する方法もあります。

### リアクションが検出されない

Slackに投稿はされるが、絵文字リアクションを付けても次の指示が送られない場合、Bot Tokenのスコープに`reactions:read`と`channels:history`が含まれているか確認してください。また、待機時間（1分→3分→5分）が経過してスクリプトがタイムアウトしている可能性があります。`~/.claude/debug/`のログでフィードバック待機の状態を確認できます。

### post-edit-lint.shでエラーが出続ける

Lint Hookが誤検知でエラーを出し続ける場合、対象ファイルの拡張子判定が原因であることが多いです。例えば、YAMLフロントマター付きMarkdownで閉じ`---`のチェックが本文中の`---`（水平線）に反応してしまうケースがあります。スクリプト内のパターンを修正するか、特定のファイルをHookの対象外にする（matcherでフィルタリング）ことで対処できます。

---

## 11. まとめ

### 10-1. Hooksのベストプラクティス

Hooksを書き始めるとあれもこれも自動化したくなりますが、最初は1つだけ設定して様子を見るのが正解です。自分の場合、いきなり5つのHookを入れたら互いに干渉して原因特定に時間がかかりました。

設計で気をつけたいのは冪等性（同じHookが何度実行されても問題ない設計）、外部サービス連携時のエラーハンドリング（Slack APIが落ちているケースなど）、そしてデバッグ用のログ出力です。処理が重いHookはバックグラウンド実行にすることで、Claude Codeの応答を遅延させずに済みます。

### 10-2. 推奨Hook構成（初心者向け）

最初に入れるなら、この3つが実用性とリスクのバランスが良いです。**post-edit-lint.sh**で編集後の構文エラーを自動検出、**protect-sensitive-files.sh**で機密ファイルへの意図しない書き込みをブロック、**slack-feedback.sh**でタスク完了をSlack通知。この3つだけで「気づいたらエラーが混入していた」「.envを上書きされた」「作業完了を見逃した」という3大ストレスがなくなります。

### 10-3. 参考リンク

- [Claude Code公式ドキュメント - Hooks](https://code.claude.com/docs/ja/hooks)
- [Hooks基本的な使い方](https://dev.classmethod.jp/articles/claude-code-hooks-basic-usage/)
- [Hooks解説 - Zenn](https://zenn.dev/buddypia/articles/99abea47607225)
- [settings.json設定ガイド](https://syu-m-5151.hatenablog.com/entry/2025/06/05/134147)

---

## 付録: 完全な設定例

```json
{
  "hooks": {
    "UserPromptSubmit": {
      "type": "command",
      "command": "python3 ~/.claude/ccnotify/ccnotify.py UserPromptSubmit"
    },
    "SessionStart": {
      "type": "command",
      "command": "bash ~/.claude/hooks/compact-reinject.sh",
      "matcher": "compact"
    },
    "PreToolUse": [
      {
        "type": "command",
        "command": "bash ~/.claude/hooks/pre-push-check.sh",
        "matcher": "Bash"
      },
      {
        "type": "command",
        "command": "bash ~/.claude/hooks/protect-sensitive-files.sh",
        "matcher": "Edit|Write"
      }
    ],
    "PostToolUse": {
      "type": "command",
      "command": "bash ~/.claude/hooks/post-edit-lint.sh",
      "matcher": "Edit|Write"
    },
    "Stop": [
      {
        "type": "command",
        "command": "bash ~/.claude/hooks/slack-feedback.sh"
      },
      {
        "type": "command",
        "command": "python3 ~/.claude/ccnotify/ccnotify.py Stop"
      }
    ],
    "Notification": {
      "type": "command",
      "command": "bash ~/.claude/hooks/slack-feedback.sh"
    }
  }
}
```

この設定により、編集後の自動チェック、機密ファイル保護、Slack連携、macOS通知がすべて動作します。
