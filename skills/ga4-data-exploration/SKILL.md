---
name: ga4-data-exploration
description: |
  チャネル別・デバイス別・時間帯別などの切り口でGA4データを集計し、課題を探索する。複数の集計軸でクロス分析を行い、改善仮説を提示する。

  Triggers: "データ探索", "課題探索", "チャネル分析", "デバイス分析", "時間帯分析", "クロス分析", "ga4-data-exploration"
---

# GA4データ集計 & 課題探索

## ワークフロー

```
1. 集計軸を決定（チャネル/デバイス/時間帯/ページなど）
   ↓
2. 複数レポートを並列取得
   ↓
3. クロス集計・比較分析
   ↓
4. 課題の洗い出しと改善仮説の提示
```

---

## Step 1: 集計軸の決定

- **property_id:** 環境変数 `GA4_PROPERTY_ID` から取得。未設定の場合はユーザーに確認する
- **デフォルト期間:** 直近28日間（`28daysAgo` ~ `yesterday`）
- ユーザーが期間や集計軸を指定した場合はそれに従う

デフォルトでは以下の5軸で分析を行う。ユーザーの指定があれば絞り込む。

| 軸 | ディメンション | 目的 |
|---|---|---|
| チャネル | `sessionDefaultChannelGroup` | 流入経路の評価 |
| デバイス | `deviceCategory` | UXの差異検出 |
| 時間帯 | `hour`, `dayOfWeek` | 利用パターンの把握 |
| ページ | `pagePath` | 機能別利用状況 |
| ユーザー属性 | `operatingSystem`, `browser` | 環境別の問題検出 |

---

## Step 2: データ取得

`mcp__analytics-mcp__run_report` を使用。**6つのレポートを並列取得**する。

### 2-1. チャネル別

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["sessionDefaultChannelGroup"],
  "metrics": ["activeUsers", "sessions", "engagementRate", "averageSessionDuration", "bounceRate", "screenPageViews"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}]
}
```

### 2-2. デバイス別

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["deviceCategory"],
  "metrics": ["activeUsers", "sessions", "engagementRate", "averageSessionDuration", "bounceRate", "screenPageViews"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}]
}
```

### 2-3. 時間帯別

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["hour"],
  "metrics": ["activeUsers", "sessions", "screenPageViews"],
  "order_bys": [{"dimension": {"dimension_name": "hour"}, "desc": false}]
}
```

### 2-4. 曜日別

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["dayOfWeek"],
  "metrics": ["activeUsers", "sessions", "screenPageViews", "engagementRate"],
  "order_bys": [{"dimension": {"dimension_name": "dayOfWeek"}, "desc": false}]
}
```

### 2-5. ページ別

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["pagePath"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate", "averageSessionDuration", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 20
}
```

### 2-6. デバイス × ページ（クロス分析）

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["deviceCategory", "pagePath"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 50
}
```

---

## Step 3: 分析・課題抽出

### 分析の視点

| 視点 | チェック内容 | 課題の兆候 |
|---|---|---|
| チャネル効率 | チャネル別のエンゲージメント率差 | 特定チャネルだけ低い → ランディング最適化不足 |
| デバイスUX | デバイス別の指標差 | モバイルだけ離脱率高い → レスポンシブ不備 |
| 時間帯パターン | ピーク時間帯と閑散時間帯 | 業務時間外の利用 → 通知タイミングの最適化 |
| 曜日パターン | 平日/週末の差 | 週末利用あり → 建設業の土曜出勤パターン |
| ページ集中度 | PVの偏り | 特定ページに集中 → 他機能の認知不足 |
| クロス課題 | デバイス × ページの組み合わせ | 特定デバイス×ページで指標悪化 → 個別最適化必要 |

---

## Step 4: レポート出力

```markdown
# GA4データ探索レポート
対象: [プロパティ名]（property_id: [ID]）
期間: YYYY/MM/DD〜YYYY/MM/DD

## 全体サマリー

| 指標 | 値 |
|---|---:|
| アクティブユーザー | [値] |
| セッション数 | [値] |
| PV数 | [値] |
| エンゲージメント率 | [値] |
| 平均滞在時間 | [値] |
| 離脱率 | [値] |

## チャネル別パフォーマンス

| チャネル | ユーザー | セッション | エンゲージメント率 | 平均滞在時間 | 離脱率 | PV |
|---|---:|---:|---:|---:|---:|---:|

### チャネル分析
- [所見]

## デバイス別パフォーマンス

| デバイス | ユーザー | セッション | エンゲージメント率 | 平均滞在時間 | 離脱率 | PV |
|---|---:|---:|---:|---:|---:|---:|

### デバイス分析
- [所見]

## 時間帯別パフォーマンス

| 時間帯 | ユーザー | セッション | PV |
|---|---:|---:|---:|

### 時間帯分析
- ピーク時間帯: [HH:00〜HH:00]
- 閑散時間帯: [HH:00〜HH:00]

## 曜日別パフォーマンス

| 曜日 | ユーザー | セッション | PV | エンゲージメント率 |
|---|---:|---:|---:|---:|

（曜日: 0=日, 1=月, ... 6=土）

### 曜日分析
- [所見]

## ページ別パフォーマンス TOP20

| # | ページ | PV | ユーザー | エンゲージメント率 | 平均滞在時間 | 離脱率 |
|---|---|---:|---:|---:|---:|---:|

## デバイス × ページ クロス分析

### 指標格差が大きいページ

| ページ | 指標 | Desktop | Mobile | 差分 |
|---|---|---:|---:|---:|

## 課題一覧 & 改善仮説

| # | 課題 | 根拠データ | 改善仮説 | 優先度 |
|---|---|---|---|---|
| 1 | [課題] | [データ] | [仮説] | 高/中/低 |

### 課題詳細

#### 課題1: [タイトル]
- **根拠:** [データ]
- **影響範囲:** [ユーザー数・ページ数]
- **改善仮説:** [具体的な提案]
- **期待効果:** [予測]

（各課題について同構成）
```

---

## 出力フォーマット規約

- 数値はカンマ区切り（例: `1,234`）
- パーセンテージは小数点1桁（例: `89.8%`）
- 時間は「○分○秒」形式（例: `5分12秒`）
- 根拠データは全て GA4 API レスポンスから直接引用。推測値には「推定」と明記
- テーブルの数値は右寄せ（`---:`）

## 注意事項

- GA4 Data APIにはサンプリング・クォータ制限がある。レスポンスの `sampling_metadatas` を確認し、サンプリングされている場合はレポートに明記すること
- 28日間のデータは1日あたりの平均値も併記すると傾向が見えやすい
- ハルシネーション防止: 数値は必ずAPIレスポンスから直接引用し、計算値は計算過程を示す
