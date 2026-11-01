# Excel Tips

## シート名をセルから参照

```excel
=INDIRECT(A1&"!A2")
```

1. 通常の別シート参照は `"Sheet1!A2"` です。
2. `INDIRECT` はセルの参照を文字列で指定する関数なので、`INDIRECT("Sheet1!A2")` と記述できます。
3. 文字列で指定するため、別セルの値との文字列結合が使えます。その結果、`INDIRECT(A1&"!A2")` と指定できます。

## 右からFIND

```excel
=FIND("★", SUBSTITUTE(A1, "/", "★", LEN(A1)-LEN(SUBSTITUTE(A1, "/", ""))))
```

1. 例えば A1 セルが `"/aaa/bbb/ccc/ddd/eee"` で、一番右の `"/"` を検索する場合を考えます。
2. `LEN(A1)` = 20文字、`LEN(SUBSTITUTE(A1, "/", ""))` = 15文字 ... つまり `"/"` が5つ存在します。
3. `SUBSTITUTE` の「置換対象の指定」を利用して、5つ目の `"/"` だけを存在しない文字（例: `"★"`）へ置き換えます。
   - 結果: `"/aaa/bbb/ccc/ddd★eee"`
   - ※上記では `"★"` ですが、TAB（`CHAR(9)`）などでも可能です（NULL文字 `CHAR(0)` は動作しません）。
4. 置き換えた文字を `FIND` で検索すれば、実質的に右から検索したことと同義になります。

## ファイルパスをパスとファイル名へ分割

**フォルダパスの取得:**

```excel
=LEFT(A1, FIND(CHAR(9), SUBSTITUTE(A1, "/", CHAR(9), LEN(A1)-LEN(SUBSTITUTE(A1, "/", ""))), 1)-1)
```

1. パスは右からFINDする応用です。`"/"` を右から探して、その直前までの文字列を `LEFT` で取得します。

**ファイル名の取得（パターン1）:**

```excel
=MID(SUBSTITUTE(A1, LEFT(A1, FIND(CHAR(9), SUBSTITUTE(A1, "/", CHAR(9), LEN(A1)-LEN(SUBSTITUTE(A1, "/", ""))), 1)-1), ""), 2, LEN(A1))
```

1. パスを取得する `LEFT(...)` までは上と同じです。
2. `SUBSTITUTE(A1, パス, "")` でパス部分を削ります。
3. `MID(残り, 2, LEN(A1))` でファイル名部分を取り出します。

**ファイル名の取得（パターン2）:**

```excel
=RIGHT(A1, LEN(A1)-FIND(CHAR(9), SUBSTITUTE(A1, "/", CHAR(9), LEN(A1)-LEN(SUBSTITUTE(A1, "/", ""))), 1))
```

1. 最後の `"/"` を探すところまでは同じです。
2. 探した `"/"` より右側（全体の長さ - 最後の `"/"` の位置）を `RIGHT` で取得します。

## 表の中を条件に当てはまる行を検索してその値を取得

```excel
=INDEX(A1:A10, MATCH("xxx", B1:B10, 0))
=INDEX(A1:K11, MATCH("COLUMN", A1:A11, 0), MATCH("ROW", A1:K1, 0))
```

`VLOOKUP` は検索列が必ず左端に無ければ使用できないため、列の配置に制約があります。
上記のように `INDEX` と `MATCH` を組み合わせると、`MATCH()` で一致した位置（行・列）を取得できるため、列位置の制約を受けません。

<!-- spell-checker: words VLOOKUP -->

## Array formulas

> 配列数式

入力確定時に以下のショートカットキーを押します：
`Shift` + `Ctrl` + `Enter`

### `SUM` / `MIN` / `MAX`

```excel
{=SUM(A1:A10*B1:B10)}
{=MIN(IF(A1:A10="xxx", B1:B10*C1:C10))}
```

### `AND` / `OR`

配列数式内では `AND`, `OR` 関数をそのまま使用することはできません（配列全体を評価してしまうため）。論理積・論理和演算や代替関数を使用します。

```excel
{=MIN(IF(ISBLANK(B1:F1), COLUMN(B1:F1), ""))}
=IF(COUNTIF($C:$C, $G2), INDEX(B:B, MAX(IF($C$2:$C$999=$G2, ROW($C$2:$C$999)))))
=INDEX(A1:A99, MATCH(TRUE, A1:A99<>0, 0))
```

<!-- spell-checker: words ISBLANK COUNTIF -->