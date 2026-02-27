# Claude Code 利用制限とサンドボックス活用ガイド

## はじめに

Claude Codeを効果的に使いこなすには、**リソース管理の仕組み**を理解することが重要です。このガイドでは、以下の3つのトピックを扱います。

1. **権限とセキュリティ** — Allowリスト、サンドボックス、機密ファイル保護
2. **コンテキストウィンドウ管理** — トークン制限、キャッシュ戦略、/compact活用
3. **利用制限の把握と対策** — 5時間/7日間制限、Usage API、最適化のコツ

このガイドを読むことで、セキュアかつ効率的にClaude Codeを使えるようになります。

---

## 2. 権限とセキュリティ

### 2-1. 権限モード（defaultMode）の違い

Claude Codeは、ツールを実行する際にユーザーの許可を求めるタイミングを4種類のモードで制御できます。

| モード | 動作 | 用途 |
|--------|------|------|
| `ask` | すべてのツール実行前に確認プロンプトを表示 | セキュリティ最優先・初心者向け |
| `auto` | Allowリストに一致すれば自動実行。それ以外は確認 | バランス型（推奨） |
| `delegate` | Claudeが自律判断。危険と判断したもののみ確認 | 作業効率優先・上級者向け |
| `plan` | 実行前に計画全体を表示し、一括承認 | 複雑なタスクを事前レビューしたい場合 |

**設定場所**: `~/.claude/settings.json`

```json
{
  "permissions": {
    "defaultMode": "delegate"
  }
}
```

**推奨設定**:
- 初心者: `ask` または `auto`
- 慣れたら: `delegate` + Allowリスト整備
- チーム共有環境: `auto` + 厳格なAllow/Denyリスト

### 2-2. Allow/Denyリスト — パターンマッチ

Allowリストに登録されたパターンに一致するツール呼び出しは、確認なしで自動実行されます。

**基本構文**:
```
Bash(command args)
Read(file_path)
Edit(file_path)
Write(file_path)
mcp__server_name__tool_name
WebSearch
WebFetch(domain:example.com)
```

**パターンマッチのルール**:
- `*` は任意の文字列にマッチ（例: `git *`, `npm *`）
- コマンド+引数の組み合わせ全体でマッチング
- シェル演算子（`&&`, `|`, `>` 等）を含むコマンドは **パターンマッチ不可**（後述）

### 2-3. よく使う設定例

