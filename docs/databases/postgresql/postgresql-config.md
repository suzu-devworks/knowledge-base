# PostgreSQL Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation](#installation)
  - [Using Docker](#using-docker)
  - [Environment Variables and Options](#environment-variables-and-options)
  - [Connect to postgres from psql](#connect-to-postgres-from-psql)
- [Troubleshoot](#troubleshoot)
  - [(SOLVED) ERROR column c.relhasoids does not exist](#solved-error-column-crelhasoids-does-not-exist)

## Overview

PostgreSQL（ポストグレスキューエル）は、拡張性と SQL 準拠を強みとするオープンソースのリレーショナルデータベース管理システム（RDBMS）です。
ACID 特性を備えたトランザクション、自動更新可能なビュー、マテリアライズドビュー、トリガー、外来キー、ストアドプロシージャなど、エンタープライズ用途に耐えうる豊富な機能を備えています

- [PostgreSQL: The world's most advanced open source database](https://www.postgresql.org/)。

## Environments

- 2020-10-22: PostgreSQL 13.0 (Docker: `postgres:13-alpine`)
- 2020-09-05: PostgreSQL 12.4 (Docker: `postgres:12-alpine`)
- 2026-11-02: PostgreSQL 18.6 (Docker: `postgres:18-alpine`)

## Installation

### Using Docker

Docker Hub: [postgres - Docker Hub](https://hub.docker.com/_/postgres)

```bash
docker run --name some-postgres \
  -e POSTGRES_PASSWORD=***** \
  -e POSTGRES_INITDB_ARGS="--encoding=UTF-8 --locale=ja_JP.UTF8" \
  -d postgres:18-alpine
```

### Environment Variables and Options

| Option / Variable | Description |
| :--- | :--- |
| `VOLUME /var/lib/postgresql/data` | データベースのデータ保存用ボリューム |
| `(VOLUME)/docker-entrypoint-initdb.d/*` | 初期化スクリプト (.sql, .sh) の配置ディレクトリ |
| `EXPOSE 5432` | PostgreSQL の標準ポート |
| `POSTGRES_USER` | デフォルトのスーパーユーザー名（未指定時は `postgres`） |
| `POSTGRES_PASSWORD` | スーパーユーザーのパスワード（必須） |
| `POSTGRES_INITDB_ARGS` | `initdb` 実行時の追加引数（エンコーディングやロケールの指定など） |

### Connect to postgres from psql

```bash
# psql -h ホスト名 -p ポート番号 -U ロール名 -d データベース名
docker exec -it some-postgres psql -U postgres
```

```console
psql (12.4)
Type "help" for help.
```

```sql
postgres=# 

-- コマンドラインの終了
postgres=# \q
```

## Troubleshoot

### (SOLVED) ERROR column c.relhasoids does not exist

```text
docker-# \d users
ERROR:  column c.relhasoids does not exist
LINE 1: ...riggers, c.relrowsecurity, c.relforcerowsecurity, c.relhasoi...
                                                             ^
```
<!-- spell-checker: words relrowsecurity relforcerowsecurity relhasoi relhasoids  -->

原因:

サーバー（PostgreSQL 12 以降）に対して、それより古いバージョンの `psql` クライアントから接続した際に発生します。（PostgreSQL 12 で `relhasoids` カラムが削除されたため）

解決策:

Debian 9 等の環境で最新の `postgresql-client` をインストールしてアップデートします。

```bash
# 古いクライアントの削除とリポジトリ追加
sudo apt remove -y postgresql-client
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list

# 最新版のインストール
sudo apt update
sudo apt install -y postgresql-client-12
```
<!-- spell-checker: words pgdg -->
