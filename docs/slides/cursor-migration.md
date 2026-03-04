# Cursorからの移行ガイド

**作成日**: 2026-02-27
**対象**: Cursor経験者向けClaude Code移行ガイド
**前提**: Cursorの基本操作（.cursorrules、Composerなど）を理解している

### こんな困りごとありませんか？

- CursorでTab補完は使いこなしているが、大規模なリファクタリングが苦手
- AIにもっと自律的にタスクを任せたいが、Cursorだと細かく指示が必要
- Cursorの設定資産（.cursorrules等）があるので、移行コストが心配

### このガイドで解決できます

CursorとClaude Codeは競合ではなく補完関係です。Cursorの素早いコード補完と、Claude Codeの自律的なタスク実行を併用する「タイムリープ戦略」で、両方の長所を活かせます。既存の.cursorrulesはsymlinkで共有できるため、移行コストもほぼゼロです。

| Before | After |
|--------|-------|
| Cursorだけで大規模変更 → 手動ステップが多い | Claude Codeに委譲 → 複数ファイル一括変更を自律実行 |
| .cursorrules → CLAUDE.md の二重管理 | symlink共有で単一ファイル管理 |
| AIツールを切り替えるたびに文脈を伝え直す | 各ツールの得意分野で使い分け、文脈はCLAUDE.mdで共有 |

### 前提条件

| 項目 | 要件 |
|------|------|
| Cursor | インストール済み（.cursorrules運用経験があるとスムーズ） |
| Claude Code | インストール済み・認証済み |
| OS | macOS 13.0+ / Windows 10 1809+ / Ubuntu 20.04+ |

---

## 1. はじめに — Cursor vs Claude Code: そもそも何が違う？

### 根本的な違い

| 項目 | Cursor | Claude Code |
|------|--------|-------------|
| **アーキテクチャ** | GUI型エディタ拡張 | CLI型自律エージェント |
| **操作方法** | エディタ内チャット・Composer | ターミナル・VSCode拡張 |
| **エージェント性** | 対話的補助 | 自律的タスク実行 |
| **設定ファイル** | `.cursorrules` | `CLAUDE.md` |
| **コンテキスト** | エディタ内コードベース | ファイル・ツール・外部API（MCP） |
| **ツール連携** | 限定的 | MCP（Model Context Protocol）で拡張可能 |

### 得意分野の違い

Cursorが圧倒的に強いのはリアルタイムのコード補完とインライン編集です。Tab補完の速度とComposerの直感的な編集体験は、日常的なコーディングの相棒として最適です。

一方、Claude Codeが本領を発揮するのは「タスクを丸ごと任せる」場面です。複数ファイルにまたがるリファクタリング、Hooksによるタスク自動化、MCPを介したSlack・GitHub・Notion連携、ターミナルコマンドの実行まで、Cursorでは手動ステップが必要な作業をエージェントとして自律的に片付けてくれます。プロジェクト横断の設定管理（3層構造のCLAUDE.md）も、複数プロジェクトを抱える開発者には大きな利点です。

**一言で整理すると**: Cursorは「コードを書く速度」、Claude Codeは「タスクを完了させる速度」に強い。

---

## 2. 移行の基本ステップ

### Step 1: .cursorrules → CLAUDE.md の変換

Cursorの `.cursorrules` は Claude Code の `CLAUDE.md` に相当する。ただし、いくつかの重要な違いがある。

#### Cursor (.cursorrules)
```markdown
# .cursorrules

- Always use TypeScript for new files
- Run `npm run lint` before committing
- Follow the component structure in /components
```

#### Claude Code (CLAUDE.md)
```markdown
# CLAUDE.md

## 技術スタック

- Always use TypeScript for new files

## Git操作ルール

- Run `npm run lint` before committing

## ディレクトリ構造

- Follow the component structure in /components
```

**主な違い:**
1. **構造化**: CLAUDE.mdは章立てを推奨（セクション分け）
2. **長さ制限**: 300行以内、指示は150〜200個が限界
3. **@importで分割**: `@README.md`, `@docs/git-instructions.md`
4. **3層マージ**: グローバル → ワークスペース → プロジェクト

**変換コマンド（参考）:**
```bash
# .cursorrules を CLAUDE.md にコピーして開始
cp .cursorrules CLAUDE.md
# Claude Codeに自動整形を依頼
claude
> `/init` コマンドで初期生成してから、既存の .cursorrules の内容をマージしてください
```

