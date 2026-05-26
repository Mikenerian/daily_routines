# Daily Routines

Claude Code Routines を使った自動タスク集。

## セットアップ

### Slack Webhook URL の設定

Claude Code on the web の Environment 設定で以下の環境変数を追加する:

```
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

Slack の Incoming Webhooks は [Slack API](https://api.slack.com/messaging/webhooks) から取得できる。

## Routines 一覧

### `wired-jp-digest` — WIRED Japan 新着記事ダイジェスト

**スケジュール:** 毎日 09:00 (UTC)

**概要:** https://wired.jp の新着記事を自動チェックし、前回未投稿の記事を日本語で要約して Slack に投稿する。

**状態管理:** `data/seen_articles.json` に処理済み記事URLを保持する（最新200件）。

## ディレクトリ構成

```
.claude/
  routines/
    wired-jp-digest.md   # ルーティン定義
  settings.json          # 権限設定
data/
  seen_articles.json     # 処理済み記事URL（自動更新）
CLAUDE.md                # このファイル
```
