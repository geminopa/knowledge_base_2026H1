# MySQL 5.6 と 8.4 の比較: 文字セットと照合順序

5.6のデフォルトは `latin1`、8.4は `utf8mb4`。照合順序も `latin1_swedish_ci` から `utf8mb4_0900_ai_ci` に変わり、**新規作成するDB・テーブルの標準と、文字列の比較結果**が変わる。

| | MySQL 5.6 | MySQL 8.4 |
|---|---|---|
| `character_set_server` | `latin1` | `utf8mb4` |
| `collation_server` | `latin1_swedish_ci` | `utf8mb4_0900_ai_ci` |
| `utf8` の指す文字セット | `utf8mb3`（最大3バイト） | `utf8mb3` の別名のまま。非推奨で、将来のメジャーリリースで削除予定（8.0・8.4の間は使える） |
| 末尾スペースの比較 | 無視される（PAD SPACE） | `0900` 系は区別する（NO PAD）。`general_ci` などの旧来の照合順序は従来どおり PAD SPACE |
| インデックスの長さ上限 | 767バイト（`DYNAMIC` 形式＋`innodb_large_prefix` 有効時は3072） | 3072バイト（`DYNAMIC` 形式。`innodb_large_prefix` は削除済み） |

## 具体例

```sql
-- 現在の設定を確認する
SHOW VARIABLES LIKE 'character_set_%';
SHOW VARIABLES LIKE 'collation_%';

-- 既存テーブルの文字セットを確認する
SELECT table_name, table_collation
FROM information_schema.tables
WHERE table_schema = 'your_db';
```

照合順序が異なる列同士を比較すると、次のエラーになる。

```
ERROR 1267 (HY000): Illegal mix of collations (utf8mb4_general_ci,IMPLICIT)
and (utf8mb4_0900_ai_ci,IMPLICIT) for operation '='
```

PHP側では、接続時に文字セットを明示する。

```php
$pdo = new PDO('mysql:host=localhost;dbname=app;charset=utf8mb4', $user, $pass);
```

## 注意点・コツ

- **既存のDB・テーブルの文字セットは、アップグレードしても自動では変わらない。** 変わるのは、移行後に新規作成するものの標準と、`my.cnf` に指定がなかったサーバ設定
- `utf8` と書くと8.4でも3バイト版のまま。絵文字などの4バイト文字を扱うなら `utf8mb4` と明記する
- `utf8mb4_0900_ai_ci` と `utf8mb4_general_ci` は比較結果が変わりうる。具体的な違いは次の節を参照
- 移行前後で新旧テーブルが混在する場合は、`COLLATE` を明示するか、テーブルを変換して揃える: `ALTER TABLE t CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;`
- 旧来の挙動に合わせたい場合は、`my.cnf` で `collation_server=utf8mb4_general_ci` を指定する

## 0900_ai_ci と general_ci の比較結果の違い

どちらも `_ci`（大文字小文字を区別しない）だが、文字の比較ルールの世代が違う。`general_ci` は古い独自ルールで、1文字ずつの単純な比較しかできない。`0900_ai_ci` はUnicode 9.0の標準ルール（UCA 9.0.0）に沿っている。

| 比較する対象 | `utf8mb4_general_ci` | `utf8mb4_0900_ai_ci` |
|---|---|---|
| `ß`（ドイツ語） と `s` | 同じ（`ß = s`） | 別（`ß = ss`） |
| 絵文字などの4バイト文字 | 全部同じ重みとして扱われ、**別の絵文字が等しいと判定される** | 文字ごとに別の重みを持ち、区別される |
| 末尾スペース（`'a '` と `'a'`） | 同じ | 別 |
| 展開・縮約・無視される文字 | 非対応 | 対応 |
| 速度 | 最も速い | UCA系の中では最も速い |

※ 絵文字の扱いは公式で「補助文字（supplementary characters）は U+FFFD の重みになる」と説明されている。`utf8mb4_unicode_ci` も同じ。

## 影響が出る場所

- **UNIQUE制約・主キー:** 「等しい」の基準が変わるので、重複と判定される範囲が変わる。
  - `general_ci` → `0900_ai_ci` に変換すると、`'ß'` と `'ss'` のように、これまで別の値だったものが同じ値になり、変換時に `Duplicate entry` で失敗しうる。
  - 逆に `0900_ai_ci` では、`'a'` と `'a '`（末尾スペース付き）は別の値として登録できる。入力のtrimをDB任せにしていたシステムでは、見た目が同じ重複データが入りうる。
- **WHERE・JOIN:** 末尾スペース付きのデータは、`general_ci` では一致していたのに `0900_ai_ci` では一致しなくなる。CHAR列に手入力された値が混ざっているデータで起きやすい。
- **ORDER BY・GROUP BY:** 並び順と、同じ値としてまとめられる範囲が変わる。並び順に依存した画面や、ページング・CSV出力の結果が変わりうる。
- **日本語のひらがな・カタカナ:** 公式で確認できたのは、日本語向けの `utf8mb4_ja_0900_as_cs`（かなを区別しない）と `utf8mb4_ja_0900_as_cs_ks`（区別する）の違いまで。`0900_ai_ci` でのかなの扱いは、次のSQLで実際に確認する。

## 変更前にSQLで確かめる

```sql
-- 同じ文字列でも、照合順序によって比較結果が変わる
SELECT
  'ß'  COLLATE utf8mb4_general_ci  = 's'    AS eszett_general,   -- 1
  'ß'  COLLATE utf8mb4_0900_ai_ci  = 's'    AS eszett_0900,      -- 0
  'a ' COLLATE utf8mb4_general_ci  = 'a'    AS space_general,    -- 1
  'a ' COLLATE utf8mb4_0900_ai_ci  = 'a'    AS space_0900,       -- 0
  'あ' COLLATE utf8mb4_0900_ai_ci  = 'ア'   AS kana_0900;        -- 環境で確認
```

現行のデータで重複が起きないかは、変換前に次のように探す。

```sql
-- 変更後の照合順序で、同じ値になってしまう組み合わせを探す
SELECT name COLLATE utf8mb4_0900_ai_ci AS name_new, COUNT(*)
FROM members
GROUP BY name_new
HAVING COUNT(*) > 1;
```

- 結果が0件なら、UNIQUE列を変換しても重複エラーにならない。1件以上あれば、その行を先に整理する。
- 本番と同じデータを入れた検証環境で、`ORDER BY` を使う主要な画面の並び順も見比べる。

## 出典

- [8.4: Server Character Set and Collation](https://dev.mysql.com/doc/refman/8.4/en/charset-server.html)
- [8.4: The utf8mb3 Character Set](https://dev.mysql.com/doc/refman/8.4/en/charset-unicode-utf8mb3.html)
- [8.4: Unicode Character Sets](https://dev.mysql.com/doc/refman/8.4/en/charset-unicode-sets.html)（`general_ci` / `unicode_ci` / `0900_ai_ci` の比較、`ß`、補助文字、日本語用照合順序）
- [8.4: Binary Collations / Pad Attribute](https://dev.mysql.com/doc/refman/8.4/en/charset-binary-collations.html)
- [8.0: What Is New in MySQL 8.0](https://dev.mysql.com/doc/refman/8.0/en/mysql-nutshell.html)（`latin1` から `utf8mb4` への変更）
- [InnoDB Limits（5.7 / 8.4）](https://dev.mysql.com/doc/refman/8.4/en/innodb-limits.html)（インデックスの長さ上限）