**汎用的なAllow設定の例**:

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(gh *)",
      "Bash(npm *)",
      "Bash(node *)",
      "Bash(python3 *)",
      "Bash(pip3 *)",
      "Bash(ls *)",
      "Bash(cd *)",
      "Bash(mkdir *)",
      "Bash(cat *)",
      "Bash(grep *)",
      "Bash(find *)",
      "Bash(curl *)",
      "Bash(jq *)",
      "Bash(tmux *)",
      "WebSearch",
      "WebFetch",
      "mcp__slack-fast-mcp__slack_post_message"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(rm -rf /*)",
      "Bash(sudo *)"
    ]
  }
}
```

**プロジェクト固有のAllow設定**:
```json
{
  "permissions": {
    "allow": [
      "Bash(./scripts/*)",
      "Bash(~/.claude/hooks/* *)",
      "Read(~/.claude/try-stock.md)",
      "Read(~/dev/.claude/*)"
    ]
  }
}
```

**TIPS**:
- コマンドのパターンを増やしすぎると保守が大変になるため、**よく使うものだけ登録**する
- MCPツールは `mcp__server_name__tool_name` の形式で登録
- `deny` リストは `allow` より優先される（誤った危険操作のブロックに使う）

### 重要: シェル演算子を使ったコマンドはAllowリスト対象外

以下のようなシェル演算子を含むコマンドは、**Allowリストのパターンマッチが効かない**ため、毎回確認プロンプトが表示されます。

**パターンマッチ不可の例**:
```bash
# NG: コマンド連結
git add . && git commit -m "message"

# NG: パイプ
cat file.txt | grep pattern

# NG: リダイレクト
echo "text" > output.txt

# NG: コマンド置換
git commit -m "$(date)"
```

**対策**:
- コマンドを分割して1つずつ実行する
- 専用ツール（Read/Write/Edit/Grep）を使う
- HEREDOCでのコミットメッセージは例外的に許容される

**正しいパターン（HEREDOC使用）**:
```bash
git commit -m "$(cat <<'EOF'
Commit message here
EOF
)"
```

---

## 3. サンドボックス

### 3-1. サンドボックスとは

サンドボックスは、Claude Codeが実行するBashコマンドを**OSレベルで隔離**し、ファイルシステムやネットワークへのアクセスを制限する機能です。

**サンドボックスの効果**:
- ファイル書き込み: ホワイトリスト以外のディレクトリへの書き込みをブロック
- ネットワーク: 指定されたホスト以外への通信をブロック
- セキュリティ: 誤操作や悪意あるコマンドの影響を限定

**サンドボックスの制約**:
- 一部のツール（例: Docker、一部のMCPサーバー）がUnixソケットアクセスを必要とする場合、サンドボックス内で動作しない
- ファイルパスの制限により、意図した動作ができないケースがある

### 3-2. サンドボックス設定

**設定場所**: `~/.claude/settings.json`

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "allowUnsandboxedCommands": true
  }
}
```

| パラメータ | 説明 | 推奨値 |
|-----------|------|--------|
| `enabled` | サンドボックスを有効化 | `true` |
| `autoAllowBashIfSandboxed` | サンドボックス内なら自動許可 | `true` |
| `allowUnsandboxedCommands` | サンドボックス外コマンドを許可 | `true`（状況次第） |

**推奨設定**:
- デフォルトは **サンドボックス有効 + 必要時のみバイパス**
- サンドボックス外コマンドが必要な場合のみ `allowUnsandboxedCommands: true` を設定

**サンドボックスのホワイトリスト（Filesystem Write）**:
```json
{
  "sandbox": {
    "filesystem": {
      "write": {
        "allowOnly": [
          "/tmp/claude",
          "/private/tmp/claude",
          ".",
          "/Users/username/.npm/_logs",
          "/Users/username/.claude/debug"
        ]
      }
    }
  }
}
```

**ポイント**:
- カレントディレクトリ（`.`）への書き込みは許可される
- 一時ファイルは `/tmp/claude` を使う（`TMPDIR` 環境変数が自動設定される）

### 3-3. --dangerously-skip-permissions の正しい使い方

`--dangerously-skip-permissions` フラグは、すべての権限チェックをバイパスするオプションです。

> **⚠️ 重要な警告: このフラグの危険性を正しく理解してください**
>
> このフラグを有効にすると、Claude Codeは**一切の確認なし**で以下を実行できます:
>
> - **ファイル破壊**: `rm -rf /` や `rm -rf ~/*` が即座に実行される
> - **機密情報の漏洩**: `.env`, `credentials.json`, SSH秘密鍵が無確認で読み書きされる
> - **Git操作の暴走**: `git push --force`, `git reset --hard` が確認なしで実行される
> - **プロジェクト外アクセス**: `~/.ssh/`, `~/.gnupg/`, `~/.aws/` など機密ディレクトリにアクセスされる
>
> **サンドボックスを併用すれば安全に使える**: サンドボックスがOSレベルでファイルシステム・ネットワークを隔離するため、上記リスクの大部分が軽減されます。

**条件付き許容（推奨構成）**:

このフラグは以下の条件を**すべて満たす**場合にのみ使用してください:

1. **サンドボックスが有効**（`sandbox.enabled: true`）
2. **permissions.deny で危険コマンドをブロック済み**
3. **tmux環境などバックグラウンド実行が必要な場合**

**正しい使用例**:
- サンドボックス制約でコマンドが動作しない場合（Docker、Unixソケット通信など）
- tmux環境で対話的な権限確認が困難な場合（サンドボックス併用前提）

**設定方法**:
```json
{
  "skipDangerousModePermissionPrompt": true
}
```

このフラグを `true` にすると、`dangerouslyDisableSandbox` オプション付きのコマンドが確認なしで実行されます。

**注意点**:
- **原則としてサンドボックス内で実行すること**（最重要）
- サンドボックス外で実行する場合は、ファイルパスやネットワークアクセスを慎重に確認
- 公開リポジトリに設定ファイルをコミットする場合、このフラグは含めない
- **初心者はこのフラグを使わず、まずはAllow/Denyリストの整備から始めること**

**ctコマンドでの自動付与**:
`ct` コマンド（Claude Code Tmux統合）を使うと、自動的に `--dangerously-skip-permissions` が付与されます。

```bash
ct "タスクを実行してください"
```

これは、tmuxセッション内での操作が信頼できる環境であることを前提としています。**ctコマンドを使う場合も、必ずサンドボックスを有効にしてください。**

---

## 4. 機密ファイル保護

### 4-0. 二段構えの保護戦略（推奨）

機密ファイルの保護は、**settings.jsonのDenyリスト**（第1防衛線）と**Hookスクリプト**（第2防衛線）の二段構えで行うことを推奨します。

> **なぜ二段構えが必要か?**
> - Hookスクリプトはファイル削除や設定ミスでバイパスされる可能性がある
> - settings.jsonのDenyリストはClaude Code本体が直接参照するため、より堅牢
> - 両方を設定することで、片方が失敗してももう一方が保護を維持する

**第1防衛線: settings.json の permissions.deny**

```json
{
  "permissions": {
    "deny": [
      "Edit(.env)",
      "Edit(.env.*)",
      "Edit(credentials.json)",
      "Edit(secrets.json)",
      "Edit(service-account*.json)",
      "Write(.env)",
      "Write(.env.*)",
      "Write(credentials.json)",
      "Write(secrets.json)",
      "Write(service-account*.json)",
      "Read(.env)",
      "Read(.env.*)",
      "Read(credentials.json)",
      "Read(secrets.json)",
      "Bash(rm -rf /)",
      "Bash(rm -rf /*)",
      "Bash(sudo *)"
    ]
  }
}
```

> **ポイント**: `deny` リストは `allow` リストより**常に優先**されます。たとえ `allow` に `Edit(*)` があっても、`deny` に `Edit(.env)` があればブロックされます。

**第2防衛線: protect-sensitive-files.sh Hook**（下記参照）

### 4-1. protect-sensitive-files.sh hookの仕組み

Claude Codeは、**PreToolUse（Edit|Write）** フックで機密ファイルへの書き込みを自動ブロックする機能を提供します。settings.jsonのDenyリストと併用することで、二重の保護を実現します。

**hookの設定例**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/protect-sensitive-files.sh"
          }
        ]
      }
    ]
  }
}
```

### 4-2. ブロック対象パターン

`protect-sensitive-files.sh` は以下のファイルへの書き込みをブロックします。

**ファイル名パターン**:
- `.env`, `.env.local`, `.env.production`, `.env.staging`, `.env.development`
- `credentials.json`, `secrets.json`, `service-account*.json`
- `id_rsa`, `id_ed25519`, `*.pem`, `*.key`

**パスパターン**:
- `*/.ssh/*` — SSH設定ディレクトリ
- `*/.gnupg/*` — GPG設定ディレクトリ

**動作例**:
```bash
# ブロック例
Edit(.env)
→ "環境変数ファイル (.env) への直接編集はブロックされています"

Write(~/.ssh/id_rsa)
→ "SSH設定ディレクトリ内のファイルへの編集はブロックされています"
```

**hook実装のポイント**:
- `exit 2` でブロック（`exit 0` は通過、`exit 1` はエラー）
- JSONでファイルパスを取得し、パターンマッチで判定
- ブロック理由を標準エラー出力に出力

**protect-sensitive-files.sh の実装例**:
```bash
#!/bin/bash
set -euo pipefail

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

if [ -z "$FILE_PATH" ]; then
  exit 0
fi

BASENAME=$(basename "$FILE_PATH")

# ファイル名パターンチェック
case "$BASENAME" in
  .env|.env.local|.env.production|.env.staging|.env.development)
    echo "環境変数ファイル (${BASENAME}) への直接編集はブロックされています" >&2
    exit 2
    ;;
  credentials.json|secrets.json|service-account*.json)
    echo "認証情報ファイル (${BASENAME}) への直接編集はブロックされています" >&2
    exit 2
    ;;
  id_rsa|id_ed25519|*.pem|*.key)
    echo "秘密鍵ファイル (${BASENAME}) への直接編集はブロックされています" >&2
    exit 2
    ;;
esac

# パスパターンチェック
case "$FILE_PATH" in
  */.ssh/*)
    echo "SSH設定ディレクトリ内のファイルへの編集はブロックされています" >&2
    exit 2
    ;;
  */.gnupg/*)
    echo "GPG設定ディレクトリ内のファイルへの編集はブロックされています" >&2
    exit 2
    ;;
esac

exit 0
```

---

## 5. コンテキストウィンドウ管理

### 5-1. コンテキストウィンドウの仕組み

Claude Codeのコンテキストウィンドウは、Claudeが「覚えている情報の量」を表します。このウィンドウがいっぱいになると、以前の指示を忘れたり、パフォーマンスが低下したりします。

**コンテキストウィンドウの構成要素**:

| 要素 | 説明 | トークン消費 |
|------|------|-------------|
| **Input Tokens** | ユーザーの指示、Readしたファイル、ツール出力 | あり |
| **Output Tokens** | Claudeの応答、生成したコード | あり |
| **Cache Write** | Claudeがキャッシュに保存した情報 | あり（初回のみ） |
| **Cache Read** | キャッシュから読み込んだ情報 | **90%割引** |

**キャッシュのメリット**:
- CLAUDE.md、よく読むファイル、長いツール出力をキャッシュすることで、次回以降のトークン消費を大幅削減
- キャッシュは5分間有効（5分以内に再度アクセスすると、Cache Readとして処理される）

**セッション累計の追跡**:
- `total_input_tokens`: セッション全体で送信したInput Tokens
- `total_output_tokens`: セッション全体でClaudeが生成したOutput Tokens
- コンテキストウィンドウの使用率（%）が80%を超えると警告

### 5-2. 使用状況の確認方法

**ステータスラインでリアルタイム確認**

Claude Codeのステータスラインは、4行のダッシュボード形式でコンテキストウィンドウ情報を表示します。

```
Line 1: [dir] [branch] | [model] | [context bar] [used%] [label] | [cost] | [duration] | [lines +/-]
Line 2: 📊 🪟 [current/total] (W:[cache_write] R:[cache_read]) | In:[total_input] Out:[total_output] | 残[remaining%]
Line 3: ⚡ [5h%] [reset] | 📅 [week%] [reset] | 🎵 [Sonnet%] [reset] | 🎹 [Opus%] [reset] | 🔥 [limit_impact]
Line 4: 🧠 CC:[memory] | App:[memory] | VSCode:[memory]
```

**Line 2の読み方**:
- `🪟 45.2k/200k` — 現在のコンテキストウィンドウ使用量/最大容量
- `W:12.3k` — キャッシュに書き込んだトークン数（初回コスト）
- `R:38.5k` — キャッシュから読み込んだトークン数（90%割引）
- `In:102k` — セッション累計Input Tokens
- `Out:25k` — セッション累計Output Tokens
- `残35%` — コンテキストウィンドウの残量

**色による警告**:
- 緑: 0-59% — OK
- 黄: 60-79% — WATCH（注意）
- 赤: 80-100% — SWITCH（新しいセッションを推奨）

### 5-3. /compact の活用

**コンパクションとは**:
コンパクションは、現在のセッションの情報を**要約・圧縮**して新しいセッションに引き継ぐ機能です。コンテキストウィンドウが満杯に近づいた際に使用します。

**使い方**:
```
/compact 以下の情報を保持してください:
- プロジェクトの構造
- 現在実装中の機能
- 未解決のバグリスト
```

**自動コンパクション**:
コンテキストウィンドウが制限に近づくと、Claudeが自動的にコンパクションを提案します。

**情報ロスへの対策**:

| 対策 | 説明 |
|------|------|
| **保持すべき情報を明示** | コンパクション時に「保持すべき情報」を具体的に指示する |
| **CLAUDE.mdで定義** | SessionStartフックでcompact後に再注入するスクリプトを設定 |
| **マイルストーン・タスク管理を外部化** | scrum/tasks.md、scrum/milestones.md で進捗を管理 |

**CLAUDE.mdでのcompact保持ルール例**:
```markdown
# コンパクション時の保持ルール

/compact 実行時、以下を必ず保持:
- プロジェクト名・概要
- 現在のスプリントID・タスクID
- 未完了タスクのリスト
- 技術スタック・依存関係
```

### 5-4. ベストプラクティス

**コンテキストウィンドウを節約するコツ**:

| 手法 | 説明 |
|------|------|
| **長いファイルは部分読み** | `Read(file_path, offset, limit)` で必要な部分だけ読む |
| **大きなタスクは分割** | 1セッションで複数の大きなタスクを実行しない |
| **定期的にcompact** | コンテキストウィンドウが60%を超えたら検討 |
| **キャッシュヒット率を上げる** | CLAUDE.md、よく使うファイルを最初に読み込む |
| **不要なReadを減らす** | 同じファイルを何度も読まない（一度読んだらキャッシュされる） |

---

## 6. 利用制限の把握

### 6-1. 制限の種類

Claude Codeには、プランに応じた利用制限が設定されています。

**制限の種類**:

| 制限 | 説明 | リセット周期 |
|------|------|------------|
| **5時間ローリングウィンドウ** | 直近5時間の利用量（プロンプト数） | ローリング（常に過去5時間分） |
| **7日間制限** | 直近7日間の利用量（トークン数） | 週単位 |
| **モデル別制限** | Sonnet/Opus それぞれの7日間制限 | 週単位 |

**プラン別の制限目安**:

| プラン | 5時間制限 | 7日間制限 |
|--------|-----------|-----------|
| Pro | 約10〜40プロンプト | 標準 |
| Max 5x | Proの5倍 | Proの5倍 |
| Max 20x | Proの20倍 | Proの20倍 |

**注意点**:
- 正確な制限値は公開されていない（利用状況によって変動する可能性がある）
- Cache Write（キャッシュ書き込み）は制限にカウントされる
- Cache Read（キャッシュ読み込み）は90%割引でカウントされる

### 6-2. 確認方法

**ステータスライン（Line 3）で確認**:

```
⚡ 45% 2h 15m | 📅 62% 4d 3h | 🎵 58% 4d 3h | 🎹 12% 4d 3h | 🔥 102k
```

| 記号 | 説明 |
|------|------|
| ⚡ | 5時間ローリングウィンドウの使用率とリセットまでの時間 |
| 📅 | 7日間制限の使用率とリセットまでの時間 |
| 🎵 | Sonnetモデルの7日間使用率 |
| 🎹 | Opusモデルの7日間使用率 |
| 🔥 | セッションの制限影響量（Input + Cache Write） |

**Usage API（limit-tracker-cache.sh）**:

Claude Codeは、Anthropic OAuth Usage APIを定期的に呼び出し、制限情報をキャッシュしています。

**キャッシュファイルの場所**:
```
~/.claude/cache/limit-tracker.json
```

**キャッシュ更新ロジック**:
- ステータスラインが描画されるたびに、キャッシュの古さをチェック
- 60秒以上古い場合、バックグラウンドで `limit-tracker-cache.sh` を実行
- ロックファイル（`limit-tracker.lock`）で多重実行を防止

**limit-tracker-cache.sh の処理フロー**:
1. macOS Keychainから Claude Code の OAuthトークンを取得
2. `https://api.anthropic.com/api/oauth/usage` にリクエスト
3. レスポンスを `limit-tracker.json` に保存
4. ステータスラインがこのJSONを読み込んで表示

**キャッシュのデータ構造**:
```json
{
  "five_hour": {
    "utilization": 45,
    "resets_at": "2026-02-27T16:30:00Z"
  },
  "seven_day": {
    "utilization": 62,
    "resets_at": "2026-03-02T12:00:00Z"
  },
  "seven_day_sonnet": {
    "utilization": 58,
    "resets_at": "2026-03-02T12:00:00Z"
  },
  "seven_day_opus": {
    "utilization": 12,
    "resets_at": "2026-03-02T12:00:00Z"
  }
}
```

### 6-3. 制限到達時の対応

**制限が近づいたときの対処法**:

| 対処法 | タイミング | 説明 |
|--------|-----------|------|
| **モデル切替（/model）** | 50%以上 | Opus → Sonnet → Haikuに切り替える |
| **セッション分割** | 60%以上 | 大きなタスクを複数のセッションに分割 |
| **制限リセット待ち** | 80%以上 | リセットまでの時間を確認し、緊急性の低いタスクは後回し |
| **Haiku活用** | 常時 | 単純なタスク（Lint、テスト実行）はHaikuで |

**モデル切替のコマンド**:
```
/model sonnet
/model opus
/model haiku
```

**セッション分割の例**:
```
# Before（1セッションで実行）
タスクA（実装） → タスクB（テスト） → タスクC（ドキュメント）

# After（セッション分割）
Session 1: タスクA（実装）
Session 2: タスクB（テスト） + タスクC（ドキュメント）
```

---

## 7. 利用制限の最適化

### 7-1. キャッシュヒット率を上げるコツ

キャッシュヒット率を上げることで、制限へのカウントを大幅に削減できます。

**キャッシュ戦略**:

| 戦略 | 説明 |
|------|------|
| **CLAUDE.mdを最初に読み込む** | プロジェクトルールをキャッシュに載せる |
| **よく使うファイルを先に読む** | scrum/tasks.md、persona/*.md などを早めに読む |
| **5分以内に再アクセス** | キャッシュの有効期限は5分。連続作業時に効果的 |
| **SessionStartフックで再注入** | compact後に必要な情報を自動再注入 |

**キャッシュの有効期限**:
- キャッシュは **5分間** 有効
- 5分以内に同じ情報にアクセスすると、Cache Read（90%割引）として処理される
- 5分以上経過すると、再度Cache Write（通常コスト）が発生

### 7-2. Haiku活用

Haikuモデルは、Sonnet/Opusよりも軽量で、制限へのカウントが少ないモデルです。

**Haiku適用シーン**:
- Lintエラーの確認
- テストの実行
- ファイルの検索・一覧表示
- 簡単なコード修正（軽微なバグ修正、フォーマット調整）

**Haiku不向きなシーン**:
- 複雑なロジックの実装
- アーキテクチャ設計
- 大規模なリファクタリング

**モデル切替のタイミング**:
- 制限が50%を超えたら、Haikuで対応可能なタスクはHaikuに切り替える
- 制限リセットまで数時間以内の場合、Haikuで時間稼ぎ

### 7-3. /context の活用

`/context` コマンドで、現在のセッションのコンテキスト情報を確認できます。

```
/context
```

**表示内容**:
- コンテキストウィンドウの使用率
- Input/Output Tokensの累計
- キャッシュのWrite/Read状況
- 制限へのカウント状況

### 7-4. その他の最適化TIPS

**不要なReadを減らす**:
- 同じファイルを何度も読まない（キャッシュに載っている情報は再利用される）
- ファイル全体ではなく、必要な部分だけ読む（`Read(file_path, offset, limit)`）

**ツール選択の工夫**:
- `Bash(cat file.txt)` ではなく `Read(file_path)` を使う（Bashコマンドより効率的）
- `Bash(grep pattern file.txt)` ではなく `Grep(pattern, file_path)` を使う

**セッション管理**:
- 長時間のセッションは定期的に /compact を実行
- 大きなタスクは事前に分割し、それぞれ独立したセッションで実行

---

## 8. TIPS

### セキュリティTIPS

- **機密ファイルは protect-sensitive-files.sh でブロック**する
- **サンドボックスを原則有効**にし、必要時のみバイパスする
- **Denyリストで危険なコマンドをブロック**する（`rm -rf /`, `sudo *` など）
- **公開リポジトリに settings.json をコミットしない**（個人情報・トークンが含まれる場合）

### コンテキスト管理TIPS

- **コンテキストウィンドウが60%を超えたら /compact を検討**
- **compact時に保持すべき情報を明示**する（プロジェクト構造、未完了タスク）
- **マイルストーン・タスク管理は外部ファイルで管理**する（scrum/tasks.md）

### 制限対策TIPS

- **制限が50%を超えたらHaikuモデルに切り替える**
- **Usage APIのキャッシュを定期確認**（`~/.claude/cache/limit-tracker.json`）
- **リセットまでの時間を把握し、緊急性の低いタスクは後回し**にする
- **キャッシュヒット率を上げるため、CLAUDE.mdやよく使うファイルを先に読む**

### Allowリスト保守TIPS

- **よく使うコマンドのみ登録**し、パターンを肥大化させない
- **シェル演算子を含むコマンドはAllowリスト対象外**（コマンドを分割する）
- **プロジェクト固有のコマンドはプロジェクトごとに設定**（`./scripts/*` など）

---

## まとめ

Claude Codeのリソース管理を理解することで、セキュアかつ効率的に作業できるようになります。

**重要ポイント**:
1. **権限モード**: `delegate` + Allowリストで効率化
2. **サンドボックス**: 原則有効、必要時のみバイパス
3. **機密ファイル保護**: PreToolUseフックで自動ブロック
4. **コンテキスト管理**: 60%超えたら /compact 検討、キャッシュ戦略を活用
5. **制限対策**: Usage APIで確認、Haiku活用、モデル切替

このガイドを参考に、あなたのワークフローを最適化してください。
