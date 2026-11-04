# SQLite Tips Guide

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Terminology](#terminology)
- [Installation](#installation)
  - [For Windows](#for-windows)
  - [For Linux on Fedora](#for-linux-on-fedora)
- [Command Line Tools(`sqlite3`)](#command-line-toolssqlite3)
- [Management Commands](#management-commands)
  - [`VACUUM`](#vacuum)
  - [Display Tables Information](#display-tables-information)
  - [Table Structure (Describe Table)](#table-structure-describe-table)
  - [Reset Auto-increment Counter](#reset-auto-increment-counter)
- [Tips](#tips)
  - [Check Data Type stored in Columns](#check-data-type-stored-in-columns)
  - [Join Tables across Multiple Database Files](#join-tables-across-multiple-database-files)

## Overview

SQLite は MySQL や PostgreSQL と同じデータベース管理システムですが、独立したサーバーとしてではなくアプリケーションに組み込んで利用される軽量なデータベースです。

- [SQLite Home Page](http://www.sqlite.org/)

主な特徴は以下の通りです。

- SQL92 の機能の多くを実装
- パブリックドメインで提供
- ファイルを使用しない揮発性のインメモリデータベースとしても利用可能
- バイトオーダーに依存しないデータ構造
- トランザクション処理のサポート
- バージョン 3.3.8 以降における全文検索（FTS1 モジュール）のサポート

## Environments

- ???: SQLite 3.7.2 on Windows XP
- ???: SQLite 3.6.22 on Fedora 13
- 2019-10-04: SQLite 3.30.0 on Fedora 31

## Terminology

| Words | Description |
| :--- | :--- |
| Database | SQLite においてはデータベースファイルそのものを指します。1つのファイルが1つのデータベースに対応します。 |
| Schema | データベースの構造を指します。Table schema や Index schema などの用途で利用されます。 |

## Installation

### For Windows

インストールは基本的に圧縮ファイルの展開とパスの設定のみで完了します。

1. SQLite 公式サイトの「Download」ページから `sqlite-tools-win32-x86-*.zip` などの Windows 用パッケージをダウンロードします。
2. 任意のフォルダに解凍し、そのディレクトリへ環境変数 `PATH` を通します。

### For Linux on Fedora

通常は標準リポジトリに含まれており、多くの環境で導入済みとなっています。

```bash
sudo dnf install sqlite
```

```console
パッケージ sqlite-3.30.0-1.fc31.x86_64 は既にインストールされています。
```

## Command Line Tools(`sqlite3`)

`sqlite3` コマンドラインツールは Linux、Windows 共に共通の操作体系を提供します。データベース名を指定せずに起動すると、トランジェントなインメモリデータベースに接続されます。

```bash
sqlite3
```

```console
SQLite version 3.30.0 2019-10-04 15:03:17
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
```

```sql
-- 文字コードの確認
sqlite> PRAGMA encoding;
UTF-8

-- テーブルの作成
sqlite> CREATE TABLE personal (id, name);
sqlite> CREATE TABLE items (id, name);
sqlite> CREATE TABLE item_owners (id, item_id, personal_id, update_at timestamp);

-- テーブル一覧の確認
sqlite> .tables
item_owners  items        personal   

-- テーブル定義（スキーマ）の確認
sqlite> .schema
CREATE TABLE personal (id, name);
CREATE TABLE items (id, name);
CREATE TABLE item_owners (id, item_id, personal_id, update_at timestamp);

sqlite> .schema item_owners
CREATE TABLE item_owners (id, item_id, personal_id, update_at timestamp);

-- SELECT 結果の出力モード変更 (標準: list)
sqlite> .mode quote

-- メモリ上のデータベースをファイルへ保存
sqlite> .save example.db

-- コマンドラインツールの終了
sqlite> .exit
```

```bash
ls
```

```console
example.db
```

作成したデータベースファイルへ再接続:

```bash
sqlite3 example.db
```

```console
SQLite version 3.30.0 2019-10-04 15:03:17
Enter ".help" for usage hints.
```

```sql
-- 接続中データベースの確認
sqlite> .databases
main: /home/akira/workspaces/example.db

-- 表示出力の設定 (カラム表示・ヘッダーON)
sqlite> .mode column
sqlite> .header on
```

## Management Commands

### `VACUUM`

`VACUUM` 命令を実行すると、未使用領域を解放してデータベースファイルのサイズを縮小できます。データを一時データベースへコピーして再構築するため、空き領域の削除と同時にデータが連続領域へ再配置されます。

> [!NOTE]
> `VACUUM` の対象となるのは `main` データベースのみです。アタッチされた別データベースは対象外となります。

```sql
sqlite> VACUUM;
```

### Display Tables Information

標準 SQL を使用してテーブル情報を確認する場合は `sqlite_master` システムテーブルを参照します。

```sql
sqlite> .mode column
sqlite> .header on
sqlite> SELECT * FROM sqlite_master;

type        name        tbl_name    rootpage    sql                               
----------  ----------  ----------  ----------  ----------------------------------
table       personal    personal    2           CREATE TABLE personal (id, name)  
table       items       items       3           CREATE TABLE items (id, name)     
table       item_owners item_owners 4           CREATE TABLE item_owners (id, ... 
```

### Table Structure (Describe Table)

Oracle DB の `DESC` コマンドと同等の機能として `pragma table_info()` を使用します。

```sql
sqlite> .mode column
sqlite> .header on
sqlite> PRAGMA table_info(item_owners);

cid         name        type        notnull     dflt_value  pk        
----------  ----------  ----------  ----------  ----------  ----------
0           id                      0                       0         
1           item_id                 0                       0         
2           personal_id             0                       0         
3           update_at   timestamp   0                       0         
```

### Reset Auto-increment Counter

SQLite には `TRUNCATE` 文が存在しないため `DELETE` を代用します。`AUTOINCREMENT` のシーケンス値をリセットする場合は、管理テーブル `sqlite_sequence` から該当レコードを削除します。

```sql
sqlite> .mode column
sqlite> .header on
sqlite> SELECT * FROM sqlite_sequence;

name        seq       
----------  ----------
sales       1         

sqlite> DELETE FROM sqlite_sequence WHERE name = 'sales';
```

## Tips

### Check Data Type stored in Columns

カラムにデータ型を指定しない場合、入力されたデータの型がそのまま格納されます。型を指定した場合は、指定型への変換を試みた上で格納されます。

```sql
sqlite> .mode column
sqlite> .header on

-- レコードの挿入
sqlite> INSERT INTO personal VALUES (1, 'hogehoge');
sqlite> INSERT INTO personal VALUES (2, 23.456);
sqlite> INSERT INTO personal VALUES (3, 123);
sqlite> INSERT INTO personal VALUES (4, null);

-- データ一覧の取得
sqlite> SELECT * FROM personal;

id          name      
----------  ----------
1           hogehoge  
2           23.456    
3           123       
4                     

-- 各データのアサイン型を確認
sqlite> SELECT id, typeof(id), name, typeof(name) FROM personal;

id          typeof(id)  name        typeof(name)
----------  ----------  ----------  ------------
1           integer     hogehoge    text        
2           integer     23.456      real        
3           integer     123         integer     
4           integer                 null        
```

### Join Tables across Multiple Database Files

`ATTACH` ステートメントを使用することで、別ファイルのデータベースを読み込み、単一クエリ内でテーブル結合が可能です。

```sql
sqlite> .mode column
sqlite> .header on

-- 現在接続しているデータベースの確認
sqlite> .databases
main: /home/akira/workspaces/example.db

-- other.db をエイリアス db1 としてアタッチ
sqlite> ATTACH 'other.db' AS db1;

-- アタッチされたデータベースの確認
sqlite> .databases
main: /home/akira/workspaces/example.db
db1: /home/akira/workspaces/other.db

-- db1 内のテーブルが参照可能に
sqlite> .tables
db1.sales
item_owners  items        personal   

-- アタッチ先テーブルとの JOIN クエリ
sqlite> SELECT * FROM db1.sales LEFT OUTER JOIN items ON (items.id = db1.sales.item_id);

-- デタッチ処理
sqlite> DETACH db1;

sqlite> .databases
main: /home/akira/workspaces/example.db
```
