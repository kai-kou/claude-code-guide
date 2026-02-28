---
title: Claude Code とは — はじめてのAIコーディングアシスタント
section: 0, 1
created: 2026-02-27
updated: 2026-02-27
target: Claude Code初心者
topics:
  - Claude Codeの概要
  - Cursorとの比較
  - 対応モデル
  - インストール・初回起動
---

# Claude Code とは — はじめてのAIコーディングアシスタント

Claude Codeは、Anthropicが開発したエージェント型AIコーディングアシスタントです。ターミナルで動作し、自然言語の指示からコードベース全体を理解した上で、ファイルの編集・コマンド実行・外部ツール連携まで自律的にこなします。このガイドでは、Claude Codeの基本的な特徴と始め方を解説します。

---

## 1. Claude Code の特徴

### エージェント型アシスタント

Claude Codeは「コード補完ツール」ではなく「エージェント」として設計されています。ユーザーが自然言語でタスクを伝えると、Claude Codeが以下を自律的に実行します。

- コードベースの探索・理解（複数ファイル横断）
- ファイルの作成・編集・削除
- ターミナルコマンドの実行（テスト、ビルド、Git操作）
- MCP経由での外部ツール連携（Slack、GitHub、Notion等）

つまり「ここを直して」ではなく「この機能を実装して」「このバグを修正して」のように、タスクレベルで指示できます。

### 動作環境

Claude Codeは以下の環境で利用できます。

| 環境 | 説明 |
|-----|------|
| **ターミナルCLI** | メインの利用方法。macOS / Linux / WSL / Windowsで動作 |
| **VSCode拡張** | エディタ内のサイドパネルでチャット。インラインdiff対応 |
| **JetBrains拡張** | IntelliJ系IDEとの統合 |
| **デスクトップアプリ** | macOS / Windows向けGUIアプリ |

### 主な用途

- 複数ファイルにまたがるリファクタリング
- テストの自動生成・実行・修正
- Gitワークフロー（コミット、ブランチ作成、PR作成）
- バグ調査・修正
- ドキュメント生成
- 外部ツール連携（MCP: Slack / GitHub / Notion等）

---

## 2. Cursor との違い

Claude Codeを検討している方の多くはCursorも選択肢に入れているかと思います。両者は根本的な設計思想が異なります。

### 設計思想の違い

| 観点 | Cursor | Claude Code |
|------|--------|-------------|
| **アーキテクチャ** | IDE-first（VS Code fork） | Agent-first（CLIが中心） |
| **操作モデル** | ユーザーが操縦し、AIがアシスト | タスクを委譲し、AIが自律実行 |
| **インターフェース** | GUIエディタ内のチャット・Composer | ターミナル / VSCode拡張 |

### 機能比較

| 機能 | Cursor | Claude Code |
|------|--------|-------------|
| **リアルタイム補完（Tab）** | 非常に強い | VSCode拡張経由で対応 |
| **インライン編集** | Composerが強力 | diff表示で対応 |
| **大規模リファクタリング** | 中程度 | 非常に強い |
| **コマンド実行** | 限定的 | Bash toolでフル対応 |
| **Plan Mode** | 相当機能なし | 実装前に計画を確認・編集可能 |
| **MCP連携** | 40ツール上限あり | 上限なし |
| **Hooks（自動化）** | なし | 全14イベントでカスタマイズ可能 |
| **Agent Teams** | 単一レベルのサブエージェント | 複数エージェントの双方向協調（実験的） |

### 使い分けのポイント

両方を併用するのが最も効率的です。

- **Cursor向き**: 素早いコード補完、インライン編集、GUIでの細かい修正
- **Claude Code向き**: 複雑なタスクの委譲、マルチファイル変更、自動化ワークフロー、外部ツール連携

---

## 3. 対応モデル

Claude Codeでは用途に応じてモデルを切り替えられます。セッション中に `/model` コマンドで変更可能です。

| モデル | ID | 特徴 | 最適な用途 |
|--------|-----|------|-----------|
| **Claude Opus 4.6** | `claude-opus-4-6` | 最高性能。複雑な推論・コーディングに強い | アーキテクチャ判断、難易度の高いデバッグ、エージェントチームのリード |
| **Claude Sonnet 4.6** | `claude-sonnet-4-6` | 速度と知性のバランス | 日常的なコーディング・リファクタリング（コスパ良好） |
| **Claude Haiku 4.5** | `claude-haiku-4-5-20251001` | 最速・軽量 | 大量のファイル処理、単純な変換タスク、コスト節約 |

