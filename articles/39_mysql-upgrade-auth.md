# MySQL 5.6→8.4: 認証プラグイン・認証方式の変更

MySQL 5.6/5.7ではデフォルト認証プラグインは `mysql_native_password` だったが、段階的に変更されている。

| バージョン | デフォルト認証プラグイン |
|---|---|
| 5.6 / 5.7 | `mysql_native_password` |
| 8.0.0〜8.0.3 | `mysql_native_password` |
| 8.0.4以降 | `caching_sha2_password` |
| 8.4 | `caching_sha2_password`（`default_authentication_plugin` は非推奨。`authentication_policy` に統合） |

## 影響・注意点

- **`caching_sha2_password` はSSL/TLS接続が前提**（非SSL接続では初回認証時にRSA公開鍵の交換が必要）。クライアントライブラリが古い場合、この認証方式に対応しておらず接続エラーになることがある（特に古いPHPのmysqlnd、古いJDBCドライバなど）。
- MySQL 8.4では `default_authentication_plugin` システム変数自体が非推奨/削除されており、代わりに **`authentication_policy`** で認証方式のポリシー（優先順位・必須/任意）を指定する方式に変わっている。5.6/5.7からの設定ファイルをそのまま流用すると、この変数の指定が無効になる点に注意。
- 既存ユーザーの認証方式は、アップグレードしても自動では変わらない（作成時のプラグインのまま）。新規ユーザー作成時のデフォルトが変わるという点がポイント。

## 対応方法の例

```sql
-- 既存ユーザーをcaching_sha2_passwordに切り替える
ALTER USER 'app_user'@'%' IDENTIFIED WITH caching_sha2_password BY 'new_password';

-- 現在の認証プラグインを確認
SELECT user, host, plugin FROM mysql.user;
```

```ini
# MySQL 8.4でmysql_native_passwordを引き続き使いたい場合(非推奨だが移行期間の緩和策として)
[mysqld]
authentication_policy=mysql_native_password,,
```

## 確認コマンド

```sql
SHOW VARIABLES LIKE 'authentication_policy';
SHOW VARIABLES LIKE 'default_authentication_plugin';
```

## 出典

- [MySQL 8.0 Reference Manual: Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)
- [MySQL Server Version Reference: Option and Variable Changes in MySQL 8.4](https://dev.mysql.com/doc/mysqld-version-reference/en/optvar-changes-8-4.html)
