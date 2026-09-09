# MySQL 5.6→8.4: InnoDB関連パラメータのデフォルト値変化

InnoDBまわりのパラメータは5.7→8.0、8.0→8.4のそれぞれでデフォルト値が大きく変わっている。性能・チューニングに直結するため、既存my.cnfのパラメータを外していると意図せず挙動が変わる。

## 5.7→8.0の変更

| パラメータ | 5.7デフォルト | 8.0デフォルト |
|---|---|---|
| `innodb_undo_tablespaces` | 0 | 2 |
| `innodb_autoinc_lock_mode` | 1（consecutive） | 2（interleaved） |
| `innodb_flush_neighbors` | 1（有効） | 0（無効） |
| `innodb_max_dirty_pages_pct_lwm` | 0% | 10% |
| `innodb_max_dirty_pages_pct` | 75% | 90% |

## 8.0→8.4の変更

| パラメータ | 8.0デフォルト | 8.4デフォルト |
|---|---|---|
| `innodb_adaptive_hash_index` | ON | OFF |
| `innodb_change_buffering` | all | none |
| `innodb_flush_method`（Linux） | fsync | O_DIRECT（対応時） |
| `innodb_io_capacity` | 200 | 10000 |
| `innodb_log_buffer_size` | 16MiB | 64MiB |
| `innodb_buffer_pool_instances` | 8（固定的） | CPU数・バッファプールサイズに応じた自動計算 |
| `innodb_doublewrite_files` | `innodb_buffer_pool_instances * 2` | 2 |
| `innodb_purge_threads` | 4 | CPU数≦16なら1、それ以外は4 |

## 影響・注意点

- **`innodb_autoinc_lock_mode` の1→2変更**: AUTO_INCREMENT採番の挙動が変わり、複数セッション同時INSERT時の採番順が変わりうる。連番の連続性に依存したロジックがある場合は要確認。
- **`innodb_io_capacity` の 200→10000**: ストレージのIOPS性能を前提にした値。旧来のHDD/低性能ディスク環境のまま8.4に上げると、フラッシュ処理が過剰にディスクへ負荷をかける可能性がある。逆にSSD/NVMe環境では性能向上が期待できる。自環境のストレージ性能に応じて明示指定を検討する。
- **`innodb_log_buffer_size` の16MB→64MB**: 大きめのトランザクションを扱う場合のログバッファ不足によるフラッシュ頻発が緩和される方向の変更。メモリ使用量は増える。
- 明示的に5.7/8.0相当の挙動を維持したい場合は、my.cnfで該当パラメータを固定値指定する。

## 確認コマンド

```sql
SHOW VARIABLES LIKE 'innodb_%' ;
SELECT variable_name, variable_source, variable_value
FROM performance_schema.variables_info
WHERE variable_name LIKE 'innodb_%';
```

## 出典

- [MySQL 8.0 Reference Manual: Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)
- [MySQL Server Version Reference: Option and Variable Changes in MySQL 8.4](https://dev.mysql.com/doc/mysqld-version-reference/en/optvar-changes-8-4.html)
