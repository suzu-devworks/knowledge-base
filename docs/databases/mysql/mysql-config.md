# MySQL Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation Using Generic Binary](#installation-using-generic-binary)
  - [Install Dependency Library](#install-dependency-library)
  - [Download Binary](#download-binary)
  - [Install](#install)
  - [Change root@localhost Password](#change-rootlocalhost-password)
  - [Start Server](#start-server)
  - [Start Server with Systemd](#start-server-with-systemd)
  - [Show Server Info](#show-server-info)
- [Installation with RPM](#installation-with-rpm)
  - [Download Repository File](#download-repository-file)
  - [Install](#install-1)
- [Installation with Docker](#installation-with-docker)
  - [Use Docker Official](#use-docker-official)
  - [Use Oracle Official Container](#use-oracle-official-container)
  - [How to set up an initial database in Docker](#how-to-set-up-an-initial-database-in-docker)
- [Configurations](#configurations)
  - [Change root@localhost Password](#change-rootlocalhost-password-1)
  - [mysql\_secure\_installation](#mysql_secure_installation)
  - [Configuration Password Policy](#configuration-password-policy)
  - [Configuration Character Set and Collation](#configuration-character-set-and-collation)
  - [Create Remote Access User](#create-remote-access-user)
- [Troubleshooting](#troubleshooting)
  - [Unicode Character Issues in Docker](#unicode-character-issues-in-docker)
  - [Remote Host Connection Issues](#remote-host-connection-issues)
- [Reference](#reference)

## Overview

MySQLは高速性と堅牢性を追求したマルチユーザ・マルチスレッドのオープンソースデータベースです。ORACLEに買収された後、本家はMariaDBへ分岐しましたが、商標権および著作権はORACLEに移管されています。
本ドキュメントでは、Linux環境における Generic Binary および RPM を使用したインストール手順と、Dockerコンテナを用いた構築・設定手順についてまとめています。

## Environments

- 2018-06-11: MySQL 8.0.11 on Fedora 28 (Linux Generic Binary & RPM)
- 2018-06-11: MySQL 8.0.11 on Docker

## Installation Using Generic Binary

### Install Dependency Library

依存ライブラリである `libaio` をインストールします。

```bash
sudo dnf install libaio
```

<!-- spell-checker: words libaio -->

### Download Binary

Linux Generic x86 - 64bit のパッケージを公式サイトから取得します。

- [MySQL :: Download MySQL Community Server](https://dev.mysql.com/downloads/mysql/)

### Install

公式マニュアルの手順に沿ってインストールを実行します。

```bash
sudo useradd -r -s /bin/false mysql
cd /usr/local

sudo tar zxvf /path/to/mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz
sudo ln -s /usr/local/mysql-8.0.11-linux-glibc2.12-x86_64/ mysql
cd mysql

# エクスポート／インポート操作を制限するためのフォルダを作成
sudo mkdir mysql-files
sudo chown mysql:mysql mysql-files
sudo chmod 750 mysql-files

# 初期化を実行（実行ログに root@localhost の初期パスワードが表示されます）
sudo bin/mysqld --initialize --user=mysql
# 例: 2018-06-12T00:46:25.017007Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: R)AC&i&t2Uua

sudo bin/mysql_ssl_rsa_setup
sudo bin/mysqld_safe --user=mysql &

# mysql コマンドへのシンボリックリンクを作成
sudo ln -s /usr/local/mysql/bin/mysql /usr/local/bin/
```
<!-- spell-checker: words mysqld zxvf -->

### Change root@localhost Password

初期パスワードを変更しないと操作が制限されます。

```bash
mysql -u root -p
```

初期パスワードを入力してEnterを押します:

```console
Enter password: ********
```

```sql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.11

-- Initial password must be changed
mysql> select host,user,plugin from mysql.user;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.

-- Change the password for root@localhost
mysql> alter user 'root'@'localhost' identified by '********';
Query OK, 0 rows affected (0.09 sec)

-- Verify the password change
mysql> select host,user,plugin from mysql.user;
+-----------+------------------+-----------------------+
| host      | user             | plugin                |
+-----------+------------------+-----------------------+
| localhost | mysql.infoschema | mysql_native_password |
| localhost | mysql.session    | mysql_native_password |
| localhost | mysql.sys        | mysql_native_password |
| localhost | root             | caching_sha2_password |
+-----------+------------------+-----------------------+
4 rows in set (0.00 sec)
```
<!-- spell-checker: words infoschema -->

### Start Server

Start the MySQL server(`mysqld`) using the following command:

```bash
sudo bin/mysqld_safe --user=mysql &
```

Stop the MySQL server using the following command:

```bash
sudo bin/mysqladmin -u root -p shutdown
```
<!-- spell-checker: words mysqladmin -->

### Start Server with Systemd

systemd へ登録するためにサービスファイルを作成します。

ファイルパス: `/etc/systemd/system/mysqld.service`

```ini
[Unit]
Description=MySQL Server
After=network.target

[Service]
Type=simple
PermissionsStartOnly=true
ExecStart=/usr/local/mysql/bin/mysqld
User=mysql
Group=mysql
Restart=always

[Install]
WantedBy=multi-user.target
```

サービスの確認:

```bash
systemctl list-unit-files --type=service | grep mysql
```

サービスの開始:

```bash
sudo systemctl start mysqld
```

サービスの停止:

```bash
sudo systemctl stop mysqld
```

サービスの状態確認:

```bash
sudo systemctl status mysqld
```

サービスのログ確認:

```bash
journalctl -xe -u mysqld
```
<!-- spell-checker: words journalctl -->

### Show Server Info

バージョンの確認:

```bash
bin/mysqladmin -u root -p version
```

接続確認:

```bash
bin/mysqlshow -u root -p
bin/mysqlshow -u root -p mysql
```
<!-- spell-checker: words mysqlshow -->

接続後にサーバ情報を確認:

```bash
mysql -u root -p -s
```

```sql
mysql> show variables like 'port';
```

## Installation with RPM

Fedora標準リポジトリでは MariaDB が提供されているため、MySQL公式リポジトリを追加してインストールします。

### Download Repository File

リポジトリファイル (`mysql80-community-release-fc28-1.noarch.rpm`) を取得します。

- [MySQL :: Download MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/)

<!-- spell-checker: words noarch -->

### Install

```bash
sudo dnf localinstall ./mysql80-community-release-fc28-1.noarch.rpm
sudo dnf repolist enabled
sudo dnf install mysql-community-server
```
<!-- spell-checker: words localinstall repolist -->

## Installation with Docker

### Use Docker Official

Docker Hub の公式イメージを使用します。こちらのほうがダウンロード数が多いです。

```bash
docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=***** -d mysql:8.0
```

### Use Oracle Official Container

Oracle Official イメージ (Oracle Linux ベース) を使用します。

```bash
docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=***** -d mysql/mysql-server:8.0
```

### How to set up an initial database in Docker

`/docker-entrypoint-initdb.d` にローカルディレクトリをマウントすることで、コンテナ起動時に初期化スクリプトを自動実行します。

`100_create.sql`:

```sql
create database test_db1;
create user test_db1_user1 identified with mysql_native_password by 'test_db1_user1';
grant SELECT,INSERT,UPDATE,DELETE,CREATE,DROP on test_db1.* to test_db1_user1;
```

`200_import.sh`:

```bash
#!/bin/bash
cd /docker-entrypoint-initdb.d/
echo "import data..."
gzip -dc ./scott.dump.gz | mysql -h localhost -u root -p******** -P 3306 test_db1
echo "done."
```

コンテナ起動コマンド:

```bash
docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=***** \
  --mount type=bind,source="$(pwd)"/initdb.d,target=/docker-entrypoint-initdb.d \
  -d mysql/mysql-server:8.0
```

---

## Configurations

### Change root@localhost Password

ログから一時パスワードを確認して変更します。

```console
$ sudo grep 'temporary password' /var/log/mysqld.log

2018-06-12T07:52:34.579110Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: ;

$ mysql -u root -p

Enter password: ********[Enter] <= 初期パスワード

mysql> alter user 'root'@'localhost' identified by '********';
Query OK, 0 rows affected (0.09 sec)
```

### mysql_secure_installation

初期セキュリティ設定を行うためのスクリプトを実行します。

```bash
sudo bin/mysql_secure_installation
```

設定内容:

- root ユーザーのパスワード変更
- VALIDATE PASSWORD プラグインのインストールと強度ポリシー設定 (0 = LOW, 1 = MEDIUM, 2 = STRONG)
- 匿名ユーザー (anonymous) の削除
- リモートからの root ログイン禁止
- test データベースの削除
- 権限テーブルの再読み込み

### Configuration Password Policy

パスワードポリシーの変更手順です。

```sql
mysql> show variables like 'validate_password%';
mysql> set global validate_password_length=6;
mysql> set global validate_password_policy=LOW;
```

### Configuration Character Set and Collation

文字コードおよび照合順序の設定例です。

```sql
mysql> create database example_sjis character set cp932;
mysql> use example_sjis
mysql> show variables like 'char%';
mysql> show variables like 'collation%';

-- 照合順序の変更
mysql> alter database example_sjis character set cp932 collate cp932_bin;
mysql> show variables like 'collation%';
```
<!-- spell-checker: words sjis -->

### Create Remote Access User

コンテナ内で非 root ユーザーを作成します。

```bash
docker exec -it <container_name> bash
```

```bash
mysql -p
```

```sql
mysql> create user ruser identified with mysql_native_password by '*****';
mysql> select host,user,plugin from mysql.user;
```
<!-- spell-checker: words ruser -->

```bash
mysql -h 127.0.0.1 -P 3306 -u ruser -p
```

## Troubleshooting

### Unicode Character Issues in Docker

Docker コンテナ内で Unicode 文字や絵文字が入力・表示できない場合、ロケール設定を行います。

Debianベース (Docker公式):

```bash
docker exec -it <container_name> bash
```

```bash
apt update && apt install task-japanese
export LANG=C.UTF-8
```

Oracle Linuxベース (Oracle公式):

```bash
docker exec -it -e LANG=en_US.utf-8 <container_name> bash
```

### Remote Host Connection Issues

MySQL 8.0.4 以降、デフォルトの認証プラグインが `caching_sha2_password` に変更されたため、旧クライアント (MariaDB 10.2等) から接続するとエラーが発生する場合があります。
接続用ユーザーの認証方式を `mysql_native_password` に変更することで回避可能です。

```sql
mysql> alter user 'ruser'@'%' identified with mysql_native_password by '*****';
```

また、パスワード再設定時に認証文字列がクリアされることがあるため、変更後に再度パスワードを設定してください。

## Reference

- [Installing MySQL on Unix/Linux Using Generic Binaries](https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html)
- [Docker Hub - mysql](https://store.docker.com/images/mysql)
- [Docker Hub - mysql/mysql-server](https://hub.docker.com/r/mysql/mysql-server/)
