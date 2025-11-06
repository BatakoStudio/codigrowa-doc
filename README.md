# Codigrowa Docs

このリポジトリ（`codigrowa-doc`）は Codigrowa のゲームエンジン／教材制作フローを公開用にまとめる Hugo 製ドキュメントサイトです。アプリ本体リポジトリ（例: [BatakoStudio/codigrowa](https://github.com/BatakoStudio/codigrowa)）で策定した仕様・ガイドラインを、閲覧しやすい形に再編して配信します。

## 必要環境

- [Hugo Extended](https://gohugo.io/getting-started/installing/) v0.121 以上（Book テーマが SCSS を利用するため Extended 版が必須）
- Node.js / npm（`themes/book` の依存取得やルートリポジトリとの一貫性のため推奨）
- **初回セットアップ手順**: クローン直後に必ず Book テーマ（`themes/book`）を取得してください。環境構築時に一度だけ次のコマンドを実行します。

  ```bash
  git submodule update --init --recursive
  ```

> macOS なら `brew install hugo` で Extended 版が入ります。既に Hugo をインストール済みの場合は `hugo version` で Extended かどうかを確認してください。

## ローカルプレビュー

```bash
hugo server -D
```

- `http://localhost:1313/` にプレビューが立ち上がります。
- `-D` を付けているため、`draft: true` の記事も含めて確認できます（公開前に `draft` を外すのを忘れずに）。
- ポートやバインドアドレスを変えたい場合は追加フラグを渡します。

  ```bash
  hugo server -D --bind 0.0.0.0 --port 1314
  ```

## 本番ビルド

```bash
hugo --minify
```

- 出力は `public/` 以下に生成されます（コミット対象は Markdown やテンプレートのみで、ビルド済みファイルは含めません）。
- CI / ホスティングへ渡す場合もこのコマンドで作った成果物をアップロードします。

## ディレクトリ構成

| パス | 役割 |
| --- | --- |
| `content/_index.md` | トップページ。サイト全体のリード文。 |
| `content/docs/runtime/` | Game / Scheduler / World / Entity などランタイム層の仕様解説。 |
| `content/docs/rendering/` | `components/game-canvas.tsx` などレンダリング層の実装ガイド。 |
| `content/docs/guides/` | Entity の追加手順やデバッグ方法など How-To 集。 |
| `static/` | 画像や添付ファイル。ビルド時にルートへコピーされます。 |
| `assets/` | Book テーマの上書き用 SCSS/JS。テーマの見た目を調整する場合に使用。 |
| `layouts/` | テンプレートのカスタマイズ。必要になったら `themes/book/layouts` からコピーして修正します。 |

## 執筆フロー

1. **Markdown を作成**
   `hugo new` を使うと Front Matter を含んだ雛形ができます。

   ```bash
   hugo new docs/runtime/world-entity.md
   ```

2. **Front Matter を設定**
   `title`（表示名）と `weight`（章内の並び順）は必須です。必要に応じて `description` や `tags` を付けます。

   ```yaml
   ---
   title: World / Entity
   weight: 30
   description: World と Entity 連携の責務分離
   tags: ["engine", "runtime"]
   draft: true
   ---
   ```

3. **章（セクション）を追加したい場合**
   ディレクトリ直下に `_index.md` を置き、`weight` を設定するとサイドバーに表示されます。例: `content/docs/runtime/_index.md`。

4. **プレビューで確認**
   `hugo server` を起動したまま保存すると自動リロードされます。図版が必要な場合は `static/img/<topic>/` に配置し、Markdown から `/img/<topic>/foo.png` で参照してください。

5. **PR / レビュー**
   Codigrowa アプリ本体の `docs/` （仕様の一次情報源）に差分がある場合はそちらも更新し、Hugo 版との整合性を維持してください。
