# IBM DB2 SQL Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation](#installation)
  - [Pull the Docker Image](#pull-the-docker-image)
  - [Environment Variables Configuration](#environment-variables-configuration)
  - [Start the Docker Container](#start-the-docker-container)
  - [Connection Verification](#connection-verification)
- [References](#references)

## Overview

IBM Db2とは、米IBM社が提供しているリレーショナルデータベース管理システム（RDBMS）およびその関連製品群のことです。

メインフレームの時代から脈々と受け継がれてきたデータベース技術です。

## Environments

- 2026-11-04: icr.io/db2_community/db2:12.1.5 on macOS 26.6.2 (Apple silicon)

## Installation

### Pull the Docker Image

公式でいくとまずは IBM の提供する Docker イメージをPullしてくるところから始まります。

- [Linux システムでの Db2 Community Edition Docker イメージのインストール](https://www.ibm.com/docs/ja/db2/12.1.x?topic=system-linux)

```bash
docker pull icr.io/db2_community/db2 --platform linux/amd64 
```

> [!WARNING]
> `--platform linux/amd64` をつけないと Apple silicon では次のエラーになります。<br>
> Error response from daemon: no matching manifest for linux/arm64/v8 in the manifest list entries:
> no match for platform in manifest: not found

### Environment Variables Configuration

`.env_list` ファイルに、Docker コンテナで使用する環境変数を定義します。

```env
LICENSE=accept
DB2INSTANCE=db2inst1
DB2INST1_PASSWORD=*****
DBNAME=testdb
BLU=false
ENABLE_ORACLE_COMPATIBILITY=false
SAMPLEDB=true
REPODB=false
PERSISTENT_HOME=true
HADR_ENABLED=false
ETCD_ENDPOINT=
ETCD_USERNAME=
ETCD_PASSWORD=
```

> SAMPLEDB=true だけ公式の説明と異なります。

| 環境変数 | 説明 |
| --- | --- |
| `LICENSE` |  このイメージに含まれる Db2 ソフトウェアの契約条件に同意します。 |
| `DB2INSTANCE` |  Db2 インスタンス名を指定します。 |
| `DB2INST1_PASSWORD` |  Db2 インスタンスのパスワードを指定します。 |
| `DBNAME` |  指定された名前で初期データベースを作成します (データベースが必要ない場合は空のままにします) |
| `BLU` |  Db2 インスタンスの BLU Acceleration を使用可能 (true) または使用不可 (false) に設定します。 |
| `ENABLE_ORACLE_COMPATIBILITY` | インスタンスの Oracle 互換性を有効 (true) または無効 (false) に設定します |
| `SAMPLEDB` |  セットアップ中に SAMPLE データベースを作成します（ true ） |
| `REPODB` |  Data Server Manager リポジトリー・データベースを作成します (true) |
| `PERSISTENT_HOME` | デフォルトで、true に設定されており、Docker for Windows を実行している場合にのみ false として指定する必要があります |
| `HADR_ENABLED` | インスタンスの Db2 HADR を構成します (true)。 以下の 3 つの環境変数は、HADR_ENABLED が true に設定されていることに依存します。|
| `ETCD_ENDPOINT` | ユーザー自身が指定した ETCD キー値ストアを指定します。 コンマ (スペースなし) を区切り文字として エンドポイントを入力します。 HADR_ENABLED が true に設定されている場合、この環境変数が必要です。|
| `ETCD_USERNAME` |  ETCD のユーザー名資格情報を指定します。 空のままにすると、 Db2 インスタンスが使用されます。 |
| `ETCD_PASSWORD` |  ETCD のパスワード資格情報を指定します。 空のままにすると、 Db2 インスタンス・パスワードが使用されます。|

### Start the Docker Container

Dockerを利用してサーバーを起動します。

```bash
docker run -h db2server --name db2server --restart=always --detach --privileged=true -p 50000:50000 \
    --env-file .env_list icr.io/db2_community/db2
```

Dockerのログに次が出るまでは我慢。

```console
(*) User chose to create testdb database
(*) Creating database testdb ... 
DB20000I  The CREATE DATABASE command completed successfully.
```

### Connection Verification

接続確認:

```bash
docker exec -ti db2server bash -c "su - db2inst1"
```

```console
db2inst1@db2server $ db2 connect to testdb

   Database Connection Information

 Database server        = DB2/LINUXX8664 12.1.5.0
 SQL authorization ID   = DB2INST1
 Local database alias   = TESTDB
```

```console
db2inst1@db2server $ db2 "SELECT SERVICE_LEVEL FROM SYSIBMADM.ENV_INST_INFO;"

SERVICE_LEVEL                                                                                                                   
-------------------------------------------------------------------------------
DB2 v12.1.5.0                                                                                                                   

  1 record(s) selected.
```

切断：

```console
db2inst1@db2server $ db2 terminate

DB20000I  The TERMINATE command completed successfully.
```

`db2` だけで実行するとインタラクティブモードになります。

```console
db2inst1@db2server $ db2

(c) Copyright IBM Corporation 1993,2007
Command Line Processor for DB2 Client 12.1.5.0

You can issue database manager commands and SQL statements from the command 
prompt. For example:
    db2 => connect to sample
    db2 => bind sample.bnd

For general help, type: ?.
For command help, type: ? command, where command can be
the first few keywords of a database manager command. For example:
 ? CATALOG DATABASE for help on the CATALOG DATABASE command
 ? CATALOG          for help on all of the CATALOG commands.

To exit db2 interactive mode, type QUIT at the command prompt. Outside 
interactive mode, all commands must be prefixed with 'db2'.
To list the current command option settings, type LIST COMMAND OPTIONS.

For more detailed help, refer to the Online Reference Manual.

db2 => 
```

## References

- [Db2 Community Edition for Docker](https://www.ibm.com/docs/ja/db2/12.1.x?topic=deployments-db2-community-edition-docker)
