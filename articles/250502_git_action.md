---
title: "Git Action🌾🐖"
emoji: "🐖"
type: "tech" # idea / tech
topics: ["ソロ開発", "Github", "GitHub Actions"]
published: true
---

# GitHub Actions で開発履歴を消してクリーンな公開用リポジトリに自動転送する方法

## 🎯 目的

- 開発履歴に本名や社内情報が含まれている場合に、それを**完全に非公開**にしたい
- `.github/workflows` や `.gitignore` などの内部設定ファイルを**公開先に含めたくない**
- 公開先のリポジトリには **`{username}` 名義で、初期コミットだけ** を含めたい

---

## 📦 構成

- 開発用リポジトリ：`{公開元username}/new_project_name`
- 公開用リポジトリ：`{username}/new_project_name`
- 自動転送には **GitHub Actions + Personal Access Token（PAT）** を使用

---

## 🛠 1. GitHub Actions ワークフローを追加

`{公開元username}` 側のリポジトリに以下のファイルを追加：

```bash
mkdir -p .github/workflows
touch .github/workflows/publish.yml
```

内容は以下の通りです：

```yaml
name: Publish to Public Repo

on:
  push:
    branches: [ main ]

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          persist-credentials: false
          fetch-depth: 0  # 全履歴を取得（filter-repo不要の場合でも重要）

      - name: Copy files and re-init git
        run: |
          TMPDIR=$(mktemp -d)
          rsync -a --exclude='.git' ./ "$TMPDIR"/
          rm -rf ./*
          rsync -a "$TMPDIR"/ ./
          rm -rf "$TMPDIR"

          rm -rf .git
          git init

          git config user.name "{username}"
          git config user.email "209776260+{username}@users.noreply.github.com"

          # 公開不要なものは削除
          rm -rf .github .gitignore

          git add .
          git commit -m "Initial public commit"

      - name: Push to public repo
        env:
          PAT: ${{ secrets.PUBLIC_REPO_TOKEN }}
        run: |
          git branch -M main
          git remote add public https://x-access-token:$PAT@github.com/{username}/new_project.git
          git push public main --force
```

---

## 🔐 2. Secrets に PAT を登録

{公開元username} リポジトリの **Settings > Secrets and variables > Actions** に  
以下のシークレットを登録します：

- `Name`: `PUBLIC_REPO_TOKEN`
- `Value`: `{username}` アカウントで発行した **Personal Access Token（repo権限付き）**

> ✅ トークンには最低限 `repo` スコープが必要です。

---

## 🚀 3. アクションを発火して公開リポジトリに転送

任意のファイルをコミット・プッシュしてアクションを発動させます：

```bash
git commit --allow-empty -m "Trigger publish"
git push origin main
```

---

## ✅ 結果

- `{username}/new_project` に初期コミットが 1 つだけ存在
- コミッターは `{username}` のみ
- `.github/` や `.gitignore` などは含まれない
- `{公開元username}` の名前・メールアドレス・履歴は一切残っていない

---

## 🔁 応用

- `.mailmap` を使って複数名義を一括で置換する構成も可能
- filter-repo を活用すれば、コミットメッセージや履歴の改ざんもより柔軟に
- 複数プロジェクトのテンプレート化にも最適

---

## 🏁 まとめ

本手法を使えば、**開発履歴や個人情報を含まず、公開用に最適化された GitHub リポジトリを自動生成**できる。  
個人開発・副業・技術ポートフォリオなどで、「企業名や実名を出さずに成果物を見せたい」場合にとても便利。
