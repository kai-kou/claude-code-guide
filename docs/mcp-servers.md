---
title: MCP サーバー連携ガイド
section: 9
created: 2026-02-27
updated: 2026-02-27
target: Claude Code中級者
topics:
  - Model Context Protocol
  - MCP設定方法
  - おすすめMCPサーバー
  - OAuth認証
  - セキュリティ
---

# MCP サーバー連携ガイド

MCP（Model Context Protocol）は、Claude Codeと外部サービスを接続するためのオープンスタンダードです。たとえば「Slack MCPサーバーを追加してね」と設定するだけで、Claude Codeのチャットから「#generalに進捗報告を投稿して」と指示できるようになります。GitHubのissue操作やNotionのページ更新も同様です。

MCPを導入する前は、ブラウザでSlackを開いて手動投稿、GitHubのPR画面を行き来してレビュー…という作業をしていました。MCP経由でClaude Codeに任せるようになってからは、ターミナルから離れる回数が激減しました。

---

## 1. MCPとは

### 概要

MCPは、AIアシスタントが外部ツールやデータソースにアクセスするための標準プロトコルです。Claude Codeに限らず、Cursorやその他のAIツールでも採用が広がっています。

### MCPで何ができるか

MCPに対応したサーバーは数多くありますが、実際によく使われるのはSlack（メッセージ投稿・チャンネル検索・スレッド読み取り）、GitHub（Issue/PR操作・コード検索・レビュー）、Notion（ページ作成・更新・データベース操作）あたりです。開発系ではSentry（エラー監視）、Playwright（ブラウザ自動操作・E2Eテスト）、Context7（ライブラリの最新ドキュメント参照）も重宝します。PostgreSQLに直接クエリを投げることもできますが、本番環境への接続には十分注意してください。

### サーバータイプ

MCPサーバーには3つのトランスポートタイプがあります。

| タイプ | 説明 | 用途 |
|--------|------|------|
| `http`（streamable-http） | HTTPベースのリモートサーバー | クラウドサービス連携（推奨） |
| `stdio` | ローカルプロセスとの通信 | ローカルツール（Playwright等） |
| `sse` | Server-Sent Events | **非推奨**。`http`への移行を推奨 |

---

## 2. 設定方法

### claude mcp add コマンド

MCPサーバーの追加は `claude mcp add` コマンドで行います。

**HTTPサーバー（リモートサービス向け）**

```bash
claude mcp add --transport http <サーバー名> <URL>
```

**stdioサーバー（ローカルツール向け）**

```bash
claude mcp add <サーバー名> -- <コマンド> [引数...]
```

**管理コマンド**

```bash
# 一覧表示
claude mcp list

# 詳細確認
claude mcp get <サーバー名>

# 削除
claude mcp remove <サーバー名>

# Claude Desktopからインポート（macOS / WSL）
claude mcp add-from-claude-desktop
```

### オプションの記述順序

`--transport`、`--scope`、`--env`、`--header` はサーバー名の**前**に置きます。

```bash
# 正しい順序
claude mcp add --transport http --scope user notion https://mcp.notion.com/mcp

# 環境変数付き（stdioサーバー）
claude mcp add --transport stdio --env API_KEY=xxx myserver -- python server.py
```

### 3つのスコープ

MCPサーバーの設定は3つのスコープに保存できます。用途に応じて使い分けます。

| スコープ | 保存先 | 用途 | 共有 |
|---------|--------|------|------|
| `local`（デフォルト） | `~/.claude.json` | 個人・現プロジェクト用 | 共有されない |
| `project` | `.mcp.json`（プロジェクトルート） | チーム共有用（Git管理） | VCSで共有 |
| `user` | `~/.claude.json` | 個人・全プロジェクト共通 | 共有されない |

**スコープの指定**

```bash
# 全プロジェクトで使いたいサーバー（個人用）
claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp

# チームで共有したいサーバー（.mcp.jsonに保存）
claude mcp add --scope project --transport http github https://api.githubcopilot.com/mcp/
```

**優先度**: `local` > `project` > `user`（同名サーバーが複数スコープに存在する場合）

---

## 3. OAuth 2.0 認証

多くのリモートMCPサーバーはOAuth 2.0認証を使用します。設定手順は以下の通りです。

**ステップ1: サーバーを追加**

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

**ステップ2: Claude Code内で認証**

```
> /mcp
```

