# MySQL 5.6→8.4: レプリケーション用語・コマンドの刷新

MySQL 8.0.22以降、レプリケーション関連の用語が `master`/`slave` から `source`/`replica` に置き換えられている。5.6/5.7時代のSQL・スクリプト・運用手順書をそのまま使っていると、コマンドが実行できなくなる。

| 5.6/5.7 | 8.4での名称 |
|---|---|
| `CHANGE MASTER TO` | `CHANGE REPLICATION SOURCE TO` |
| `SHOW MASTER STATUS` | `SHOW BINARY LOG STATUS` |
| `RESET MASTER` | `RESET BINARY LOGS AND GTIDS` |
| `START SLAVE` | `START REPLICA` |
| `STOP SLAVE` | `STOP REPLICA` |
| `SHOW SLAVE STATUS` | `SHOW REPLICA STATUS` |
| `master_info_repository` | `source_info_repository`（変数名も変更） |
| `slave_pending_jobs_size_max` | `replica_pending_jobs_size_max` |

## 影響・注意点

- 旧コマンド（`CHANGE MASTER TO` 等）はMySQL 8.4時点でも**エイリアスとして動作はする**ものが多いが、非推奨扱いであり将来のバージョンで削除される可能性がある。運用スクリプト・手順書・監視ツールの設定は新用語に更新しておくのが望ましい。
- **`expire_logs_days` は廃止され、`binlog_expire_logs_seconds` に統合**された。秒単位指定に変わっている点に注意（`expire_logs_days=7` → `binlog_expire_logs_seconds=604800`）。
- **`log_bin`（バイナリログ）のデフォルトが8.0以降ON**になっている。5.6/5.7でレプリケーションを使っておらずバイナリログも明示的にOFFにしていなかった環境では、アップグレード後にバイナリログが有効化され、ディスク使用量が増える可能性がある。
- MySQL 8.4ではGTIDに「タグ」を付与できる新フォーマット（`UUID:TAG:NUMBER`）が追加されている。既存のGTID運用への影響は基本的にないが、新機能として把握しておくとよい。

## 確認コマンド

```sql
SHOW REPLICA STATUS\G
SHOW BINARY LOG STATUS;
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'binlog_expire_logs_seconds';
```

## 出典

- [MySQL Server Version Reference: Option and Variable Changes in MySQL 8.4](https://dev.mysql.com/doc/mysqld-version-reference/en/optvar-changes-8-4.html)
- [MySQL 8.0 Reference Manual: Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)
