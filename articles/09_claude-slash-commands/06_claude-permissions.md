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

`/permissions` の画面を使わず、設定ファイルに直接書いてもよい。`permissions` の中に `allow` / `ask` / `deny` の配列を置き、ルールを1つずつ文字列で並べる。

```json
{
  "permissions": {
    "allow": [
      "Bash(./vendor/bin/phpunit *)",
      "Bash(git diff *)",
      "Bash(php -l *)"
    ],
    "ask": [
      "Bash(composer require *)",
      "Edit(./config/database.php)"
    ],
    "deny": [
      "Bash(git push *)",
      "Read(./.env)",
      "Read(./secrets/**)"
    ]
  }
}
```

- `Bash(./vendor/bin/phpunit *)`: `phpunit` で始まるコマンドは確認なし
- `Bash(composer require *)`: 実行前に毎回確認する
- `Bash(git push *)`: pushは禁止（人が自分で行う）
- `Read(./.env)` / `Read(./secrets/**)`: 秘密情報のファイル・ディレクトリは読ませない
- ルールの書式は `ツール名` または `ツール名(条件)`。ツール名だけ（例: `WebFetch`）なら、そのツールの全操作が対象になる

## ルールの書き方

**Bashの `*`**

| 書き方 | 一致する | 一致しない |
|---|---|---|
| `Bash(git diff *)` | `git diff`、`git diff --stat` | `git status` |
| `Bash(php -l *)` | `php -l`、`php -l index.php` | `php index.php` |
| `Bash(./vendor/bin/phpunit)` | `./vendor/bin/phpunit` そのもの | `./vendor/bin/phpunit --filter Foo` |

- `*` は空白を含む任意の文字列に一致する。**末尾の `*` の前には空白を入れる。** `Bash(ls *)` は `lsof` に一致しないが、`Bash(ls*)` は一致する
- `*` はサブコマンドの後ろに置く。`Bash(git * main)` のように前に置くと、`git push origin main` にも一致してしまう
- `Bash(git diff:*)` のように末尾を `:*` と書く形も同じ意味で使える

**ファイルのパス（`Read` / `Edit`）**

| 書き方 | 意味 | 例 |
|---|---|---|
| `./path` または `path` | 今いるディレクトリからの相対 | `Read(./.env)` |
| `/path` | 設定ファイルの基準位置からの相対（プロジェクト設定なら、プロジェクトのルートから） | `Edit(/src/**/*.php)` |
| `~/path` | ホームディレクトリから | `Read(~/.ssh/**)` |
| `//path` | ファイルシステムのルートからの絶対パス | `Read(//etc/passwd)` |

- `/Users/xxx/file` のように先頭に `/` 1つだけで書くと、絶対パスではなく「設定ファイルの基準位置からの相対」になる。絶対パスは `//` で始める
- `Edit` のルールは、ファイルを書き換える組み込みツール全般に効く。`Write` のルールは効かないので `Edit` で書く
- `Read` のdenyは、同じパスの編集・新規作成もブロックする

## 設定ファイルの置き場所

| ファイル | 範囲 |
|---|---|
| `.claude/settings.json` | プロジェクト共通（gitでチームと共有） |
| `.claude/settings.local.json` | 自分だけ・このプロジェクトだけ |
| `~/.claude/settings.json` | 自分だけ・全プロジェクト |

確認ダイアログで「Yes, and don't ask again」を選んだ場合も、`.claude/settings.local.json` に自動で保存される。

- **複数のファイルにまたがっても、denyが常に勝つ。** ユーザー設定でallowしていても、プロジェクト設定でdenyしていれば実行されない。逆も同じ
- 組織の管理者が配る管理設定（managed settings）のルールは、他のどの設定でも上書きできない
- 書式に問題のあるルールは、起動時に警告が表示される。`claude doctor` で、読み込まれた設定と警告を確認できる

## 注意点

- `*` は「その位置に何が来てもよい」という意味。`Bash(git *)` にすると `git reset --hard` なども許可されるので、`Bash(git diff *)` のように範囲を絞る
- **Bashのdenyは万能ではない。** `Bash(git push *)` は `git -C . push` のように書き換えられたコマンドには一致しない。確実に止めたいものは、別の手段（フックやサンドボックス）も検討する
- 迷ったら「読むだけの操作はallow、書き込み・外部送信はask、取り返しがつかない操作はdeny」を目安にする

出典: [Claude Code公式ドキュメント Configure permissions](https://code.claude.com/docs/en/permissions)
