# Oracle Database Express Edition(XE) 18c Installation

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Spec limitations](#spec-limitations)
- [Installation on Docker](#installation-on-docker)
  - [Preparation](#preparation)
  - [Create Docker Image](#create-docker-image)
  - [Run Docker Image](#run-docker-image)
  - [Connect Database](#connect-database)
- [EM Express](#em-express)
- [Use Sample schema](#use-sample-schema)
  - [Download Sample Schema](#download-sample-schema)
  - [Run Create Script](#run-create-script)
- [Troubleshoots](#troubleshoots)
  - [\[SOLVED\] ORA-12547: TNS:lost contact](#solved-ora-12547-tnslost-contact)
- [References](#references)

## Overview

Oracle Database 18c Express Edition（Oracle 18c XE）は、オラクル社が提供する無償の軽量版リレーショナル・データベースです。開発・テストだけでなく、条件を満たせば本番環境でのデプロイ（商用利用）も無償で認められています。

## Environments

- 2021-09-12: Oracle Database 18c Express (18.4), Docker Community 20.10.8 on Fedora 34 (5.13.14-200.fc34.x86_64)
- 2021-09-13: Oracle Database 18c Express (18.4), Docker Desktop 4.0.0 (Docker Engine 20.10.8) on macOS 11.5.2

## Spec limitations
  
- CPU: 最大2コア（スレッド）まで使用可能
- メモリ（RAM）: 最大2GBまで（SGA/PGA合計）
- ユーザーデータ容量: 最大12GBまで
- プラガブルデータベース（PDB）: 最大3つまで作成可能

## Installation on Docker

### Preparation

Dockerfile を公式リポジトリから取得する。

```bash
git clone https://github.com/oracle/docker-images.git
```

Oracle Database 18c Express Edition for Linux x64 をダウンロードし、指定ディレクトリへ移動する。

- [Oracle Database XE 18c Download](https://www.oracle.com/jp/database/technologies/xe18c-downloads.html)

```bash
mv oracle-database-xe-18c-1.0-1.x86_64.rpm ~/repos/docker-images/OracleDatabase/SingleInstance/dockerfiles/18.4.0
```

配置確認:

```console
$ ls

Dockerfile.xe                       oracle-database-xe-18c-1.0-1.x86_64.rpm
setPassword.sh                      checkDBStatus.sh
oracle-xe-18c.conf                  checkSpace.sh
runOracle.sh
```

### Create Docker Image

ビルドコマンドを実行する。

```bash
./buildContainerImage.sh -x -v 18.4.0
```

完了時に以下のメッセージが出力される。

```console
...
Oracle Database container image for 'xe' version 18.4.0 is ready to be extended:
    --> oracle/database:18.4.0-xe
Build completed in 481 seconds.
```

イメージ生成結果の確認:

```console
$ docker image ls

REPOSITORY                     TAG         IMAGE ID       CREATED          SIZE
oracle/database                18.4.0-xe   5f087c7d1933   3 minutes ago    5.89GB
oracle/database                21.3.0-ee   9760ae144b78   18 minutes ago   8.01GB
oraclelinux                    7-slim      0a28ba78f4c9   2 months ago     132MB
```

### Run Docker Image

コンテナを起動する。

```bash
docker run --name ora18xe \
    -p 1521:1521 -p 5500:5500 \
    -v oradata_xe:/opt/oracle/oradata \
    oracle/database:18.4.0-xe
```

設定値一覧:

| Parameters | Value |
| --- | --- |
| ORACLE_SID | XE (変更不可) |
| ORACLE_PDB | XEPDB1 (変更不可) |
| ORACLE_PWD | 起動ログへ出力 (ORACLE PASSWORD FOR SYS, SYSTEM AND PDBADMIN) |
| ORACLE_CHARACTERSET | AL32UTF8 |

起動完了メッセージ:

```console
#########################
DATABASE IS READY TO USE!
#########################
```

### Connect Database

コンテナ内に入り、CDB へ接続する。

```bash
docker exec -it --user oracle ora18xe bash
```

```sql
[oracle@e1b2cc3975c9 /]$ sqlplus sys/<your password>@//localhost:1521/XE as sysdba

SQL*Plus: Release 18.0.0.0.0 - Production on Sun Sep 12 09:50:12 2021
Version 18.4.0.0.0

Copyright (c) 1982, 2018, Oracle.  All rights reserved.

Connected to:
Oracle Database 18c Express Edition Release 18.0.0.0.0 - Production
Version 18.4.0.0.0

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO

SQL> show CON_NAME

CON_NAME
------------------------------
XE
```

PDB へスイッチする。

```sql
SQL> connect sys/<your password>@//localhost:1521/xepdb1 as sysdba
Connected.

SQL> show con_name

CON_NAME
------------------------------
XEPDB1
```

接続文字列の省略パターン:

```sql
SQL> connect sys/<your password>@//localhost:1521/xepdb1 as sysdba [OK]
SQL> connect sys/<your password>@localhost:1521/xepdb1 as sysdba [OK]
SQL> connect sys/<your password>@localhost/xepdb1 as sysdba [OK]
SQL> connect sys/<your password>@xepdb1 as sysdba [OK]
SQL> connect / as sysdba [connect to CDB$ROOT]
```

## EM Express

インストール直後はローカル接続のみのため、Docker ホストからの接続を許可する設定を行う。

```console
[oracle@c5f6c9c17eb3 ~]$ sqlplus /nolog
SQL> conn system/d79c47e9a980043f
Connected.

SQL> EXEC DBMS_XDB.SETLISTENERLOCALACCESS(FALSE);
PL/SQL procedure successfully completed.
```

ブラウザから `https://localhost:5500/em/login` にアクセスする。

> [!WARNING]
> 18c の EM Express は Adobe Flash を使用して作成されているため、Flash サポート終了に伴いそのままでは利用できない点に注意。
> 19c 以降では Oracle JET に変更されている。

## Use Sample schema

### Download Sample Schema

コンテナ内で curl を利用してサンプルスキーマを取得する。

```bash
curl -OL https://github.com/oracle/db-sample-schemas/archive/refs/tags/v18c.zip
```

- [Database Sample Schema](https://docs.oracle.com/cd/F19136_01/comsc/preface.html)

データを保持するため `/opt/oracle/oradata` 以下に解凍する。

```console
unzip v18c.zip
cd db-sample-schemas-18c
```

`__SUB__CWD__` を現在のパスへ置換する。

```bash
grep -lr '__SUB__CWD__' *.sql */*.sql */*.dat | xargs sed -i.bak 's#__SUB__CWD__#'$(pwd)'#g'
```

環境変数を読み込む。

```console
$ source /usr/local/bin/oraenv

ORACLE_SID = [XE] ? 
The Oracle base remains unchanged with value /opt/oracle
```

### Run Create Script

サンプル作成スクリプトを実行する。

```console
$ sqlplus system/<your password>@//localhost:1521/xepdb1

SQL> @mksample <your password> <your password> hrPassword1 oePassword1 pmPassword1 ixPassword1 shPassword1 biPassword1 users temp /opt/oracle/product/18c/dbhomeXE/demo/schema/log/ localhost:1521/xepdb1
```

データの確認:

<!-- spell-checker: disable -->
```console
SQL> select * from hr.employees fetch first 5 rows only;

EMPLOYEE_ID FIRST_NAME           LAST_NAME                 EMAIL
----------- -------------------- ------------------------- -------------------------
PHONE_NUMBER         HIRE_DATE JOB_ID         SALARY COMMISSION_PCT MANAGER_ID
-------------------- --------- ---------- ---------- -------------- ----------
DEPARTMENT_ID
-------------
        100 Steven               King                      SKING
515.123.4567         17-JUN-03 AD_PRES         24000               90
         101 Neena                Kochhar                   NKOCHHAR
515.123.4568         21-SEP-05 AD_VP           17000                       100
         90
        102 Lex                  De Haan                   LDEHAAN
515.123.4569         13-JAN-01 AD_VP           17000                       100
         90
        103 Alexander            Hunold                    AHUNOLD
590.423.4567         03-JAN-06 IT_PROG          9000                       102
         60
        104 Bruce                Ernst                     BERNST
590.423.4568         21-MAY-07 IT_PROG          6000                       103
         60

5 rows selected.
```
<!-- spell-checker: enable -->

## Troubleshoots

### [SOLVED] ORA-12547: TNS:lost contact

エラー現象:

```console
bash-4.2# sqlplus / as sysdba

ERROR:
ORA-12547: TNS:lost contact
```

原因と対処:

root ユーザーで実行したことによる権限問題。`oracle` ユーザーで実行することで解決する。

```console
$ docker exec -it --user oracle e1b2cc3975c9 bash
[oracle@e1b2cc3975c9 /]$ sqlplus / as sysdba

Connected to:
Oracle Database 18c Express Edition Release 18.0.0.0.0 - Production
Version 18.4.0.0.0
```

## References

- [Oracle Database Express Edition Installation Guide, 18c for Linux x86-64](https://docs.oracle.com/cd/E96517_01/xeinl/)
