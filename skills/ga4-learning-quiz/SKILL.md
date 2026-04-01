---
name: ga4-learning-quiz
description: |
  実データに基づいたGA4学習用のクイズ・問題を生成する。標準レポートの理解から探索レポートの応用まで、難易度別に出題する。

  Triggers: "GA4クイズ", "GA4学習", "GA4問題", "分析クイズ", "ga4-learning-quiz"
---

# GA4学習コンテンツ作成

## ワークフロー

```
1. 実データを取得（run_report）
   ↓
2. データに基づいた設問を作成
   ↓
3. 選択肢と解説を生成
   ↓
4. 難易度別に出力
```

---

## Step 1: パラメータ確認

`AskUserQuestion` で以下を確認する:

- **難易度:** 初級 / 中級 / 上級 / ミックス（デフォルト: ミックス）
- **問題数:** デフォルト 5問
- **property_id:** 環境変数 `GA4_PROPERTY_ID` から取得。未設定の場合はユーザーに確認する
- **テーマ:** 特定テーマの指定があれば（例: 「チャネル分析」）

---

## Step 2: データ取得

`mcp__analytics-mcp__run_report` を使用。**問題生成に必要なデータを並列取得**する。

### 2-1. ページ別データ

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

### 2-2. チャネル別データ

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["sessionDefaultChannelGroup"],
  "metrics": ["activeUsers", "sessions", "engagementRate", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}]
}
```

### 2-3. デバイス別データ

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["deviceCategory"],
  "metrics": ["activeUsers", "sessions", "engagementRate", "averageSessionDuration", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "activeUsers"}, "desc": true}]
}
```

### 2-4. 前期比較

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [
    {"start_date": "28daysAgo", "end_date": "yesterday", "name": "Current"},
    {"start_date": "56daysAgo", "end_date": "29daysAgo", "name": "Previous"}
  ],
  "dimensions": ["pagePath"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 10
}
```

---

## Step 3: 問題作成

### 難易度別の出題範囲

| 難易度 | 出題範囲 | 問題タイプ |
|---|---|---|
| 初級 | 指標の定義・読み方 | 4択クイズ |
| 中級 | データの解釈・比較分析 | 4択 + 記述 |
| 上級 | 原因分析・改善提案 | 記述・応用問題 |

### 出題テンプレート

#### 初級: 指標理解

```
Q: [指標名]が[値]%とは、どういう意味ですか？
A) [正解の説明]
B) [誤答1]
C) [誤答2]
D) [誤答3]
```

#### 初級: データ読み取り

```
Q: 以下のデータで、最もPVが多いページはどれですか？
[テーブル形式でデータを提示]
A) [選択肢]
```

#### 中級: 比較分析

```
Q: 以下のデバイス別データから、モバイルUXに課題があると判断できる根拠を選んでください。
[テーブル形式でデータを提示]
A) [選択肢]
```

#### 中級: 前期比較

```
Q: 以下のページで前月比で最も改善が見られるのはどれですか？その根拠は？
[テーブル形式でデータを提示]
```

#### 上級: 原因分析

```
Q: ページXのエンゲージメント率が[値]%で、前期比[変化]%です。
考えられる原因を3つ挙げ、それぞれの確認方法を述べてください。
```

#### 上級: 改善提案

```
Q: 以下のデータに基づいて、最も優先度の高い改善施策を提案してください。
理由とともに説明してください。
[複数のテーブルデータを提示]
```

---

## Step 4: 出力フォーマット

```markdown
# GA4学習クイズ
対象: [プロパティ名]（property_id: [ID]）
データ期間: YYYY/MM/DD〜YYYY/MM/DD
難易度: [初級/中級/上級/ミックス]

---

## 問題 1 / [総数]（難易度: ⭐ 初級）

### データ

[テーブルまたは数値の提示]

### 設問

[問題文]

<details>
<summary>選択肢を見る</summary>

A) [選択肢A]
B) [選択肢B]
C) [選択肢C]
D) [選択肢D]

</details>

<details>
<summary>正解と解説を見る</summary>

**正解: [A/B/C/D]**

**解説:**
[なぜその答えが正しいのか、GA4の指標の意味を含めて解説]

**学習ポイント:**
- [この問題で学べるGA4の概念]

</details>

---

## 問題 2 / [総数]（難易度: ⭐⭐ 中級）

（同構成で繰り返し）

---

## 問題 [N] / [総数]（難易度: ⭐⭐⭐ 上級）

（同構成で繰り返し）

---

## まとめ

### 学習ポイント一覧

| # | テーマ | ポイント |
|---|---|---|
| 1 | [テーマ] | [学べること] |

### おすすめの次のステップ
- [さらに学ぶための提案1]
- [さらに学ぶための提案2]
```

---

## 出力フォーマット規約

- 数値はカンマ区切り（例: `1,234`）
- パーセンテージは小数点1桁（例: `89.8%`）
- 時間は「○分○秒」形式（例: `5分12秒`）
- 根拠データは全て GA4 API レスポンスから直接引用。推測値には「推定」と明記
- テーブルの数値は右寄せ（`---:`）
- `<details>` タグで回答を隠す（ユーザーが自分で考えてから確認できるように）

## 注意事項

- 問題の数値は全て実データから引用し、架空の数値を使わない
- 選択肢の誤答はもっともらしいが明確に間違いであること
- 解説では GA4 の公式ドキュメントの概念に基づいて説明する
- ハルシネーション防止: 数値は必ずAPIレスポンスから直接引用し、計算値は計算過程を示す
