---
name: ga4-page-issue
description: |
  GA4データからページ別の利用状況を分析し、課題を抽出。重要度・緊急度で改善案を3つに絞り、ユーザー確認後にGitHub Issueを作成する。

  Triggers: "GA4分析", "ページ分析", "アクセス分析", "改善Issue", "利用状況分析", "ga4-page-issue"
---

# GA4ページ別分析 → 改善Issue作成

## ワークフロー

```
1. データ取得（GA4 API）
   ↓
2. 課題抽出（ページ別の問題点を洗い出し）
   ↓
3. 優先度評価（重要度 × 緊急度で3つに絞る）
   ↓
4. レポート出力 & ユーザー確認
   ↓
5. GitHub Issue作成（確認後）
```

---

## Step 1: データ取得

`mcp__analytics-mcp__run_report` を使用。**4つのレポートを並列取得**する。

- **property_id:** 環境変数 `GA4_PROPERTY_ID` から取得。未設定の場合はユーザーに確認する
- **デフォルト期間:** 直近7日間（`7daysAgo` ~ `yesterday`）
- ユーザーが期間を指定した場合はそれに従う

### 1-1. ページ別 PV・エンゲージメント

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["pagePath"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate", "averageSessionDuration", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 20
}
```

### 1-2. ページ × デバイス別

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["pagePath", "deviceCategory"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 50
}
```

### 1-3. ランディングページ

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [{"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"}],
  "dimensions": ["landingPage"],
  "metrics": ["sessions", "activeUsers", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "sessions"}, "desc": true}],
  "limit": 10
}
```

### 1-4. 前期比較（ページ別）

```json
{
  "property_id": "<GA4_PROPERTY_ID>",
  "date_ranges": [
    {"start_date": "7daysAgo", "end_date": "yesterday", "name": "Current"},
    {"start_date": "14daysAgo", "end_date": "8daysAgo", "name": "Previous"}
  ],
  "dimensions": ["pagePath"],
  "metrics": ["screenPageViews", "activeUsers", "engagementRate", "bounceRate"],
  "order_bys": [{"metric": {"metric_name": "screenPageViews"}, "desc": true}],
  "limit": 20
}
```

---

## Step 2: 課題抽出

取得したデータから、以下の判定基準でページごとに課題を特定する。

| 課題タイプ | 判定基準 | 意味 |
|---|---|---|
| 低エンゲージメント | engagementRate < 70% | ページ内容がユーザー期待と不一致 |
| 高離脱率 | bounceRate > 50% | ランディング後すぐ離脱 |
| 利用率低下 | 前期比 PV -20%以上 | 機能の価値低下または導線不備 |
| 滞在時間異常（短） | averageSessionDuration < 30秒 | 目的を達成できていない |
| 滞在時間異常（長） | averageSessionDuration > 15分 | UIが複雑/操作に迷っている可能性 |
| 機能未発見 | PVが全体の1%未満 | 導線/ディスカバラビリティ不足 |
| デバイス間格差 | 同一ページでデバイス別の指標差が大きい | レスポンシブ対応の不備 |

**注意:** 判定基準はあくまで目安。プロダクト特性（B2B SaaS、建設業向け）を考慮して解釈すること。

---

## Step 3: 優先度評価

課題を重要度 × 緊急度で評価し、**上位3つ**に絞る。

### 重要度（Impact）

| レベル | 条件 |
|---|---|
| 高 | activeUsersが多い主要ページの課題（dashboard, calendar, reports） |
| 中 | 設定系・休暇申請など中規模ページの課題 |
| 低 | オンボーディング等の低頻度ページの課題 |

### 緊急度（Urgency）

| レベル | 条件 |
|---|---|
| 高 | 前期比で急激な悪化（-30%以上）、bounceRate急増 |
| 中 | じわじわ悪化傾向、基準値を下回る |
| 低 | 改善余地はあるが安定している |

### スコアリング

- ★★★: 重要度・緊急度ともに高い
- ★★☆: 一方が高く他方が中
- ★☆☆: それ以外

---

## Step 4: レポート出力 & ユーザー確認

以下の構成でレポートを出力する。

```markdown
# GA4ページ別分析レポート
期間: YYYY/MM/DD〜YYYY/MM/DD（前期比較: YYYY/MM/DD〜YYYY/MM/DD）

## ページ別サマリー

| ページ | PV | ユーザー | エンゲージメント率 | 平均滞在時間 | 離脱率 | 前期比PV |
|---|---:|---:|---:|---:|---:|---:|

## デバイス別内訳

### Desktop
| ページ | PV | ユーザー | エンゲージメント率 | 離脱率 |
|---|---:|---:|---:|---:|

### Mobile
| ページ | PV | ユーザー | エンゲージメント率 | 離脱率 |
|---|---:|---:|---:|---:|

### Tablet（データがあれば）
| ページ | PV | ユーザー | エンゲージメント率 | 離脱率 |
|---|---:|---:|---:|---:|

## 抽出された課題一覧

| # | ページ | 課題タイプ | 根拠データ |
|---|---|---|---|

## 改善提案 TOP3（重要度 × 緊急度）

| # | ページ | 課題 | 重要度 | 緊急度 | スコア |
|---|---|---|---|---|---|

### 提案1: [タイトル]
- **ページ:** [pagePath]
- **課題:** [詳細]
- **データ根拠:** [数値]（デバイス別の差異があれば記載）
- **改善案:** [具体的な提案]
- **期待効果:** [予測]

### 提案2: [タイトル]
（同構成）

### 提案3: [タイトル]
（同構成）
```

### ユーザー確認

レポート出力後、`AskUserQuestion` で確認する：
- 「この3つの改善案でGitHub Issueを作成しますか？」
- 選択肢: 「このまま作成」「修正してから作成」「やめる」
- 修正要望があれば対応してから再度確認

---

## Step 5: GitHub Issue作成

ユーザーの承認後、3つの改善案をそれぞれ `gh issue create` で起票する。

### Issueフォーマット

```bash
gh issue create --title "[改善] ページ名: 課題の要約" --body "$(cat <<'EOF'
## 概要
[課題の要約]

## データ根拠
- **対象ページ:** `[pagePath]`
- **分析期間:** [日付範囲]
- **[指標名]:** [数値]（前期比: [変化率]%）
- **デバイス別:** Desktop [値] / Mobile [値]

## 課題
[データから読み取れる問題点の詳細]

## 改善案
[具体的な改善提案 - プロダクトの画面に紐づけて記述]

## 期待効果
[改善後に期待される数値変化]

---
🤖 GA4データに基づきClaude Codeが自動生成
EOF
)" --label "enhancement"
```

3つのIssueを作成したら、Issue番号とURLをまとめて報告する。

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
- セグメント機能は未対応。`dimension_filter` で代替する
- ハルシネーション防止: 数値は必ずAPIレスポンスから直接引用し、計算値は計算過程を示す
