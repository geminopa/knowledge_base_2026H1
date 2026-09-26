# MySQL 5.6 と 8.4 の比較: ZERO DATE（0000-00-00）問題

5.6では `0000-00-00` を日付として保存できたが、8.4では標準の `sql_mode` が `NO_ZERO_DATE` / `NO_ZERO_IN_DATE` を含むため、**INSERT・UPDATEや、テーブル定義でエラー**になる。

| | MySQL 5.6 | MySQL 8.4 |
|---|---|---|
| `NO_ZERO_DATE` / `NO_ZERO_IN_DATE` | デフォルトでは無効 | デフォルトで有効（ただし非推奨） |
| strictモードとの関係 | — | 厳格モード（`STRICT_TRANS_TABLES` 等）と組み合わせたときに拒否される。単独では警告のみで通る |
| `INSERT ... '0000-00-00'` | 通る | エラー（strictモード時） |
| `DEFAULT '0000-00-00'` を含む `CREATE/ALTER TABLE` | 通る | エラー |
| 既存データの `SELECT` | 読める | 読める |
| `explicit_defaults_for_timestamp` | `OFF` | `ON`（TIMESTAMP列の暗黙の動作が変わる） |

## 具体例

移行後に起きやすいのは、既存データが原因の**後からのエラー**。

```sql
-- 既存の 0000-00-00 が残っていると、テーブルの再構築を伴う ALTER で失敗することがある
ALTER TABLE orders ADD COLUMN memo VARCHAR(100);
-- ERROR 1292 (22007): Incorrect date value: '0000-00-00' for column ...
```

対処は、ゼロ日付を `NULL` に置き換える（列が `NULL` 許可であること）。

```sql
-- 該当データの確認
SELECT COUNT(*) FROM orders WHERE shipped_at = '0000-00-00';

-- NULL に置換（列定義も DEFAULT NULL にしておく）
ALTER TABLE orders MODIFY shipped_at DATE NULL DEFAULT NULL;
UPDATE orders SET shipped_at = NULL WHERE shipped_at = '0000-00-00';
```

PHP側も見直す。

```php
// 5.6時代: 未設定の日付として '0000-00-00' を入れていた
$stmt->execute([':shipped_at' => '0000-00-00']);   // 8.4ではエラー

// 8.4: 未設定は NULL
$stmt->execute([':shipped_at' => null]);
```

## 注意点・コツ

- **ゼロ日付の有無は、移行前に5.6側で洗い出す。** 日付列ごとに `WHERE col = '0000-00-00'` で件数を確認する。`0000-00-00` だけでなく、`2024-00-10` のような月日が0の値（`NO_ZERO_IN_DATE` の対象）もある
- PHPで `'0000-00-00'` を `strtotime()` や `DateTime` に渡すと、意図しない日付になる。`NULL` にしておくとPHP側の判定も単純になる
- `mysqldump` で書き出したデータを戻すときも、ゼロ日付が原因で失敗することがある
- `sql_mode` から `NO_ZERO_DATE` を外せば通るが、この2つは非推奨。公式は「将来、別のモード名としては削除され、厳格モードの効果に含まれる」としている。データ側の修正を優先する

## 出典

- [MySQL 8.4 Reference Manual: Server SQL Modes](https://dev.mysql.com/doc/refman/8.4/en/sql-mode.html)（`NO_ZERO_DATE` / `NO_ZERO_IN_DATE` の非推奨と厳格モードとの関係）
