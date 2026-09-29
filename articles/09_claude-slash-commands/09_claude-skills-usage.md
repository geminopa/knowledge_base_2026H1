# Skillsの使い方いろいろ

Skillは作って終わりではなく、呼び出し方や設定次第でいろいろな使い方ができる。代表的な4パターンを紹介する。

## 呼び出し方の種類

| 種類 | 指定方法 | 向いている場面 |
|---|---|---|
| 手動呼び出し | `/{command_name} 引数` | 自分の判断で使いたいとき |
| 自動発動 | 何も打たず、会話内容が`description`に合致 | よくある相談を毎回Skill化したいとき |
| 人だけが呼べる | `disable-model-invocation: true` | デプロイなど副作用のある操作 |
| Claudeだけが使う | `user-invocable: false` | 背景知識として持たせたいだけのとき |

## 具体例1: 引数を受け取る

```markdown
---
name: fix-issue
description: 指定した番号のバグを調査・修正する
arguments: [issue-number]
---

課題番号 $issue-number の不具合を調査し、修正案を出してください。
```

呼び出し: `/fix-issue 123` → `$issue-number` に `123` が入る。

## 具体例2: 実行結果を埋め込む

```markdown
---
description: 未コミットの変更点を要約する
---

## 現在の差分

!`git diff HEAD`

## 指示

上記の差分を2〜3行で要約し、リスクがあれば指摘してください。
```

`` !`git diff HEAD` `` の部分は実行結果（実際のdiff）に置き換わってからClaudeに渡される。想像ではなく実際の差分をもとに回答させたいときに使う。

## 具体例3: 副作用のある操作は人だけが呼べるようにする

```markdown
---
name: deploy
description: 本番環境へデプロイする
disable-model-invocation: true
allowed-tools: Bash(php artisan migrate --force)
---

1. テストを実行する
2. マイグレーションを実行する
3. デプロイ完了をチームに報告する
```

`disable-model-invocation: true` を付けると、Claudeが「コードが良さそうだから」と勝手にデプロイを実行することはなくなり、`/deploy` と人が打ったときだけ動く。

## コツ

- 1つのメッセージで複数のSkillを同時に呼び出すこともできる（例: `/write-tests /fix-issue 123`）
- `allowed-tools` はそのSkill実行中だけ有効な一時的な許可。次のメッセージを送ると効果は切れる
- 迷ったら、まずは手動呼び出し（`disable-model-invocation` なし）から始め、慣れてきたら自動発動や実行制御を足していくとよい
