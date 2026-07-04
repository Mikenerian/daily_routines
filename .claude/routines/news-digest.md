---
schedule: "0 22 * * 0"
description: "エンジニアブログの新着記事をチェックして Slack に日本語要約を投稿する（毎週月曜 07:00 JST）"
---

# エンジニアブログ 新着記事ダイジェスト

## 目的
毎週月曜の朝、複数のエンジニアブログをチェックし、前回以降の新着記事の中から **「データ分析」や「生成AI」の利活用に関する応用的なトピック** を中心に **全体で最大10件** ピックアップして日本語で要約し Slack に投稿する。

> スケジュールは `0 22 * * 0`（UTC）= **毎週月曜 07:00 JST** に実行される。
> 曜日フィールドの `0` は「UTC の日曜」を指すが、JST（UTC+9）に換算すると月曜 07:00 になるため、これで「月曜朝」の実行になる（`1` にすると火曜朝にずれるので注意）。

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

全サイトの未掲載記事を集めて、次のステップで横断的に選抜する（サイトごとの件数上限は設けない）。

### 3. トピックで選抜（全体で最大10件）
集めた候補の中から、以下の観点で **関連度の高い記事を優先** して **全体で最大10件** を選ぶ:

- **主軸トピック:** 「データ分析（データ基盤・分析・BI・データエンジニアリング等）」または「生成AI（LLM・RAG・エージェント・生成AIの製品/業務への組み込み等）」の **利活用に関する応用的なトピック**
- 単なる製品リリース告知・採用/イベント情報・ポエム的な記事より、**実装・アーキテクチャ・導入事例・ベストプラクティスなど実務に役立つ応用寄りの内容** を優先する
- 主軸トピックに合致する候補が10件未満の場合はその件数でよい（無理に他ジャンルで埋めない）
- 主軸トピックに合致する候補が0件の場合は、該当なしとして Slack 投稿をスキップする（ステップ4参照）

タイトルやリード文から判断が難しい場合は、記事ページを確認してトピック該当性を判定してよい。

### 4. 各新着記事を要約
選抜した記事それぞれについて:
1. 記事ページをフェッチして本文を取得する
2. **言語にかかわらず日本語で3〜5文**に要約する
3. タイトル・URL・公開日時（取得できた場合）・日本語要約をリストに追加する

### 5. Slack に投稿
選抜記事が1件以上あった場合のみ投稿する。投稿フォーマット（サイトごとにまとめて表示）:

```
📰 *エンジニアブログ 新着記事（データ分析 / 生成AI）- {today}*

*Salesforce Engineering*  ← 選抜記事ありのサイトのみ表示
• *{タイトル}* ({公開日時 or 省略})
  {URL}
  {日本語要約}

• *{タイトル}*
  ...

*GitHub Blog - Engineering*
• ...
```

主軸トピックに合致する選抜記事が0件の場合は Slack 投稿をスキップしてログ出力のみ行う。

`SLACK_WEBHOOK_URL` 環境変数が未設定の場合も Slack 投稿をスキップしてコンソールに出力する。

投稿は `curl` コマンドを使う:
```bash
curl -s -X POST "$SLACK_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d "{\"text\": \"...(メッセージ内容)...\"}"
```

### 6. 既読リストの更新
新しく処理した記事の URL と処理日時を `data/seen_articles.json` に追記する。
最新 200 件のみ保持し、古いものは削除する。

### 7. 変更をコミット・プッシュ
新しいブランチを作成したり PR を作成したりしない。必ず main ブランチに直接プッシュする。

```bash
git add data/seen_articles.json
git commit -m "chore: update seen articles - {today}"
git push origin HEAD:main
```

## 注意事項
- 選抜は **全体で最大10件**。主軸トピック（データ分析 / 生成AI の利活用・応用）に合致するものを優先し、合致が少なければ件数も少なくてよい（0件なら投稿スキップ）
- サイトごとの件数上限は設けない（1サイトから複数件選ばれてもよい）
- 日付データの優先順位: ページ内の日付テキスト → `<time>` タグ → `pubDate`（RSS）→ 取得不可の場合は省略
- 英語記事でも要約は必ず日本語で書く
- ネットワークエラーが発生したサイトはスキップしてログ出力し、次のサイトの処理を続ける
- 記事の要約は正確さを重視し、センセーショナルな表現を避ける
- 新しいブランチを作成したり、PR を作成したりしない。必ず main ブランチに直接プッシュすること
