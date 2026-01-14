third fix.
## 🧩 質問①

> git fetch origin → git rebase origin/main をすると毎回 force が必要？正しいやり方は？

### ✅ 解答

- **rebase はコミットIDを書き換える**ため、一度 push したブランチで再度 rebase すると
 **`-force-with-lease` が必要**。
- **まだ push していないローカルブランチ**なら force は不要。
- PR（プルリク）作成後は、履歴を変えないように merge で追従するのが安全。

### 💻 コマンドまとめ

```bash
# 通常のrebase運用
git fetch origin
git rebase origin/main

# rebase後にpush（公開済みブランチなら必ず）
git push --force-with-lease origin feature/your-branch

# forceを避けたいならmerge運用
git fetch origin
git merge origin/main
git push origin feature/your-branch
```

---

## 🧩 質問②

> 一度PRを作成して、その間に main で修正があったらマージするしかないの？


### ✅ 解答

- いいえ。PR作成後でも3つの方法があります：
    1. `merge origin/main`（安全・force不要）
    2. `rebase origin/main`（履歴きれい・force-with-lease必要）
    3. 新しいブランチを切って PR 作り直し

### 💻 コマンドまとめ

```bash
# (1) merge方式（安全・force不要）

git fetch origin
git switch feature/branch
git merge origin/main
git push origin feature/branch

# (2) rebase方式（履歴きれい・force必要）

git fetch origin
git switch feature/branch
git rebase origin/main
git push --force-with-lease origin feature/branch
```

---

## 🧩 質問③

> mainブランチを更新する場合、fetchしたらmergeするしかない？
> マージコミットを作りたくない・pushが面倒。

### ✅ 解答

- **ただ最新化したいだけなら merge 不要**。
Fast-forward 更新（`-ff-only`）でコミットも push も不要。
- main にローカル変更がある場合は rebase か reset。

### 💻 コマンドまとめ

```bash
# ① 最新化だけ（マージコミットなし・push不要）
git switch main
git pull --ff-only

# 同じ意味（手動fetch+merge）
git fetch origin
git merge --ff-only origin/main

# ② mainにローカル変更がある場合（リモートに合わせる）
git fetch origin
git reset --hard origin/main   # ローカルmainをリモートと完全一致させる

# ③ ローカル変更を保ちたいなら
git pull --rebase

# 設定で常にマージコミットを作らない
git config --global pull.ff only
```

---
## 🧩 質問④

> PR（Pull Request）の変更をローカルで検証するためのコマンドを教えて
> 

### ✅ 解答

GitHub 上の PR は、`refs/pull/番号/head` という特別なリファレンスとして取得できます。

ローカルでその変更内容をチェックアウトして動作確認が可能です。

### 💻 コマンドまとめ

```bash
# PRをローカルにfetchしてチェックアウト
git fetch origin pull/<PR番号>/head:pr-<PR番号>
git switch pr-<PR番号>

# 例）PR #12 を検証したい場合
git fetch origin pull/12/head:pr-12
git switch pr-12

# 作業後は削除してクリーンに
git branch -D pr-12
```

---

## 🧭 総まとめ

| 状況 | 推奨操作 | force要否 |
| --- | --- | --- |
| 初回 push 前の rebase | OK | 不要 |
| PR 作成後の rebase | できるがやったらダメ | `--force-with-lease` 必須 |
| main 更新（追従のみ） | `git pull --ff-only` | 不要 |
| main にローカル変更あり | `git pull --rebase` or `reset --hard` | 要push or 不要 |
| 安全に済ませたい | merge 運用 | 不要 |
| PR のローカル検証 | `git fetch origin pull/<番号>/head:pr-<番号>` | 不要 |