### Step 2: .cursor/rules → .claude/rules の対応

Cursor の `.cursor/rules/*.mdc` ファイルは、そのまま `.claude/rules/` にsymlinkを張ることで共有可能。

```bash
# 共存スクリプト例
ln -s ../../.cursor/rules .claude/rules
```

### Step 3: 設定ファイルの移行

#### Cursor設定（Cursorアプリ内）
- GUI設定画面でモデル選択・権限設定

#### Claude Code設定（settings.json）
```json
{
  "defaultMode": "auto",
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true
  },
  "allowedTools": ["Read", "Edit", "Bash"],
  "allowList": {
    "bashCommands": ["npm", "git", "ls"]
  }
}
```

**配置先:**
- `~/.claude/settings.json` — グローバル（全プロジェクト共通）
- `~/dev/.claude/settings.local.json` — ワークスペース
- `./.claude/settings.local.json` — プロジェクト固有

---

## 3. CLAUDE.mdの書き方（Cursor経験者向け）

### .cursorrulesとの対比

| .cursorrules | CLAUDE.md | 備考 |
|-------------|-----------|------|
| 箇条書きの羅列 | セクション構造化 | 見出しで整理 |
| 無制限 | 300行以内推奨 | LLMの順守限界 |
| 1ファイルのみ | 複数ファイル`@import`可能 | `@docs/git-rules.md` |
| プロジェクトローカル | 3層自動マージ | グローバル→WS→PJ |
| 静的 | 動的注入可能 | `` !`command` `` |

### CLAUDE.mdの3層構造

Claude Codeは複数の `CLAUDE.md` を自動的にマージして読み込む:

```
~/.claude/CLAUDE.md              ← レベル1: グローバル（全PJ共通）
  ↓ マージ
~/dev/CLAUDE.md                  ← レベル2: ワークスペース（PJ横断）
  ↓ マージ
~/dev/01_active/my-app/CLAUDE.md ← レベル3: プロジェクト固有
```

**活用例:**
- **グローバル**: キャラクター設定、Git操作ルール、Bashコマンド制約
- **ワークスペース**: デグレ防止ルール、コミット規律、スプリントワークフロー
- **プロジェクト**: 技術スタック、API設計方針、テストルール

### @importによる分割

長くなりがちなルールは外部ファイルに分割できる:

```markdown
# CLAUDE.md

## プロジェクト概要
...

## Git操作ルール
@docs/git-instructions.md

## API設計ルール
@docs/api-design.md
```

**分割の目安:**
- 特定トピックが50行以上
- 複数プロジェクトで再利用したいルール
- 技術的詳細（例: API仕様、DB schema）

### ベストプラクティス

#### 1. 簡潔に保つ
```markdown
❌ NG（冗長）
TypeScriptを使う場合、必ずtsconfigでstrictモードを有効にして、
型チェックを厳格に行い、any型の使用を避けること。

✅ OK（簡潔）
- TypeScript strict mode必須
- `any`型禁止
```

#### 2. 削除テスト
> 「このルールを削除したらClaude Codeが間違いを犯すか？」

犯さないなら削除する。

#### 3. 具体的な例を含める
```markdown
❌ NG（抽象的）
コンポーネントは適切に設計すること

✅ OK（具体的）
コンポーネント構造:
- /components/Button/Button.tsx
- /components/Button/Button.test.tsx
- /components/Button/index.ts
```

#### 4. 優先度を明示
```markdown
## 最重要ルール（必須）
- セキュリティに関わる変更は必ずレビュー

## 推奨ルール
- コミット前にlint実行
```

---

## 4. 共存戦略（タイムリープ戦略）

### 完全移行ではなく使い分ける

CursorとClaude Codeは**共存させて使い分ける**のが最も効率的。

| タスク | 推奨ツール | 理由 |
|-------|----------|------|
| **リアルタイム補完** | Cursor | Tab補完が高速 |
| **単一ファイル編集** | Cursor Composer | インライン編集が直感的 |
| **複数ファイルリファクタ** | Claude Code | エージェント性による自律実行 |
| **テスト自動生成** | Claude Code | ファイル横断、実行確認まで一貫 |
| **ドキュメント生成** | Claude Code | Readツールで画像も扱える |
| **外部API連携** | Claude Code | MCP（Slack, GitHub, Notion等） |
| **定型タスク自動化** | Claude Code | Hooksで完全自動化 |

