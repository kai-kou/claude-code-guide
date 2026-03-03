# Claude Code 利用ガイド — スライドファイル

## ファイル構成

| ファイル | セクション | スライド数 | 内容 |
|---------|-----------|-----------|------|
| `01-入門とセットアップ.md` | 0-3 | 16枚 | タイトル・目次・Claude Codeとは・セットアップ・CLAUDE.md |
| `02-セキュリティと自動化.md` | 4-5 | 13枚 | 権限設定・サンドボックス・Hooks |
| `03-効率的な運用.md` | 6-7 | 10枚 | コンテキスト管理・利用制限 |
| `04-上級機能とまとめ.md` | 9-13 | 15枚 | MCP・スキル・エージェント・移行・まとめ |
| **合計** | | **54枚** | |

## 形式

- **Marp** (Markdown Presentation Ecosystem) 形式
- テーマ: default、アクセントカラー: #2D5B8A（ダークブルー）
- フォント: Noto Sans JP

## Google Slides への変換方法

### 方法1: Marp CLI → PPTX → Google Slides（推奨）

```bash
# Marp CLIのインストール
npm install -g @marp-team/marp-cli

# PPTX変換（ファイルごと）
marp slides/01-入門とセットアップ.md --pptx -o slides/output/01.pptx
marp slides/02-セキュリティと自動化.md --pptx -o slides/output/02.pptx
marp slides/03-効率的な運用.md --pptx -o slides/output/03.pptx
marp slides/04-上級機能とまとめ.md --pptx -o slides/output/04.pptx

# Google Driveにアップロード → 「Google スライドで開く」
```

### 方法2: Marp CLI → PDF

```bash
marp slides/01-入門とセットアップ.md --pdf -o slides/output/01.pdf
```

### 方法3: VSCode Marp拡張

1. VSCodeで「Marp for VS Code」拡張をインストール
2. .mdファイルを開いてプレビュー表示
3. エクスポート: Cmd+Shift+P → "Marp: Export Slide Deck"

## ファイルサイズ目安

テキストのみの場合、各ファイルは以下の範囲:
- Markdown: 3-6 KB
- PPTX変換後: 100-500 KB
- Google Slides上限: 100 MB（余裕あり）

画像を追加する場合は、assets/ に格納し各スライドから参照。
