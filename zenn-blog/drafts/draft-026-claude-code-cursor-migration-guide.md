---
title: "CursorユーザーのためのClaude Code移行ガイド：完全移行より「使い分け」が答え"
emoji: "⏱️"
type: "tech"
topics: ["claudecode", "cursor", "ai", "migration"]
published: false
---

Cursor で十分では？——正直に言うと、多くの場面でその通りです。

「Claude Code に乗り換えるべきか」という問いに対して、答えは「乗り換えない」です。Cursor の Tab 補完は手放せません。自分もいまも毎日 Cursor を開いています。ただ、「Cursor だけでは完結しにくい仕事」が確実に存在していて、そこに Claude Code を組み合わせると生産性が変わります。

この記事では、Cursor を日常的に使っている方が Claude Code を導入するときの「移行コスト」と「使い分けの判断基準」を整理します。設定の共存方法からタイムリープ戦略まで、順を追って説明します。

---

## Cursor と Claude Code は何が違うのか

まず根本的な違いを整理しましょう。一言で言えば「コードを書く速度」対「タスクを完了させる速度」です。

| 項目 | Cursor | Claude Code |
|------|--------|-------------|
| アーキテクチャ | GUI 型エディタ拡張 | CLI 型自律エージェント |
| 操作感 | エディタ内チャット・Composer | ターミナル・VSCode 拡張 |
| 強み | リアルタイム補完、インライン編集 | 複数ファイル横断、タスク自律実行 |
| 設定ファイル | `.cursorrules` | `CLAUDE.md` |
| 外部連携 | 限定的 | MCP（Slack, GitHub, Notion 等） |

Cursor が圧倒的に強いのは日常的なコーディングの補助です。Tab 補完の速度、Composer のインライン編集体験、リアルタイムのサジェストは、コードを「書く」場面では最高の相棒です。

Claude Code が本領を発揮するのは「タスクを丸ごと任せる」場面です。複数ファイルにまたがるリファクタリング、テストの自動生成、Slack・GitHub への投稿、ターミナルコマンドの実行まで、Cursor では細かく手動ステップが必要な作業を、エージェントとして自律的に片付けてくれます。

---

## どちらを使うか——タスク別の判断基準

使い分けに迷ったとき、自分が参考にしている基準を表にまとめます。

| タスク | 推奨ツール |
|-------|----------|
| リアルタイム補完、インライン編集 | Cursor |
| 単一ファイルの軽微な修正 | Cursor Composer |
| 複数ファイルのリファクタリング | Claude Code |
| テストの自動生成・実行確認 | Claude Code |
| ドキュメント生成 | Claude Code |
| Slack・GitHub 等の外部 API 連携 | Claude Code |
| 定型タスクの自動化（Hooks） | Claude Code |

判断に迷ったら「自分の手が何度動くか」で考えてみてください。Cursor Composer で 2〜3 ステップ以内に終わる作業は Cursor のほうが速い。それ以上になるなら Claude Code に任せたほうが楽です。

---

## 移行コストはほぼゼロ——設定ファイルの共存

「Cursor の設定資産があるから移行したくない」という心理的ハードルは、実際にはほぼ存在しません。

### `.cursorrules` を `CLAUDE.md` に変換する

Cursor の `.cursorrules` は Claude Code の `CLAUDE.md` に対応します。フォーマットはほぼ同じで、コピーしてそのまま動きます。ただし、Claude Code では **300 行以内・セクション構造化** が推奨です。

変換の手順は実質 2 コマンドです。

```bash
cp .cursorrules CLAUDE.md
```

その後、Claude Code を起動して `/init` を実行すると、Claude Code が既存の内容を読んで適切に整形してくれます。「今の `.cursorrules` を見て Claude Code 向けに最適化して」と言うだけでも同じことをやってくれます。自分で設定ファイルを書く必要はありません。

`.cursorrules` と `CLAUDE.md` の書き方の違いを比較すると、以下のようになります。

```markdown
# .cursorrules（変換前）
- Always use TypeScript for new files
- Run `npm run lint` before committing
- Follow the component structure in /components
```

