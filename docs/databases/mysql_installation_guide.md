# MySQL インストール＆設定ガイド (Linux & Docker)

## 目次

* [概要](#概要)
* [1. Linux への直接インストール](#1-linux-への直接インストール)
  * [1.1. Generic Binary を使用したインストール](#11-generic-binary-を使用したインストール)
  * [1.2. RPM (dnf/yum) を使用したインストール (Fedora等)](#12-rpm-dnfyum-を使用したインストール-fedora等)
  * [1.3. 初期セキュリティ設定 (mysql_secure_installation)](#13-初期セキュリティ設定-mysql_secure_installation)
  * [1.4. パスワードポリシー・文字コードの設定](#14-パスワードポリシー文字コードの設定)
* [2. Docker を使用した環境構築](#2-docker-を使用した環境構築)
  * [2.1. コンテナの起動 (Docker公式 / ORACLE公式)](#21-コンテナの起動-docker公式--oracle公式)
  * [2.2. ホスト側からの接続とリモートユーザー作成](#22-ホスト側からの接続とリモートユーザー作成)
  * [2.3. 初期データベース・スクリプトの自動実行](#23-初期データベーススクリプトの自動実行)
* [3. トラブルシューティング](#3-トラブルシューティング)
  * [3.1. 絵文字・Unicode文字が入力/表示できない](#31-絵文字unicode文字が入力表示できない)
  * [3.2. 外部・ホストから接続できない / 認証プラグインエラー (`caching_sha2_password`)](#32-外部ホストから接続できない--認証プラグインエラー-caching_sha2_password)
* [参考資料](#参考資料)
* [変更履歴](#変更履歴)

---

## 概要

MySQLは高速性と堅牢性を追求したマルチユーザー・マルチスレッド型のオープンソース・リレーショナルデータベース管理システム (RDBMS) です。商標権および著作権は Oracle Corporation に帰属します。

---

## 1. Linux への直接インストール

### 1.1. Generic Binary を使用したインストール

汎用バイナリパッケージ（`glibc` 版）を使用した手動インストールの手順です。

#### 依存ライブラリの導入
```bash
$ sudo dnf install libaio
```

#### インストール手順
```bash
# 実行ユーザーの作成
$ sudo useradd -r -s /bin/false mysql

# アーカイブの展開とシンボリックリンクの作成
$ cd /usr/local
$ sudo tar zxvf /path/to/mysql-8.0.11-linux-glibc2.12-x86_64.tar.gz
$ sudo ln -s /usr/local/mysql-8.0.11-linux-glibc2.12-x86_64/ mysql

# ディレクトリ作成と権限設定
$ cd mysql
$ sudo mkdir mysql-files
$ sudo chown mysql:mysql mysql-files
$ sudo chmod 750 mysql-files

# データベースの初期化 (一時パスワードがコンソールに出力されます)
$ sudo bin/mysqld --initialize --user=mysql
# 例: temporary password is generated for root@localhost: R)AC&i&t2Uua

# SSL/TLS 鍵の生成と起動
$ sudo bin/mysql_ssl_rsa_setup
$ sudo bin/mysqld_safe --user=mysql &

# コマンドのシンボリックリンク作成
$ sudo ln -s /usr/local/mysql/bin/mysql /usr/local/bin/
```

#### root 初期パスワードの変更
初回ログイン後、パスワード変更を行うまで他の操作は実行できません。
```bash
$ mysql -u root -p
```
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NEW_PASSWORD';
```

#### サーバーの停止・情報確認
```bash
# 停止
$ sudo bin/mysqladmin -u root -p shutdown

# サーバー情報の確認
$ bin/mysqladmin -u root -p version
$ bin/mysqlshow -u root -p
```

#### Systemd サービス登録
`/etc/systemd/system/mysqld.service` を以下の内容で作成します。

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

```bash
# サービスの起動とステータス確認
$ sudo systemctl daemon-reload
$ sudo systemctl start mysqld
$ sudo systemctl status mysqld
```

---

### 1.2. RPM (dnf/yum) を使用したインストール (Fedora等)

Fedora等の標準リポジトリで MariaDB が提供されている場合は、MySQL公式 Yum/DNF リポジトリを追加してインストールします。

#### リポジトリの追加とインストール
```bash
# リポジトリパッケージの取得とインストール
$ sudo dnf localinstall ./mysql80-community-release-fc28-1.noarch.rpm

# MySQL Server のインストール
$ sudo dnf install mysql-community-server

# サービスの起動
$ sudo systemctl start mysqld
$ sudo systemctl status mysqld
```

#### 初期パスワードの確認と変更
ログファイルから一時パスワードを取得します。
```bash
$ sudo grep 'temporary password' /var/log/mysqld.log
```
ログイン後、パスワードを変更します。
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NEW_PASSWORD';
```

---

### 1.3. 初期セキュリティ設定 (mysql_secure_installation)

セキュリティ強化スクリプトを実行し、不要なユーザーやデータベースを削除します。

```bash
$ sudo mysql_secure_installation
```

**設定項目:**
* **VALIDATE PASSWORD PLUGIN**: パスワード強度検証プラグインの導入有無およびレベル（LOW / MEDIUM / STRONG）の設定
* **Change root password**: root パスワードの変更
* **Remove anonymous users**: 匿名ユーザーの削除
* **Disallow root login remotely**: root のリモートログイン禁止
* **Remove test database**: test データベースの削除
* **Reload privilege tables**: 権限テーブルのリロード

---

### 1.4. パスワードポリシー・文字コードの設定

#### パスワードポリシーの調整
開発環境などでパスワードの制約を緩和したい場合の設定例です。
```sql
-- 現在の設定確認
SHOW VARIABLES LIKE 'validate_password%';

-- ポリシー緩和設定 (例: 6文字以上)
SET GLOBAL validate_password.policy = LOW;
SET GLOBAL validate_password.length = 6;
```

#### データベースごとの文字コード・照合順序設定
データベース作成時や変更時に文字コードを指定します。
```sql
-- cp932 (Shift_JIS拡張) で作成
CREATE DATABASE example_sjis CHARACTER SET cp932;

-- 照合順序 (Collation) の変更
ALTER DATABASE example_sjis CHARACTER SET cp932 COLLATE cp932_bin;

-- 確認
SHOW VARIABLES LIKE 'char%';
SHOW VARIABLES LIKE 'collation%';
```

---

## 2. Docker を使用した環境構築

### 2.1. コンテナの起動 (Docker公式 / ORACLE公式)

#### Docker 公式イメージ (`mysql:8.0`)
```bash
$ docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hogehoge -d mysql:8.0
```

#### ORACLE 公式イメージ (`mysql/mysql-server:8.0`)
```bash
$ docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hogehoge -d mysql/mysql-server:8.0
```

---

### 2.2. ホスト側からの接続とリモートユーザー作成

MySQL 8.0 のデフォルト認証方式 (`caching_sha2_password`) に古いクライアントが対応していない場合があるため、`mysql_native_password` を明示してユーザーを作成します。

#### コンテナ内でのユーザー作成
```bash
$ docker exec -it <container_name_or_id> bash
# container-bash# mysql -p
```
```sql
-- リモート接続用ユーザーの作成 (従来型の認証プラグインを指定)
CREATE USER 'ruser'@'%' IDENTIFIED WITH mysql_native_password BY 'ruserpass';

-- 権限付与 (必要に応じて)
GRANT ALL PRIVILEGES ON *.* TO 'ruser'@'%';
FLUSH PRIVILEGES;
```

#### ホストからの接続確認
```bash
$ mysql -h 127.0.0.1 -P 3306 -u ruser -p
```

---

### 2.3. 初期データベース・スクリプトの自動実行

`/docker-entrypoint-initdb.d` にスクリプトをマウントすると、コンテナ初回起動時に自動で実行されます。

#### ディレクトリ構造とスクリプトの準備
ローカルに `initdb.d` ディレクトリを作成し、以下のようなファイルを配置します。

* **`100_create.sql`**
  ```sql
  CREATE DATABASE test_db1;
  CREATE USER 'test_db1_user1'@'%' IDENTIFIED WITH mysql_native_password BY 'test_db1_user1';
  GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP ON test_db1.* TO 'test_db1_user1'@'%';
  ```

* **`200_import.sh`** (シェルスクリプトでダンプを投入する場合)
  ```bash
  #!/bin/bash
  echo "Importing data..."
  gzip -dc /docker-entrypoint-initdb.d/scott.dump.gz | mysql -h localhost -u root -p"$MYSQL_ROOT_PASSWORD" test_db1
  echo "Done."
  ```

#### コンテナの起動
```bash
$ docker run --rm -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=hogehoge \
  --mount type=bind,source="$(pwd)"/initdb.d,target=/docker-entrypoint-initdb.d \
  -d mysql/mysql-server:8.0
```

---

## 3. トラブルシューティング

### 3.1. 絵文字・Unicode文字が入力/表示できない

Dockerコンテナ内のロケール設定や日本語パッケージが不足している場合に発生します。

**解決策 (Debian系ベースコンテナの場合):**
```bash
$ docker exec -it <container_name> bash
# apt update && apt install -y task-japanese
# export LANG=C.UTF-8
```

---

### 3.2. 外部・ホストから接続できない / 認証プラグインエラー (`caching_sha2_password`)

#### 現象 1: ソケット接続エラー
`mysql -u root -p` 実行時に `Can't connect to local MySQL server through socket` が発生する。
* **原因**: ホスト側から `-h 127.0.0.1` または `-h <IP>` を指定しない場合、UNIXドメインソケット接続を試みようとするため。
* **対処**: ネットワーク経由接続にするため `-h 127.0.0.1 -P 3306` オプションを明示する。

#### 現象 2: `Plugin caching_sha2_password could not be loaded` エラー
クライアント（例: MariaDB付属の古いクライアントなど）から接続しようとするとエラーが発生する。
* **原因**: MySQL 8.0.4 以降、デフォルト認証プラグインが `caching_sha2_password` に変更されたが、接続側クライアントが未対応のため。
* **対処**: 該当ユーザーの認証方式を `mysql_native_password` に変更する。

```sql
-- 認証プラグインの変更
ALTER USER 'ruser'@'%' IDENTIFIED WITH mysql_native_password;

-- パスワードが再設定（クリア）された場合は再設定を行う
ALTER USER 'ruser'@'%' IDENTIFIED BY 'ruserpass';
```

---

## 参考資料

* [日本MySQLユーザ会](http.www.mysql.gr.jp/)
* [MySQL Documentation - Installing MySQL on Unix/Linux Using Generic Binaries](https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html)
* [Docker Hub - mysql/mysql-server](https://hub.docker.com/r/mysql/mysql-server/)
* [MySQL 8.0.4におけるデフォルト認証形式の変更 (日々の覚書)](https://yoku0825.blogspot.com/2018/01/mysql-804.html)

---

## 変更履歴

* **2018-06-11**: Linux Generic Binary / RPM でのインストール手順作成 (MySQL 8.0.11 / Fedora 28)
* **2018-06-11**: Docker 環境での構築および MySQL 8.0 認証プラグイン・文字コードのトラブルシューティング記録追加