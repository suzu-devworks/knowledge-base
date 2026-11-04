# PostgreSQL Tips

## Table of Contents <!-- omit in toc -->

- [Environments](#environments)
- [Commands with `psql`](#commands-with-psql)
  - [Connecting to PostgreSQL](#connecting-to-postgresql)
  - [`\help`, `\h`, `\?` Help](#help-h--help)
  - [`\encoding` Encoding Information](#encoding-encoding-information)
  - [`\conninfo` Connection Information](#conninfo-connection-information)
- [Database Information and Meta Commands](#database-information-and-meta-commands)
  - [`\list`, `\l` Databases Information](#list-l-databases-information)
  - [`\connect`, `\c` Connect to Database](#connect-c-connect-to-database)
  - [Database version](#database-version)
  - [Current Database](#current-database)
  - [Database Creation and Deletion](#database-creation-and-deletion)
- [Table Information and Meta Commands](#table-information-and-meta-commands)
  - [`\dt` List Tables](#dt-list-tables)
  - [`\d <table_name>` Table Details](#d-table_name-table-details)
  - [Create Tables](#create-tables)
- [Variables](#variables)
- [Tips](#tips)
  - [Character Types](#character-types)
  - [`RESTART IDENTITY`](#restart-identity)
  - [Transaction Identifier](#transaction-identifier)
  - [Use uuid](#use-uuid)

## Environments

- 2026-11-02: PostgreSQL 18.6 (Docker: `postgres:18-alpine`)

## Commands with `psql`

### Connecting to PostgreSQL

```text
# psql -h ホスト名 -p ポート番号 -U ロール名 -d データベース名
```

```bash
psql -U postgres
```

```console
psql (18.6)
Type "help" for help.
```

### `\help`, `\h`, `\?` Help

ヘルプの表示:

```sql
postgres=# \?
-- \? [commands]          バックスラッシュコマンドのヘルプ表示
-- \? options             psql コマンドラインオプションのヘルプ表示
-- \? variables           特別な変数のヘルプ表示
-- \h [NAME]              SQLコマンドの構文ヘルプ表示（* で全コマンド）
```

### `\encoding` Encoding Information

文字コード確認と設定:

```sql
postgres=# \encoding
UTF8

-- 設定
postgres=# SET encoding TO 'UTF8';
```

### `\conninfo` Connection Information

```sql
postgres=# \conninfo
           Connection Information
      Parameter       |        Value        
----------------------+---------------------
 Database             | postgres
 Client User          | postgres
 Socket Directory     | /var/run/postgresql
 Server Port          | 5432
 Options              | 
 Protocol Version     | 3.0
 Password Used        | false
 GSSAPI Authenticated | false
 Backend PID          | 78
 SSL Connection       | false
 Superuser            | on
 Hot Standby          | off
(12 rows)
```

## Database Information and Meta Commands

### `\list`, `\l` Databases Information

データベース一覧:

```sql
postgres-# \l
                                                    List of databases
   Name    |  Owner   | Encoding | Locale Provider |  Collate   |   Ctype    | Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+------------+------------+--------+-----------+-----------------------
 postgres  | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | 
 template0 | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
           |          |          |                 |            |            |        |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | en_US.utf8 | en_US.utf8 |        |           | =c/postgres          +
           |          |          |                 |            |            |        |           | postgres=CTc/postgres
(3 rows)
```

### `\connect`, `\c` Connect to Database

データベースに接続:

```sql
postgres=# \c db
You are now connected to database "db" as user "postgres".

db=# \connect postgres
You are now connected to database "postgres" as user "postgres".
```

### Database version

接続先データベースのバージョンを確認:

```sql
postgres=# SELECT version();
                                            version                                            
-----------------------------------------------------------------------------------------------
 PostgreSQL 18.6 on aarch64-unknown-linux-musl, compiled by gcc (Alpine 15.2.0) 15.2.0, 64-bit
(1 row)
```

### Current Database

接続中のデータベースを確認（プロンプトでわかるけど）:

```sql
postgres=# SELECT current_database();
 current_database 
------------------
 postgres
(1 row)
```

### Database Creation and Deletion

```sql
-- データベース作成
postgres=# CREATE DATABASE db;
CREATE DATABASE

-- データベース切り替え
postgres=# \c db
You are now connected to database "db" as user "postgres".

-- 接続中のデータベースは削除不可
postgres=# DROP DATABASE db;
ERROR:  cannot drop the currently open database

db=# \connect postgres
You are now connected to database "postgres" as user "postgres".

-- データベース削除
postgres=# DROP DATABASE db;
DROP DATABASE
```

## Table Information and Meta Commands

### `\dt` List Tables

デーブル一覧を表示:

```sql
postgres=# \dt
Did not find any tables.

postgres=# \dtS
                      List of tables
   Schema   |           Name           | Type  |  Owner   
------------+--------------------------+-------+----------
 pg_catalog | pg_aggregate             | table | postgres
 pg_catalog | pg_am                    | table | postgres
 pg_catalog | pg_amop                  | table | postgres
 pg_catalog | pg_amproc                | table | postgres
 pg_catalog | pg_attrdef               | table | postgres
 -- More --
```

> `\dtS` でシステムテーブルも含めて表示できます

### `\d <table_name>` Table Details

テーブルの詳細情報を確認:

<!-- spell-checker: disable -->
```sql
postgres=# \d pg_database;
               Table "pg_catalog.pg_database"
     Column     |   Type    | Collation | Nullable | Default 
----------------+-----------+-----------+----------+---------
 oid            | oid       |           | not null | 
 datname        | name      |           | not null | 
 datdba         | oid       |           | not null | 
 encoding       | integer   |           | not null | 
 datlocprovider | "char"    |           | not null | 
 datistemplate  | boolean   |           | not null | 
 datallowconn   | boolean   |           | not null | 
 dathasloginevt | boolean   |           | not null | 
 datconnlimit   | integer   |           | not null | 
 datfrozenxid   | xid       |           | not null | 
 datminmxid     | xid       |           | not null | 
 dattablespace  | oid       |           | not null | 
 datcollate     | text      | C         | not null | 
 datctype       | text      | C         | not null | 
 datlocale      | text      | C         |          | 
 daticurules    | text      | C         |          | 
 datcollversion | text      | C         |          | 
 datacl         | aclitem[] |           |          | 
Indexes:
    "pg_database_oid_index" PRIMARY KEY, btree (oid), tablespace "pg_global"
    "pg_database_datname_index" UNIQUE CONSTRAINT, btree (datname), tablespace "pg_global"
Tablespace: "pg_global"
```
<!-- spell-checker: enable -->

### Create Tables

```sql
-- テーブル作成
postgres=# CREATE TABLE personals (id INT PRIMARY KEY, name CHARACTER VARYING(60));
CREATE TABLE

postgres=# CREATE TABLE items (id INT PRIMARY KEY, name VARCHAR(120));
CREATE TABLE

postgres=# CREATE TABLE item_owners (id INT PRIMARY KEY, item_id INT, personal_id INT, update_at TIMESTAMPTZ);
CREATE TABLE

postgres=# \dt
             List of tables
 Schema |    Name     | Type  |  Owner   
--------+-------------+-------+----------
 public | item_owners | table | postgres
 public | items       | table | postgres
 public | personals   | table | postgres
(3 rows)
```

## Variables

psqlは一般的なUnixコマンドシェルに似た変数の置換機能を提供します。
例えば、`\set`コマンドを使って変数を設定し、SQL文内で使用することができます。

```sql
postgres=# \set myvar 'Hello, World!'
```

この変数を確認するには、`\echo`コマンドを使用します。

```sql
postgres=# \echo :myvar
Hello, World!
```

もちろん、SQL文内でも変数を使用することができます。

```sql
postgres=# SELECT :'myvar';
   ?column?    
---------------
 Hello, World!
(1 row)
```

バインド変数ではなく、psqlの変数置換機能であることに注意してください。

```sql
postgres=# SELECT :myvar;
ERROR:  syntax error at or near ";"
LINE 1: SELECT Hello, World!;

postgres=# SELECT ':myvar';
 ?column? 
----------
 :myvar
(1 row)
```

引数をまったく取らない\setは、現在設定されているpsql変数すべての名前と値を表示します。

```sql
postgres=# \set
 ...
myvar='Hello, World!'
```

## Tips

### Character Types

PostgreSQLでは、文字列を格納するためのデータ型としてつぎのものがありますが、公式ドキュメントで注意書きがあります。

- `CHAR(n)` / `CHARACTER(n)`:
  固定長の文字列を格納します。指定した長さに満たない場合は空白で埋められます。
- `CHARACTER VARYING(n)` / `VARCHAR(n)`:
  可変長の文字列を格納します。最大長`n`を指定できます。
- `TEXT`:
  長さ制限のない文字列を格納します。

これら3つの型の間で、パフォーマンス上の違いはありません。
ただし、空白埋め（blank-padded）型を使用する場合はストレージ容量が増加するほか、
長さが制限された列にデータを格納する際には、長さを確認するためのCPU処理がわずかに余分に発生します。

他のデータベースシステムでは `character(n)` 型がパフォーマンス面で有利な場合がありますが、PostgreSQLにはそのような利点はありません。
実際、`character(n)` 型はストレージ負荷が余分にかかるため、これら3つの中で最も低速になるのが一般的です。

ほとんどの状況において、代わりに `text` 型や `character varying` 型を使用すべきです。

### `RESTART IDENTITY`

テーブルの全データを削除する際、自動採番（IDENTITY / SERIAL）を初期値にリセットします。

```sql
-- テーブル作成
postgres=# CREATE TABLE users (
    id INT GENERATED ALWAYS AS IDENTITY,
    name VARCHAR(50)
);
CREATE TABLE

-- データ挿入
postgres=# INSERT INTO users (name) VALUES ('Alice'), ('Bob'), ('Charlie');
INSERT 0 3

-- 3. データの確認（idが1, 2, 3になっている）
postgres=# SELECT * FROM users;
 id |  name   
----+---------
  1 | Alice
  2 | Bob
  3 | Charlie
(3 rows)

-- 4. 通常のデータの切り捨て
postgres=# TRUNCATE TABLE users;
TRUNCATE TABLE

-- 5. データ挿入（idが4になる）
postgres=# INSERT INTO users (name) VALUES ('Dave')
INSERT 0 1

postgres=# SELECT * FROM users;
 id | name 
----+------
  4 | Dave
(1 row)

-- 6. データの切り捨てとIDのリセット
postgres=# TRUNCATE TABLE users RESTART IDENTITY;
TRUNCATE TABLE

-- 7. データ挿入（idが1になる）
postgres=# INSERT INTO users (name) VALUES ('Eve');
INSERT 0 1

postgres=# SELECT * FROM users;
 id | name 
----+------
  1 | Eve
(1 row)
```

### Transaction Identifier

- [Transaction ID and Snapshot Information Functions](https://www.postgresql.org/docs/current/functions-info.html#FUNCTIONS-INFO-SNAPSHOT)

テーブルにトランザクションが関与している場合、そのトランザクションのIDを取得することができます。
例えば、`pg_current_xact_id()` 関数を使用すると、現在のトランザクションのIDを取得できます。

```sql
postgres=# SELECT pg_current_xact_id();
 pg_current_xact_id 
--------------------
                775
(1 row)
```

この値を使えば、例えばデータ挿入時のトランザクションIDを記録したり、特定のトランザクションに関連する操作を追跡することができます。

```sql
postgres=# CREATE TABLE order_logs (
    id SERIAL PRIMARY KEY,
    order_id INT,
    status VARCHAR(50),
    -- トランザクションIDを記録するカラム
    tx_id bigint DEFAULT pg_current_xact_id()::text::bigint,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE TABLE

-- データ挿入例
postgres=# INSERT INTO order_logs (order_id, status) VALUES 
  (101, '注文受付'), 
  (101, '発送準備中');
INSERT 0 2

postgres=# INSERT INTO order_logs (order_id, status) VALUES 
  (101, '発送済み');
INSERT 0 1

-- データの確認
postgres=# SELECT * FROM order_logs;
 id | order_id |   status   | tx_id |         created_at         
----+----------+------------+-------+----------------------------
  1 |      101 | 注文受付   |   780 | 2026-11-03 13:03:06.382832
  2 |      101 | 発送準備中 |   780 | 2026-11-03 13:03:06.382832
  3 |      101 | 発送済み   |   781 | 2026-11-03 13:03:45.277192
(3 rows)
```

> [!WARNING]
> トランザクションIDは有限の範囲で循環するため、長期間にわたって同じテーブルに大量のトランザクションを記録する場合、IDが再利用される可能性があります。
> これにより、古いトランザクションと新しいトランザクションのIDが重複することがあるため注意が必要です。
> 長期間保存する場合は、トランザクションIDの再利用に注意し、必要に応じて別の方法でトランザクションを識別することを検討してください。

### Use uuid

PostgreSQL 13.0 以降では、拡張機能無しで UUIDを使用することができます。

```sql
postgres=# SELECT gen_random_uuid();
           gen_random_uuid            
--------------------------------------
 230678e7-33a5-4082-a692-c9b66f6f9872
(1 row)
```

PostgreSQL 18では、タイムスタンプベースで時系列ソート可能なUUIDv7のデータベース側生成がサポートされました。

```sql
postgres=# select uuidv7() ;
                uuidv7                
--------------------------------------
 01a1a185-b75d-77eb-a8c3-edb2ecd73bcc
(1 row)

postgres=# select uuidv7() ;
                uuidv7                
--------------------------------------
 01a1a185-bf19-7fd9-a22b-1d944779536b
(1 row)
```

> なんとなく並びそう

uuidv7() が追加されたことにより gen_random_uuid() を uuidv4() のエイリアスで使用できるようになりました。

```sql
postgres=# select uuidv4() ;
                uuidv4                
--------------------------------------
 380c0e1b-ad1d-4887-8010-276f6532c75e
(1 row)
```
