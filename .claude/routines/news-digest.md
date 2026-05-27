---
schedule: "0 9 * * *"
description: "エンジニアブログの新着記事をチェックして Slack に日本語要約を投稿する"
---

# エンジニアブログ 新着記事ダイジェスト

## 目的
毎日複数のエンジニアブログをチェックし、前回以降の新着記事をサイトごとに最大3件ピックアップして日本語で要約し Slack に投稿する。

## 対象ブログ

| サイト名 | URL |
|---|---|
| Salesforce Engineering | https://engineering.salesforce.com |
| Snowflake AI Research | https://www.snowflake.com/en/product/ai/ai-research |
| Google Developers Blog (JA) | https://developers.googleblog.com/ja |
| AWS Blog (JP) | https://aws.amazon.com/jp/blogs/news |
| Meta Engineering | https://engineering.fb.com |
| GitHub Blog - Engineering | https://github.blog/engineering |
| OpenAI Developer Blog | https://developers.openai.com/blog |
| OpenAI News (JA) | https://openai.com/ja-JP/news |
| Anthropic Engineering | https://www.anthropic.com/engineering |
| Slack Engineering Blog | https://slack.com/blog/collections/digital-hq-for-engineering |
| Atlassian Blog | https://atlassianblog.wpengine.com |
| Docker Blog | https://www.docker.com/blog |

## 手順

### 1. 既読記事リストの読み込み
`data/seen_articles.json` を読み込む。ファイルが存在しない場合は `{"articles": []}` として扱う。

### 2. 各ブログの記事一覧を取得
対象ブログを上から順に1サイトずつ処理する。各サイトについて:

1. トップページをフェッチして記事の **タイトル・URL・公開日時** を抽出する
2. 公開日時が取得できる場合は新しい順に並べる。取得できない場合はページ上の掲載順（通常は新しい順）を使う
3. `data/seen_articles.json` の既読 URL と照合し、未掲載の記事のみを候補にする
4. 候補から最大3件を選ぶ（候補が0件のサイトはスキップ）

### 3. 各新着記事を要約
候補記事それぞれについて:
1. 記事ページをフェッチして本文を取得する
2. **言語にかかわらず日本語で3〜5文**に要約する
3. タイトル・URL・公開日時（取得できた場合）・日本語要約をリストに追加する

### 4. Slack に投稿
新着記事が1件以上あった場合のみ投稿する。投稿フォーマット:

```
📰 *エンジニアブログ 新着記事 - {today}*

*Salesforce Engineering*  ← 新着ありのサイトのみ表示
• *{タイトル}* ({公開日時 or 省略})
  {URL}
  {日本語要約}

• *{タイトル}*
  ...

*GitHub Blog - Engineering*
• ...
```

新着記事が全サイト合計で0件の場合は Slack 投稿をスキップしてログ出力のみ行う。

`SLACK_WEBHOOK_URL` 環境変数が未設定の場合も Slack 投稿をスキップしてコンソールに出力する。

投稿は `curl` コマンドを使う:
```bash
curl -s -X POST "$SLACK_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d "{\"text\": \"...(メッセージ内容)...\"}"
```

### 5. 既読リストの更新
新しく処理した記事の URL と処理日時を `data/seen_articles.json` に追記する。
最新 200 件のみ保持し、古いものは削除する。

### 6. 変更をコミット・プッシュ
新しいブランチを作成したり PR を作成したりしない。必ず main ブランチに直接プッシュする。

```bash
git add data/seen_articles.json
git commit -m "chore: update seen articles - {today}"
git push origin HEAD:main
```

## 注意事項
- 各サイトは最大3件。0件でもよい（全体の合計件数に下限なし）
- 日付データの優先順位: ページ内の日付テキスト → `<time>` タグ → `pubDate`（RSS）→ 取得不可の場合は省略
- 英語記事でも要約は必ず日本語で書く
- ネットワークエラーが発生したサイトはスキップしてログ出力し、次のサイトの処理を続ける
- 記事の要約は正確さを重視し、センセーショナルな表現を避ける
- 新しいブランチを作成したり、PR を作成したりしない。必ず main ブランチに直接プッシュすること
