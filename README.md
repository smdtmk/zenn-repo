# Zenn Contents

このリポジトリは[Zenn](https://zenn.dev/)の記事管理用リポジトリです。

## セットアップ

```bash
# 依存パッケージのインストール
npm install
```

## 使用方法

### プレビューの起動

```bash
npm run preview
```

### 新規記事の作成

```bash
npm run new:article
```

このコマンドを実行すると：
- 現在のブランチ名をベースに記事のslugが生成されます
- 記事用の画像ディレクトリが自動的に作成されます

### 未使用の画像ディレクトリの削除

```bash
npm run clean:images
```

## ディレクトリ構造 
.
├── articles/ # 記事ファイル
├── images/ # 画像ファイル
└── package.json # 依存関係の設定

## ライセンス

ISC