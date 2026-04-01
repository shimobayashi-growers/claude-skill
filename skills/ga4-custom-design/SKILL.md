---
name: ga4-custom-design
description: |
  事業戦略に基づいてGA4のカスタムイベント・ディメンション・セグメントの設計を支援する。現在の設定を確認し、KPIに基づいた計測設計を提案する。

  Triggers: "カスタム設計", "イベント設計", "GA4設計", "計測設計", "ディメンション設計", "ga4-custom-design"
---

# GA4カスタム設定の設計支援

## ワークフロー

```
1. 現在のカスタム定義を取得
   ↓
2. 事業目標・KPIをヒアリング
   ↓
3. 必要なカスタムイベント・ディメンションを設計
   ↓
4. 実装コード例（GTMタグ/dataLayer）を提示
```

---

## Step 1: 現在の設定取得

`mcp__analytics-mcp__get_custom_dimensions_and_metrics` を使用。

```json
{
  "property_id": "<GA4_PROPERTY_ID>"
}
```

- **property_id:** 環境変数 `GA4_PROPERTY_ID` から取得。未設定の場合はユーザーに確認する

---

## Step 2: ヒアリング

`AskUserQuestion` で以下を確認する:

### 必須ヒアリング項目

1. **事業目標:** 何を達成したいか
   - 例: ユーザー定着率向上、機能利用促進、チャーン予防
2. **主要KPI:** 何を計測したいか
   - 例: DAU/MAU比率、機能別利用率、オンボーディング完了率
3. **ユーザーセグメント:** どんなユーザー分類が必要か
   - 例: プラン別、ロール別、組織規模別

### テンプレート提示

ヒアリングが曖昧な場合、以下のテンプレートを提示:

| # | KPIカテゴリ | 計測例 | 必要なカスタム定義 |
|---|---|---|---|
| 1 | 利用定着 | DAU/MAU、機能利用頻度 | カスタムイベント + ユーザープロパティ |
| 2 | 機能価値 | 各機能のアクティブ率・完了率 | カスタムイベント + パラメータ |
| 3 | オンボーディング | ステップ完了率、離脱ステップ | カスタムイベント（ファネル用） |
| 4 | チャーン予防 | 最終ログイン日、利用頻度低下 | ユーザープロパティ |
| 5 | ビジネス成果 | プラン別利用状況、upsell指標 | ユーザープロパティ + イベント |

---

## Step 3: カスタム設定の設計

### 設計原則

1. **命名規則:** `snake_case` で統一
2. **イベント命名:** `[カテゴリ]_[アクション]` 形式（例: `schedule_create`, `report_export`）
3. **パラメータ命名:** `[対象]_[属性]` 形式（例: `site_count`, `member_role`）
4. **スコープの選択:**
   - EVENT: 行動に紐づく属性（ページ名、操作対象など）
   - USER: ユーザーに紐づく属性（プラン、ロール、組織IDなど）
   - ITEM: アイテムに紐づく属性（現場カテゴリなど）

### 設計ドキュメントのフォーマット

```markdown
# GA4カスタム設定 設計書
対象: [プロパティ名]（property_id: [ID]）
作成日: YYYY/MM/DD

## 現状の設定

### カスタムディメンション（使用: X / 上限: 50）

| # | 名前 | スコープ | 説明 | ステータス |
|---|---|---|---|---|

### カスタムメトリクス（使用: X / 上限: 50）

| # | 名前 | スコープ | 説明 | ステータス |
|---|---|---|---|---|

## KPIと計測設計

### KPI 1: [KPI名]

**目標:** [数値目標]

#### 必要なカスタムイベント

| イベント名 | 説明 | トリガー条件 | パラメータ |
|---|---|---|---|
| `[event_name]` | [説明] | [条件] | `param1`, `param2` |

#### 必要なカスタムディメンション

| 名前 | スコープ | パラメータ名 | 説明 |
|---|---|---|---|
| `[dimension_name]` | EVENT/USER | `[param]` | [説明] |

#### 必要なカスタムメトリクス

| 名前 | スコープ | パラメータ名 | 単位 | 説明 |
|---|---|---|---|---|
| `[metric_name]` | EVENT | `[param]` | 標準/通貨/時間 | [説明] |

（KPIごとに繰り返し）

## 実装ガイド

### 枠の使用見積もり

| カテゴリ | 現在 | 追加 | 合計 | 上限 | 余裕 |
|---|---:|---:|---:|---:|---:|
| カスタムディメンション（イベント） | X | X | X | 50 | X |
| カスタムディメンション（ユーザー） | X | X | X | 25 | X |
| カスタムメトリクス | X | X | X | 50 | X |
```

---

## Step 4: 実装コード例

### GTM dataLayer形式

```javascript
// イベント送信
dataLayer.push({
  event: '[event_name]',
  [param_name]: [value],
  // ...
});

// ユーザープロパティ設定
dataLayer.push({
  event: 'user_properties_set',
  user_properties: {
    [property_name]: [value],
  }
});
```

### gtag.js形式

```javascript
// イベント送信
gtag('event', '[event_name]', {
  [param_name]: [value],
});

// ユーザープロパティ設定
gtag('set', 'user_properties', {
  [property_name]: [value],
});
```

### Next.js実装例

```typescript
// lib/analytics.ts
export function trackEvent(eventName: string, params?: Record<string, unknown>) {
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('event', eventName, params);
  }
}

// 使用例
trackEvent('feature_use', {
  feature_name: 'export',
  format: 'csv',
});
```

---

## 出力フォーマット規約

- 命名はすべて `snake_case`
- イベント名は動詞を含める（`create`, `update`, `delete`, `view`, `export`）
- パラメータ値の型を明記（string / number / boolean）
- 根拠データは全て GA4 API レスポンスから直接引用。推測値には「推定」と明記

## 注意事項

- カスタムディメンション・メトリクスの上限に注意（イベントスコープ50、ユーザースコープ25）
- 一度作成したカスタム定義はアーカイブできるが枠は戻らない（GA4の仕様）
- PII（個人識別情報）をカスタムディメンションに含めてはいけない
- 設計変更は影響が大きいため、段階的な導入を推奨する
