# Oracle MultiTenant Database Tips of Managing Pluggable Databases

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Local Relocation using XML](#local-relocation-using-xml)
  - [Preparation](#preparation)
  - [Unplug PDB using XML](#unplug-pdb-using-xml)
  - [Compatibility check](#compatibility-check)
  - [Plug PDB using XML](#plug-pdb-using-xml)
- [Local Relocation using PDB Archive](#local-relocation-using-pdb-archive)
  - [Preparation](#preparation-1)
  - [Unplug PDB using PDB Archive](#unplug-pdb-using-pdb-archive)
  - [Compatibility check](#compatibility-check-1)
  - [Plug PDB using PDB Archive](#plug-pdb-using-pdb-archive)
- [References](#references)

## Overview

マルチテナントにおける Pluggable Database (PDB) の操作・再配置に関するメモ。
文字セット (character set) は PDB 個別で変更できるわけではなく、異なる character set の CDB から PDB を Plug することで対応する。
必要に応じて Shift_JIS 用の PDB$SEED を用意しておくとよい。

## Environments

- 2021-09-14: Oracle Database 18c Express Edition (18.4) on Oracle Linux 7.9

## Local Relocation using XML

簡易な方法。使用していない PDB をメモリ節約のため保存しておき、後で使用する場合などに適している。

1. PDB の切断 (Unplug): XML メタデータ・ファイルを作成し、切断する。
2. PDB の削除 (Drop): データファイルを保持したまま削除する。
3. PDB の再接続 (Plug): XML メタデータ・ファイルとデータファイルを使用して再接続する。

### Preparation

必要な環境変数を確認し、メタデータの出力先フォルダを作成する。
sqlplus で CDB へ接続した後、PDB を Close する。

```bash
[oracle@41f115dfbb6e /]$ printenv | grep ^ORACLE
ORACLE_DOCKER_INSTALL=true
ORACLE_SID=XE
ORACLE_BASE=/opt/oracle
ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE

# Create Directory
[oracle@41f115dfbb6e /]$ mkdir -p /opt/oracle/share/unplug
[oracle@41f115dfbb6e /]$ sqlplus / as sysdba

SQL*Plus: Release 18.0.0.0.0 - Production on Sat Sep 18 08:56:32 2021
Version 18.4.0.0.0

Copyright (c) 1982, 2018, Oracle.  All rights reserved.

Connected to:
Oracle Database 18c Express Edition Release 18.0.0.0.0 - Production
Version 18.4.0.0.0
```

```sql
SQL> show con_name

CON_NAME
------------------------------
CDB$ROOT

SQL> alter pluggable database xepdb2 close immediate;
alter pluggable database xepdb2 close immediate
*
ERROR at line 1:
ORA-65020: pluggable database XEPDB2 already closed
```

### Unplug PDB using XML

UNPLUG INTO 句に XML メタデータ・ファイルの名前と場所を指定する。

```sql
-- on CDB
SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         MOUNTED

-- Unplug
SQL> alter pluggable database XEPDB2 unplug into '/opt/oracle/share/unplug/XEPDB2.xml';

Pluggable database altered.

-- Drop (keep datafiles)
SQL> drop pluggable database xepdb2 keep datafiles;

Pluggable database dropped.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
```

### Compatibility check

PDB と CDB の互換性をチェックする。出力結果が「NO」の場合は互換性がないため、PDB_PLUG_IN_VIOLATIONS ビューを確認する。

```sql
-- on CDB
SQL> SET SERVEROUTPUT ON
SQL> DECLARE
       compatible CONSTANT VARCHAR2(3) :=
          CASE DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
                pdb_descr_file => '/opt/oracle/share/unplug/XEPDB2.xml',
                pdb_name       => 'XEPDB2')
         WHEN TRUE THEN 'YES'
         ELSE 'NO'
     END;
     BEGIN
        DBMS_OUTPUT.PUT_LINE(compatible);
     END;
     /
YES

PL/SQL procedure successfully completed.
```

### Plug PDB using XML

XML メタデータ・ファイルを指定する。PDB のデータファイルは削除していないため、NOCOPY を指定する。

```sql
-- on CDB
SQL> create pluggable database XEPDB2 using '/opt/oracle/share/unplug/XEPDB2.xml' nocopy;

Pluggable database created.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         MOUNTED

SQL> alter pluggable database XEPDB2 open;

Pluggable database altered.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         READ WRITE NO

SQL> conn system/<your password>@//localhost:1521/xepdb2
Connected.
```

## Local Relocation using PDB Archive

アーカイブ形式による再配置。

1. PDB の切断 (Unplug): PDB アーカイブ・ファイル (XML メタデータ・ファイルと PDB のデータファイルなどを含む圧縮ファイル) を作成し、切断する。
2. PDB の削除 (Drop): データファイルも削除する。
3. PDB の再接続 (Plug): PDB アーカイブ・ファイルを使用して再接続する。

### Preparation

「Local Relocation using XML」の準備手順と同様。

### Unplug PDB using PDB Archive

UNPLUG INTO 句に PDB アーカイブ・ファイルの名前と場所を指定する。

```sql
-- on CDB
SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         MOUNTED

-- Unplug
SQL> alter pluggable database XEPDB2 unplug into '/opt/oracle/share/unplug/XEPDB2.pdb';

Pluggable database altered.

-- Drop (including datafiles)
SQL> drop pluggable database XEPDB2 including datafiles;

Pluggable database dropped.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
```

### Compatibility check

PDB と CDB の互換性をチェックする。

```sql
-- on CDB
SQL> SET SERVEROUTPUT ON
SQL> DECLARE
       compatible CONSTANT VARCHAR2(3) :=
          CASE DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
                pdb_descr_file => '/opt/oracle/share/unplug/XEPDB2.pdb',
                pdb_name       => 'XEPDB2')
         WHEN TRUE THEN 'YES'
         ELSE 'NO'
     END;
     BEGIN
        DBMS_OUTPUT.PUT_LINE(compatible);
     END;
     /
YES

PL/SQL procedure successfully completed.
```

### Plug PDB using PDB Archive

PDB アーカイブ・ファイルを指定する。データファイルは削除済みのため、FILE_NAME_CONVERT 句を指定してアーカイブからコピーされるように指定する。

```sql
-- on CDB
SQL> create pluggable database XEPDB2 using '/opt/oracle/share/unplug/XEPDB2.pdb'
     FILE_NAME_CONVERT = ('/opt/oracle/share/unplug/', '/opt/oracle/oradata/XE/XEPDB2plug/')
     /

Pluggable database created.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         MOUNTED

SQL> alter pluggable database XEPDB2 open;

Pluggable database altered.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         READ WRITE NO

SQL> conn system/<your password>@//localhost:1521/xepdb2
Connected.
```

PDB アーカイブ・ファイル出力例:

```console
$ ls -lh /opt/oracle/share/unplug/

total 180M
-rw-r--r--. 1 oracle oinstall 180M Sep 18 09:58 XEPDB2.pdb
-rw-r--r--. 1 oracle oinstall 7.1K Sep 18 09:02 XEPDB2.xml
```

## References

- [Multitenant : Overview of Container Databases (CDB) and Pluggable Databases (PDB)](https://oracle-base.com/articles/12c/multitenant-overview-container-database-cdb-12cr1)
