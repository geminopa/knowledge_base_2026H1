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

## 良い点・悪い点・具体例

### 良い点

- master/slaveという主従関係を連想させる用語から、より中立的な `source`/`replica` に変わったことで、社外向けドキュメントや対外的な説明でも配慮しやすくなった。
- 新コマンド体系（`SHOW REPLICA STATUS`等）は、レプリケーション構成の要素（source, replica, binary log）がコマンド名から直感的に分かりやすくなっている。

### 悪い点（注意が必要な点）

- 既存の運用Runbook、監視スクリプト、Ansible/Terraformなどの構成管理コード、障害対応手順書で `CHANGE MASTER TO` や `SHOW SLAVE STATUS` をハードコードしている箇所が軒並み見直し対象になる。見落とすと、いざという時に「手順書通りのコマンドが通らない」事態になる。
- 監視ツール（Zabbix、Datadog等）のMySQL連携プラグインが古いバージョンだと、新しい`SHOW REPLICA STATUS`の出力フォーマットや新設の`SHOW BINARY LOG STATUS`に対応しておらず、監視項目が取得できなくなることがある。

### 具体例

```sql
-- 障害対応の手順書に書かれていたコマンド(5.6/5.7時代)
CHANGE MASTER TO MASTER_HOST='db-primary', MASTER_USER='repl', MASTER_PASSWORD='xxx';
START SLAVE;
SHOW SLAVE STATUS\G

-- 8.4での書き換え後
CHANGE REPLICATION SOURCE TO SOURCE_HOST='db-primary', SOURCE_USER='repl', SOURCE_PASSWORD='xxx';
START REPLICA;
SHOW REPLICA STATUS\G
```

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
