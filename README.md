# claude-skill

Claude Code のスキルをまとめたリポジトリです。プラグインマーケットプレイス経由でインストールできます。

## インストール方法

Claude Code で以下のコマンドを実行してください。

```
/plugin marketplace add shimobayashi-growers/claude-skill
/plugin install growers@claude-skill
```

## 使い方

インストール後、各スキルは `/growers:skill-name` で呼び出せます。

## 収録スキル

### デザイン

| スキル名 | 説明 |
|----------|------|
| `design-principles` | Linear, Notion, Stripe にインスパイアされたミニマルデザインシステム |
| `ui-ux-pro-max` | 50種のUIスタイル、21パレット、50フォントペアリングを検索可能なデザインDB（Python必須） |

### GA4分析

GA4 MCP連携が必要です。下記の [GA4 MCPセットアップ](#ga4-mcpセットアップ) を参照してください。

| スキル名 | 説明 |
|----------|------|
| `ga4-adhoc-analysis` | 自然言語でGA4アドホック分析を対話的に実行 |
| `ga4-anomaly-detect` | デプロイ/GTM変更前後のデータ異常を自動検出 |
| `ga4-custom-design` | 事業KPIに基づくカスタムイベント・ディメンションの設計支援 |
| `ga4-data-exploration` | チャネル・デバイス・時間帯別のクロス分析と課題探索 |
| `ga4-demo-report` | Google公式デモアカウントでのレポート出力・練習用 |
| `ga4-learning-quiz` | 実データに基づくGA4学習クイズの生成 |
| `ga4-page-issue` | ページ別分析から改善提案を抽出しGitHub Issue化 |
| `ga4-property-audit` | GA4プロパティ設定の監査と推奨構成チェック |

### ワークフロー

| スキル名 | 説明 |
|----------|------|
| `implement-to-pr` | 実装タスクからPR作成までの一貫ワークフロー |

### コマンド

| コマンド名 | 説明 |
|------------|------|
| `create-pr` | フォーマット→simplify→細かいコミット→Mermaid図付きPR作成を自動化 |

## GA4 MCPセットアップ

GA4スキルは [analytics-mcp](https://github.com/nicholasgriffintn/analytics-mcp) 経由で GA4 Data API にアクセスします。

### 1. Google Cloud Console で API を有効化

[GA4 Data API](https://console.cloud.google.com/apis/library/analyticsdata.googleapis.com) を有効にしてください。

### 2. サービスアカウントを作成

[IAM](https://console.cloud.google.com/iam-admin/serviceaccounts) でサービスアカウントを作成し、JSONキーをダウンロードします。

### 3. GA4プロパティで権限を付与

GA4管理画面 → プロパティ → プロパティのアクセス管理 → サービスアカウントのメールアドレスを「閲覧者」として追加。

### 4. Claude Code に MCP サーバーを設定

`~/.claude/settings.json` またはプロジェクトの `.claude/settings.json` に追加:

```json
{
  "mcpServers": {
    "analytics-mcp": {
      "command": "npx",
      "args": ["-y", "analytics-mcp"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/service-account-key.json"
      }
    }
  }
}
```

### 5. プロジェクトの CLAUDE.md に property ID を設定

```markdown
## GA4分析
- GA4_PROPERTY_ID: <your-property-id>
```

GA4プロパティIDは GA4管理画面 → プロパティ設定 で確認できます。

## スキルの追加方法

1. `skills/<skill-name>/SKILL.md` を作成する（`template/SKILL.md` を参考に）
2. frontmatter の `name` をディレクトリ名と一致させ、`description` にスキルの用途を記載する
3. `.claude-plugin/marketplace.json` の `skills` 配列に `"./skills/<skill-name>"` を追加する
4. commit & push する

### SKILL.md の書き方

```yaml
---
name: my-skill
description: このスキルをいつ使うべきかの説明
---

# スキルのタイトル

Claude への指示をここに記載する
```
