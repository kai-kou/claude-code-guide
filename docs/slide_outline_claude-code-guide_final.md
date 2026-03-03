# スライド構成（最終版）: Claude Code 利用ガイド

**対象読者**: Claude Code初心者〜中級者
**総枚数**: 48枚
**アクセントカラー**: #2D5B8A（ダークブルー）

---

## Slide 0-1: タイトルスライド
- タイトル: Claude Code 利用ガイド & ベストプラクティス
- サブタイトル: Claude Code初心者〜中級者向け | 2026年3月

## Slide 0-2: 目次
- 見出し: このガイドで学べること
- 本文:
  - Claude Code の基本と始め方
  - CLAUDE.md — AIへの指示書
  - 権限設定とセキュリティ
  - Hooks — 自動化の要
  - コンテキスト管理とステータスライン
  - 利用制限の把握と対策
  - MCP サーバー連携
  - カスタムスキル & エージェント
  - Cursorからの移行

---

## Slide 1-1: Claude Code とは
- 見出し: Claude Code とは
- 本文:
  - Anthropic開発のエージェント型AIコーディングアシスタント
  - ターミナルで動作し、コードベース全体を理解して自律実行
  - 「ここを直して」ではなく「この機能を実装して」とタスクレベルで指示
  - ファイル編集・コマンド実行・外部ツール連携まで一貫して実行

## Slide 1-2: Cursorとの違い
- 見出し: Cursor vs Claude Code — 設計思想の違い
- 本文:
  - 結論: 両方を併用するのが最も効率的
  - Cursor: IDE-first。リアルタイムTab補完、インライン編集が強力
  - Claude Code: Agent-first。大規模リファクタ、MCP連携、Hooks自動化
  - Cursor → ユーザーが操縦しAIがアシスト
  - Claude Code → タスクを委譲しAIが自律実行

## Slide 1-3: 対応モデル
- 見出し: 3つのモデルと使い分け
- 本文:
  - Opus 4.6 (claude-opus-4-6): 最高性能。アーキテクチャ判断、難解なデバッグ
  - Sonnet 4.6 (claude-sonnet-4-6): 速度と知性のバランス。日常コーディング推奨
  - Haiku 4.5 (claude-haiku-4-5): 最速・軽量。大量ファイル処理、コスト節約
  - セッション中に /model コマンドで切り替え可能

---

## Slide 2-1: インストール
- 見出し: インストールと認証
- 本文:
  - 推奨: curl -fsSL https://claude.ai/install.sh | bash（自動更新あり）
  - その他: brew install --cask claude-code（手動更新）
  - 要件: macOS 13+ / Windows 10+ / Ubuntu 20.04+、4GB RAM
  - アカウント: Claude Pro / Max / Teams / Enterprise（無料プラン非対応）
  - 初回起動: cd your-project → claude → ブラウザで認証

## Slide 2-2: VSCodeで使う
- 見出し: VSCode拡張のセットアップ
- 本文:
  - 拡張機能で「Claude Code」を検索してインストール
  - セカンダリサイドバーに配置（右クリック → Move to Secondary Side Bar）
  - Cmd+Opt+B でサイドバー表示切替、Ctrl+Esc でチャット起動
  - レイアウト: ファイルツリー(左) | エディタ(中央) | Claude Code(右)

## Slide 2-3: tmuxで使う
- 見出し: tmux + ctコマンドで強力なワークフロー
- 本文:
  - ctコマンド: tmux + Claude Code の統合ラッパー
  - セッション永続化（SSH切断しても継続）
  - --dangerously-skip-permissions を自動付与（サンドボックス併用必須）
  - Agent Teamsの各エージェントをSplit Panesで並列表示
  - ct "タスクの説明" でプロンプト付き起動

## Slide 2-4: 初回起動と /init
- 見出し: /init でプロジェクト設定
- 本文:
  - /init: プロジェクト構造を分析してCLAUDE.mdを自動生成
  - よく使うコマンド:
    - /model — モデル切替
    - /plan — Plan Mode（計画確認）
    - /compact — コンテキスト圧縮
    - /clear — 会話リセット

---

## Slide 3-1: CLAUDE.mdとは
- 見出し: CLAUDE.md — AIへの永続的な指示書
- 本文:
  - プロジェクトのルール・規約をAIに伝える設定ファイル
  - Cursorの .cursorrules に相当
  - セッション開始時に自動的に読み込まれる
  - プロジェクト固有のルールをMarkdownで記述