### 共存スクリプトによる一括設定

両方のツールで同じプロジェクトルールを参照するスクリプトを用意すると便利です:

```bash
#!/bin/bash
# 共存環境を構築するスクリプト例

# 1. CLAUDE.md を Source of Truth として作成
if [ ! -f CLAUDE.md ]; then
  touch CLAUDE.md
  echo "# Project Rules" >> CLAUDE.md
fi

# 2. .cursorrules を CLAUDE.md へのsymlinkに変換
if [ -f .cursorrules ] && [ ! -L .cursorrules ]; then
  mv .cursorrules .cursorrules.backup
fi
ln -sf CLAUDE.md .cursorrules

# 3. .cursor/rules を .claude/rules から参照
mkdir -p .claude/rules
if [ -d .cursor/rules ]; then
  ln -sf ../../.cursor/rules .claude/rules/cursor
fi

echo "共存設定完了"
```

**ポイント**: 自分のプロジェクト構成に合わせてカスタマイズしてください。

### AGENTS.md — 複数AIツール共存の新標準

AGENTS.mdはLinux Foundation管理のオープン標準で、複数のAIコーディングツール間でルールを共通管理できます。

- **対応ツール**: Cursor / GitHub Copilot / OpenAI Codex 等
- **Claude Codeでの利用**: 未ネイティブ対応だが `@import` で利用可能
- **設定方法**: CLAUDE.mdに `@AGENTS.md` と1行追加するだけ

```markdown
# CLAUDE.md
@AGENTS.md

（以下、Claude Code固有のルール）
```

**メリット**: AGENTS.mdに共通ルールを書き、CLAUDE.mdにはClaude Code固有の設定のみ記述することで、複数ツールの設定を効率的に管理できます。

### タイムリープ戦略 — /resume & /rewindで巻き戻し

Claude Codeにはセッションを巻き戻す強力な機能があります:

| コマンド | 動作 | 用途 |
|---------|------|------|
| `/resume` | 過去セッションの会話を復元して再開 | 中断した作業の継続 |
| `/rewind` (Esc+Esc) | コードと会話を特定時点に巻き戻し | 失敗した変更の取り消し |
| `--fork-session` | 元セッションを保護しつつ別アプローチを試す | 安全な実験 |

**重要な注意点**:
- **Write/Edit経由の変更のみ**がチェックポイント対象
- **Bash操作（git push, rm等）は巻き戻せない** — 外部への副作用は元に戻らない
- `/rewind`はコードの変更とClaude Codeの会話履歴の両方を巻き戻す

**実践例**: リファクタリングの試行錯誤
1. Claude Codeにリファクタを指示
2. 結果が期待と違う → `/rewind` で巻き戻し
3. 別のアプローチで再指示
4. 成功 → そのまま作業継続

---

## 5. 画像の扱い

### Cursorでの画像参照

```
チャット欄に画像を貼り付け
→ Cursorがアップロード
→ 画像の内容を質問
```

**制約:**
- 一度に1枚ずつ
- セッション終了で消失
- 参照先がURLにならない

### Claude Codeでの画像参照

```bash
claude
> この画像を分析してUIコンポーネント化して
> /path/to/screenshot.png
```

**Read toolで直接参照:**
- ローカルファイルパスを直接指定
- セッション再開後も再利用可能
- 複数画像を一度に参照可能

**ペーストキャッシュの仕組み:**
1. 画像をクリップボードにコピー
2. Claude Codeのチャットで `Cmd+V` 貼り付け
3. 自動的に `/tmp/claude/` に保存され、Read toolで読み込み

**画像の永続化:**
```bash
# 画像を明示的にプロジェクトに保存
cp /tmp/claude/paste-123.png ./docs/assets/wireframe.png

# Claude Codeに参照を指示
claude
> ./docs/assets/wireframe.png を元にコンポーネントを実装して
```

### 画像活用の実践例

**UIモックアップからコンポーネント生成:**
```bash
claude
> この3枚のワイヤーフレームを見て、Reactコンポーネント構成を提案してください
> ./design/home.png
> ./design/detail.png
> ./design/settings.png
```

**エラー画面のデバッグ:**
```bash
# スクリーンショットを撮影
screenshot error-screen.png

claude
> ./error-screen.png のエラーを解析して、原因と修正方法を提案して
```