```markdown
# CLAUDE.md（変換後）

## 技術スタック
- Always use TypeScript for new files

## Git 操作ルール
- Run `npm run lint` before committing

## ディレクトリ構造
- Follow the component structure in /components
```

変換ポイントは「箇条書きの羅列」から「セクション構造」への変換です。Claude Code は指示の数が多くなると守れなくなります。「このルールを削除しても Claude Code が間違いを犯さないか？」を自問しながら削除テストをかけると、300 行以内に収まることがほとんどです。

### symlink で両方のツールが同じファイルを参照する

Cursor と Claude Code を共存させる最もシンプルな方法は symlink です。ただし方向性に注意が必要です。

**Cursor はシンボリックリンク先を認識しない**という制約があります。そのため、Cursor 側をオリジナル、Claude Code 側を symlink にする必要があります。

```bash
# CLAUDE.md を Source of Truth にする場合
cp .cursorrules CLAUDE.md
rm .cursorrules
ln -sf CLAUDE.md .cursorrules
```

これで `.cursorrules` は `CLAUDE.md` へのリンクになり、どちらのツールも同じルールを参照します。ルールの更新は `CLAUDE.md` 一箇所だけで済みます。

---

## CLAUDE.md の 3 層構造——Cursor にない機能

Cursor の `.cursorrules` は 1 プロジェクト 1 ファイルです。Claude Code にはプロジェクト横断で設定を管理する 3 層構造があります。

```
~/.claude/CLAUDE.md              ← グローバル（全プロジェクト共通）
~/dev/CLAUDE.md                  ← ワークスペース（複数 PJ 横断）
~/dev/01_active/my-app/CLAUDE.md ← プロジェクト固有
```

複数のリポジトリを日常的に行き来している場合、グローバルに「日本語で答えること」「コミット前に lint を実行すること」を書いておけば、全プロジェクトで同じルールが適用されます。プロジェクト固有の技術スタックやテストルールはプロジェクトの `CLAUDE.md` に書く——この分離が効いてきます。

長いルールは `@import` で外部ファイルに切り出すこともできます。

```markdown
# CLAUDE.md

## プロジェクト概要
...

## Git 操作ルール
@docs/git-instructions.md
```

Cursor の `.cursorrules` が 1 枚のフラットなファイルだったのに対して、Claude Code は「継承する設定」として設計されています。

---

## タイムリープ戦略——Cursor 経験者に刺さる機能

Cursor を使っていた方が Claude Code に乗り換えて最初に戸惑うのが「会話のやり直し」です。Cursor では前の会話に戻って再編集することで、失敗した変更をなかったことにできました。

Claude Code にはこれに対応する 2 つのコマンドがあります。

| コマンド | 動作 |
|---------|------|
| `/resume` | 過去セッションの会話を復元して再開する |
| `/rewind` (Esc + Esc) | コードと会話を特定の時点に巻き戻す |

`/rewind` はリファクタリングの試行錯誤で特に役立ちます。

1. Claude Code にリファクタを指示する
2. 結果が期待と違う → `/rewind` で指示前の状態に戻る
3. 別のアプローチで再指示する
4. うまくいったらそのまま継続する

ただし **重要な制約** があります。`/rewind` が巻き戻せるのは、Claude Code の Write/Edit ツールを通じたファイル変更のみです。`git push` や `rm` といった Bash 操作の副作用は元に戻りません。リファクタが失敗した時点でコミットやプッシュをしていなければ問題ありませんが、外部に影響が出た操作は手動で対処する必要があります。

`--fork-session` オプションを使うと、元のセッションを保護したまま別アプローチを試すこともできます。「どちらのアプローチが良いか確認したい」という場面で活用できます。

---

## 画像の扱い方——Cursor との違い

画像については Cursor のほうが直感的です。チャット欄に貼り付けるだけで認識してくれます。Claude Code はファイルパスの指定が基本です。

ただし、Claude Code でも `Cmd+V` で画像を貼り付けること自体はできます。その場合、画像は `/tmp/claude/paste-*.png` に自動保存されるので、そのパスを指定すれば参照できます。

長期的に再利用する画像は、プロジェクト内に明示的に保存しておくほうが確実です。

