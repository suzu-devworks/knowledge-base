# SQL Server Tips of Indexes

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Various Indices](#various-indices)
  - [Clustered Index](#clustered-index)
  - [NonClustered Index](#nonclustered-index)
  - [Unique Index](#unique-index)
  - [Complex Index](#complex-index)
  - [Included Columns in Indexes](#included-columns-in-indexes)
  - [Filtered Indexes](#filtered-indexes)
  - [Partitioned Indexes](#partitioned-indexes)
- [Tips](#tips)
  - [Table Structure](#table-structure)
  - [Checking Index Effectiveness](#checking-index-effectiveness)
  - [Index Fragmentation and Remediation](#index-fragmentation-and-remediation)

## Overview

効率的なインデックスのデザインは、データベースとアプリケーションの高パフォーマンスを実現するための最優先事項である。

## Various Indices

### Clustered Index

- 基本的にはBツリー構造で実装される
- リーフページ＝データ行となる
- キー値によって並べ替えて格納されている
- 1つのテーブルに設定できるクラスタ化インデックスは1つだけである
- BETWEEN、>、>=、<、<= などの演算子を使用した、範囲検索で効率的に動作する
- ORDER BY 句または GROUP BY 句になる場合、効率的に動作する
- 一意となる列、もしくはID列をインデックスにした場合、効率的に動作する
- 頻繁に更新される列をインデックスにした場合、行全体を移動させる必要があるので非効率である

```sql
CREATE [UNIQUE] CLUSTERED INDEX <Index Name>
    ON <スキーマ名>.<テーブル名> (<列名> [ASC|DESC], ...)
    WITH (OPTION, ...)
    ON [PRIMARY]
```

### NonClustered Index

- Bツリー構造で実装される
- リーフ行に行ロケーター（Row Identifier: RID）が格納されている
- キー値によって並べ替えて格納されている
- text, ntext, image, varchar(max), nvarchar(max), varbinary(max) は指定できない
- 1つのテーブルに設定できる非クラスタ化インデックスは 999個まで作成できる
- 範囲検索となる場合、該当列がインデックスに含まれない場合、非効率な動作結果となる
- JOIN、GROUP BY を行う列に効率的である

```sql
CREATE [UNIQUE] NONCLUSTERED INDEX <Index Name>
    ON <スキーマ名>.<テーブル名> (<列名> [ASC|DESC], ...)
    WITH (OPTION, ...)
    ON [PRIMARY]
```

### Unique Index

一意インデックスを使用すると、インデックス キーの値が重複することがないので、テーブルのすべての行を一意にすることができます。

- NULL値も1つの値とみなされるためNULL値を複数行もつ列には設定できない

### Complex Index

テーブルに範囲検索をかける場合、インデックス検索から行ロケータによる行識別子参照（RID Lookup）が行われるため、実際にデータを持たない非クラスタ化インデックスではパフォーマンスが低下する場合がある。条件となる列やSELECTの選択リストを非クラスタ化インデックスに含めることで、行識別子参照が発生しなくなる。

- 複合インデックスに含めることができる列数は最大16列まで
- インデックスのサイズは900バイトまで

### Included Columns in Indexes

付加列と呼ばれる非キー列は、非クラスター化インデックスのリーフ レベルに追加でき、クエリに対応することによりクエリ パフォーマンスを向上できる。

- 非キー列は、非クラスター化インデックスにのみ定義できる
- 複合インデックスの最大16列、900バイトまでの制限を受けない
- text, ntext, image を除いたすべてのデータ型を使用できる

```sql
CREATE [UNIQUE] NONCLUSTERED INDEX <Index Name>
    ON <スキーマ名>.<テーブル名> (<列名> [ASC|DESC], ...)
    INCLUDE (<列名>, ...)
    WITH (OPTION, ...)
    ON [PRIMARY]
```

### Filtered Indexes

フィルター選択されたインデックスは、最適化された非クラスター化インデックスであり、テーブルから選択する行の少ないクエリに適している。NULL値などあらかじめ検索対象とならない列がある場合にストレージを節約しパフォーマンスを向上できる。

```sql
CREATE [UNIQUE] NONCLUSTERED INDEX <Index Name>
    ON <スキーマ名>.<テーブル名> (<列名> [ASC|DESC], ...)
    WHERE <where条件>
    WITH (OPTION, ...)
    ON [PRIMARY]
```

### Partitioned Indexes

Enterprise Editionで使用可能。パーティション分割されたテーブルと同様の方法で作成および維持されるが、通常のインデックスのように、個別のデータベース オブジェクトとして扱われる。

| Index | Description |
| :--- | :--- |
| 固定されたインデックス | 対応するテーブルと同じパーティション構成に基づいて構築されたインデックス。インデックスとベース テーブルのパーティション関数は、本質的に同一である必要がある。 |
| 固定されていないインデックス | インデックスのパーティション構成が異なっているか、インデックスがベース テーブルとは別のファイル グループに配置されている。 |

## Tips

### Table Structure

```sql
USE <database>
SELECT *
FROM sys.dm_db_partition_stats
WHERE OBJECT_NAME(object_id) = '<TableName>'
```

| Column | Description |
| :--- | :--- |
| index_id | 0: Heap構造（クラスタ化インデックスが作成されていない）<br>1: クラスタ化インデックス構造<br>2以上: 非クラスタ化インデックスの情報 |
| in_row_reserved_page_count | 予約済みページ数 |
| in_row_used_page_count | 使用済みページ数 |
| in_row_data_page_count | 使用済みデータページ数<br>`in_row_used_page_count` - `in_row_data_page_count` = IAM (Index Allocation Map) |

### Checking Index Effectiveness

インデックスの効果を確認するためには、実際にテーブルからデータを読み込む必要があるので、バッファーキャッシュを削除した後、クエリを実行して実行プランと物理ページに対するI/O数を確認する。

```sql
USE <database>
-- バッファーキャッシュに置かれているページを削除
DBCC DROPCLEANBUFFERS
-- ページのI/O数を確認
SET STATISTICS IO ON
GO
<Query>
GO
```

実行結果例:

```text
テーブル '<TableName>'。スキャン回数 1、論理読み取り数 101、物理読み取り数 0、先行読み取り数 101、LOB論理読み取り数 0、LOB物理読み取り数 0、LOB先行読み取り数 0。
```

### Index Fragmentation and Remediation

インデックスの断片化には2種類ある。

- 内部断片化：インデックスページ内に必要以上の空白があり領域を消費している状態
- 外部断片化：ページ分割により物理的なページの並び順が論理的な並び順と一致しない

```sql
SELECT *
FROM sys.dm_db_index_physical_stats
    ( DB_ID(N'<database>'), OBJECT_ID(N'<tableName>'), NULL, NULL, 'DETAILED')
GO
```

| Column | Description |
| :--- | :--- |
| avg_fragmentation_in_percent | 5〜30%: インデックスの再構成を推奨する<br>30%以上: インデックスの再構築を推奨する |

```sql
USE <database>
-- テーブルに作成した全てのインデックスを再構成する
ALTER INDEX ALL ON <TableName> REORGANIZE
GO
-- テーブルに作成した全てのインデックスを再構築する
ALTER INDEX ALL ON <TableName> REBUILD
    WITH (FILLFACTOR = 70, SORT_IN_TEMPDB = ON, ONLINE = ON )
GO
```

avg_fragmentation_in_percent（インデックスの論理的な断片化）確認クエリ:

```sql
SELECT * FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, NULL) AS ps
INNER JOIN sys.indexes i ON (
    ps.object_id = i.object_id
    AND ps.index_id = i.index_id
)
WHERE ps.index_id > 0
```
