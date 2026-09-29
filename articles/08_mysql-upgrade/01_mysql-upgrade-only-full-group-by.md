# MySQL 5.6 と 8.4 の比較: ONLY_FULL_GROUP_BY

5.6では `GROUP BY` に含めていない列を `SELECT` に書いてもエラーにならなかったが、**8.4では `ONLY_FULL_GROUP_BY` が標準で有効**のため、同じSQLがエラーになる。

| | MySQL 5.6 | MySQL 8.4 |
|---|---|---|
| `sql_mode` のデフォルト | `NO_ENGINE_SUBSTITUTION` のみ（※） | `ONLY_FULL_GROUP_BY`, `STRICT_TRANS_TABLES`, `NO_ZERO_IN_DATE`, `NO_ZERO_DATE`, `ERROR_FOR_DIVISION_BY_ZERO`, `NO_ENGINE_SUBSTITUTION` |
| `ONLY_FULL_GROUP_BY` | 指定すれば使えるが、デフォルトでは無効 | デフォルトで有効 |
| 集約されない列の扱い | 任意の1行の値が返る（不定） | 選択リスト・HAVING・ORDER BY のいずれも、GROUP BY に無く関数従属でもない列はエラー 1055 |
| 関数従属性の判定 | 5.7.5より前は検出されない（主キーで `GROUP BY` しても他列を書くと有効時はエラー） | 検出される。GROUP BY列から一意に決まる列は通る |

※ 5.6のサーバ本体の標準値。新規インストール時に作られる設定ファイルの雛形には `sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES` が書かれているため、環境によって異なる。

## 具体例

```sql
-- 5.6では通る（name は不定の1行が返る）。8.4ではエラー 1055
SELECT dept_id, name, COUNT(*) FROM members GROUP BY dept_id;
```

直し方は2通り。

```sql
-- 1. GROUP BY に列を足す（意味が変わらないか確認）
SELECT dept_id, name, COUNT(*) FROM members GROUP BY dept_id, name;

-- 2. 集約する（意図が最大値なら MAX など）
SELECT dept_id, MAX(name), COUNT(*) FROM members GROUP BY dept_id;
```

## 注意点・コツ

- **`sql_mode` を外して回避するのは最終手段。** 5.6時代の「不定の値が返る」挙動に依存したSQLは、もともと結果が保証されていない。エラーが出た機会に直すのが安全
- `ORDER BY` や `HAVING` に集約されない列を使っている場合も同じエラーになる
- 主キー（または一意キー）で `GROUP BY` している場合は、他の列を `SELECT` してもエラーにならない（関数従属性の判定。5.7.5以降。公式ドキュメントの定義は8.4で確認済み）
- 影響範囲は、アプリ内のSQLを `GROUP BY` で検索して洗い出す。PHPのコード内のSQL文字列も対象
- 現在の `sql_mode` は `SELECT @@GLOBAL.sql_mode;` で確認できる

## 出典

- [MySQL 8.4 Reference Manual: Server SQL Modes](https://dev.mysql.com/doc/refman/8.4/en/sql-mode.html)（8.4の標準値、`ONLY_FULL_GROUP_BY` の定義）
- 5.6の標準値: [MySQL 5.6 Release Notes 5.6.8](https://docs.oracle.com/cd/E17952_01/mysql-5.6-relnotes-en/news-5-6-8.html)、[5.6 Reference Manual: Default Configuration File](https://docs.oracle.com/cd/E17952_01/mysql-5.6-en/server-default-configuration-file.html)
