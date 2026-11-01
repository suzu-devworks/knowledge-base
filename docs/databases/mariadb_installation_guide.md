# MariaDB インストール＆設定ガイド (Linux & Docker)

## 目次
- [概要](#概要)
- [1. Linux (Fedora) への直接インストール](#1-linux-fedora-への直接インストール)
  - [1.1. インストール](#11-インストール)
  - [1.2. サービスの起動と停止](#12-サービスの起動と停止)
  - [1.3. 初期セキュリティ設定 (mysql_secure_installation)](#13-初期セキュリティ設定-mysql_secure_installation)
  - [1.4. 動作確認と情報表示](#14-動作確認と情報表示)
- [2. Docker を使用した環境構築](#2-docker-を使用した環境構築)
  - [2.1. コンテナの起動](#21-コンテナの起動)
  - [2.2. コマンドラインツールの導入とリモート接続](#22-コマンドラインツールの導入とリモート接続)
  - [2.3. UTF-8 (utf8mb4) 文字コードの設定](#23-utf-8-utf8mb4-文字コードの設定)
  - [2.4. 初期データベース・スクリプトの自動実行](#24-初期データベーススクリプトの自動実行)
- [参考資料](#参考資料)
- [変更履歴](#変更履歴)

---

## 概要
MariaDBは、MySQLのオリジナル作者である Michael "Monty" Widenius 氏によって開発されたオープンソースの関係データベース管理システム (RDBMS) です。ライセンスは GNU General Public License (GPL) v2 で配布されています。

---

## 1. Linux (Fedora) への直接インストール

### 1.1. インストール
Fedora環境における `dnf` パッケージマネージャーを使用したインストール手順です。

```bash
$ sudo dnf install mariadb-server
```

### 1.2. サービスの起動と停止
`systemctl` を用いて MariaDB サービスの起動・確認・停止を行います。

```bash
# サービスの確認
$ systemctl list-unit-files --type=service | grep mariadb

# サービスの起動
$ sudo systemctl start mariadb

# ステータスの確認
$ sudo systemctl status mariadb

# サービスの停止
$ sudo systemctl stop mariadb

# ログの確認
$ journalctl -xe -u mariadb
```

### 1.3. 初期セキュリティ設定 (mysql_secure_installation)
本番環境での安全な運用のために、対話型スクリプトを実行して初期セキュリティ設定を行います。

```bash
$ sudo mysql_secure_installation
```

**設定プロンプトの例:**
* **Enter current password for root**: 空文字のまま `[Enter]`
* **Set root password?**: `Y` (パスワードを設定)
* **Remove anonymous users?**: `Y` (匿名ユーザーの削除)
* **Disallow root login remotely?**: `Y` (rootの外部接続を禁止)
* **Remove test database and access to it?**: `Y` (テストDBの削除)
* **Reload privilege tables now?**: `Y` (権限テーブルのリロード)

### 1.4. 動作確認と情報表示
MariaDB にログインし、データベース一覧やルートユーザーの権限、使用ポートを確認します。

```bash
$ mysql -u root -p
```

```sql
-- データベース一覧の表示
MariaDB [(none)]> show databases;

-- ユーザー情報の確認
MariaDB [(none)]> select host,user,plugin from mysql.user;

-- ポート番号の確認 (デフォルト: 3306)
MariaDB [(none)]> show variables like 'port';
```

---

## 2. Docker を使用した環境構築

### 2.1. コンテナの起動
Docker Hub の公式 MariaDB イメージを使用してコンテナを起動します。

```bash
$ docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hogehoge -d mariadb:10.3
```

**主な環境変数・オプション:**
| オプション / 環境変数 | 説明 |
| :--- | :--- |
| `VOLUME /var/lib/mysql` | データベースファイルの永続化領域 |
| `EXPOSE 3306` | コンテナが開放するポート |
| `MYSQL_ROOT_PASSWORD` | root ユーザーのパスワード |

### 2.2. コマンドラインツールの導入とリモート接続

#### ホスト側にクライアントツールを導入
```bash
$ sudo dnf install mariadb
$ mysql --version
```

#### リモート接続用ユーザーの作成 (コンテナ内部での作業)
```bash
$ docker exec -it <container_name_or_id> bash
bash-4.2# mysql -p
```

```sql
-- ユーザーの新規作成
MariaDB [(none)]> create user ruser identified by 'ruserpass';

-- ユーザー作成と権限の確認
MariaDB [(none)]> select host,user,password,plugin from mysql.user;
MariaDB [(none)]> show grants for ruser;
```

#### ホストマシンからの接続テスト
```bash
$ mysql -h 127.0.0.1 -P 3306 -u ruser -p
```

### 2.3. UTF-8 (utf8mb4) 文字コードの設定
デフォルトの MariaDB の `utf8` は 3 バイト制限があるため、4 バイト文字（絵文字等）に対応する `utf8mb4` の利用を推奨します。

#### サーバー側の設定 (Docker起動時)
起動時引数に `--character-set-server=utf8mb4` を指定します。

```bash
$ docker run --rm -p 3306:3306 -e MYSQL_ROOT_PASSWORD=hogehoge -d mariadb:10.3 --character-set-server=utf8mb4
```

#### クライアント側の設定
`/etc/my.cnf.d/utf8mb4.cnf` ファイルを作成し、以下を記述します。

```ini
[client]
default-character-set=utf8mb4
```

#### 文字コード設定の確認
```sql
MariaDB [(none)]> show variables like 'char%';
MariaDB [(none)]> show variables like 'collation%';
```

### 2.4. 初期データベース・スクリプトの自動実行

#### 準備
ローカルの `./initdb.d` ディレクトリを作成し、コンテナの `/docker-entrypoint-initdb.d` にマウントすることで、初回起動時に `.sql` スクリプト等を自動実行させることができます。

**ファイル構成例 (`initdb.d/100_create.sql`):**
```sql
create database test_db1;
create user test_db1_user1 identified by 'test_db1_user1';
grant SELECT,INSERT,UPDATE,DELETE,CREATE,DROP on test_db1.* to test_db1_user1;
```

#### コンテナの起動 (バインドマウント付き)
```bash
$ docker run --rm -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=hogehoge \
  -e MYSQL_DATABASE=test_db1 \
  --mount type=bind,source="$(pwd)"/initdb.d,target=/docker-entrypoint-initdb.d \
  -d mariadb:10.3
```

#### サンプルDDL / DML (`scott` スキーマ)
```sql
create database scott;
use scott;

create table dept(
  deptno integer,
  dname  varchar(14),
  loc    varchar(13),
  constraint pk_dept primary key (deptno)
);

create table emp(
  empno    integer,
  ename    varchar(10),
  job      varchar(9),
  mgr      integer,
  hiredate date,
  sal      decimal(7,2),
  comm     decimal(7,2),
  deptno   INTEGER,
  constraint pk_emp primary key (empno),
  constraint fk_deptno foreign key (deptno) references dept (deptno)
);

create table bonus(
  ename varchar(10),
  job   varchar(9),
  sal   decimal(7,2),
  comm  decimal(7,2)
);

create table salgrade(
  grade integer,
  losal decimal(7,2),
  hisal decimal(7,2)
);

insert into dept values(10, 'ACCOUNTING', 'NEW YORK');
insert into dept values(20, 'RESEARCH', 'DALLAS');
insert into dept values(30, 'SALES', 'CHICAGO');
insert into dept values(40, 'OPERATIONS', 'BOSTON');

insert into emp values(7839, 'KING', 'PRESIDENT', null, str_to_date('17-11-1981','%d-%m-%Y'), 5000, null, 10);
insert into emp values(7698, 'BLAKE', 'MANAGER', 7839, str_to_date('1-5-1981','%d-%m-%Y'), 2850, null, 30);
insert into emp values(7782, 'CLARK', 'MANAGER', 7839, str_to_date('9-6-1981','%d-%m-%Y'), 2450, null, 10);
insert into emp values(7566, 'JONES', 'MANAGER', 7839, str_to_date('2-4-1981','%d-%m-%Y'), 2975, null, 20);
insert into emp values(7788, 'SCOTT', 'ANALYST', 7566, str_to_date('13-JUL-87','%d-%M-%y') - INTERVAL 85 DAY, 3000, null, 20);
insert into emp values(7902, 'FORD', 'ANALYST', 7566, str_to_date('3-12-1981','%d-%m-%Y'), 3000, null, 20);
insert into emp values(7369, 'SMITH', 'CLERK', 7902, str_to_date('17-12-1980','%d-%m-%Y'), 800, null, 20);
insert into emp values(7499, 'ALLEN', 'SALESMAN', 7698, str_to_date('20-2-1981','%d-%m-%Y'), 1600, 300, 30);
insert into emp values(7521, 'WARD', 'SALESMAN', 7698, str_to_date('22-2-1981','%d-%m-%Y'), 1250, 500, 30);
insert into emp values(7654, 'MARTIN', 'SALESMAN', 7698, str_to_date('28-9-1981','%d-%m-%Y'), 1250, 1400, 30);
insert into emp values(7844, 'TURNER', 'SALESMAN', 7698, str_to_date('8-9-1981','%d-%m-%Y'), 1500, 0, 30);
insert into emp values(7876, 'ADAMS', 'CLERK', 7788, str_to_date('13-JUL-87', '%d-%M-%y') - INTERVAL 51 DAY, 1100, null, 20);
insert into emp values(7900, 'JAMES', 'CLERK', 7698, str_to_date('3-12-1981','%d-%m-%Y'), 950, null, 30);
insert into emp values(7934, 'MILLER', 'CLERK', 7782, str_to_date('23-1-1982','%d-%m-%Y'), 1300, null, 10);

insert into salgrade values (1, 700, 1200);
insert into salgrade values (2, 1201, 1400);
insert into salgrade values (3, 1401, 2000);
insert into salgrade values (4, 2001, 3000);
insert into salgrade values (5, 3001, 9999);

commit;
```

#### mysqldump によるバックアップ
```bash
# データベース単位のダンプ (CREATE DATABASE を含める場合)
$ mysqldump -h 127.0.0.1 -u ruser -p --databases scott | gzip > scott.sql.gz
```

---

## 参考資料
* [MariaDB 公式サイト](https://mariadb.org/)
* [MariaDB ナレッジベース (日本語)](https://mariadb.com/kb/ja/mariadb/)
* [mariadb - Docker Hub](https://hub.docker.com/_/mariadb)
* [Server World - Fedora 28 : MariaDB : インストール](https://www.server-world.info/query?os=Fedora_28&p=mariadb&f=1)

## 変更履歴
* **2018-06-11**: Docker 環境での構築手順作成 (MariaDB 10.3.7)
* **2018-06-14**: Linux (Fedora 28) への直接インストール手順作成 (MariaDB 10.2.14)