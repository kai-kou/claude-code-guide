# Cursorからの移行ガイド

**作成日**: 2026-02-27
**対象**: Cursor経験者向けClaude Code移行ガイド
**前提**: Cursorの基本操作（.cursorrules、Composerなど）を理解している

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

### 得意分野の比較

| 作業タイプ | Cursor | Claude Code |
|----------|--------|-------------|
| **リアルタイムコード補完** | ⭐⭐⭐⭐⭐ | ⭐（VSCode拡張経由） |
| **インラインコード編集** | ⭐⭐⭐⭐⭐（Composer） | ⭐⭐⭐（diff表示） |
| **複雑なリファクタリング** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐（エージェント性） |
| **複数ファイル横断操作** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **自動化タスク** | ⭐⭐ | ⭐⭐⭐⭐⭐（Hooks） |
| **外部ツール連携** | ⭐⭐ | ⭐⭐⭐⭐⭐（MCP） |
| **ターミナル操作** | ⭐⭐ | ⭐⭐⭐⭐⭐（Bash tool） |
| **画像解析** | ⭐⭐⭐（貼り付け）| ⭐⭐⭐⭐（Read tool） |
| **プロジェクト横断管理** | ⭐⭐ | ⭐⭐⭐⭐⭐（設定3層構造） |

**結論**: Cursorは「コード編集の速度」、Claude Codeは「タスク全体の自律実行」に強い。

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
# 共存スクリプト（後述のsetup-coexistence.sh）
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

### setup-coexistence.sh による共存設定

両方のツールで同じプロジェクトルールを参照する仕組み:

```bash
#!/bin/bash
# setup-coexistence.sh

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

echo "✅ Cursor & Claude Code 共存設定完了"
echo "  - CLAUDE.md: Source of Truth"
echo "  - .cursorrules → CLAUDE.md (symlink)"
echo "  - .claude/rules/cursor → .cursor/rules (symlink)"
```

**使い方:**
```bash
chmod +x setup-coexistence.sh
./setup-coexistence.sh
```

### タイムリープ戦略の実践例

**シナリオ**: 新機能実装

1. **Cursor Composer**: 新しいReactコンポーネントの基本形を生成
2. **Cursor Tab補完**: propsの型定義を高速補完
3. **Claude Code**: 関連する10ファイルのimport文を一括更新
4. **Claude Code**: テストケースを自動生成・実行
5. **Cursor**: 失敗したテストを見ながら手動で微調整
6. **Claude Code**: `/commit` でコミットメッセージ生成・Git操作

**ポイント**: 各ツールの強みを活かして、**手戻りを最小化**する。

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
- [ ] `setup-coexistence.sh` 実行（Cursor共存）
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

### Q6. Cursorで使っていた拡張機能がClaude Codeで使えない

**A**: Claude Codeは**MCPサーバー**で外部ツール連携します。以下が可能です:
- GitHub（Issue・PR管理）
- Slack（通知・コマンド実行）
- Notion（ドキュメント参照）
- Sentry（エラー解析）

詳しくは「Section 9: MCP サーバー連携」を参照してください。

### Q7. Cursorのキーボードショートカットに慣れている

**A**: VSCode拡張を使えば、以下のショートカットが使えます:
- `Ctrl+Esc`: Claude Code チャット起動
- `Ctrl+G`: Plan編集（Plan Mode時）
- `/コマンド`: スキル呼び出し

### Q8. コンテキストウィンドウがすぐ満杯になる

**A**: 以下を実践してください:
1. **タスク完了ごとに `/clear`** — 最も重要
2. **大きなファイルは部分読み** — `Read /path/to/file.ts --lines 1-50`
3. **CLAUDE.mdを簡潔に** — 300行以内
4. **`/compact`で選択的圧縮** — `/compact Keep API changes only`

### Q9. Cursorの「Apply All」のような一括適用はある？

**A**: Claude Codeは**Edit tool**で自動的にファイルを編集します。確認せずに適用されるため、**settings.jsonでdefaultMode: "ask"**にすることで、変更前に確認可能です。

### Q10. Cursorのプロジェクト間移動が便利だったが、Claude Codeは？

**A**: **tmux + ctコマンド**の活用を推奨します:
```bash
# プロジェクトAでClaude Code起動
ct project-a

# 別ペインでプロジェクトBを起動（Agent Teams対応）
ct project-b
```

詳しくは「tmux活用ガイド」を参照してください。

---

## まとめ

### Cursorからの移行の心構え

1. **完全移行ではなく、使い分け** — 両方の強みを活かす
2. **CLAUDE.mdは段階的に改善** — 最初から完璧を目指さない
3. **コンテキスト管理が最重要** — `/clear`を習慣化
4. **Plan Modeで大きなタスク** — 事前計画で手戻り防止
5. **MCPで外部ツール連携** — Cursorでできなかったことが可能に

### 次のステップ

- [ ] `setup-coexistence.sh` を実行してCursorとの共存環境を構築
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

**移行はゆっくりでOK。焦らず、自分のペースで進めてください。** 🚀
