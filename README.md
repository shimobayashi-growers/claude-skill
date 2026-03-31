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

| スキル名 | 説明 |
|----------|------|
| `design-principles` | Linear, Notion, Stripe にインスパイアされたミニマルデザインシステム。ダッシュボードや管理画面の構築時に使用 |

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