## Slide 3-2: 3層構造
- 見出し: CLAUDE.md の3層自動マージ
- 本文:
  - Level 1: ~/.claude/CLAUDE.md（ユーザーグローバル・全PJ共通の個人設定）
  - Level 2: 親ディレクトリのCLAUDE.md（ワークスペース共通・複数PJ横断ルール）
  - Level 3: ./CLAUDE.md（プロジェクト固有の技術スタック・規約）
  - セッション開始時に3層が自動的にマージされて読み込まれる
  - 下位の設定を上位が上書き。300行以内推奨

## Slide 3-3: 書き方のコツ
- 見出し: CLAUDE.md のベストプラクティス
- 本文:
  - 300行以内、150〜200指示が限界
  - 「削除したらClaudeが間違いを犯すか？」テストで不要ルールを削減
  - @importで分割: @docs/git-instructions.md
  - 簡潔に: 「TypeScript strict mode必須、any禁止」
  - 具体例を含める（ディレクトリ構造、コード例）
  - 優先度を明示: 「最重要ルール（必須）」「推奨ルール」

## Slide 3-4: 実例 — Bashルール
- 見出し: 実例: シェル演算子禁止ルール
- 本文:
  - Allowリストのパターンマッチが && や | を含むコマンドに効かない
  - ルール: 1コマンド = 1回のBashツール呼び出し
  - NG: mkdir -p /path && cd /path && git init
  - OK: mkdir -p /path → git init を順次実行
  - cat file | grep → Grepツールを使用
  - echo "text" > file → Writeツールを使用

## Slide 3-5: Cursor共存
- 見出し: CursorとClaude Codeの共存設定
- 本文:
  - CLAUDE.md を Source of Truth として管理
  - .cursorrules → CLAUDE.md へのsymlink
  - setup-coexistence.sh で一括設定
  - .cursor/rules → .claude/rules からの参照も可能
  - 両方のツールで同じプロジェクトルールを共有

---

## Slide 4-1: 権限モード
- 見出し: 5つの権限モード（defaultMode）
- 本文:
  - default: ツール初回使用時に確認プロンプト表示。初心者向け標準モード
  - acceptEdits: ファイル編集を自動承認、Bashは確認あり。普段使いの推奨モード
  - plan: 読み取り専用。分析・計画はできるが変更不可。大規模タスクの事前計画用
  - dontAsk: Allowリスト未登録ツールは自動拒否。リスト厳密管理の上級者向け
  - bypassPermissions: 全確認スキップ。コンテナ・VM等の隔離環境専用（--dangerously-skip-permissions）
  - 切替: Shift+Tab で default ↔ acceptEdits ↔ plan
  - 設定: settings.json の permissions.defaultMode

## Slide 4-2: Allow/Deny リスト
- 見出し: Allow/Deny リスト — パターンマッチ
- 本文:
  - Allow: Bash(git *)、Bash(npm *)、WebSearch 等
  - Deny: Bash(rm -rf /)、Bash(sudo *) — Allowより優先
  - パターン: * で任意文字列にマッチ
  - 重要: シェル演算子(&&, |, >)を含むコマンドはマッチ不可
  - MCPツール: mcp__server_name__tool_name 形式

## Slide 4-3: サンドボックス
- 見出し: サンドボックス — OSレベルの隔離
- 本文:
  - ファイル書き込み: ホワイトリスト以外をブロック
  - ネットワーク: 指定ホスト以外への通信をブロック
  - 設定: sandbox.enabled: true
  - autoAllowBashIfSandboxed: サンドボックス内なら自動許可
  - カレントディレクトリ(.)と/tmp/claudeは書き込み許可

## Slide 4-4: --dangerously-skip-permissions
- 見出し: --dangerously-skip-permissions の正しい使い方
- 本文:
  - 全ての権限チェックをバイパスするオプション
  - 危険: rm -rf, 機密ファイル読み書き, git push --forceが確認なし実行
  - 使用条件:
    1. サンドボックスが有効
    2. permissions.deny で危険コマンドをブロック済み
  - 用途: tmux環境などバックグラウンド実行が必要な場合

