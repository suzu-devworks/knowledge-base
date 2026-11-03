# DB2 SQL Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Useful System Catalog Queries](#useful-system-catalog-queries)

## Overview

IBM DB2 の基本的なシステム カタログ クエリのヒントを提供します。

## Environments

- ???

## Useful System Catalog Queries

次のクエリを使用すると、システム カタログ テーブルでテーブル メタデータ、列定義、およびビュー構造を検索できます。

```sql
-- Search for a specific table by name
SELECT * FROM SYSIBM.SYSTABLES WHERE NAME = 'XXX';

-- Search for column information for a specific table or column name
SELECT * FROM SYSIBM.SYSCOLUMNS WHERE NAME = 'XXX';

-- Retrieve all system views
SELECT * FROM SYSIBM.SYSVIEWS;
```
<!-- spell-checker: words SYSIBM SYSVIEWS SYSCOLUMNS SYSTABLES -->