### モデル選択の目安

- **迷ったら Sonnet 4.6**: 速度と品質のバランスが良く、ほとんどのタスクに対応
- **難しいタスクは Opus 4.6**: 複雑なアーキテクチャ判断や難解なバグ修正
- **コスト重視なら Haiku 4.5**: 定型作業やサブエージェントの実行に最適

---

## 4. インストールと初回起動

### 前提条件

| 項目 | 要件 |
|------|------|
| **OS** | macOS 13.0+ / Windows 10 1809+ / Ubuntu 20.04+ / Debian 10+ |
| **RAM** | 4GB以上 |
| **シェル** | Bash / Zsh / PowerShell / CMD |
| **ネットワーク** | インターネット接続必須 |
| **アカウント** | Claude Pro / Max / Teams / Enterprise / Consoleのいずれか（無料プランは非対応） |

### インストール

**推奨方法（Native Installer）**

```bash
# macOS / Linux / WSL
curl -fsSL https://claude.ai/install.sh | bash
```

**その他の方法**

```bash
# Homebrew（macOS）— 自動更新なし
brew install --cask claude-code

# npm（非推奨・旧来の方法）
npm install -g @anthropic-ai/claude-code
```

Native Installerはバックグラウンドで自動更新されます。Homebrew / npmは手動更新が必要です。

### 初回起動

```bash
# プロジェクトディレクトリに移動して起動
cd your-project
claude
```

初回起動時にブラウザが開き、認証フローが始まります。Claude.aiアカウント（Pro以上）でログインしてください。

### /init でプロジェクト設定

初めてのプロジェクトでは `/init` コマンドでCLAUDE.mdを自動生成するのがおすすめです。

```
> /init
```

Claude Codeがプロジェクト構造を分析し、そのプロジェクトに最適化されたCLAUDE.md（AIへの指示書）を生成します。

---

## 5. 基本的な操作

### よく使うコマンド

| コマンド | 説明 |
|---------|------|
| `/init` | CLAUDE.mdの自動生成 |
| `/model` | モデル切り替え |
| `/plan` | Plan Mode（実装前に計画を確認） |
| `/compact` | コンテキストの圧縮 |
| `/clear` | 会話のリセット |
| `/help` | ヘルプ表示 |
| `Ctrl+C` | 現在の処理を中断 |
| `Esc` | 入力をキャンセル |

### Plan Mode

複雑なタスクでは、まずPlan Modeで計画を確認してから実装に進むワークフローが効果的です。

```
> /plan この機能を実装したい
（Claudeが実装計画を提示）
（計画を確認・修正）
> /normal
（Claudeが実装を開始）
```

### @メンション

チャット内で `@` を入力すると、ファイルやMCPリソースを直接参照できます。

```
> @src/main.ts このファイルにエラーハンドリングを追加して
> @github:issue://123 このissueを修正して
```

---

## 6. 主要機能の全体像

Claude Codeには多くの機能がありますが、以下の順序で学んでいくのがおすすめです。

| 順序 | 機能 | 難易度 | ガイド |
|------|------|--------|--------|
| 1 | CLAUDE.md（AIへの指示書） | 初級 | docs/vscode-guide.md Section 3 |
| 2 | 権限設定（settings.json） | 初級 | docs/usage-limits-sandbox.md |
| 3 | VSCode / tmux での利用 | 初級 | docs/vscode-guide.md, docs/tmux-guide.md |
| 4 | コンテキスト管理 | 中級 | docs/usage-limits-sandbox.md |
| 5 | Hooks（自動化） | 中級 | docs/slack-integration.md |
| 6 | MCP サーバー連携 | 中級 | docs/mcp-servers.md |
| 7 | カスタムスキル・エージェント | 上級 | docs/skills-agents.md |
| 8 | Agent Teams | 上級 | docs/skills-agents.md |

---

## 参考リンク

- [Claude Code 公式ドキュメント](https://code.claude.com/docs/en/overview)
- [Claude Code セットアップ](https://code.claude.com/docs/en/setup)
- [Claude Code 認証](https://code.claude.com/docs/en/authentication)
- [モデル一覧](https://platform.claude.com/docs/en/about-claude/models/overview)