**ドキュメント生成:**
```bash
claude
> ./architecture-diagram.png を元に、システム設計書を作成してください
```

---

## 6. Plan Mode（Cursorのcomposerに相当する機能）

### Cursorのcomposerとの違い

| 項目 | Cursor Composer | Claude Code Plan Mode |
|------|-----------------|----------------------|
| **起動** | エディタ内パネル | `/plan` コマンド |
| **編集方法** | インライン編集 | `Ctrl+G` でプラン編集 |
| **実行** | 自動適用 | Plan確認後、Normal Modeで実行 |
| **終了** | パネルを閉じる | `/normal` コマンド |
| **中断** | ❌ 難しい | ✅ Plan確認後に中断可能 |

### Plan Modeの操作まとめ

| 操作 | コマンド/ショートカット |
|------|---------------------|
| Plan Mode開始 | `/plan` |
| プラン編集 | `Ctrl+G` |
| Normal Modeに戻る | `/normal` |

> Plan Modeの詳細な使い方・4フェーズワークフロー・活用例については、[VSCode利用ガイド Section 7.5](./vscode-guide.md#75-plan-modeの活用) を参照してください。

### Cursor経験者向けのポイント

**Plan Modeの利点（Composer比較）:**
- ✅ 実装前に全体像を確認できる
- ✅ プランを編集してから実行可能
- ✅ 途中で中断・修正しやすい
- ✅ コンテキストウィンドウを節約（探索フェーズで最小限のRead）

**ComposerからPlan Modeへの移行TIPS:**
- Composerで「全自動適用」に慣れている場合でも、Plan Modeで一度計画を確認する習慣をつけると手戻りが減る
- 小さなタスクはNormal Modeで直接実行してOK。Plan Modeは複雑なタスク向け

---

## 7. 移行チェックリスト

### セットアップ（初回のみ）

- [ ] Claude Code インストール（`npm install -g @anthropic-ai/claude-code`）
- [ ] 認証設定（APIキー or OAuth）
- [ ] VSCode拡張インストール（オプション）
- [ ] `~/.claude/CLAUDE.md` 作成（グローバルルール）
- [ ] `~/.claude/settings.json` 設定（権限・Hooks）

### プロジェクト移行（プロジェクトごと）

- [ ] `.cursorrules` → `CLAUDE.md` に変換
- [ ] `CLAUDE.md` の長さを確認（300行以内に調整）
- [ ] 共存スクリプトを実行（Cursor共存環境構築）
- [ ] `.claude/settings.local.json` 作成（プロジェクト固有権限）
- [ ] プロジェクト固有ルールを `CLAUDE.md` に記述

### 日常ワークフロー

- [ ] Cursorで高速コード補完（Tab補完・Composer）
- [ ] Claude Codeで複雑なリファクタリング・自動化
- [ ] 画像は `./docs/assets/` に保存してClaude Codeで参照
- [ ] Plan Modeで大きなタスクの計画・実行
- [ ] `/clear` でコンテキストをこまめにクリア

### 定期メンテナンス

- [ ] `CLAUDE.md` を定期的にレビュー（削除テスト）
- [ ] `settings.json` の権限設定を見直し
- [ ] 不要なHooksを削除
- [ ] コンテキスト使用量を確認（`/context`）

---

## 8. FAQ

### Q1. Cursorを完全に置き換えるべき？

**A**: いいえ。**使い分けが最適**です。Cursorは高速補完・インライン編集に強く、Claude Codeは複雑なタスク・自動化に強い。両方を併用することで生産性が最大化されます。

### Q2. .cursorrulesをそのままCLAUDE.mdにコピーしても大丈夫？

**A**: 動作はしますが、**構造化して300行以内に収める**ことを推奨します。長すぎるとClaude Codeが指示を順守できなくなります。

### Q3. settings.jsonの設定が多すぎて混乱する

**A**: **3層構造**を活用しましょう:
- グローバル（`~/.claude/settings.json`）: 全PJ共通の基本設定
- ワークスペース（`~/dev/.claude/settings.local.json`）: 特定ディレクトリ配下のPJ共通
- プロジェクト（`./.claude/settings.local.json`）: PJ固有の追加設定

### Q4. Cursorのcomposerに慣れているので、Plan Modeが使いづらい

**A**: **Normal Modeでも同様の操作が可能**です。`/plan` を使わず、直接「このファイルをリファクタして」と指示してもOKです。Plan Modeは「大きなタスクで事前に全体像を確認したい」場合に有効です。

### Q5. 画像をチャットに貼り付けたら「ファイルが見つかりません」と言われる

**A**: Claude Codeは**ファイルパスを直接指定**する必要があります。`Cmd+V` で貼り付けた画像は `/tmp/claude/paste-*.png` に保存されるので、そのパスを指定するか、プロジェクト内に明示的にコピーしてください。

> **応用FAQ**: 拡張機能の代替、キーボードショートカット、コンテキスト管理、一括適用、プロジェクト間移動については公式ドキュメントを参照してください。

---

## トラブルシューティング

### symlinkが機能しない（.cursorrulesが読み込まれない）

共存スクリプトでsymlinkを作ったのにCursorが`.cursorrules`を認識しない場合、Cursorがsymlinkを辿れていない可能性があります。`ls -la .cursorrules`でリンク先が正しいか確認してください。リンクが壊れている場合は一度削除してから再作成します（`rm .cursorrules`→`ln -sf CLAUDE.md .cursorrules`）。Windowsの場合、管理者権限なしではsymlinkが作れないことがあるため、WSL経由での作成を推奨します。

### CLAUDE.mdが読み込まれない

Claude Codeを起動してもCLAUDE.mdのルールが適用されていないように感じる場合、ファイルの配置場所を確認してください。CLAUDE.mdはClaude Codeを起動したディレクトリ（カレントディレクトリ）に置く必要があります。サブディレクトリから起動した場合、親ディレクトリのCLAUDE.mdは読み込まれますが、兄弟ディレクトリのものは読み込まれません。`claude`コマンドを実行するディレクトリがプロジェクトルートであることを確認してください。

### settings.jsonの設定が反映されない

`~/.claude/settings.json`を編集したのに動作が変わらない場合、JSONの構文エラーが原因であることが多いです。`jq empty ~/.claude/settings.json`でパースエラーがないか確認してください。よくあるミスはカンマの付け忘れ・余分なカンマ・クォートの閉じ忘れです。構文が正しい場合は、Claude Codeを再起動して再読み込みさせてください。

### CursorとClaude Codeでファイルの競合が起きる

CursorとClaude Codeを同時に起動して同じファイルを編集すると、片方の変更がもう片方に上書きされることがあります。対策としては、同じファイルを同時に編集しないワークフローにするか、Claude Codeにはtmuxで別タスク（テスト実行、ドキュメント生成など）を任せてファイル編集はCursorに集中させるといった役割分担が有効です。

### 画像の貼り付けがうまくいかない

`Cmd+V`で画像を貼り付けたのにClaude Codeが認識しない場合、クリップボードに画像データが入っているか確認してください。スクリーンショットツールによっては、ファイルとして保存するだけでクリップボードにコピーしないものがあります。確実なのは画像ファイルをプロジェクト内に保存してからパスを直接指定する方法です（例: `./docs/assets/screenshot.png を見て`）。

---

## まとめ

### Cursorからの移行の心構え

Cursorをやめる必要はありません。自分もCursorのTab補完は毎日使い続けています。大事なのは「どちらが得意なタスクか」で切り替えること。CLAUDE.mdは最初から完璧に書こうとせず、使いながら育てていくものだと割り切ると気が楽です。コンテキスト管理（タスク完了ごとの`/clear`）は最初に身につけたい習慣で、これだけで応答品質が安定します。

### 次のステップ

- [ ] 共存スクリプトを実行してCursorとの共存環境を構築
- [ ] 小さなタスクでClaude Codeを試す（例: 単一ファイルのリファクタ）
- [ ] Plan Modeで複雑なタスクを計画・実行
- [ ] Hooksを1つ追加して自動化を体験
- [ ] MCPサーバーを1つ追加して外部ツール連携

### 参考リンク

- [Claude Code 公式ドキュメント](https://code.claude.com/docs/ja/overview)
- [CLAUDE.md ベストプラクティス](https://zenn.dev/imohuke/articles/claude-code-best-practices-2026)
- [Hooks解説](https://zenn.dev/buddypia/articles/99abea47607225)
- [MCP接続ガイド](https://code.claude.com/docs/ja/mcp)

---

移行は一気にやる必要はありません。まずは1つの小さなタスクをClaude Codeに任せてみて、手応えを感じたら少しずつ使う場面を増やしていってください。
