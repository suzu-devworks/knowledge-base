# IBM DB2 System Catalog Tips

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Useful System Catalog Queries](#useful-system-catalog-queries)
  - [Database list](#database-list)
  - [User tables](#user-tables)
  - [Dummy Query](#dummy-query)
- [References](#references)

## Overview

IBM DB2 の基本的なシステム カタログ クエリのヒントを提供します。

## Useful System Catalog Queries

### Database list

```console
db2 => list db directory

 System Database Directory

 Number of entries in the directory = 2

Database 1 entry:

 Database alias                       = TESTDB
 Database name                        = TESTDB
 Local database directory             = /database/data
 Database release level               = 16.00
 Comment                              =
 Directory entry type                 = Indirect
 Catalog database partition number    = 0
 Alternate server hostname            =
 Alternate server port number         =

Database 2 entry:

 Database alias                       = SAMPLE
 Database name                        = SAMPLE
 Local database directory             = /database/data
 Database release level               = 16.00
 Comment                              =
 Directory entry type                 = Indirect
 Catalog database partition number    = 0
 Alternate server hostname            = 
 Alternate server port number         =
```

### User tables

`SYSIBM.SYSTABLES` や `SYSIBM.SYSCOLUMNS` などのシステム カタログ テーブルを直接参照するのは一般的には推奨されません。代わりに `SYSCAT` ビューを使用することが推奨されます。

```sql
-- Retrieve all user tables
SELECT TABSCHEMA, TABNAME, TYPE FROM SYSCAT.TABLES WHERE TYPE = 'T' AND OWNERTYPE = 'U';
```

```sql
-- Retrieve column information for a specific table
SELECT COLNO, COLNAME, TYPENAME, LENGTH, SCALE, NULLS FROM SYSCAT.COLUMNS WHERE TABSCHEMA = 'DB2INST1' AND TABNAME = 'STAFF' ORDER BY COLNO;
```

### Dummy Query

いわゆる DUAL表です。

```sql
db2 => SELECT 1 FROM SYSIBM.SYSDUMMY1;
```

## References

- [Road map to the catalog views | Db2 for Linux, UNIX and Windows 12.1.x | IBM Documentation](https://www.ibm.com/docs/ja/db2/12.1.x?topic=views-road-map-catalog)
