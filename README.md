# qiita-archive

Qiita記事をローカルで管理するリポジトリです。[qiita-cli](https://github.com/increments/qiita-cli) を使用しています。

## ディレクトリ構成

```
qiita-archive/
├── public/                     # 記事ファイル (.md)
│   └── <article-id>.md         # 各記事（qiita-cliが自動生成）
├── .github/
│   └── workflows/
│       └── publish.yml         # GitHub Actions（mainへのpushで自動投稿）
├── qiita.config.json           # qiita-cli設定
├── package.json
└── README.md
```

## セットアップ

```bash
npm install
export QIITA_TOKEN=<your_token>
npx qiita login
```

## 使い方

### 新規記事を作成

```bash
npx qiita new <記事のファイル名>
```

### プレビュー

```bash
npx qiita preview
```

### 手動で投稿

```bash
npx qiita publish <記事のファイル名>
```

### 既存のQiita記事をすべて取得

```bash
npx qiita pull
```

## GitHub連携による自動投稿

`main` ブランチにpushすると、GitHub Actionsが自動でQiitaに記事を投稿します。

### 初回設定

GitHubリポジトリの **Settings > Secrets and variables > Actions** に以下を追加：

| シークレット名 | 値 |
|---|---|
| `QIITA_TOKEN` | QiitaのAPIトークン |