## Slide 4-5: 機密ファイル保護
- 見出し: 機密ファイル保護 — 二段構えの防衛
- 本文:
  - 第1防衛線: settings.json の permissions.deny
    - Edit(.env), Write(credentials.json), Read(secrets.json) 等
  - 第2防衛線: protect-sensitive-files.sh Hook
    - PreToolUse(Edit|Write)で自動チェック
    - exit 2 でブロック、exit 0 で通過
  - ブロック対象: .env*, credentials.json, *.pem, *.key, ~/.ssh/*

---

## Slide 5-1: Hooksとは
- 見出し: Hooks — イベント駆動の自動化基盤
- 本文:
  - Claude Codeのイベントに応じてスクリプトを自動実行
  - 活用例: Slack通知、自動Lint、Push前チェック、コンテキスト復元
  - settings.json の hooks セクションで定義
  - command型: bashスクリプト実行（高速・定型処理向き）
  - matcher: ツール名のフィルタ（正規表現対応）

## Slide 5-2: イベント一覧
- 見出し: Hookイベント一覧（主要5つ）
- 本文:
  - PreToolUse: ツール実行前。検証・ブロック・事前チェック
  - PostToolUse: ツール実行後。フォーマット・Lint・後処理
  - Stop: 停止時。完了通知・Slack投稿
  - Notification: 通知発生時。macOS通知・外部連携
  - SessionStart: セッション開始時。コンテキスト再注入
  - 他にも9種: UserPromptSubmit, SubagentStop, PreCompact 等

## Slide 5-3: Slack連携
- 見出し: 実例: Slack連携（slack-feedback.sh）
- 本文:
  - Stopイベント → Slack投稿 → リアクション待機 → 指示変換
  - リアクションマッピング:
    - 👍 コミット&プッシュ / 👀 レビュー / 🔁 やり直し
    - ❌ 破棄 / 🚀 テスト / 📝 ドキュメント
  - 段階的待機: 1分→3分→5分
  - トークン管理: macOS Keychain推奨（環境変数は非推奨）

## Slide 5-4: Lint自動チェック
- 見出し: 実例: Lint自動チェック（post-edit-lint.sh）
- 本文:
  - PostToolUse(Edit|Write) で発火
  - 対応フォーマット:
    - Markdown: YAMLフロントマター閉じチェック
    - Shell: bash -n 構文チェック
    - JSON: jq empty 構文チェック
  - exit 1 → Claude Codeが自動修正を試みる

## Slide 5-5: Push前チェック
- 見出し: 実例: Push前チェック（pre-push-check.sh）
- 本文:
  - PreToolUse(Bash) でgit pushコマンドを検出
  - チェック項目:
    - 未コミット変更の確認
    - プッシュ対象コミットの表示
    - 高リスクファイル(.env, .secret, .key)の変更検出
  - exit code: 0=正常、1=エラー、2=ブロック（ユーザー確認要求）

## Slide 5-6: コンテキスト再注入
- 見出し: 実例: コンテキスト再注入（compact-reinject.sh）
- 本文:
  - /compact後の情報ロスを防ぐ
  - SessionStart(matcher: "compact")で再注入:
    - プロジェクト名・現在ブランチ
    - 直近5コミット
    - スプリント情報
    - 未コミット変更
  - CLAUDE.mdに「圧縮時の保持ルール」を記述するのも有効

---

## Slide 6-1: コンテキストの仕組み
- 見出し: コンテキストウィンドウの仕組み
- 本文:
  - Claudeが「覚えている情報の量」= コンテキストウィンドウ
  - 構成: Input Tokens + Output Tokens + Cache
  - Cache Write: 新規情報の保存（初回コスト発生）
  - Cache Read: 保存済み情報の再利用（90%割引）
  - キャッシュ有効期限: 5分（5分以内の再アクセスで割引適用）

## Slide 6-2: ステータスラインで状況を把握
- 見出し: ステータスライン — リアルタイム4行ダッシュボード
- 本文:
  - settings.json の statusLine 設定で有効化（カスタムスクリプト）
  - Line 1: プロジェクト | モデル | コンテキスト使用率 | コスト | 時間
  - Line 2: コンテキスト詳細（現在/最大, W:キャッシュ書込, R:読取）
  - Line 3: 利用制限（⚡5時間使用率 | 📅7日間使用率 | 🎵Sonnet | 🎹Opus）
  - Line 4: メモリ使用量（CLI | App | VSCode）

## Slide 6-3: /compact と色の読み方
- 見出し: /compact の活用とステータスの色
- 本文:
  - /compact: 情報を圧縮して引き継ぐ。保持すべき情報を指示で明示
  - コンテキスト使用率の色:
    - 緑(〜60% OK): 安全、作業継続
    - 黄(60-80% WATCH): 注意。/compact検討
    - 赤(80%〜 SWITCH): /compact or /clear 推奨

## Slide 6-4: コンテキスト管理のベストプラクティス
- 見出し: コンテキスト管理のコツ
- 本文:
  - タスク完了ごとに /clear（最重要）
  - 大きなファイルは部分読み（offset, limitを指定）
  - 同じファイルを何度も読まない（キャッシュを活用）
  - CLAUDE.mdやよく使うファイルをセッション序盤で読み込む
  - 60%超えたら /compact を検討

---

## Slide 7-1: 制限の種類
- 見出し: 利用制限の種類
- 本文:
  - 5時間ローリングウィンドウ: 直近5時間の利用量
  - 7日間制限: 週単位の利用量
  - モデル別制限: Sonnet/Opus それぞれの7日間制限
  - プラン別: Pro（標準）、Max 5x（5倍）、Max 20x（20倍）
  - Cache Writeは制限にカウント、Cache Readは90%割引

## Slide 7-2: 確認方法
- 見出し: 制限の確認方法
- 本文:
  - ステータスライン Line 3 で常時確認
  - ⚡ 5時間制限 + リセットまでの時間
  - 📅 7日間制限 + リセットまでの時間
  - 🎵 Sonnet使用率 / 🎹 Opus使用率
  - 色: 緑(〜70% Safe) / 黄(70-85% Caution) / 赤(85%〜 Danger)

## Slide 7-3: 最適化のコツ
- 見出し: 利用制限の最適化
- 本文:
  - キャッシュヒット率向上: CLAUDE.md・よく使うファイルを先に読む
  - 5分以内の再アクセスでCache Read（90%割引）
  - Haiku活用: Lint、テスト、検索などの軽作業はHaikuで
  - 専用ツール優先: cat/grepではなくRead/Grepツールを使う
  - 不要なReadを減らす: 一度読んだファイルはキャッシュから再利用

## Slide 7-4: 制限到達時の対応
- 見出し: 制限に近づいたら
- 本文:
  - 50%超: Haikuモデルに切り替え（/model haiku）
  - 60%超: タスクを分割して複数セッションに
  - 80%超: リセット待ち、緊急性の低いタスクは後回し
  - モデル切替: /model sonnet / /model opus / /model haiku
  - セッション分割例: 実装→テスト→ドキュメントを各セッションに

---

## Slide 9-1: MCPとは
- 見出し: MCP — 外部ツール連携の標準プロトコル
- 本文:
  - Model Context Protocol: AIと外部ツールを接続するオープンスタンダード
  - Slack投稿、GitHub Issue操作、Notion更新を自然言語で実行可能
  - 3つのトランスポート: http（推奨）、stdio（ローカル）、sse（廃止予定）
  - @メンションで直接参照: @github:issue://123

## Slide 9-2: 設定方法
- 見出し: MCPサーバーの設定方法
- 本文:
  - HTTPサーバー: claude mcp add --transport http 名前 URL
  - stdioサーバー: claude mcp add 名前 -- コマンド [引数]
  - 3スコープ: local（個人）、project（チーム共有）、user（全PJ共通）
  - OAuth 2.0: /mcp コマンドでブラウザ認証、トークンはKeychain保存
  - 管理: claude mcp list / get / remove

## Slide 9-3: おすすめMCPサーバー
- 見出し: おすすめMCPサーバー6選
- 本文:
  - Slack: メッセージ投稿・検索・リアクション
  - GitHub: Issue/PR操作・コード検索・レビュー
  - Sentry: エラー監視・パフォーマンスデータ
  - Notion: ページ作成・DB操作
  - Playwright: ブラウザ自動操作・E2Eテスト
  - Context7: ライブラリ最新ドキュメント参照（AI幻覚防止）

---

## Slide 10-1: スキルとは
- 見出し: カスタムスキル — /コマンドで呼べるプロンプト
- 本文:
  - 繰り返し使うワークフローを再利用可能に
  - 例: /review-pr, /deploy-staging, /sprint-start
  - 配置: ~/.claude/skills/ (Personal) / .claude/skills/ (Project)
  - SKILL.md ファイルで定義（YAMLフロントマター + Markdown）
  - Agent Skillsオープン標準に準拠

## Slide 10-2: スキルの作り方
- 見出し: SKILL.md の書き方
- 本文:
  - フロントマター: name, description, user-invocable, model, context
  - 動的コンテキスト注入: !`gh pr diff` で実行結果を埋め込み
  - $ARGUMENTS で引数を受け取り
  - references/ フォルダで補足情報を分離
  - context: fork で独立コンテキストで実行

## Slide 10-3: エージェントとは
- 見出し: カスタムエージェント — 専門家AIサブプロセス
- 本文:
  - 特定タスクに特化した独立サブプロセス
  - 独自のコンテキスト・システムプロンプト・ツール権限
  - 配置: .claude/agents/名前.md
  - ツール制限: 最小権限の原則（調査用は読み取りのみ等）
  - 組み込み: Explore(Haiku), Plan(継承), general-purpose(全ツール)

## Slide 10-4: Agent Teams
- 見出し: Agent Teams — 複数AIの協調（実験的）
- 本文:
  - Team Lead + Teammates で並列タスク実行
  - 共有タスクリスト + メッセージングで連携
  - 有効化: CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
  - 表示: in-process(Shift+Down切替) / split panes(tmux)
  - 制限: 1チームのみ、セッション再開不可、ネスト不可

---

## Slide 11-1: 移行のポイント
- 見出し: Cursorからの移行 — 基本ステップ
- 本文:
  - .cursorrules → CLAUDE.md: 構造化して300行以内に
  - .cursor/rules → .claude/rules: symlinkで共有可能
  - settings.json: 権限・Hooks・サンドボックスを設定
  - setup-coexistence.sh: CLAUDE.mdをSource of Truthに
  - /init コマンドで初期生成→既存ルールをマージ

## Slide 11-2: タイムリープ戦略
- 見出し: タイムリープ戦略 — 完全移行ではなく使い分け
- 本文:
  - Cursor: リアルタイム補完、単一ファイル編集、インライン修正
  - Claude Code: 複数ファイルリファクタ、テスト自動生成、外部API連携
  - 実践フロー:
    1. Cursor Composer で基本コンポーネント生成
    2. Cursor Tab で型定義を高速補完
    3. Claude Code で10ファイルのimport一括更新
    4. Claude Code でテスト生成・Git操作

## Slide 11-3: 画像の扱い
- 見出し: 画像の扱い方
- 本文:
  - Cursor: チャットに貼り付け（一度に1枚、セッション終了で消失）
  - Claude Code: Read toolでファイルパスを直接指定
  - ペーストキャッシュ: Cmd+V → /tmp/claude/ に自動保存
  - 画像の永続化: プロジェクト内にコピーしてパス指定
  - 用途: UIモック→コンポーネント生成、エラー画面デバッグ

---

## Slide 12-1: macOS通知
- 見出し: macOS通知 — terminal-notifier + ccnotify.py
- 本文:
  - タスク完了時にmacOSネイティブ通知を表示
  - 通知クリックでVSCodeを開く
  - ccnotify.py: プロンプト記録（SQLite）+ 通知送信
  - Hookイベント: UserPromptSubmit（記録）、Stop（通知）

## Slide 12-2: Slack連携フロー
- 見出し: Slack連携 — リアクション駆動の遠隔操作
- 本文:
  - フロー: タスク完了 → Slack投稿 → リアクション待機 → 指示変換
  - PCから離れてもスマホでリアクション → 次の指示を自動送信
  - MCP経由: claude mcp add --transport http slack（トークン管理不要）
  - 直接API: slack-feedback.sh + Keychain管理

---

## Slide 13-1: 推奨学習順序
- 見出し: Claude Code 学習ロードマップ
- 本文:
  1. CLAUDE.md（AIへの指示書）— 初級
  2. 権限設定（settings.json）— 初級
  3. VSCode / tmux での利用 — 初級
  4. コンテキスト管理 — 中級
  5. Hooks（自動化）— 中級
  6. MCPサーバー連携 — 中級
  7. カスタムスキル・エージェント — 上級
  8. Agent Teams — 上級

## Slide 13-2: ベストプラクティス6選
- 見出し: 今日から実践できるベストプラクティス
- 本文:
  1. /clear をタスク完了ごとに実行（コンテキスト管理の基本）
  2. Plan Modeで大きなタスクは計画してから実行
  3. CLAUDE.mdに検証基準を明記（300行以内・簡潔に）
  4. サンドボックスを有効にしてHooksで自動化
  5. キャッシュヒット率を上げてコスト・制限を節約
  6. MCPで外部ツールと連携して作業範囲を拡大

## Slide 13-3: 参考リンク
- 見出し: 参考リンク
- 本文:
  - Claude Code 公式: https://code.claude.com/docs/en/overview
  - セットアップ: https://code.claude.com/docs/en/setup
  - Hooks: https://code.claude.com/docs/en/hooks
  - MCP: https://code.claude.com/docs/en/mcp
  - Skills: https://code.claude.com/docs/en/skills
  - Agent Teams: https://code.claude.com/docs/en/agent-teams
