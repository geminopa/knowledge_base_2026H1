# MySQL 5.6 と 8.4 の比較: my.cnf の設定

5.6用の `my.cnf`（XAMPPでは `my.ini`）をそのまま8.4に持ち込むと、**削除された変数があるとサーバが起動しない**。移行前に、設定ファイルの項目を洗い出しておく。

| 5.6の設定 | 8.4での扱い | 対応 |
|---|---|---|
| `query_cache_type` / `query_cache_size` | クエリキャッシュ自体が8.0で削除。認識されない変数のため起動に失敗する | 削除する |
| `sql_mode` に `NO_AUTO_CREATE_USER` | 8.0で削除された値（旧来の動作が標準になった）。指定するとエラー | 値から外す |
| `tx_isolation` | 削除 | `transaction_isolation` に書き換える |
| `innodb_file_format` / `innodb_large_prefix` | 削除（8.0以降は常に大きいプレフィックス対応） | 削除する |
| `expire_logs_days` | 8.0で非推奨、8.4で削除 | `binlog_expire_logs_seconds` に書き換える |
| `log_warnings` | 8.0で削除 | `log_error_verbosity` に置き換える |
| `default_authentication_plugin` | 8.0.27で非推奨、8.4で削除 | `authentication_policy` を使う（書式が変わっている） |
| `mysql_native_password` を使うユーザー | 8.4は標準で無効 | 使い続けるなら `mysql_native_password=ON`。推奨は `caching_sha2_password` への移行 |

## 具体例

```ini
[mysqld]
# 5.6の設定（8.4では起動しない）
# query_cache_type = 1
# query_cache_size = 64M
# tx_isolation = READ-COMMITTED
# expire_logs_days = 7

# 8.4向けの書き換え
transaction_isolation = READ-COMMITTED
binlog_expire_logs_seconds = 604800   # 7日
character_set_server = utf8mb4
collation_server = utf8mb4_general_ci  # 旧来の並び順を維持したい場合
```

起動できるかは、本番前に検証環境で確認する。公式は、アップグレード前に MySQL Shell の Upgrade Checker Utility（`util.checkForServerUpgrade()`）で互換性を確認することを推奨している。

## 注意点・コツ

- **PHPの接続は認証方式に注意。** 8.4の標準は `caching_sha2_password`。PHP 7.4未満のmysqlndは非対応で、接続に失敗する。PHPのバージョンも合わせて確認する
- **5.6から8.4へは直接上げられない。** 8.4のドキュメントでは 5.7 → 8.0 → 8.4 の経路が示されており、LTSシリーズは飛ばせない。5.6は 5.7 を経由する必要がある
- 8.0以降は `binlog_format` の標準が `ROW`、`explicit_defaults_for_timestamp` が有効になるなど、**書いていない設定の標準値**も変わる
- 各項目の詳細は公式ドキュメントの「What Is New in MySQL 8.4」「Removed」の一覧で最終確認する

## 出典

- [MySQL 8.4 Reference Manual: What Is New in MySQL 8.4](https://dev.mysql.com/doc/refman/8.4/en/mysql-nutshell.html)（`expire_logs_days` / `default_authentication_plugin` の削除、`mysql_native_password` の既定無効）
- [MySQL 8.0 Reference Manual: What Is New in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)（クエリキャッシュ、`tx_isolation`、`innodb_file_format`、`innodb_large_prefix`、`log_warnings`、`NO_AUTO_CREATE_USER` の削除）
- [MySQL 8.4 Reference Manual: Upgrade Paths](https://dev.mysql.com/doc/refman/8.4/en/upgrade-paths.html)
