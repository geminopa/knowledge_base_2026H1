# /permissions で「毎回の確認」を減らしつつ、危険な操作は止める

Claude Codeはコマンド実行やファイル編集の前に確認を求める。毎回同じテストコマンドで確認が出るなら、許可ルールを登録すると作業が止まらなくなる。

## 使い方

`/permissions`（別名 `/allowed-tools`）を実行すると画面が開き、ルールの確認・追加・削除ができる。ルールは3種類ある。

| 種類 | 動き |
|---|---|
| `allow` | 確認なしで実行する |
| `ask` | 毎回確認する |
| `deny` | 実行させない |

**評価順は deny → ask → allow**。どれか1つでもdenyに当てはまれば、allowに書いてあっても実行されない。

## allow / ask / deny をどう選ぶか

| 種類 | 選ぶ基準 | 具体例 |
|---|---|---|
| `allow` | 副作用がない、または何度もやり直せる操作。頻繁に使うのに毎回確認が出ると作業が止まる | ① `Bash(./vendor/bin/phpunit *)` テスト実行は失敗しても実害がない<br>② `Bash(git diff *)` / `Bash(git status)` 読み取るだけの操作<br>③ `Bash(php -l *)` PHPの構文チェック（ファイルを書き換えない） |
| `ask` | 実行してよいが、そのとき何をするか都度確認したい中リスクの操作 | ① `Bash(composer require *)` 依存パッケージの追加は内容を見てから許可したい<br>② `Bash(mysql -u * -p * < *.sql)` SQLファイルの流し込みはDBを変更するため都度確認<br>③ `Edit(./config/database.php)` 接続先が変わりうる設定ファイルの編集 |
| `deny` | 取り返しがつかない操作、または秘密情報に触れる操作 | ① `Bash(git push *)` 本番相当のリモートへの反映は人が行う<br>② `Read(./.env)` DBのパスワードなど秘密情報が入ったファイル<br>③ `Bash(rm -rf *)` 復元できない削除 |

## 具体例: settings.json に書く場合

```json
{
  "permissions": {
    "allow": [
      "Bash(./vendor/bin/phpunit *)",
      "Bash(git diff *)"
    ],
    "deny": [
      "Bash(git push *)",
      "Read(./.env)"
    ]
  }
}
```

- `Bash(./vendor/bin/phpunit *)`: `phpunit` で始まるコマンドは確認なし
- `Bash(git push *)`: pushは禁止（人が自分で行う）
- `Read(./.env)`: 秘密情報のファイルは読ませない

## 設定ファイルの置き場所

| ファイル | 範囲 |
|---|---|
| `.claude/settings.json` | プロジェクト共通（gitでチームと共有） |
| `.claude/settings.local.json` | 自分だけ・このプロジェクトだけ |
| `~/.claude/settings.json` | 自分だけ・全プロジェクト |

確認ダイアログで「Yes, and don't ask again」を選んだ場合も、`.claude/settings.local.json` に自動で保存される。

## 注意点

- `*` は「その位置に何が来てもよい」という意味。`Bash(git *)` にすると `git reset --hard` なども許可されるので、`Bash(git diff *)` のように範囲を絞る
- 迷ったら「読むだけの操作はallow、書き込み・外部送信はask、取り返しがつかない操作はdeny」を目安にする