```bash
cp /tmp/claude/paste-123.png ./docs/assets/wireframe.png
```

その後、Claude Code に「`./docs/assets/wireframe.png` を元にコンポーネントを実装して」と指示すればセッションをまたいで参照できます。

---

## Plan Mode vs Cursor Composer

Cursor Composer は変更をインラインで自動適用するスタイルです。Claude Code の Plan Mode はそれとは少し異なります。

| 項目 | Cursor Composer | Claude Code Plan Mode |
|------|-----------------|----------------------|
| 起動 | エディタ内パネル | `/plan` コマンド |
| 編集方法 | インライン自動適用 | 計画を確認してから実行 |
| 中断 | 難しい | 計画確認後に中断可能 |

Composer に慣れていると Plan Mode を「面倒」と感じるかもしれませんが、大きなタスクでは逆に便利です。実装を始める前に Claude Code の計画を確認できるので、「意図と違う方向に進んでいた」という手戻りが減ります。コンテキストウィンドウの消費も抑えられます。

小さなタスクは `/plan` を使わず Normal Mode で直接指示してかまいません。Plan Mode は「複雑なタスクで全体像を先に確認したい」場面向けです。

---

## settings.json は自分で書かなくていい

「設定ファイルの移行が大変そう」と思っている方へ——Claude Code の `settings.json` は自分で書く必要がありません。Claude Code に「この設定を追加して」と言えば書いてくれます。

実際、設定ファイルを自分で開いたことがないという方も少なくありません。Claude Code に「npm と git コマンドを許可して、sandbox を有効にして」と言えば、対応する JSON を生成してくれます。

設定が複雑になってきたら 3 層構造を活用してください。

| 配置先 | 用途 |
|--------|------|
| `~/.claude/settings.json` | 全プロジェクト共通の基本設定 |
| `~/dev/.claude/settings.local.json` | ワークスペース固有設定 |
| `./.claude/settings.local.json` | プロジェクト固有の追加設定 |

---

## AGENTS.md——複数 AI ツールの共通管理（発展）

使い分けが進んでくると「Cursor と Claude Code で別々にルールを管理するのが面倒」になってきます。そこで使えるのが AGENTS.md という標準仕様です。

AGENTS.md は Linux Foundation が管理するオープン標準で、Cursor・GitHub Copilot・OpenAI Codex など複数の AI コーディングツールが共通で参照できます。Claude Code はネイティブ対応していませんが、`@import` で取り込めます。

```markdown
# CLAUDE.md
@AGENTS.md

（以下、Claude Code 固有のルール）
```

共通ルールを AGENTS.md に書いて、Claude Code 固有の設定だけを CLAUDE.md に書く——この分離が整うと複数ツールの管理がシンプルになります。

---

## 日常のワークフローに組み込む

最終的には「どのツールを使うか」ではなく「手が止まらないこと」が目的です。自分が落ち着いたワークフローを参考に紹介します。

Cursor を開いてコーディングしながら、並行して Claude Code をターミナルで動かしています。Cursor でコードを書きつつ、Claude Code にはテスト生成・ドキュメント更新・スプリント管理を任せる形です。同じファイルを同時に編集すると競合が起きるので、役割を分けています。Cursor はファイル編集、Claude Code には Cursor が触らないファイルか、Bash 操作を任せる——そのくらいの分担が現実的です。

コンテキスト管理として、タスクが一段落したら `/clear` でコンテキストを落とす習慣をつけると Claude Code の応答品質が安定します。これは Cursor にはない概念ですが、慣れると自然になります。

---

## まず 1 つのタスクを任せてみる

移行は一気にやる必要がありません。今の `.cursorrules` をコピーして `CLAUDE.md` にするだけで Claude Code は動き始めます。

最初の一歩として、Cursor では手間がかかると感じているタスクを 1 つ選んで、Claude Code に任せてみてください。複数ファイルにまたがるリファクタリング、テストの自動生成、あるいは PR 説明の自動作成——何でもかまいません。その手応えをもとに、少しずつ使う場面を広げていくのが自然な進め方です。

Cursor をやめる必要はありません。使い分けが答えです。
