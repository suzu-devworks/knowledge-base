# Oracle MultiTenant Database Tips

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Composition](#composition)
- [Shutdown and Startup](#shutdown-and-startup)
  - [Restart CDB](#restart-cdb)
  - [Restart PDB on PDB](#restart-pdb-on-pdb)
  - [Restart PDB on CDB](#restart-pdb-on-cdb)
  - [Restart PDB$SEED on CDB](#restart-pdbseed-on-cdb)
  - [Configure Auto Restart (Save State)](#configure-auto-restart-save-state)
- [Create PDB](#create-pdb)
  - [From Scratch (PDB$SEED)](#from-scratch-pdbseed)
  - [Cold Cloning a Local PDB](#cold-cloning-a-local-pdb)
  - [Hot Cloning a Local PDB](#hot-cloning-a-local-pdb)
  - [Refreshable Clone PDBs](#refreshable-clone-pdbs)
- [Removing PDB](#removing-pdb)
- [References](#references)

## Overview

12c で導入され、20c からは必須になる Multitenant 構成の整理メモです。Enterprise エディションでは高額なオプションとして提供され、Standard Edition 2 (19c 以降) では最大 3 PDB まで作成可能です。
シングルインスタンス構成における CDB/PDB の使い方や操作コマンドを整理しています。

## Environments

- 2021-09-14: Oracle Database 18c Express Edition (18.4) on Oracle Linux 7.9

## Composition

| CON_NAME | Description |
| --- | --- |
| CDB$ROOT | Oracle提供のメタデータおよび共通ユーザーが格納されます。 |
| PDB$SEED | PDB作成用のテンプレートです。 |
| PDB | ユーザー用のDBです。 |

## Shutdown and Startup

### Restart CDB

CDBを停止する場合は PDB もまとめて停止します。
開始の場合、設定によっては PDB が自動起動します。
状態の遷移（IDLE → NOMOUNT → MOUNT → OPEN）は従来のシングル構成と変わりません。

```sql
-- On CDB
SQL> show con_name
CON_NAME
------------------------------
CDB$ROOT

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO

-- Shutdown (OPEN → IDLE)
SQL> shutdown immediate
Database closed.
Database dismounted.
ORACLE instance shut down.

-- Startup (IDLE → NOMOUNT)
SQL> startup nomount
ORACLE instance started.
Total System Global Area  624949720 bytes
Fixed Size                  8899032 bytes
Variable Size             310378496 bytes
Database Buffers          301989888 bytes
Redo Buffers                3682304 bytes

SQL> show con_name
CON_NAME
------------------------------
CDB$ROOT

-- Startup (NOMOUNT → MOUNT)
SQL> alter database mount;
Database altered.

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       MOUNTED
         3 XEPDB1                         MOUNTED

SQL> select instance_number,instance_name,status from v$instance;
INSTANCE_NUMBER INSTANCE_NAME    STATUS
--------------- ---------------- --------
              1 XE               MOUNTED
1 row selected.

-- Startup (MOUNT → OPEN)
SQL> alter database open;
Database altered.

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO

SQL> select instance_number,instance_name,status from v$instance;
INSTANCE_NUMBER INSTANCE_NAME    STATUS
--------------- ---------------- ------------
              1 XE               OPEN
```

### Restart PDB on PDB

```sql
-- On PDB
SQL> conn sys/<your password>@xepdb1 as sysdba
Connected.

SQL> show con_name
CON_NAME
------------------------------
XEPDB1

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         3 XEPDB1                         READ WRITE NO

-- Shutdown PDB (OPEN → MOUNT)
SQL> shutdown immediate
Pluggable Database closed.

SQL> show con_name
CON_NAME
------------------------------
XEPDB1

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         3 XEPDB1                         MOUNTED

-- Startup PDB (MOUNT → OPEN)
SQL> startup open
Pluggable Database opened.

-- or
SQL> alter database open;

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         3 XEPDB1                         READ WRITE NO
```

### Restart PDB on CDB

```sql
-- On CDB
SQL> alter session set container=CDB$ROOT;
Session altered.

SQL> show con_name
CON_NAME
------------------------------
CDB$ROOT

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB1                           READ WRITE NO
         4 PDB2                           READ WRITE NO

-- All PDBs (Exclude PDB$SEED)
SQL> alter pluggable database all close immediate;

-- All PDBs (Exclude PDB$SEED, PDB2)
SQL> alter pluggable database all except PDB2 close immediate;

-- Select PDB
SQL> alter pluggable database PDB1 close immediate;

-- Select PDBs
SQL> alter pluggable database PDB1,PDB2 close immediate;

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB1                           MOUNTED
         4 PDB2                           MOUNTED

-- All PDBs (Exclude PDB$SEED)
SQL> alter pluggable database all open;

-- All PDBs (Exclude PDB$SEED, PDB2)
SQL> alter pluggable database all except PDB2 open;

-- Select PDB
SQL> alter pluggable database PDB1 open;

-- Select PDBs
SQL> alter pluggable database PDB1,PDB2 open;

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 PDB1                           READ WRITE NO
         4 PDB2                           READ WRITE NO
```

### Restart PDB$SEED on CDB

```sql
-- On CDB
SQL> alter session set container=CDB$ROOT;
Session altered.

SQL> show con_name
CON_NAME
------------------------------
CDB$ROOT

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO

-- Shutdown PDB (OPEN → MOUNT)
SQL> alter pluggable database pdb$seed close immediate;
Pluggable database altered.

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       MOUNTED
         3 XEPDB1                         READ WRITE NO

-- Startup PDB (OPEN → MOUNT) Mistake!!
SQL> alter pluggable database pdb$seed open;
Pluggable database altered.

SQL> show pdbs;
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ WRITE NO
         3 XEPDB1                         READ WRITE NO

-- Re-Shutdown PDB (OPEN → MOUNT)
SQL> alter pluggable database pdb$seed close immediate;
Pluggable database altered.

-- Startup PDB (OPEN → MOUNT read only)
SQL> alter pluggable database pdb$seed open read only;
Pluggable database altered.

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
```

### Configure Auto Restart (Save State)

```sql
-- On CDB
SQL> alter session set container=CDB$ROOT;
Session altered.

SQL> show con_name
CON_NAME
------------------------------
CDB$ROOT

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO

-- Shutdown PDB (OPEN → MOUNT)
SQL> alter pluggable database xepdb1 close immediate;

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         MOUNTED

-- Save State!!
SQL> alter pluggable database all save state;
Pluggable database altered.

-- Restart CDB
SQL> shutdown immediate;
Database closed.
Database dismounted.
ORACLE instance shut down.
ERROR:
ORA-12514: TNS:listener does not currently know of service requested in connect descriptor
Warning: You are no longer connected to ORACLE.

SQL> conn / as sysdba
Connected to an idle instance.

SQL> startup open;
ORACLE instance started.
Total System Global Area  624949720 bytes
Fixed Size                  8899032 bytes
Variable Size             310378496 bytes
Database Buffers          301989888 bytes
Redo Buffers                3682304 bytes
Database mounted.
Database opened.

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         MOUNTED
```

## Create PDB

### From Scratch (PDB$SEED)

XE 環境でも PDB の新規作成をテストできます。

- [PDBおよびアプリケーション・コンテナの作成および削除](https://docs.oracle.com/cd/F19136_01/multi/creating-pdbs.html#GUID-9A250D93-5B3B-4643-BE85-F32CC7B0E413)

```sql
-- On CDB
SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO

SQL> alter session set container=PDB$SEED;
Session altered.

SQL> show con_name
CON_NAME
------------------------------
PDB$SEED

SQL> select file_name from dba_data_files;
FILE_NAME
--------------------------------------------------------------------------------
/opt/oracle/oradata/XE/pdbseed/system01.dbf
/opt/oracle/oradata/XE/pdbseed/sysaux01.dbf
/opt/oracle/oradata/XE/pdbseed/undotbs01.dbf

SQL> alter session set container=CDB$ROOT;
Session altered.

SQL> show con_name
CON_NAME
------------------------------
CDB$ROOT

-- Creation
SQL> create pluggable database xepdb2 admin user pdb2admin identified by "<your password>" 
  file_name_convert = ('/opt/oracle/oradata/XE/pdbseed/', '/opt/oracle/oradata/XE/xepdb2/');
Pluggable database created.

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         MOUNTED

SQL> connect sys/98f76e0a6e8b762d@//localhost/xepdb2 as sysdba
Connected.

SQL> startup 
Pluggable Database opened.

SQL> select file_name from dba_data_files;
FILE_NAME
--------------------------------------------------------------------------------
/opt/oracle/oradata/XE/xepdb2/system01.dbf
/opt/oracle/oradata/XE/xepdb2/sysaux01.dbf
/opt/oracle/oradata/XE/xepdb2/undotbs01.dbf
```

### Cold Cloning a Local PDB

コピー元 (Source) の PDB を読み取り専用 (READ ONLY) に変更し、更新が発生しない状態でクローンします。
クローン実行中はコピー元 PDB を更新することはできません。

```sql
-- On CDB
SQL> show pdbs
    CON_ID CON_NAME     OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED     READ ONLY  NO
         3 XEPDB1     READ WRITE NO

-- Change to READ ONLY
SQL> alter pluggable database xepdb1 close immediate;
Pluggable database altered.

SQL> alter pluggable database xepdb1 open read only;
Pluggable database altered.

SQL> show pdbs
    CON_ID CON_NAME     OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED     READ ONLY  NO
         3 XEPDB1     READ ONLY  NO

-- Cloning
SQL> create pluggable database XEPDB2 from XEPDB1 
  path_prefix = '/opt/oracle/oradata/XE/XEPDB2'
  file_name_convert = ('/opt/oracle/oradata/XE/XEPDB1/', '/opt/oracle/oradata/XE/XEPDB2/');
Pluggable database created.

SQL> show pdbs
    CON_ID CON_NAME     OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED     READ ONLY  NO
         3 XEPDB1     READ ONLY  NO
         4 XEPDB2     MOUNTED
```

- 補足: Remote PDB の場合は Database Link を使用し `pdb1@dblink` のように指定して実行します。

### Hot Cloning a Local PDB

Hot（稼働中）クローンを行うには、コピー元 CDB が以下の要件を満たしている必要があります。
これはクローン作成したデータファイルに対して、クローン開始以降のコピー元 UNDO/REDO をクローン先に適用する仕組みとなっているためです。

- CDB がローカル UNDO モードであること
- CDB が ARCHIVELOG モードであること (※テスト環境で ARCHIVELOG まで更新されない場合は NOARCHIVELOG でも動作することがあります)

```sql
-- On CDB
SQL> select property_name, property_value from database_properties where property_name = 'LOCAL_UNDO_ENABLED';
PROPERTY_NAME
--------------------------------------------------------------------------------
PROPERTY_VALUE
--------------------------------------------------------------------------------
LOCAL_UNDO_ENABLED
TRUE

SQL> archive log list
Database log mode        Archive Mode
Automatic archival        Enabled
Archive destination        /opt/oracle/product/18c/dbhomeXE/dbs/arch
Oldest online log sequence     1
Next log sequence to archive   2
Current log sequence        2
```

```sql
-- On CDB
SQL> show pdbs
    CON_ID CON_NAME     OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED     READ ONLY  NO
         3 XEPDB1     READ WRITE NO
         4 XEPDB2     MOUNTED

-- Cloning
SQL> create pluggable database XEPDB3 from XEPDB1 
  path_prefix = '/opt/oracle/oradata/XE/XEPDB3'
  file_name_convert = ('/opt/oracle/oradata/XE/XEPDB1/', '/opt/oracle/oradata/XE/XEPDB3/');
Pluggable database created.

SQL> show pdbs
    CON_ID CON_NAME     OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED     READ ONLY  NO
         3 XEPDB1     READ WRITE NO
         4 XEPDB2     MOUNTED
         5 XEPDB3     MOUNTED
```

### Refreshable Clone PDBs

`REFRESH MODE` オプションを指定することで、クローン先 PDB へソース側の更新内容を後から反映・同期させることができます。

```sql
-- On XEPDB1 
SQL> create table sample_table (id number(10) primary key, name varchar2(200) not null, date_of_birth date);
Table created.

SQL> insert all 
  into sample_table values (1, 'Alice', TO_DATE('2000-02-29','YYYY-MM-DD')) 
  into sample_table values (2, 'Bob', TO_DATE('2000-04-01','YYYY-MM-DD')) 
  into sample_table values (3, 'Charlie', null) 
  select 0 from dual;
3 rows created.

SQL> commit;
Commit complete.

SQL> set pagesize 10000
SQL> column name format a20
SQL> select * from sample_table;
  ID NAME                 DATE_OF_B
---- -------------------- ---------
   1 Alice                29-FEB-00
   2 Bob                  01-APR-00
   3 Charlie
```

```sql
-- On CDB
SQL> show pdbs
    CON_ID CON_NAME     OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED     READ ONLY  NO
         3 XEPDB1     READ WRITE NO

-- Cloning
SQL> create pluggable database XEPDB4 from XEPDB1 refresh mode manual
  path_prefix = '/opt/oracle/oradata/XE/XEPDB4'
  file_name_convert = ('/opt/oracle/oradata/XE/XEPDB1/', '/opt/oracle/oradata/XE/XEPDB4/');
create pluggable database XEPDB4 from XEPDB1 refresh mode manual
                                             *
ERROR at line 1:
ORA-00922: missing or invalid option

-- Cloning
SQL> create pluggable database XEPDB4 from XEPDB1 refresh mode every 2 minutes
  path_prefix = '/opt/oracle/oradata/XE/XEPDB4'
  file_name_convert = ('/opt/oracle/oradata/XE/XEPDB1/', '/opt/oracle/oradata/XE/XEPDB4/');
create pluggable database XEPDB4 from XEPDB1 refresh mode every 2 minutes
                                             *
ERROR at line 1:
ORA-00922: missing or invalid option
```

※ DBLink 経由での指定が必要な場合があります。

## Removing PDB

```sql
-- On CDB
SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         MOUNTED

SQL> alter pluggable database PDB1 close immediate;
Pluggable Database closed.

-- Dropping with datafiles
SQL> drop pluggable database xepdb2 including datafiles;
Pluggable database dropped.

-- Keep datafiles option
SQL> drop pluggable database xepdb2 keep datafiles;

SQL> show pdbs
    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
```

## References

- [ALTER PLUGGABLE DATABASE - Oracle Documentation](https://docs.oracle.com/cd/F19136_01/sqlrf/ALTER-PLUGGABLE-DATABASE.html#GUID-A29491AD-8F0F-4E52-9D94-57FC3FF8FBC7)
- [12c R2のPDB移行・クローン新機能 - Oracle Technical Resources](https://www.oracle.com/jp/technical-resources/article/cosol-db12cr2/cosol.html#2)
