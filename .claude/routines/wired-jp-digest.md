---
schedule: "0 9 * * *"
description: "WIRED Japan の新着記事をチェックして Slack に要約を投稿する"
---

# WIRED Japan 新着記事ダイジェスト

## 目的
毎日 https://wired.jp のトップページと記事一覧をチェックし、前回以降の新しい記事を見つけてSlackに投稿する。

## 手順

### 1. 既読記事リストの読み込み
`data/seen_articles.json` を読み込む。ファイルが存在しない場合は空リスト `{"articles": []}` として扱う。

### 2. WIRED Japan のトップページから記事一覧を取得
`https://wired.jp` をフェッチして、記事のタイトル・URL・公開日時を抽出する。
記事URLは `https://wired.jp/article/` または `https://wired.jp/series/` で始まるものを対象とする。
最大20件の最新記事を取得する。

### 3. 新しい記事の特定
取得した記事URLと `data/seen_articles.json` 内の既読URLを比較し、まだ投稿していない記事を特定する。
新しい記事がない場合は「本日の新着記事はありませんでした」とログ出力して終了する。

### 4. 各新記事の内容を要約
新しい記事それぞれについて:
1. 記事ページをフェッチして本文を取得する
2. 記事の内容を **日本語で3〜5文** に要約する
3. タイトル・URL・要約をリストに追加する

### 5. Slack に投稿
環境変数 `SLACK_WEBHOOK_URL` に設定された Webhook URL に対して、以下のフォーマットで投稿する:

```
📰 *WIRED Japan 新着記事 - {today}*

[記事数]件の新着記事があります。

*1. {タイトル}*
{URL}
{要約}

*2. {タイトル}*
{URL}
{要約}

...
```

Slackへの投稿は `curl` コマンドを使い、`SLACK_WEBHOOK_URL` 環境変数が未設定の場合はSlack投稿をスキップし、代わりにコンソールに出力する。

### 6. 既読リストの更新
新しく処理した記事のURLと処理日時を `data/seen_articles.json` に追記する。
`seen_articles.json` は最新200件のみ保持し、古いものは削除する。

### 7. 変更をコミット・プッシュ
```bash
git add data/seen_articles.json
git commit -m "chore: update seen articles - {today}"
git push -u origin HEAD
```

## 注意事項
- `SLACK_WEBHOOK_URL` 環境変数が未設定でも処理は続行する（コンソール出力にフォールバック）
- ネットワークエラーが発生した場合はリトライせず、エラー内容をログ出力して終了する
- 記事の要約は正確さを重視し、センセーショナルな表現を避ける
