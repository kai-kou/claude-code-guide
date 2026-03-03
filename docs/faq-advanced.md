# Claude Code 応用FAQ

基本的なFAQ（Q1〜Q5）は [Cursorからの移行ガイド](./cursor-migration.md#8-faq) を参照してください。ここでは、ある程度Claude Codeを使い始めた後に出てくる疑問をまとめています。

---

## Q6. Cursorで使っていた拡張機能がClaude Codeで使えない

**A**: Claude Codeは**MCPサーバー**で外部ツール連携します。以下が可能です:
- GitHub（Issue・PR管理）
- Slack（通知・コマンド実行）
- Notion（ドキュメント参照）
- Sentry（エラー解析）

詳しくは「MCP サーバー連携ガイド」（docs/mcp-servers.md）を参照してください。

---

## Q7. Cursorのキーボードショートカットに慣れている

**A**: VSCode拡張を使えば、以下のショートカットが使えます:
- `Ctrl+Esc`: Claude Code チャット起動
- `Ctrl+G`: Plan編集（Plan Mode時）
- `/コマンド`: スキル呼び出し

Cursorのショートカットとは体系が異なりますが、主要な操作は2〜3個覚えれば十分です。ターミナルベースのClaude Codeでは、キーボードショートカットよりも「スラッシュコマンド」（`/clear`, `/compact`, `/model`など）の方が重要になります。

---

## Q8. コンテキストウィンドウがすぐ満杯になる

**A**: 以下を実践してください:
1. **タスク完了ごとに `/clear`** — 最も重要
2. **大きなファイルは部分読み** — `Read /path/to/file.ts --lines 1-50`
3. **CLAUDE.mdを簡潔に** — 300行以内
4. **`/compact`で選択的圧縮** — `/compact Keep API changes only`

詳しくは「コンテキスト管理ガイド」（docs/context-management.md）を参照してください。

---

## Q9. Cursorの「Apply All」のような一括適用はある？

**A**: Claude Codeは**Edit tool**で自動的にファイルを編集します。確認せずに適用されるため、**settings.jsonでdefaultMode: "ask"**にすることで、変更前に確認可能です。

Cursorの「Apply All」は提案された複数の変更を一括で受け入れる機能ですが、Claude Codeでは最初からエージェントが判断して変更を適用する設計です。心配な場合はPlan Modeで事前に変更計画を確認してから実行に移すワークフローがおすすめです。

---

## Q10. Cursorのプロジェクト間移動が便利だったが、Claude Codeは？

**A**: **tmux + ctコマンド**の活用を推奨します:
```bash
# プロジェクトAでClaude Code起動
ct project-a

# 別ペインでプロジェクトBを起動（Agent Teams対応）
ct project-b
```

tmuxのウィンドウ切り替え（`Ctrl+B` → 数字キー）で、Cursorのプロジェクト間移動と同様の体験が得られます。詳しくは「tmux活用ガイド」（docs/tmux-guide.md）を参照してください。