`/mcp` コマンドを実行すると、ブラウザが開いてOAuth認証フローが始まります。認証が完了すると、トークンがmacOS Keychainに安全に保存されます。

認証トークンは自動更新されるため、通常は再認証不要です。失効した場合は `/mcp` から「Clear authentication」で再認証できます。

---

## 4. おすすめMCPサーバー

### Slack

```bash
claude mcp add --transport http slack https://mcp.slack.com/mcp
```

OAuth認証が必要です。初回接続時にブラウザでSlackワークスペースへの認証を行います。Workspace管理者の承認が必要な場合があります。

できること: メッセージ投稿・検索、チャンネル一覧、スレッド読み取り、リアクション操作

### GitHub

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/
```

OAuth認証でGitHubアカウントと連携します。

できること: Issue/PR作成・更新、コード検索、ファイル内容取得、レビューコメント

### Sentry

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
```

Sentryのエラー監視データにアクセスできます。

できること: エラー一覧・詳細、パフォーマンスデータ、リリース情報

### Notion

```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

Notionのワークスペースにアクセスできます。

できること: ページ作成・更新、データベース操作、検索

### Playwright

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

stdioタイプのローカルサーバーです。ブラウザの自動操作ができます。

できること: Webページのスクリーンショット取得、要素操作、フォーム入力、E2Eテスト

### Context7

```bash
# リモート（推奨）
claude mcp add --scope user --transport http context7 https://mcp.context7.com/mcp

# ローカル
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp
```

ライブラリやフレームワークの最新ドキュメントをAIのトレーニングデータに依存せず参照できます。AIが古い情報に基づいてコードを生成するリスクを軽減します。

できること: ライブラリドキュメントの最新版検索、APIリファレンス参照

---

## 5. @メンションによるリソース参照

MCPサーバーが公開する「リソース」は、チャット内で `@` メンションで直接参照できます。

```
@github:issue://123
@docs:file://api/authentication
@postgres:schema://users
```

`@` を入力するとオートコンプリートが起動し、MCPリソースとローカルファイルを横断検索できます。

---

## 6. セキュリティ考慮点

### 信頼できるサーバーのみ使用する

MCPサーバーはClaude Codeに外部からデータを注入する経路でもあります。Anthropicが検証していないサードパーティサーバーは、プロンプトインジェクション（AIの指示を改ざんする攻撃）のリスクがあるため、提供元の信頼性を確認してから導入してください。

### projectスコープの承認フロー

チームの`.mcp.json`に定義されたMCPサーバーは、初回接続時に承認ダイアログが表示されます。リポジトリをcloneした直後に見覚えのないサーバーが表示された場合は、安易に承認せず、チームメンバーに確認を取ってください。サプライチェーン攻撃の入り口になり得ます。

### 環境変数でのトークン管理

`.mcp.json`をGit管理する場合、APIキーやトークンは環境変数で参照します。

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

`${変数名}` でシェルの環境変数を展開します。デフォルト値も設定可能: `${API_BASE_URL:-https://api.example.com}`

### 出力トークン上限

MCPサーバーからの応答は、デフォルトで10,000トークンを超えると警告、25,000トークンを超えると切り捨てられます。大量のデータを扱うサーバーでは、環境変数 `MAX_MCP_OUTPUT_TOKENS` で上限を調整できます。

---

## 7. トラブルシューティング

### サーバーが接続できない

```bash
# サーバーの状態を確認
claude mcp get <サーバー名>

# 認証を再実行
# Claude Code内で /mcp → 対象サーバーの「Clear authentication」
```

### MCP Tool Search

MCPサーバーを多数追加すると、利用可能なツール数がコンテキストウィンドウの10%を超える場合があります。この場合、Claude CodeはMCP Tool Searchを自動的に有効化し、必要なツールだけをコンテキストに含めるように最適化します。

### Windowsでのnpx問題

Windowsネイティブ環境（WSLではない場合）でnpxベースのMCPサーバーを使う場合、`cmd /c` ラッパーが必要な場合があります。

---

## 参考リンク

- [MCP 公式ドキュメント](https://code.claude.com/docs/en/mcp)
- [MCP プロトコル仕様](https://modelcontextprotocol.io)
- [Context7 セットアップガイド](https://context7.com/docs/clients/claude-code)
- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [Playwright MCP Server](https://github.com/microsoft/playwright-mcp)
