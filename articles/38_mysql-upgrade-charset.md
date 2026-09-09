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

## 良い点・悪い点・具体例

### 良い点

- 絵文字（😀など）や一部の難読漢字といった「4バイト文字」が正しく保存できるようになる。旧来の `utf8`（MySQLの`utf8`は実は3バイトまでしか使えない不完全な実装）では、こうした文字を含む文字列はエラーになったり文字化けしたりしていた。
- グローバル対応のアプリ（多言語対応、SNS投稿機能など）で発生しがちな文字化けトラブルが根本的に減る。

### 悪い点（注意が必要な点）

- 同じ文字列でも1文字あたりの最大バイト数が3→4に増えるため、`VARCHAR(255)`などにインデックスを張っている場合、InnoDBのキー長上限（デフォルト767バイト、`innodb_large_prefix`等が絡む）に抵触しやすくなる。
- 照合順序（collation）が変わることで、`ORDER BY`の並び順や`DISTINCT`の重複判定結果が変わる可能性がある。
- 既存テーブルが旧文字コードのまま、新規テーブルだけutf8mb4という「混在状態」になっていると、JOIN時にエラーになる。

### 具体例

```sql
-- 旧utf8(3バイト)のテーブルに絵文字を入れようとするとエラーになる
CREATE TABLE old_table (name VARCHAR(50) CHARACTER SET utf8);
INSERT INTO old_table (name) VALUES ('こんにちは😀');
-- ERROR 1366 (HY000): Incorrect string value: '\xF0\x9F\x98\x80' for column 'name'

-- utf8mb4なら問題なく入る
CREATE TABLE new_table (name VARCHAR(50) CHARACTER SET utf8mb4);
INSERT INTO new_table (name) VALUES ('こんにちは😀'); -- 成功

-- 文字コードが異なるテーブル同士をJOINするとエラーになる例
SELECT a.*, b.* FROM old_table a JOIN new_table b ON a.name = b.name;
-- ERROR 1267 (HY000): Illegal mix of collations
--   (utf8_general_ci,IMPLICIT) and (utf8mb4_0900_ai_ci,IMPLICIT) for operation '='
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
