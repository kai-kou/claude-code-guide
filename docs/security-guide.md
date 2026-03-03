# Claude Code 権限とセキュリティガイド

## はじめに

### こんな困りごとありませんか？

- 毎回「このコマンドを実行していいですか？」の確認が煩わしい
- `.env`や秘密鍵をClaude Codeが勝手に読み書きしないか不安
- サンドボックスの設定方法がよくわからない

### このガイドで解決できます

Allowリスト・サンドボックス・機密ファイル保護の3層防御を設定すれば、安全性を保ちつつ作業テンポを大幅に改善できます。

| Before | After |
|--------|-------|
| 毎回のツール確認ダイアログでテンポが悪い | Allowリスト + サンドボックスで信頼済み操作は自動実行 |
| 機密ファイルを誤って編集するリスク | Denyリスト + Hookの二段構えで自動ブロック |
| サンドボックスの挙動がわからず不安 | 設定を理解し、必要時のみバイパスする運用を確立 |

### 前提条件

| 項目 | 要件 |
|------|------|
| OS | macOS 13.0+ / Windows 10 1809+ / Ubuntu 20.04+（サンドボックスはWSL 2が必要） |
| Claude Code | インストール済み・認証済み |
| オプション | jq（settings.json編集の確認用） |

---

## 1. 権限モード（defaultMode）の違い

Claude Codeには4つの権限モードがあり、ツール実行時にどこまで自動化するかを制御します。

`ask`モードはすべての操作で確認プロンプトが表示されるため、Claude Codeが何をしようとしているか逐一把握できます。初心者やセキュリティを最優先したい場面に向いています。`auto`モードはAllowリストに登録したパターンは自動実行し、それ以外は確認を求めるバランス型です。`delegate`モードはClaudeが自律的に判断し、危険と判断した操作のみ確認します。作業テンポを重視する上級者向けです。`plan`モードは実行前に計画全体を表示して一括承認する方式で、複雑なタスクの事前レビューに使います。

最初は`ask`か`auto`で始めて、Allowリストを育てながら徐々に`delegate`に移行するのが、安全性と効率のバランスが取りやすいやり方です。

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

---

## 2. Allow/Denyリスト — パターンマッチ

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

### よく使う設定例

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

### サンドボックスとは

サンドボックスは、Claude Codeが実行するBashコマンドを**OSレベルで隔離**し、ファイルシステムやネットワークへのアクセスを制限する機能です。

**サンドボックスの効果**:
- ファイル書き込み: ホワイトリスト以外のディレクトリへの書き込みをブロック
- ネットワーク: 指定されたホスト以外への通信をブロック
- セキュリティ: 誤操作や悪意あるコマンドの影響を限定

**サンドボックスの制約**:
- 一部のツール（例: Docker、一部のMCPサーバー）がUnixソケットアクセスを必要とする場合、サンドボックス内で動作しない
- ファイルパスの制限により、意図した動作ができないケースがある

### サンドボックス設定

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

### --dangerously-skip-permissions の正しい使い方

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

### 二段構えの保護戦略（推奨）

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

### protect-sensitive-files.sh hookの仕組み

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

### ブロック対象パターン

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

## 5. TIPS

### セキュリティTIPS

- **機密ファイルは protect-sensitive-files.sh でブロック**する
- **サンドボックスを原則有効**にし、必要時のみバイパスする
- **Denyリストで危険なコマンドをブロック**する（`rm -rf /`, `sudo *` など）
- **公開リポジトリに settings.json をコミットしない**（個人情報・トークンが含まれる場合）

### Allowリスト保守TIPS

- **よく使うコマンドのみ登録**し、パターンを肥大化させない
- **シェル演算子を含むコマンドはAllowリスト対象外**（コマンドを分割する）
- **プロジェクト固有のコマンドはプロジェクトごとに設定**（`./scripts/*` など）

---

## 6. トラブルシューティング

### サンドボックスでコマンドがブロックされる

「Operation not permitted」エラーが出る場合、サンドボックスのファイルシステム制限に引っかかっています。プロジェクトディレクトリ外への書き込みが必要な場合は、settings.jsonの`sandbox.filesystem.write.allowOnly`にパスを追加してください。Dockerやデータベースツールなど、Unixソケット通信が必要なツールはサンドボックス内で動作しないことがあります。その場合、Claude Codeが自動的に`dangerouslyDisableSandbox`オプション付きで再実行を提案します。

### Allowリストに登録したのに毎回確認が出る

パターンを登録したのに確認プロンプトが消えない場合、コマンドにシェル演算子（`&&`、`|`、`>`など）が含まれていないか確認してください。シェル演算子を含むコマンドはAllowリストのパターンマッチが効きません。コマンドを分割して1つずつ実行するか、専用ツール（Read/Write/Grep）に置き換える必要があります。パターンの書き方が間違っている場合もあるため、`Bash(git *)`のように`ツール名(パターン)`の形式になっているか確認してください。

### protect-sensitive-files.shが機密ファイルをブロックしない

Hookを設定したのに`.env`への書き込みがブロックされない場合、settings.jsonでのHook設定を確認してください。`matcher`が`"Edit|Write"`になっているか、スクリプトのパスが正しいかをチェックします。Hookスクリプトが正常に動作しているかは、ターミナルで直接実行してテストできます（`echo '{"tool_input":{"file_path":".env"}}' | bash ~/.claude/hooks/protect-sensitive-files.sh`）。念のため、settings.jsonの`permissions.deny`にも`.env`パターンを登録しておくと二段構えで安全です。

---

## まとめ

Allowリストとサンドボックスで権限を整え、protect-sensitive-files.shとDenyリストの二段構えで機密ファイルを守る。この設定を一度済ませてしまえば、日常的に意識するのは「サンドボックスが有効になっているか」と「新しいプロジェクトでAllow設定を追加するか」くらいです。安全性と作業テンポの両立は、最初の30分の設定で決まります。
