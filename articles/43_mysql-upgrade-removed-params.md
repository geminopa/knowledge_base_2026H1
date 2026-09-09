# MySQL 5.6→8.4: 廃止・削除された主要パラメータ一覧

5.6時代のmy.cnfをそのまま8.4に持ち込むと、**存在しないパラメータの指定でmysqldが起動できなくなる**ことがある。事前にどのパラメータが消えているかを洗い出しておく。

## 8.0で削除された主なもの

| パラメータ/機能 | 内容 |
|---|---|
| `query_cache_type` / `query_cache_size` 等 | クエリキャッシュ機能自体が8.0で完全削除 |
| `innodb_file_format` / `innodb_file_format_max` | 8.0では常にBarracudaフォーマット相当となり、変数自体廃止 |
| `old_passwords` | pre-4.1ハッシュ関連。8.0で削除 |
| `secure_auth` | 5.7でno-op化されたのち8.0で削除 |
| `NO_AUTO_CREATE_USER`（sql_modeの値） | GRANTでの暗黙ユーザー作成自体が禁止されたため不要に |
| 互換系sql_mode（`DB2`, `MSSQL`, `ORACLE`, `POSTGRESQL`, `MYSQL323`, `MYSQL40` 等） | 8.0で削除 |
| `INNODB_SYS_*` 系INFORMATION_SCHEMAビュー | `INNODB_*`（SYS無し）に名称統合 |

## 8.4で削除・非推奨化された主なもの

| パラメータ/機能 | 内容 |
|---|---|
| `expire_logs_days` | `binlog_expire_logs_seconds` に統合（秒指定に変更） |
| `--master-info-file` / `--relay-log-info-file` | source/replica用語への刷新に伴い削除 |
| `binlog_transaction_dependency_tracking` | 削除 |
| `default_authentication_plugin` | `authentication_policy` に統合（非推奨） |
| `WAIT_UNTIL_SQL_THREAD_AFTER_GTIDS()` 関数 | 削除。`WAIT_FOR_EXECUTED_GTID_SET()` を使用 |
| `avoid_temporal_upgrade` / `show_old_temporals` | 5.6時代の日時型アップグレード関連。8.4で削除 |
| `--no-dd-upgrade` | `--upgrade=NONE` に統合 |

## 対応方法

アップグレード前に、現行my.cnfで指定しているパラメータが移行先バージョンで有効かどうかを機械的にチェックする。

```bash
# 現行設定の一覧を出力しておき、各パラメータを新バージョンのドキュメントと突き合わせる
mysqld --verbose --help 2>/dev/null | head -n 1
mysqld --print-defaults
```

**Percona Toolkitの`pt-upgrade`や、MySQL公式の`mysqlcheck --check-upgrade`、8.0以降であれば`mysql_upgrade`相当の内部チェック（8.0.16以降は起動時に自動実行）を併用して、起動前に潰しておくのが安全。**

## 出典

- [MySQL 8.0 Reference Manual: Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)
- [MySQL Server Version Reference: Option and Variable Changes in MySQL 8.4](https://dev.mysql.com/doc/mysqld-version-reference/en/optvar-changes-8-4.html)
