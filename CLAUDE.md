# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

[qiita-cli](https://github.com/increments/qiita-cli) を使ってQiita記事をローカルで管理するリポジトリ。`main` ブランチへのpushでGitHub Actionsが自動的にQiitaへ記事を投稿・更新する。

## よく使うコマンド

```bash
# 新規記事を作成（public/<name>.md が生成される）
npx qiita new <ファイル名>

# ブラウザでプレビュー（localhost:8888）
npx qiita preview

# 記事を手動で投稿・更新
npx qiita publish <ファイル名>

# Qiitaの既存記事をすべてローカルに取得
npx qiita pull
```

## 記事ファイルの構造

記事は `public/` ディレクトリに `.md` ファイルとして保存される。各ファイルの先頭にはqiita-cliが管理するフロントマターが必要：

```yaml
---
title: 記事タイトル
tags:
  - tag1
  - tag2
private: false        # trueにすると限定共有記事
updated_at: ''
id: null              # 投稿後にQiitaのIDが自動で入る
organization_url_name: null
slide: false
ignore_publish: false # trueにするとpushしても投稿されない
---
```

## 投稿フロー

1. `npx qiita new <name>` で記事作成
2. `npx qiita preview` でプレビュー確認
3. `git add / commit / push` → GitHub Actionsが自動投稿

## 設定

- `qiita.config.json`: プレビューサーバーの設定（port: 8888）、`includePrivate: false`（限定共有記事はpull対象外）
- GitHub Actions: `QIITA_TOKEN` シークレットが必要（設定済み）

## ドキュメント（docs/）

記事作成・レビューに関するガイドラインは `docs/` に格納：

| ファイル | 内容 |
|---|---|
| `general.md` | 全体的なガイドライン |
| `writing-workflow.md` | 記事執筆フロー |
| `ai-collaboration-guide.md` | AIツールとの協働指針 |
| `naming-conventions.md` | 命名規則 |
| `publishing-checklist.md` | 公開前チェックリスト |
| `review-process.md` | レビュープロセス |
| `template-usage.md` | テンプレート使用方法 |
