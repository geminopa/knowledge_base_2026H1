# MySQL 5.6→8.4: その他の運用系パラメータのデフォルト変更

上記の各観点（文字コード・認証・sql_mode・InnoDB・レプリケーション・削除パラメータ）以外にも、地味だが見落としがちなデフォルト変更がある。

| パラメータ | 5.7以前 | 8.0以降 | 影響 |
|---|---|---|---|
| `max_allowed_packet` | 4MB | 64MB | 大きめのパケットを許容するようになる。逆に4MB前提で制限をかけていたアプリの挙動が変わりうる |
| `table_open_cache` | 2000 | 4000 | オープンファイル数（テーブル数×インスタンス数）が増える方向。`open_files_limit`（OS側のファイルディスクリプタ上限）との兼ね合いを確認 |
| `back_log` | `50 + (max_connections/5)` | `max_connections` と同値の自動計算 | 接続待ちキューが広がる |
| `event_scheduler` | OFF | ON | イベントスケジューラが5.6/5.7では意識せず無効だった環境で、8.0以降は既存のイベント定義があれば動き出す可能性がある |
| `log_error_verbosity` | 3（Notes含む） | 2（Warning以上） | エラーログの出力量が減る方向。Notesレベルのログに依存した監視をしている場合は要調整 |
| `local_infile` | ON | OFF | `LOAD DATA LOCAL INFILE` がデフォルトで使えなくなる（セキュリティ強化のため）。バッチ処理などで使っている場合は明示的にONにする必要 |
| `optimizer_trace_max_mem_size` | 16KB | 1MB | オプティマイザトレースのメモリ上限拡大。通常運用への影響は小さい |

## 影響・注意点

- **`event_scheduler` のOFF→ON**は特に見落としやすい。5.6/5.7時代に「作ったが無効化していたイベント」がある場合、8.0以降のデフォルトのまま起動すると意図せず動き出す。アップグレード前に `SHOW EVENTS;` で棚卸ししておく。
- **`local_infile` のON→OFF**は、バッチ処理やデータ移行スクリプトで `LOAD DATA LOCAL INFILE` を使っている場合に影響する。エラーになった場合は、サーバー側・クライアント側両方で `local_infile=1` を明示する必要がある（セキュリティ上のリスクとのトレードオフのため、必要な範囲に限定するのが望ましい）。
- **`table_open_cache` の増加**はメモリ使用量増にも直結するため、コンテナ環境などメモリ制約が厳しい場合は明示指定を検討する。

## 確認コマンド

```sql
SHOW EVENTS;
SHOW VARIABLES LIKE 'event_scheduler';
SHOW VARIABLES LIKE 'local_infile';
SHOW VARIABLES LIKE 'table_open_cache';
```

## 出典

- [MySQL 8.0 Reference Manual: Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)
