# Oracle Database Tips of MultiTenant use JA16SJISTILDE

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Pluggable SJIS PDB on Docker](#pluggable-sjis-pdb-on-docker)
  - [Create Destination Database](#create-destination-database)
  - [Create Source Database](#create-source-database)
  - [Create Source PDB](#create-source-pdb)
  - [Plug SJIS PDB](#plug-sjis-pdb)
- [Remote Clone SJIS PDB on Docker](#remote-clone-sjis-pdb-on-docker)
  - [Create Destination Database](#create-destination-database-1)
  - [Create Source Database](#create-source-database-1)
  - [Create Source PDB](#create-source-pdb-1)
  - [Remote Clone SJIS PDB](#remote-clone-sjis-pdb)
- [References](#references)

## Overview

結局仕事でShift_JISのデータベースはまだまだ生き残るらしい。さっさとなくなればいいのに。
Oracleでマルチキャラクターセットって聞いてたけど。PDBの移動でホストはできるけど別途キャラクタセットを指定したPDBを作成できるわけでは無いらしい。

## Environments

- 2021-09-23: Oracle Database 18c Express (18.4) on Oracle Linux 7.9

## Pluggable SJIS PDB on Docker

Dockerホストに共有フォルダを作成してアーカイブファイルを作成し、アーカイブファイル経由でUNICODEデータベースへSJISをホストさせます。

### Create Destination Database

共有フォルダを作成するのがポイント。

```bash
docker run --name ora18xe \
        -p 1521:1521 \
        -v oradata_xe:/opt/oracle/oradata \
        -v <your host folder>/opt/oracle/share \
        oracle/database:18.4.0-xe

docker exec -it --user oracle ora18xe base
```

```bash
# in container
sqlplus / as sysdba
```

```sql
SQL> COLUMN parameter FORMAT A30
SQL> COLUMN value FORMAT A30
SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               AL32UTF8
```

### Create Source Database

Shift_JISデータベースを作成する。

```bash
docker run --name ora18sjis --rm \
        -p 1522:1521 \
        -e ORACLE_CHARACTERSET=JA16SJISTILDE \
        -v <your host folder>/opt/oracle/share \
        oracle/database:18.4.0-xe

docker exec -it --user oracle ora18sjis base
```

```bash
# in container
sqlplus / as sysdba
```

```sql
SQL> COLUMN parameter FORMAT A30
SQL> COLUMN value FORMAT A30
SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               JA16SJISTILDE
```

### Create Source PDB

PDB名 XESJIS$SEED はOKらしい、XE$SJISはダメだった。

```sql
-- on ora18sjis
SQL> create pluggable database xesjis$seed admin user pdbadmin identified by "<your password>" 
     file_name_convert = ('/opt/oracle/oradata/XE/pdbseed/', '/opt/oracle/oradata/XE/xesjisseed/');
Pluggable database created.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         5 XESJIS$SEED                    MOUNTED

SQL> alter pluggable database XESJIS$SEED unplug into '/opt/oracle/share/unplug/XESJISSEED.pdb';
Pluggable database altered.

SQL> !ls -lh /opt/oracle/share/unplug/XESJISSEED.pdb
-rw-r--r--. 1 oracle oinstall 156M Sep 23 08:50 /opt/oracle/share/unplug/XESJISSEED.pdb
```

### Plug SJIS PDB

```sql
-- on ora18xe
SQL> !ls -lh /opt/oracle/share/unplug/XESJISSEED.pdb
-rw-r--r--. 1 oracle oinstall 156M Sep 23 08:50 /opt/oracle/share/unplug/XESJISSEED.pdb

SQL> SET SERVEROUTPUT ON
SQL> DECLARE
  compatible CONSTANT VARCHAR2(3) := 
     CASE DBMS_PDB.CHECK_PLUG_COMPATIBILITY(
           pdb_descr_file => '/opt/oracle/share/unplug/XESJISSEED.pdb',
           pdb_name       => 'XESJIS$SEED')
    WHEN TRUE THEN 'YES'
    ELSE 'NO'
END;
BEGIN
   DBMS_OUTPUT.PUT_LINE(compatible);
END;
/
YES

PL/SQL procedure successfully completed.

-- !!Creating!!
SQL> create pluggable database XESJIS$SEED using '/opt/oracle/share/unplug/XESJISSEED.pdb'
  FILE_NAME_CONVERT = ('/opt/oracle/share/unplug/', '/opt/oracle/oradata/XE/XESJISSEED/');
Pluggable database created.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         4 XEPDB2                         MOUNTED
         5 XESJIS$SEED                    MOUNTED

SQL> alter pluggable database XESJIS$SEED open;
Pluggable database altered.

SQL> SET LINESIZE 200
SQL> COLUMN time FORMAT A30
SQL> COLUMN name FORMAT A30
SQL> COLUMN cause FORMAT A30
SQL> COLUMN message FORMAT A30
SQL> SELECT time, name, cause, message FROM pdb_plug_in_violations
     WHERE time > TRUNC(SYSTIMESTAMP) ORDER BY time;

TIME                           NAME                           CAUSE                          MESSAGE
------------------------------ ------------------------------ ------------------------------ ------------------------------
23-SEP-21 08.56.18.565581 AM   XESJIS$SEED                    PDB not Unicode                Character set mismatch: PDB ch
                                                                                             aracter set JA16SJISTILDE. CDB
                                                                                             character set AL32UTF8.

SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               AL32UTF8

SQL> ALTER SESSION SET CONTAINER=XESJIS$SEED;
Session altered.

SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               JA16SJISTILDE
```

## Remote Clone SJIS PDB on Docker

### Create Destination Database

あまりかわらない。

```bash
$ docker run --name ora18xe \
        -p 1521:1521 \
        -v oradata_xe:/opt/oracle/oradata \
        oracle/database:18.4.0-xe

docker exec -it --user oracle ora18xe bash
```

<!-- spell-checker: disable -->
```console
# in container
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
9: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

$ sqlplus / as sysdba
```
<!-- spell-checker: enable -->

```sql
SQL> COLUMN parameter FORMAT A30
SQL> COLUMN value FORMAT A30
SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               AL32UTF8
```

### Create Source Database

Shift_JISデータベースを作成する。
Dockerコンテナ間でRemoteクローンするのでipアドレスを確認しておきます。

```bash
docker run --name ora18sjis --rm \
        -p 1522:1521 \
        -e ORACLE_CHARACTERSET=JA16SJISTILDE \
        oracle/database:18.4.0-xe

docker exec -it --user oracle ora18sjis bash
```

<!-- spell-checker: disable -->
```console
# in container
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
11: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

$ sqlplus / as sysdba
```
<!-- spell-checker: enable -->

```sql
SQL> COLUMN parameter FORMAT A30
SQL> COLUMN value FORMAT A30
SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               JA16SJISTILDE
```

### Create Source PDB

リモートホットクローンするのでいろいろ設定します。

```sql
-- on source CDB
SQL> create pluggable database xesjis$seed admin user pdbadmin identified by "<your password>" 
     file_name_convert = ('/opt/oracle/oradata/XE/pdbseed/', '/opt/oracle/oradata/XE/xesjisseed/');
Pluggable database created.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         5 XESJIS$SEED                    MOUNTED

-- Create Database link user
SQL> CREATE USER c##remote_clone_user IDENTIFIED BY remote_clone_user CONTAINER=ALL;
User created.

SQL> GRANT CREATE SESSION, CREATE PLUGGABLE DATABASE TO c##remote_clone_user CONTAINER=ALL;
Grant succeeded.

-- Check Hot Clone
SQL> SHUTDOWN IMMEDIATE
SQL> STARTUP MOUNT
SQL> ALTER DATABASE ARCHIVELOG;
SQL> ALTER DATABASE OPEN;
SQL> SELECT property_name, property_value FROM database_properties WHERE property_name = 'LOCAL_UNDO_ENABLED';

PROPERTY_NAME                                                                   PROPERTY_VALUE
-------------------------------------------------------------------------------- --------------------------------------------------------------------------------
LOCAL_UNDO_ENABLED                                                              TRUE

SQL> SELECT log_mode FROM v$database;

LOG_MODE
------------
ARCHIVELOG
```

### Remote Clone SJIS PDB

DB Link経由でリモートクローンします。

```sql
-- on destination CDB
SQL> CREATE DATABASE LINK clone_link CONNECT TO c##remote_clone_user IDENTIFIED BY remote_clone_user USING '//172.17.0.3:1521/XE';
Database link created.

SQL> DESC user_tables@clone_link 
 Name                                      Null?    Type
 ----------------------------------------- -------- ----------------------------
 TABLE_NAME                                NOT NULL VARCHAR2(128)
 TABLESPACE_NAME                                    VARCHAR2(30)
 CLUSTER_NAME                                       VARCHAR2(128)
...

-- !!Cloning!!
SQL> create PLUGGABLE DATABASE XESJIS$SEED FROM XESJIS$SEED@clone_link 
     file_name_convert = ('/opt/oracle/oradata/XE/XESJISSEED/', '/opt/oracle/oradata/XE/XESJISSEED/');
Pluggable database created.

SQL> show pdbs

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
         5 XESJIS$SEED                    MOUNTED

SQL> ALTER PLUGGABLE DATABASE XESJIS$SEED OPEN;
Pluggable database altered.

SQL> SET LINESIZE 200
SQL> COLUMN time FORMAT A30
SQL> COLUMN name FORMAT A30
SQL> COLUMN cause FORMAT A30
SQL> COLUMN message FORMAT A30
SQL> SELECT time, name, cause, message FROM pdb_plug_in_violations
     WHERE time > TRUNC(SYSTIMESTAMP) ORDER BY time;

TIME                           NAME                           CAUSE                          MESSAGE
------------------------------ ------------------------------ ------------------------------ ------------------------------
23-SEP-21 08.56.18.565581 AM   XESJIS$SEED                    PDB not Unicode                Character set mismatch: PDB ch
                                                                                             aracter set JA16SJISTILDE. CDB
                                                                                             character set AL32UTF8.

SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               AL32UTF8

SQL> ALTER SESSION SET CONTAINER=XESJIS$SEED;
Session altered.

SQL> SELECT * FROM nls_database_parameters WHERE parameter = 'NLS_CHARACTERSET';

PARAMETER                      VALUE
------------------------------ ------------------------------
NLS_CHARACTERSET               JA16SJISTILDE
```

## References

- [Multitenant : Overview of Container Databases (CDB) and Pluggable Databases (PDB)](https://oracle-base.com/articles/12c/multitenant-overview-container-database-cdb-12cr1)
- [Multitenant : PDBs With Different Character Sets to the CDB in Oracle Database 12c Release 2 (12.2)](https://oracle-base.com/articles/12c/multitenant-pdb-character-sets-12cr2)
