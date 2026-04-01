---
name: ga4-anomaly-detect
description: |
  デプロイ/GTM変更前後のデータ変化を検出し、原因を調査する。基準日の前後7日間を比較し、急増・急減を自動検出して原因仮説を提示する。

  Triggers: "異常値検出", "データ異常", "アノマリ", "急増", "急減", "デプロイ影響", "ga4-anomaly-detect"
---

# GA4異常値検出・原因調査

## ワークフロー

```
1. 基準日の前後7日間を比較（イベント別・ページ別）
   ↓
2. 急増（130%以上）・急減（70%以下）を自動検出
   ↓
3. 異常の原因仮説を提示
   ↓
4. 重要な異常はGitHub Issue化（任意）
```

---

## Step 1: パラメータ確認

ユーザーに以下を確認する（`AskUserQuestion` 使用）:

- **基準日:** デプロイ日やGTM変更日（デフォルト: `yesterday`）
- **property_id:** 環境変数 `GA4_PROPERTY_ID` から取得。未設定の場合はユーザーに確認する
- **比較対象:** イベント / ページ / 両方（デフォルト: 両方）

---

## Step 2: データ取得

`mcp__analytics-mcp__run_report` を使用。**4つのレポートを並列取得**する。

基準日を `D` とし、前期 `D-14〜D-8`、後期 `D-7〜D-1` で比較する。

### 2-1. イベント別比較

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [
    {"start_date": "7daysAgo", "end_date": "yesterday", "name": "After"},
    {"start_date": "14daysAgo", "end_date": "8daysAgo", "name": "Before"}
  ],
  "dimensions": ["eventName"],
  "metrics": ["eventCount", "activeUsers"],
  "order_bys": [{"metric": {"metric_name": "eventCount"}, "desc": true}],
  "limit": 30
}
```

### 2-2. ページ別比較

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [
    {"start_date": "7daysAgo", "end_date": "yesterday", "name": "After"},
    {"start_date": "14daysAgo", "end_date": "8daysAgo", "name": "Before"}
  ],
  "dimensions": ["pagePath"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 20
}
```

### 2-3. 日別トレンド（イベント数）

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "14daysAgo", "end_date": "yesterday", "name": "Full"}],
  "dimensions": ["date"],
  "metrics": ["eventCount", "activeUsers", "screenPageViews", "sessions"],
  "order_bys": [{"dimension": {"dimension_name": "date"}, "desc": false}]
}
```

### 2-4. イベント × 日別（異常イベントの詳細確認用）

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "14daysAgo", "end_date": "yesterday", "name": "Full"}],
  "dimensions": ["date", "eventName"],
  "metrics": ["eventCount"],
  "order_bys": [{"dimension": {"dimension_name": "date"}, "desc": false}],
  "limit": 200
}
```

---

## Step 3: 異常値検出

### 検出基準

| 異常タイプ | 判定基準 | 重要度 |
|---|---|---|
| 急増 | 後期/前期 ≥ 130% | 🟡 要確認 |
| 大幅急増 | 後期/前期 ≥ 200% | 🔴 要調査 |
| 急減 | 後期/前期 ≤ 70% | 🟡 要確認 |
| 大幅急減 | 後期/前期 ≤ 50% | 🔴 要調査 |
| 消失 | 前期にあり後期で0 | 🔴 実装エラーの可能性 |
| 新規出現 | 前期になく後期で出現 | 🟡 新機能 or 誤実装 |

### フィルタ条件

- イベント: 前期 eventCount ≥ 10 のもののみ対象（ノイズ除外）
- ページ: 前期 PV ≥ 5 のもののみ対象

---

## Step 4: 原因仮説の提示

検出された異常ごとに、以下のカテゴリで原因仮説を提示する。

| カテゴリ | 典型的な原因 | 確認方法 |
|---|---|---|
| 実装ミス | イベントタグの設置ミス、パラメータ欠落 | GTM/コードの差分確認 |
| デプロイ影響 | 新機能追加、UI変更、ルーティング変更 | Gitログ確認 |
| 外部要因 | SEO変動、キャンペーン、季節要因 | チャネル別データ確認 |
| データ品質 | ボット、スパム、フィルタ変更 | ユーザー属性確認 |
| GTM変更 | トリガー条件変更、タグ追加/削除 | GTMバージョン履歴確認 |

---

## Step 5: レポート出力

```markdown
# GA4異常値検出レポート
対象: [プロパティ名]（property_id: [ID]）
基準日: YYYY/MM/DD
比較期間: [前期] vs [後期]

## 日別トレンド

| 日付 | イベント数 | ユーザー | PV | セッション |
|---|---:|---:|---:|---:|

（基準日に `◀` マークを付与）

## 検出された異常

### 🔴 重大な異常（要調査）

| # | 対象 | タイプ | 前期 | 後期 | 変化率 |
|---|---|---|---:|---:|---:|
| 1 | [イベント/ページ名] | 急増/急減 | [値] | [値] | [%] |

### 🟡 要確認

| # | 対象 | タイプ | 前期 | 後期 | 変化率 |
|---|---|---|---:|---:|---:|

## 原因分析

### 異常1: [対象名] — [異常タイプ]
- **データ:** [前期値] → [後期値]（[変化率]%）
- **日別推移:** [変化が始まった日付を特定]
- **仮説1:** [原因仮説]（確度: 高/中/低）
- **仮説2:** [原因仮説]（確度: 高/中/低）
- **確認アクション:** [次に確認すべきこと]

## まとめ
- 🔴 重大: [X]件
- 🟡 要確認: [X]件
- ✅ 正常範囲: それ以外
```

---

## Step 6: GitHub Issue化（任意）

レポート出力後、`AskUserQuestion` で確認：
- 「重大な異常についてGitHub Issueを作成しますか？」
- 選択肢: 「作成する」「しない」

### Issueフォーマット

```bash
gh issue create --title "[調査] [対象名]の[異常タイプ]を検出（[変化率]%）" --body "$(cat <<'EOF'
## 概要
GA4データの異常値検出により、[対象]の[異常タイプ]を検出。

## データ
- **比較期間:** [前期] vs [後期]
- **前期値:** [値]
- **後期値:** [値]
- **変化率:** [%]
- **変化開始日:** [日付]

## 原因仮説
1. [仮説1]（確度: [高/中/低]）
2. [仮説2]（確度: [高/中/低]）

## 確認アクション
- [ ] [アクション1]
- [ ] [アクション2]

---
🤖 GA4異常値検出により自動生成
EOF
)" --label "bug"
```

---

## 出力フォーマット規約

- 数値はカンマ区切り（例: `1,234`）
- パーセンテージは小数点1桁（例: `89.8%`）
- 時間は「○分○秒」形式（例: `5分12秒`）
- 前期比は `(前期比: +12.3%)` or `(前期比: -8.5%)` 形式
- 根拠データは全て GA4 API レスポンスから直接引用。推測値には「推定」と明記
- テーブルの数値は右寄せ（`---:`）

## 注意事項

- GA4 Data APIにはサンプリング・クォータ制限がある。レスポンスの `sampling_metadatas` を確認し、サンプリングされている場合はレポートに明記すること
- ハルシネーション防止: 数値は必ずAPIレスポンスから直接引用し、計算値は計算過程を示す
