# MariaDB Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation on Linux](#installation-on-linux)
  - [Use DNF on Fedora](#use-dnf-on-fedora)
  - [Start Server](#start-server)
  - [Setup Default Security](#setup-default-security)
  - [Show Server Info](#show-server-info)
- [Installation with Docker](#installation-with-docker)
  - [Use Docker Hub](#use-docker-hub)
  - [How to set up an initial database in Docker](#how-to-set-up-an-initial-database-in-docker)
- [Install Command Line Tools（`mysql`）](#install-command-line-toolsmysql)
- [Create Remote Access User](#create-remote-access-user)
  - [Create User](#create-user)
  - [Verify User](#verify-user)
- [Configure UTF-8(utf8mb4) Character Set](#configure-utf-8utf8mb4-character-set)
  - [Check the current character encoding settings](#check-the-current-character-encoding-settings)
  - [Server-side configuration](#server-side-configuration)
  - [Client side configuration](#client-side-configuration)
- [Backup](#backup)
- [Reference](#reference)

## Overview

MariaDBは、MySQLの派生として開発されているオープンソースの関係データベース管理システム（RDBMS）です。
MySQLのオリジナルコードの作者である Michael "Monty" Widenius により立ち上げられたプロジェクトによって開発が進められています。
配布ライセンスは GNU General Public License (GPL) バージョン2 です。MariaDBの名前は、Wideniusの2番目の娘の名前に由来しています。

- [MariaDB Official Site](https://mariadb.org/)

<!-- spell-checker: words rdbms  Widenius -->

## Environments

- 2018-06-11: MariaDB 10.3.7 on Docker
- 2018-06-14: MariaDB 3:10.2.14-1.fc28 on Fedora 28

## Installation on Linux

### Use DNF on Fedora

DNFパッケージマネージャーを使用して MariaDB サーバーをインストールします。

```bash
sudo dnf install mariadb-server
```

### Start Server

サービスの状態を確認し、起動します。

```bash
systemctl list-unit-files --type=service | grep mariadb
```

```console
mariadb.service                             disabled     
mariadb@.service                            disabled     
```

```bash
sudo systemctl start mariadb
sudo systemctl status mariadb
```

```console
● mariadb.service - MariaDB 10.2 database server
       Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
       Active: active (running) since Sun 2018-06-17 13:06:48 JST; 2min 13s ago
     Main PID: 5988 (mysqld)
       Status: "Taking your SQL requests now..."
        Tasks: 35 (limit: 4915)
       Memory: 74.2M
     CGroup: /system.slice/mariadb.service
               └─5988 /usr/libexec/mysqld --basedir=/usr
```
<!-- spell-checker: words libexec mysqld  -->

```bash
sudo systemctl stop mariadb
journalctl -xe -u mariadb
```
<!-- spell-checker: words journalctl -->

### Setup Default Security

初期セキュリティ設定スクリプトを実行し、不要なユーザーやテストDBの削除、ルートパスワードの設定を行います。

```bash
sudo mysql_secure_installation
```

実行時のインタラクティブな入力例：

1. `Enter current password for root (enter for none):` [Enter]
2. `Set root password? [Y/n]` y
3. `New password:` （任意のパスワードを設定）
4. `Remove anonymous users? [Y/n]` y
5. `Disallow root login remotely? [Y/n]` y
6. `Remove test database and access to it? [Y/n]` y
7. `Reload privilege tables now? [Y/n]` y

### Show Server Info

接続してデータベース一覧やルートユーザーの接続元、ポートを確認します。

```bash
mysql -u root -p
```

```sql
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]> select host,user,plugin from mysql.user;
+-----------+------+--------+
| host      | user | plugin |
+-----------+------+--------+
| localhost | root |        |
| 127.0.0.1 | root |        |
| ::1       | root |        |
+-----------+------+--------+
3 rows in set (0.00 sec)

MariaDB [(none)]> show variables like 'port';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| port          | 3306  |
+---------------+-------+
1 row in set (0.00 sec)
```

## Installation with Docker

### Use Docker Hub

- [mariadb - Docker Hub](https://store.docker.com/images/mariadb)

```bash
docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=***** -d mariadb:10.3
```

| オプション | 説明 |
| :- | :- |
| `VOLUME /var/lib/mysql` | データベースの永続化ボリューム |
| `EXPOSE 3306` | コンテナのポート公開 |
| `MYSQL_ROOT_PASSWORD` | root ユーザーのパスワード設定 |

### How to set up an initial database in Docker

ホスト側のローカルディレクトリを `/docker-entrypoint-initdb.d` にマウントすることで、起動時に初期化スクリプトを実行可能です。

作成例 (`100_create.sql`):

```sql
create database test_db1;
create user test_db1_user1 identified by 'test_db1_user1';
grant SELECT,INSERT,UPDATE,DELETE,CREATE,DROP on test_db1.* to test_db1_user1;
```

> `sql.gz` 等でデータを流し込む際、データベース選択がされていないとエラーになる場合があります。
> `MYSQL_DATABASE` 環境変数を指定するか、SQL内に `CREATE DATABASE` / `USE` 文を含める必要があります。

初期化用SQLを配置してコンテナを起動します。

```bash
ls initdb.d
```

```console
100_create.sql  200_scott.sql.gz
```

```bash
docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=***** \
     -e MYSQL_DATABASE=test_db1 \
     --mount type=bind,source="$(pwd)"/initdb.d,target=/docker-entrypoint-initdb.d \
     -d mariadb:10.3
```

```bash
mysql -h 127.0.0.1 -P 3306 -u test_db1_user1 -p test_db1
```

```sql
-- Show tables in test_db1
MySQL [test_db1]> show tables;
+--------------------+
| Tables_in_test_db1 |
+--------------------+
| bonus              |
| dept               |
| emp                |
| salgrade           |
+--------------------+
4 rows in set (0.01 sec)
```
<!-- spell-checker: words salgrade -->

## Install Command Line Tools（`mysql`）

ホスト側にクライアントツールをインストールします。

```bash
sudo dnf install mariadb
```

バージョンを確認します。

```bash
mysql --version
```

```console
mysql  Ver 15.1 Distrib 10.2.14-MariaDB, for Linux (x86_64) using readline 5.1
```

## Create Remote Access User

### Create User

コンテナにログインし、リモート接続用のユーザーを作成します（初期状態で root は `%` 接続が可能です）。

```bash
docker exec -it mariadb1 bash
```

```bash
mysql -p
```

```sql
-- Show existing users before creating a new one
MariaDB [(none)]> select host,user,password,plugin from mysql.user;
+-----------+-------+-------------------------------------------+--------+
| host      | user  | password                                  | plugin |
+-----------+-------+-------------------------------------------+--------+
| localhost | root  | *0B025000DCF68B1D374153A7005639E320D41D5B |        |
| %         | root  | *0B025000DCF68B1D374153A7005639E320D41D5B |        |
+-----------+-------+-------------------------------------------+--------+
2 rows in set (0.001 sec)

-- Create a new user for remote access
MariaDB [(none)]> create user ruser identified by '*****';
Query OK, 0 rows affected (0.001 sec)

-- Show users after creating the new one
MariaDB [(none)]> select host,user,password,plugin from mysql.user;
+-----------+-------+-------------------------------------------+--------+
| host      | user  | password                                  | plugin |
+-----------+-------+-------------------------------------------+--------+
| localhost | root  | *0B025000DCF68B1D374153A7005639E320D41D5B |        |
| %         | ruser | *3F3585570B7B5B3F6CE8C8F6E636FCD3A6F02C41 |        |
| %         | root  | *0B025000DCF68B1D374153A7005639E320D41D5B |        |
+-----------+-------+-------------------------------------------+--------+
3 rows in set (0.00 sec)

-- Show grants for the new user
MariaDB [(none)]> show grants for ruser;
+-----------------------------------------------------------------------+
| Grants for ruser@%                                                    |
+-----------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'ruser'@'%' IDENTIFIED BY PASSWORD '*3F35xxxxx' |
+-----------------------------------------------------------------------+
1 row in set (0.000 sec)
```

<!-- spell-checker: words ruser -->

### Verify User

ホスト側から作成したユーザーで接続を確認します（権限を未付与のため操作制限があります）。

```bash
mysql -h 127.0.0.1 -P 3306 -u ruser -p
```

```sql
-- Show databases and grants for the new user
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
+--------------------+
1 row in set (0.00 sec)

-- Show grants for the current user
MariaDB [(none)]> show grants;
+-----------------------------------------------------------------------+
| Grants for ruser@%                                                    |
+-----------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'ruser'@'%' IDENTIFIED BY PASSWORD '*3F35xxxxx' |
+-----------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## Configure UTF-8(utf8mb4) Character Set

### Check the current character encoding settings

デフォルト状態の文字コード設定を確認します。

```sql
MariaDB [(none)]> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

MariaDB [(none)]> show variables like 'collation%';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | utf8_general_ci   |
| collation_database   | latin1_swedish_ci |
| collation_server     | latin1_swedish_ci |
+----------------------+-------------------+
3 rows in set (0.00 sec)
```
<!-- spell-checker: words charsets -->

標準の `utf8` 設定では最大3バイトまでしか扱えないため、絵文字などを扱う場合は `utf8mb4` への変更が必要です。

```sql
MariaDB [(none)]> show character set;
+----------+-----------------------------+---------------------+--------+
| Charset  | Description                 | Default collation   | Maxlen |
+----------+-----------------------------+---------------------+--------+
| utf8     | UTF-8 Unicode               | utf8_general_ci     |      3 |
| utf8mb4  | UTF-8 Unicode               | utf8mb4_general_ci  |      4 |
+----------+-----------------------------+---------------------+--------+
```
<!-- spell-checker: words maxlen -->

### Server-side configuration

Docker コンテナ起動時に文字コードの初期値を設定します。

```bash
docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=**** -d mariadb:10.3 --character-set-server=utf8mb4
```

```bash
mysql -h 127.0.0.1 -u root -p
```

```sql
-- Check current character encoding settings
MariaDB [(none)]> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+

-- Check current collation settings
MariaDB [(none)]> show variables like 'collation%';
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8_general_ci    |
| collation_database   | utf8mb4_general_ci |
| collation_server     | utf8mb4_general_ci |
+----------------------+--------------------+
```

### Client side configuration

クライアント側の設定ファイル (`/etc/my.cnf.d/utf8mb4.cnf`) を修正します。

```ini
[client]
default-character-set=utf8mb4
```

設定後の接続確認結果を確認します。

```bash
mysql -h 127.0.0.1 -u root -p
```

```sql
-- Check current character encoding settings after client configuration
MariaDB [(none)]> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+

-- Check current collation settings after client configuration
MariaDB [(none)]> show variables like 'collation%';
+----------------------+--------------------+
| Variable_name        | Value              |
+----------------------+--------------------+
| collation_connection | utf8mb4_general_ci |
| collation_database   | utf8mb4_general_ci |
| collation_server     | utf8mb4_general_ci |
+----------------------+--------------------+
```

## Backup

バックアップ（ダンプ）の作成：

```bash
mysqldump -h 127.0.0.1 -u ruser -p scott | gzip > scott.sql.gz
```

CREATE DATABASE & USE を含めてダンプする場合:

```bash
mysqldump -h 127.0.0.1 -u ruser -p --databases scott | gzip > scott.sql.gz
```
<!-- spell-checker: words mysqldump -->

## Reference

- [MariaDB Documentation](https://mariadb.com/docs/)
