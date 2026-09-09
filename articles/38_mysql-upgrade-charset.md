# MySQL 5.6→8.4: 文字コード・照合順序のデフォルト変更

MySQL 5.6/5.7では `character_set_server` のデフォルトは `latin1` だったが、**MySQL 8.0以降は `utf8mb4` がデフォルト**になった。照合順序（collation）も `latin1_swedish_ci` から `utf8mb4_0900_ai_ci` に変わっている。

| パラメータ | 5.7以前のデフォルト | 8.0以降のデフォルト |
|---|---|---|
| `character_set_server` | `latin1` | `utf8mb4` |
| `collation_server` | `latin1_swedish_ci` | `utf8mb4_0900_ai_ci` |

## 影響・注意点

- 設定ファイルで明示的に `character_set_server` / `collation_server` を指定していない環境は、アップグレード後にDB作成時の暗黙のデフォルトが変わる。**既存DBの文字コードが変わるわけではない**が、新規作成するDB/テーブルのデフォルトが変わる点に注意。
- `utf8mb4_0900_ai_ci` はUnicode 9.0ベースの新しい照合順序で、5.7以前の `utf8mb4_general_ci` や `utf8mb4_unicode_ci` とはソート順・比較結果が異なるケースがある。JOINやORDER BYの結果が変わりうるため、アプリ側で照合順序に依存した処理がないか要確認。
- 明示的に旧来の挙動を維持したい場合は、`my.cnf` に以下を指定する。
  ```ini
  [mysqld]
  character_set_server=utf8mb4
  collation_server=utf8mb4_general_ci
  ```

## 確認コマンド

```sql
SHOW VARIABLES LIKE 'character_set_server';
SHOW VARIABLES LIKE 'collation_server';
SELECT TABLE_SCHEMA, TABLE_NAME, TABLE_COLLATION
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA NOT IN ('mysql','information_schema','performance_schema','sys');
```

## 出典

- [MySQL 8.0 Reference Manual: Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)
