---
name: ga4-demo-report
description: |
  GA4デモアカウント（Google Merchandise Store）を対象にレポートを出力し、スキルの動作確認やデータ分析の練習に使う。

  Triggers: "デモレポート", "GA4デモ", "デモアカウント分析", "GA4練習", "ga4-demo-report"
---

# GA4デモアカウント出力確認

## ワークフロー

```
1. デモアカウントのproperty_idでレポート取得
   ↓
2. 基本レポート（PV、ユーザー、コンバージョンなど）を出力
   ↓
3. 出力フォーマットの確認・結果の解説
```

---

## Step 1: デモアカウント情報

- **property_id:** `372541897`（Google Merchandise Store デモアカウント）
- **サイト:** merchandisestore.google.com
- **用途:** GA4のデータ分析練習、スキル動作確認

> ユーザーが別のproperty_idを指定した場合はそれに従う

---

## Step 2: レポート取得

`mcp__analytics-mcp__run_report` を使用。**4つのレポートを並列取得**する。

### 2-1. ページ別PV・エンゲージメント

```json
{
  "property_id": 372541897,
  "date_ranges": [{"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["pagePath"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate", "averageSessionDuration", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 20
}
```

### 2-2. チャネル別ユーザー

```json
{
  "property_id": 372541897,
  "date_ranges": [{"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["sessionDefaultChannelGroup"],
  "metrics": ["activeUsers", "sessions", "engagementRate", "conversions", "totalRevenue"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}],
  "limit": 10
}
```

### 2-3. デバイス別

```json
{
  "property_id": 372541897,
  "date_ranges": [{"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["deviceCategory"],
  "metrics": ["activeUsers", "sessions", "engagementRate", "conversions", "totalRevenue"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}]
}
```

### 2-4. 日別トレンド

```json
{
  "property_id": 372541897,
  "date_ranges": [{"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["date"],
  "metrics": ["activeUsers", "screenPageViews", "sessions", "conversions"],
  "order_bys": [{"dimension": {"dimension_name": "date"}, "desc": false}]
}
```

---

## Step 3: レポート出力

以下の構成でレポートを出力する。

```markdown
# GA4デモアカウント レポート
対象: Google Merchandise Store（property_id: 372541897）
期間: YYYY/MM/DD〜YYYY/MM/DD

## 全体サマリー

| 指標 | 値 |
|---|---:|
| アクティブユーザー | [値] |
| セッション数 | [値] |
| PV数 | [値] |
| エンゲージメント率 | [値] |
| コンバージョン数 | [値] |
| 総収益 | $[値] |

## 日別トレンド

| 日付 | ユーザー | PV | セッション | コンバージョン |
|---|---:|---:|---:|---:|

## チャネル別パフォーマンス

| チャネル | ユーザー | セッション | エンゲージメント率 | コンバージョン | 収益 |
|---|---:|---:|---:|---:|---:|

## デバイス別パフォーマンス

| デバイス | ユーザー | セッション | エンゲージメント率 | コンバージョン | 収益 |
|---|---:|---:|---:|---:|---:|

## ページ別PV TOP20

| # | ページ | PV | ユーザー | エンゲージメント率 | 平均滞在時間 | 離脱率 |
|---|---|---:|---:|---:|---:|---:|

## 所見
- [データから読み取れるポイントを3〜5つ記載]
```

---

## 出力フォーマット規約

- 数値はカンマ区切り（例: `1,234`）
- パーセンテージは小数点1桁（例: `89.8%`）
- 時間は「○分○秒」形式（例: `5分12秒`）
- 収益はドル表記（デモアカウントはUSD）
- 根拠データは全て GA4 API レスポンスから直接引用。推測値には「推定」と明記
- テーブルの数値は右寄せ（`---:`）

## 注意事項

- デモアカウントのデータはGoogleが提供するサンプルデータであり、実際の販売データではない
- property_idが無効な場合、`mcp__analytics-mcp__get_account_summaries` でアクセス可能なプロパティを確認する
- ハルシネーション防止: 数値は必ずAPIレスポンスから直接引用し、計算値は計算過程を示す
