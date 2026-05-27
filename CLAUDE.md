# Daily Routines

Claude Code Routines を使った自動タスク集。

## セットアップ

### Slack Webhook URL の設定

Claude Code on the web の Environment 設定で以下の環境変数を追加する:

```
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

Slack の Incoming Webhooks は [Slack API](https://api.slack.com/messaging/webhooks) から取得できる。

### ネットワーク許可リストの設定

Environment のネットワークポリシーを `Custom` に設定し、以下のドメインを許可する:

```
engineering.salesforce.com
www.snowflake.com
developers.googleblog.com
aws.amazon.com
engineering.fb.com
github.blog
developers.openai.com
openai.com
www.anthropic.com
slack.com
atlassianblog.wpengine.com
www.docker.com
```

## Routines 一覧

### `news-digest` — エンジニアブログ 新着記事ダイジェスト

**スケジュール:** 毎日 09:00 (UTC)

**概要:** 複数のエンジニアブログを自動チェックし、前回未投稿の新着記事をサイトごとに最大3件、日本語で要約して Slack に投稿する。英語記事も日本語で要約する。

**対象サイト:**
- Salesforce Engineering
- Snowflake AI Research
- Google Developers Blog (JA)
- AWS Blog (JP)
- Meta Engineering
- GitHub Blog - Engineering
- OpenAI Developer Blog
- OpenAI News (JA)
- Anthropic Engineering
- Slack Engineering Blog
- Atlassian Blog
- Docker Blog

**状態管理:** `data/seen_articles.json` に処理済み記事URLを保持する（最新200件）。

## ディレクトリ構成

```
.claude/
  routines/
    news-digest.md       # ルーティン定義
  settings.json          # 権限設定
data/
  seen_articles.json     # 処理済み記事URL（自動更新）
CLAUDE.md                # このファイル
```
