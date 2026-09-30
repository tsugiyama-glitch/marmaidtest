# marmaidtest

Mermaid で AWS 構成図 (CloudFront → ALB → EC2) を描くお試しディレクトリ。

## ファイル

| ファイル | 内容 |
|---|---|
| `aws-architecture.md` | Mermaid ソース (詳細版 / シンプル版 / シーケンス図) |
| `rendered/*.svg`, `rendered/*.png` | mermaid-cli でレンダリングした出力 |
| `puppeteer-config.json` | devcontainer 内で Chromium を起動するための設定 (`--no-sandbox`) |

## プレビュー方法

- **GitHub**: `aws-architecture.md` を開けば ```` ```mermaid ```` ブロックがそのまま描画される
- **VS Code**: 拡張 "Markdown Preview Mermaid Support" (bierner.markdown-mermaid) を入れて Markdown プレビュー
- **CLI**: 下記コマンドで SVG / PNG に出力

```sh
cd marmaidtest
# SVG
npx -y @mermaid-js/mermaid-cli -p puppeteer-config.json -i aws-architecture.md -o rendered/aws-architecture.md
# PNG (背景白・2倍スケール)
npx -y @mermaid-js/mermaid-cli -p puppeteer-config.json -i aws-architecture.md -o rendered/aws-architecture.png -b white -s 2
```

`-p puppeteer-config.json` は devcontainer のように unprivileged user namespace が無効な環境で
Chromium が `No usable sandbox!` で落ちるのを回避するためのもの。ローカル Mac 等では不要。
