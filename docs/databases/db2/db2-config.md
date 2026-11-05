# IBM DB2 SQL Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation](#installation)
  - [Pull the Docker Image](#pull-the-docker-image)
  - [Environment Variables Configuration](#environment-variables-configuration)
  - [Start the Docker Container](#start-the-docker-container)
  - [Connection Verification](#connection-verification)
- [Localizations](#localizations)
  - [Checking the character encoding and collation of the current database](#checking-the-character-encoding-and-collation-of-the-current-database)
  - [Create a Database with Shift\_JIS Encoding Japanese Territory](#create-a-database-with-shift_jis-encoding-japanese-territory)
  - [Collation](#collation)
  - [Time Zone](#time-zone)
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

## Localizations

初期データベースは UTF-8 US で作成されます。
それで困ることも少なくなってきていますが、データベースの文字コードや照合順序が重要な場合は、データベースの再作成が必要になるので最初に検討する必要があります。

- [Choosing the code page, territory, and collation for your database | Db2 for Linux, UNIX and Windows 12.1.x | IBM Documentation](https://www.ibm.com/docs/ja/db2/12.1.x?topic=support-choosing-code-page-territory-collation-your-database)

### Checking the character encoding and collation of the current database

現在の称号順の確認：

```console
db2inst1@db2server $ db2 get db cfg for testdb | grep -iE "code set|code page|territory|collating"

 Database territory                                      = us
 Database code page                                      = 1208
 Database code set                                       = utf-8
 Database collating sequence                             = IDENTITY
 Alternate collating sequence              (ALT_COLLATE) = 
```

> コード・ページ 1208（CCSID 1208）は、IBM製品やDB2などにおいて UTF-8 を示す識別番号（CCSID: Coded Character Set Identifier）です。

### Create a Database with Shift_JIS Encoding Japanese Territory

```console
db2inst1@db2server $ db2 "CREATE DATABASE testdb2 USING CODESET IBM-943 TERRITORY JP"

DB20000I  The CREATE DATABASE command completed successfully.
```

> IBM-943 は、Shift_JIS を示す IBM のコードセットです。

確認：

```console
[db2inst1@db2server ~]$ db2 get db cfg for testdb2 | grep -iE "code set|code page|territory|collating"
 Database territory                                      = JP
 Database code page                                      = 943
 Database code set                                       = IBM-943
 Database collating sequence                             = UNIQUE
 Alternate collating sequence              (ALT_COLLATE) = 
```

### Collation

DB2では、照合順序はデータベース作成時に指定して変更できません。
初期データーベースでは、照合順序はデフォルトで IDENTITY (バイナリ照合) となります。

Unicode称号順とそのほかのコードページだと照合順序の扱いが異なることがあるようです。

- [Collating sequences | Db2 for Linux, UNIX and Windows 12.1.x | IBM Documentation](https://www.ibm.com/docs/ja/db2/12.1.x?topic=collation-collating-sequences )

Shift_JIS の場合、照合順序は SYSTEM　または IDENTITY が指定することができ、デフォルトでは SYSTEM（UNIQUE） が使用されます。

UTF-8 の場合、デフォルト照合順序は IDENTITY が使用されます。

日本語関連でいくと次のような照合順を指定できます

- CLDRCOLL_LJA (Japanese: CLDRCOLL_LJA_AN_CX_EX_FX_HX_NX_S3)
- CLDRCOLL_LJA_KUNIHAN (Japanese: CLDRCOLL_LJA_KUNIHAN_AN_CX_EX_FX_HX_NX_S3)

詳細は次を参照してください。

- [Unicode Collation Algorithm-based collations | Db2 for Linux, UNIX and Windows 12.1.x | IBM Documentation](https://www.ibm.com/docs/ja/db2/12.1.x?topic=collation-unicode-algorithm-based-collations)

### Time Zone

DB2ではタイムゾーンは実行しているサーバーのローカルタイムゾーンが使用されます。
そのためタイムゾーン設定はDB2ではなく、サーバー側で行う必要があります。

たとえば docker の場合には環境変数で指定します。

```bash
docker run -e TZ=Asia/Tokyo ...
```

## References

- [Db2 Community Edition for Docker](https://www.ibm.com/docs/ja/db2/12.1.x?topic=deployments-db2-community-edition-docker)
