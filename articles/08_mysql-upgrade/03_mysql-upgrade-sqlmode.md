# MySQL 5.6→8.4: sql_modeとSQL挙動の厳格化

`sql_mode` のデフォルトは5.6→5.7→8.0で段階的に厳格化されている。5.6時代に動いていたSQLが、アップグレード後にエラーになるケースの多くはここが原因。

| バージョン | デフォルトsql_modeに追加された主なモード |
|---|---|
| 5.6 | （比較的緩い。空文字に近い状態） |
| 5.7.5〜 | `ONLY_FULL_GROUP_BY`, `STRICT_TRANS_TABLES` |
| 5.7.7〜 | `NO_AUTO_CREATE_USER` |
| 5.7.8〜 | `ERROR_FOR_DIVISION_BY_ZERO`, `NO_ZERO_DATE`, `NO_ZERO_IN_DATE`, `NO_ENGINE_SUBSTITUTION` |
| 8.0〜 | `NO_AUTO_CREATE_USER` は**廃止**（GRANTでの暗黙ユーザー作成自体が8.0で禁止されたため不要に） |

## 影響・注意点

- **`ONLY_FULL_GROUP_BY`**: `GROUP BY` している列以外を `SELECT` に含めると（集約関数を使わない限り）エラーになる。5.6ではこれが緩く許容されていたため、既存クエリがそのままだとエラーになりやすい最大の要因。
- **`STRICT_TRANS_TABLES`**: 不正な値をINSERT/UPDATEした際、5.6では警告のみで丸められていた値が、エラーで弾かれるようになる（例: カラム長を超える文字列、NOT NULL列へのNULL挿入など）。
- **`NO_ZERO_DATE` / `NO_ZERO_IN_DATE`**: `0000-00-00` や `2024-00-01` のような不正日付がエラーになる。5.6時代のダミーデータや初期値運用に依存していると要注意。
- **`GROUP BY … ASC/DESC`構文の削除（8.0.13〜）**: `GROUP BY col DESC` のようなソート指定つきGROUP BYは廃止され、`ORDER BY` で明示する必要がある。

## 良い点・悪い点・具体例

### 良い点

- `ONLY_FULL_GROUP_BY` により、「たまたま特定の1行の値が返ってきているだけ」という曖昧なGROUP BYクエリを事前にエラーとして弾ける。本番データが増減した時に集計結果が不安定になる、というバグを未然に防げる。
- `STRICT_TRANS_TABLES` により、桁あふれや不正な値がサイレントに丸められて保存される事故を防げる。データ品質そのものが上がる。

### 悪い点（注意が必要な点）

- 今まで警告だけで動いていたクエリ・バッチ処理が、アップグレード後は突然エラーで止まるようになる。事前にテストしないまま本番へ適用すると、深夜バッチが軒並み失敗するといった障害になりやすい。
- 特に古いORMや、文字列を組み立てて実行する動的SQLで、暗黙的に緩いsql_modeへ依存している場合は影響範囲の洗い出しに時間がかかる。

### 具体例

```sql
-- 5.6ではエラーにならず、department内の「どれか1行」のnameが返っていた(結果が不定)
SELECT department, name, salary FROM employees GROUP BY department;

-- 8.0(ONLY_FULL_GROUP_BY)ではエラーになる
-- ERROR 1055 (42000): 'employees.name' isn't in GROUP BY

-- 正しく直すには、集約関数を使うかGROUP BYの対象列を明確にする
SELECT department, MAX(name) AS name, MAX(salary) AS salary
FROM employees GROUP BY department;
```

```sql
-- STRICT_TRANS_TABLES: カラム長を超える値を入れた場合
CREATE TABLE users (name VARCHAR(5));
INSERT INTO users VALUES ('123456789');
-- 5.6: 警告のみで 'name' は "12345" に切り詰められて保存される（気づきにくいデータ破損）
-- 8.0: ERROR 1406 (22001): Data too long for column 'name' at row 1
```

## 対応方法

いきなり全て厳格化するのではなく、移行期間中は明示的に緩めたsql_modeを設定して段階移行することも可能。

```ini
[mysqld]
sql_mode=STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION
```

ただし恒久対応としては、アプリ側のクエリ・データ登録処理を新しいsql_modeに適合させる方が望ましい。

## 確認コマンド

```sql
SELECT @@GLOBAL.sql_mode;
SELECT @@SESSION.sql_mode;
```

## 出典

- [MySQL 5.7 Reference Manual: Server SQL Modes](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html)
- [MySQL Installation Guide: Changes in MySQL 5.7](https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/upgrading-from-previous-series.html)
- [MySQL 8.0 Reference Manual: Changes in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)
