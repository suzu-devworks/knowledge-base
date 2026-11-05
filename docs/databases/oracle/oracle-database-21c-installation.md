# Oracle Database 21c Installation

## Table of Contents <!-- omit in toc -->

## Overview

Oracle Database のコンテナイメージは Docker Hub では 12c で提供が終了している。公式の方法で 21c まで作成できる情報があったため、SQL の学習用にインストール手順をまとめる。

## Environments

- 2021-09-11: Oracle Database 21c (21.3), Docker Community 20.10.8 on Fedora 34 (5.13.14-200.fc34.x86_64)

## Installation on Docker

### Preparation

公式リポジトリから Dockerfile を取得する。

```bash
git clone https://github.com/oracle/docker-images.git
```

Oracle Database 21c Linux x86_64 を公式からダウンロードし、指定ディレクトリへ移動する。

- [Oracle Database 21c Download for Linux (Intel x86-64) (64-bit)](https://www.oracle.com/database/technologies/oracle21c-linux-downloads.html)

> [!NOTE]
> Oracle アカウントでログインが必要です。

```bash
mv LINUX.X64_213000_db_home.zip ~/repos/docker-images/OracleDatabase/SingleInstance/dockerfiles/21.3.0
```

配置確認:

```console
$ ls

Checksum.ee           createDB.sh           runOracle.sh
Checksum.se2          createObserver.sh     runUserScripts.sh
Dockerfile            db_inst.rsp           setPassword.sh
LINUX.X64_213000_db_home.zip  dbca.rsp.tmpl setupLinuxEnv.sh
checkDBStatus.sh      installDBBinaries.sh  startDB.sh
checkSpace.sh         relinkOracleBinary.sh
```

### Create Docker Image

一つ上のディレクトリへ移動してビルドスクリプトを実行する。

```bash
./buildContainerImage.sh -e -v 21.3.0 -o '--build-arg SLIMMING=false'
```

`'--build-arg SLIMMING=false'` は 19c 以降でビルドする場合に必要なオプション。
完了時に以下のメッセージが出力される。

```console
...
Oracle Database container image for 'ee' version 21.3.0 is ready to be extended:
    --> oracle/database:21.3.0-ee
Build completed in 404 seconds.
```

イメージ生成結果の確認:

```console
$ docker image ls

REPOSITORY                     TAG         IMAGE ID       CREATED         SIZE
oracle/database                21.3.0-ee   9760ae144b78   2 minutes ago   8.01GB
oraclelinux                    7-slim      0a28ba78f4c9   2 months ago    132MB
```

### Run Docker Image

コンテナを起動する。

```bash
docker run --name ora21c \
    -p 1521:1521 -p 5500:5500 \
    -e ORACLE_EDITION=enterprise \
    -v oradata:/opt/oracle/oradata \
    oracle/database:21.3.0-ee
```

設定値一覧:

| Parameters | Value |
| --- | --- |
| ORACLE_SID | ORCLCDB |
| ORACLE_PDB | ORCLPDB1 |
| ORACLE_PWD | 起動ログへ出力 (ORACLE PASSWORD FOR SYS, SYSTEM AND PDBADMIN) |
| ORACLE_CHARACTERSET | AL32UTF8 |
| ENABLE_ARCHIVELOG | false |

### Connect Database

コンテナ内に入り、CDB へ接続する。

```bash
docker exec -it ora21c bash
```

```sql
$ export ORACLE_SID=ORCLCDB
$ sqlplus / as sysdba

SQL*Plus: Release 21.0.0.0.0 - Production on Wed Sep 15 12:36:21 2021
Version 21.3.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.

Connected to:
Oracle Database 21c Enterprise Edition Release 21.0.0.0.0 - Production
Version 21.3.0.0.0

SQL> select * from v$instance;
SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 ORCLPDB1                       READ WRITE NO

SQL> show con_name

CON_NAME
------------------------------

CDB$ROOT
```

PDB へスイッチする。

```sql
SQL> connect sys/<your password>@//localhost:1521/ORCLPDB1 as sysdba
Connected.

SQL> show con_name

CON_NAME
------------------------------
ORCLPDB1
```

## EM Express

ポート 5500 は EM Express (Oracle Enterprise Manager Database Express) で使用する。
ホストのブラウザから以下に接続する。

- <https://localhost:5500/em/>

ログインアカウントには SYSDBA ロールが必要となる。

## Troubleshoots

### [SOLVED] ORA-12162: TNS:net service name is incorrectly specified

エラー現象:

```console
[oracle@97308d7a6c2c ~]$ sqlplus / as sysdba

ERROR:
ORA-12162: TNS:net service name is incorrectly specified
```

原因と対処:
環境変数 `ORACLE_SID` が未設定のため発生する。

```console
[oracle@97308d7a6c2c ~]$ export ORACLE_SID=ORCLCDB
[oracle@97308d7a6c2c ~]$ sqlplus / as sysdba

Connected to:
Oracle Database 21c Enterprise Edition Release 21.0.0.0.0 - Production
Version 21.3.0.0.0
```
