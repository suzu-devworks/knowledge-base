# Apache Cassandra Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Features](#features)
- [Terminology](#terminology)
- [Single Node Server Installation with Docker](#single-node-server-installation-with-docker)
- [Multi Node Cluster Installation with Docker](#multi-node-cluster-installation-with-docker)
  - [Create Docker Network](#create-docker-network)
  - [Run Multi Node Cassandra Cluster](#run-multi-node-cassandra-cluster)
  - [Check Cluster Nodes](#check-cluster-nodes)
  - [Check Cluster Status](#check-cluster-status)
- [Connect to Cassandra](#connect-to-cassandra)
- [DDL](#ddl)
  - [Keyspace](#keyspace)
  - [Table (ColumnFamily)](#table-columnfamily)
  - [Data Types](#data-types)
  - [Sample Table Creation](#sample-table-creation)
- [Cassandra Workbench (VSCode extension)](#cassandra-workbench-vscode-extension)
- [Troubleshooting](#troubleshooting)
  - [(SOLVED) UPDATE Failure Without WHERE Clause](#solved-update-failure-without-where-clause)
  - [(SOLVED) Error occurred during initialization of VM (nodetool)](#solved-error-occurred-during-initialization-of-vm-nodetool)
- [References](#references)

## Overview

Apache Cassandra は、オープンソースの分散データベース管理システムである。元はFacebook社において大規模データの格納のために開発されたものである。  
Dynamo の完全分散システムデザインと Bigtable のカラムファミリに基づいたデータモデルをあわせ持つ、スケーラビリティの高い第2世代分散データベースである。  

## Environments

- 2020-09-27: Apache Cassandra 3.11.8 (cassandra:3.11) on Docker (Fedora 32)
- 2026-11-05: Apache Cassandra 5.0.9 (cassandra:latest) on Docker (macOS 26.6.2)

## Features

- 単一障害点 ( SPOF ) がない
- 高速パフォーマンス
- リニアなスケーラビリティ性能
- 柔軟なNoSQLデータモデル
- データの整合性を選択可能
- CQL (Cassandra Query Language)
- スキーマを持つ
- Java で実行している  

Cassandra はクラスタでスケールアウトすることを前提に作られている。また、GUIツール（Cassandra-client等）を利用した管理も可能である。

## Terminology

| Term | Description |
| :- | :- |
| Cluster | 論理的なCassandraインスタンスのこと。クラスタは複数のキースペースを持つことができる。 |
| Keyspace | ColumnFamilies用のネームスペース。一般的には1アプリケーションに1キースペース。RDBMSでいうところのデータベース。 |
| ColumnFamily | カラムまたはスーパーカラムの集合を扱う単位。RDBMSでいうところのテーブル。 |
| SuperColumns | サブカラムを持つカラムのこと。キー (アクセス用) と値 (保持するカラムの配列) を保持する。 |
| Key | カラムへのアクセス単位。ColumnFamiliesへアクセスするにはkeyをWHERE句に指定してアクセスを行う。PRIMARY KEY と PARTITION KEY がある。 |
| Column | データの最小単位。name (キー)、value (値)、timestamp (更新日時) を保持する。 |

データの指定方法:

- 4次元の場合: [キースペース] [カラムファミリ] [キー] [カラム]
- 5次元の場合: [キースペース] [カラムファミリ] [キー] [スーパーカラム] [カラム]

## Single Node Server Installation with Docker

- [cassandra - Docker Hub](https://hub.docker.com/_/cassandra)

```bash
docker run -d --network my-cassandra-network --name my-cassandra cassandra:5
```

| Option / Volume / Port | Description |
| :- | :- |
| VOLUME /var/lib/cassandra | Cassandra Data Directory |
| EXPOSE 7000 | intra-node communication |
| EXPOSE 7001 | TLS intra-node communication |
| EXPOSE 7199 | JMX |
| EXPOSE 9042 | CQL |
| EXPOSE 9160 | thrift service |

## Multi Node Cluster Installation with Docker

### Create Docker Network

Isolation bridge networkを作成する。

```bash
docker network create --driver=bridge my-cassandra-network
```

docker network ls でネットワーク情報を確認する。

```console
% docker network ls

NETWORK ID     NAME                   DRIVER    SCOPE
5b86f47d0398   bridge                 bridge    local
5ffcc8754699   host                   host      local
660690bae265   my-cassandra-network   bridge    local
095bebe7f938   none                   null      local

% docker network inspect my-cassandra-network
[
    {
        "Name": "my-cassandra-network",
        "Id": "660690bae26541b9ee0db807803fd8f61f561bbeca55b9db1c7994785ff2e1cf",
        "Created": "2026-11-05T09:22:05.856869292Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Options": {
            "com.docker.network.enable_ipv4": "true",
            "com.docker.network.enable_ipv6": "false"
        },
        "Labels": {},
        "Containers": {},
        "Status": {
            "IPAM": {
                "Subnets": {
                    "172.18.0.0/16": {
                        "IPsInUse": 3,
                        "DynamicIPsAvailable": 65533
                    }
                }
            }
        }
    }
]
```

### Run Multi Node Cassandra Cluster

Cassandra の一貫性保証において Quorum ((ノード数+1)/2 の数分の一貫性を保証) を採用しているため、クラスタのノード数は 3台以上 にすることが推奨される。  

デフォルトでは 1node あたり約 4GB メモリを消費するため `JVM_OPTS="-Xms1024M -Xmx1024M"` でヒープサイズを小さく調整して起動する。

```bash
# 1台目起動
$ docker run -d --rm --network my-cassandra-network --name my-cassandra1 -e JVM_OPTS="-Xms512m -Xmx512m" cassandra:latest

# 2台目起動
$ docker run -d --rm --network my-cassandra-network --name my-cassandra2 -e CASSANDRA_SEEDS=my-cassandra1 -e JVM_OPTS="-Xms512m -Xmx512m" cassandra:latest

# 3台目起動 (ノード起動完了まで少し間隔を空ける必要あり)
$ docker run -d --rm --network my-cassandra-network --name my-cassandra3 -e CASSANDRA_SEEDS=my-cassandra1 -e JVM_OPTS="-Xms512m -Xmx512m" cassandra:latest
```

| Option | Description |
| :- | :- |
| CASSANDRA_CLUSTER_NAME | クラスタの名前。cassandra.yaml の cluster_name 値を設定する。 |
| CASSANDRA_SEEDS | クラスターに参加する新しいノードをブートストラップするために使用するIPアドレスのコンマ区切りリスト。cassandra.yaml の seeds 値を設定する。 |
| CASSANDRA_BROADCAST_ADDRESS | 別マシンで起動する場合、他のノードにアドバタイズするIPアドレスを指定する。 |

### Check Cluster Nodes

Cassandra クラスタのノードが正しく起動しているか確認するには、`docker ps` コマンドでコンテナの状態を確認します。

```console
% docker ps

CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS           NAMES
6dd494f57fdc   cassandra:5   "docker-entrypoint.s…"   11 seconds ago   Up 10 seconds   7000-7001/tcp, 7199/tcp, 9042/tcp,9160/tcp   my-cassandra3
c5837e7380a5   cassandra:5   "docker-entrypoint.s…"   18 seconds ago   Up 17 seconds   7000-7001/tcp, 7199/tcp, 9042/tcp,9160/tcp   my-cassandra2
4a92bd7a904b   cassandra:5   "docker-entrypoint.s…"   3 minutes ago    Up 3 minutes    7000-7001/tcp, 7199/tcp, 9042/tcp,9160/tcp   my-cassandra1
```

Docker stats で各コンテナのリソース使用状況を確認する。

```bash
docker stats
```

```console
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O         BLOCK I/O     PIDS
bb779181af91   my-cassandra2   2.24%     1.198GiB / 7.748GiB   15.46%    342kB / 327kB   0B / 14.9MB   76
71b6280a0b7b   my-cassandra1   6.04%     1.176GiB / 7.748GiB   15.18%    332kB / 381kB   0B / 15.2MB   80
24c72b930bac   my-cassandra3   2.55%     1.183GiB / 7.748GiB   15.27%    269kB / 232kB   0B / 15MB     77
```

各ノードのIPアドレスを確認する場合は、以下のコマンドを実行する。

```console
% docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)

/my-cassandra3 - 172.18.0.4
/my-cassandra2 - 172.18.0.3
/my-cassandra1 - 172.18.0.2
```

### Check Cluster Status

Cassandra クラスタのステータスを確認するには、`nodetool status` コマンドを使用する。
`nodetool` は Cassandra クラスタの管理ツールなのでどこかのノードに接続して実行する必要がある。

```bash
docker exec -it my-cassandra1 bash
```

```console
root@71b6280a0b7b:/# JVM_OPTS="-Xms128m -Xmx128m" nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack 
UN  172.18.0.3  80.05 KiB  16      59.3%             7aa74c11-0070-4ed8-b5b6-76a4340d3de1  rack1
UN  172.18.0.2  119.8 KiB  16      64.7%             48f0672a-46ab-4efe-a9b5-39fc73348469  rack1
UN  172.18.0.4  85.11 KiB  16      76.0%             62749fd6-8935-4099-9540-a2ed11360bf6  rack1
```

## Connect to Cassandra

Cassandra に接続するには、`cqlsh` コマンドを使用する。

```bash
docker run -it --network my-cassandra-network --rm cassandra:latest cqlsh my-cassandra1
```

```console
ATTENTION: All commands will be saved to history file: /root/.cassandra/cqlsh_history
This may include sensitive information such as passwords.
To disable history, use --disable-history or set 'disabled = true' in the [history] section of cqlshrc.
See https://cassandra.apache.org/doc/latest/tools/cqlsh.html for more information.

Connected to Test Cluster at my-cassandra1:9042
[cqlsh 6.2.0 | Cassandra 5.0.9 | CQL spec 3.4.7 | Native protocol v5]
Use HELP for help.
cqlsh> 

# コマンドラインの終了
cqlsh> exit
```

## DDL

### Keyspace

作成(`CREATE KEYSPACE`):

```sql
cqlsh> CREATE KEYSPACE example WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 1};
```

変更(`ALTER KEYSPACE`):

```sql
cqlsh> ALTER KEYSPACE example WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};
```

> [!NOTE]
>
> ```text
> Warnings :
> When increasing replication factor you need to run a full (-full) repair to distribute the data.
> ```
>
> これは、レプリケーションファクターを増加させた場合に、データを正しく分散させるためにフルリペアを実行する必要があることを示しています。
> フルリペアは, `nodetool repair --full <キースペース名>` コマンドで実行できます。

削除(`DROP KEYSPACE`)

```sql
cqlsh:example> DROP KEYSPACE example;
```

一覧表示 (KEYSPACES):

```sql
cqlsh> DESC KEYSPACES

system       system_distributed  system_traces  system_virtual_schema
system_auth  system_schema       system_views 

-- CQLの場合
cqlsh> SELECT * FROM system_schema.keyspaces;

 keyspace_name      | durable_writes | replication
--------------------+----------------+-------------------------------------------------------------------------------------
        system_auth |           True | {'class': 'org.apache.cassandra.locator.SimpleStrategy', 'replication_factor': '1'}
      system_schema |           True |                             {'class': 'org.apache.cassandra.locator.LocalStrategy'}
 system_distributed |           True | {'class': 'org.apache.cassandra.locator.SimpleStrategy', 'replication_factor': '3'}
             system |           True |                             {'class': 'org.apache.cassandra.locator.LocalStrategy'}
      system_traces |           True | {'class': 'org.apache.cassandra.locator.SimpleStrategy', 'replication_factor': '2'}

(5 rows)
```

詳細表示 (KEYSPACE):

```sql
cqlsh> DESC KEYSPACE system_auth;

CREATE KEYSPACE system_auth WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '1'}  AND durable_writes = true;

CREATE TABLE system_auth.cidr_groups (
...
```

選択(`USE`)

```sql
cqlsh> USE example;
cqlsh:example>
```

### Table (ColumnFamily)

作成(`CREATE TABLE`):

```sql
-- 作成例 1
cqlsh:example> CREATE TABLE monkey_species (
    species text PRIMARY KEY,
    common_name text,
    population varint,
    average_size int
) WITH comment='Important biological records';

-- 作成例 2
cqlsh:example> CREATE TABLE timeline (
    userid uuid,
    posted_month int,
    posted_time uuid,
    body text,
    posted_by text,
    PRIMARY KEY (userid, posted_month, posted_time)
) WITH compaction = { 'class' : 'LeveledCompactionStrategy' };

-- 作成例 3
cqlsh:example> CREATE TABLE loads (
    machine inet,
    cpu int,
    mtime timeuuid,
    load float,
    PRIMARY KEY ((machine, cpu), mtime)
) WITH CLUSTERING ORDER BY (mtime DESC);
```

変更(`ALTER TABLE`):

```sql
-- 例: カラムの追加
cqlsh:example> ALTER TABLE monkey_species ADD habitat text;

-- 例: カラムの削除
cqlsh:example> ALTER TABLE monkey_species DROP habitat;
```

削除(`DROP TABLE`):

```sql
cqlsh:example> DROP TABLE monkey_species;
```

一覧表示（TABLES）:

```sql
cqlsh:example> DESC TABLES

loads  monkey_species  timeline      
```

詳細表示 (TABLE):

```sql
DESC TABLE monkey_species

CREATE TABLE example.monkey_species (
    species text PRIMARY KEY,
    average_size int,
    common_name text,
    population varint
) WITH additional_write_policy = '99p'
    AND allow_auto_snapshot = true
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND cdc = false
    AND comment = 'Important biological records'
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '16', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND memtable = 'default'
    AND crc_check_chance = 1.0
    AND default_time_to_live = 0
    AND extensions = {}
    AND gc_grace_seconds = 864000
    AND incremental_backups = true
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair = 'BLOCKING'
    AND speculative_retry = '99p';
```

### Data Types

| Category | Types |
| :- | :- |
| 文字列型 (例: lastname = 'Smith') | ascii, text, varchar |
| 数値型 (例: age = 31) | tinyint, smallint, int, bigint, varint, decimal, float, double |
| 日付および時間型 (例: purchase_date = '2017-05-12') | date, duration, time, timestamp |
| 一意識別子 | uuid, timeuuid |
| 特殊な型 | BLOB, boolean, counter, inet |
| コレクション型 | frozen<collection_definition>, list<data_type>, map<data_type, data_type>, set<data_type>, tuple<data_type, ...> |
| ユーザー定義型 (UDT) | ユーザー定義構造 |

### Sample Table Creation

```sql
-- Keyspace 作成
cqlsh> CREATE KEYSPACE example WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};
cqlsh> USE example;

-- Table 作成
cqlsh:example> CREATE TABLE personal ( id UUID PRIMARY KEY, name text, birthday timestamp, nationality text, weight text, height text );

-- データ追加
cqlsh:example> INSERT INTO personal (id, name, birthday, nationality) VALUES (uuid(), 'user1', '1990-01-22', 'jp');
cqlsh:example> INSERT INTO personal (id, name, birthday, nationality) VALUES (uuid(), 'user2', '1990-02-11', 'jp');
cqlsh:example> INSERT INTO personal (id, name, birthday, nationality) VALUES (uuid(), 'user3', '1990-03-04', 'jp');

-- データ確認
cqlsh:example> SELECT * FROM personal;
 id                                   | birthday                        | height | name  | nationality | weight
--------------------------------------+---------------------------------+--------+-------+-------------+--------
 4a4ccc60-dcdb-4f4c-b5bb-463414eb6e6b | 1990-03-04 00:00:00.000000+0000 |   null | user3 |          jp |   null
 2dcddd27-c1ce-48e9-8679-585fdf5b83fe | 1990-01-22 00:00:00.000000+0000 |   null | user1 |          jp |   null
 518c12f4-771a-4a00-8153-23d3a8ccd7f3 | 1990-02-11 00:00:00.000000+0000 |   null | user2 |          jp |   null
```

## Cassandra Workbench (VSCode extension)

だいぶ古いけど、それほど苦もなく接続できたので。

- [kdcro101/vscode-cassandra](https://github.com/kdcro101/vscode-cassandra)

ただ、いくつか手順があります。

まず、dockerで cql のポート(9042)を開放しておく必要があります。

```bash
docker run -d --rm --network my-cassandra-network --name my-cassandra1 -e JVM_OPTS="-Xms512m -Xmx512m" -p 9042:9042 cassandra:latest
```

そして、VSCode に Cassandra Workbench 拡張をインストールしたのち、
コマンドパレットで [Cassandra Workbench: Generate Configuration] を選択する。

作成される `.cassandraWorkbench.jsonc` を調整

```json
[
  // AllowAllAuthenticator
  {
    "name": "Cluster AllowAllAuthenticator",
    "contactPoints": [
      "127.0.0.1"
    ]
  }
  // //PasswordAuthenticator
  // {
  //   "name": "Cluster PasswordAuthenticator",
  //   "contactPoints": [
  //     "127.0.0.1"
  //   ],
  //   "authProvider": {
  //     "class": "PasswordAuthenticator",
  //     "username": "yourUsername",
  //     "password": "yourPassword"
  //   }
  // }
]
```

これで保存して、VSCode の Cassandra Workbench 拡張から接続を試みる。

## Troubleshooting

### (SOLVED) UPDATE Failure Without WHERE Clause

UPDATE文ではWHERE句を省略した場合はエラーとなる。

```console
cqlsh:example> update personal set nationality = 'jp' ;
SyntaxException: line 1:39 mismatched input ';' expecting K_WHERE (...personal set nationality = 'jp' [;])
```

### (SOLVED) Error occurred during initialization of VM (nodetool)

JVM_OPTS でヒープサイズを指定した際、コンテナ内の nodetool 実行時にヒープ設定エラーとなる場合の対応。環境変数を設定して回避する。

```bash
export MAX_HEAP_SIZE=512m
export HEAP_NEWSIZE=512m
nodetool status
```

もしくは上で記載したように実行時に指定する

```bash
JVM_OPTS="-Xms128m -Xmx128m" nodetool status
```

## References

- [The Cassandra Query Language (CQL) | Apache Cassandra Documentation](https://cassandra.apache.org/doc/5.0.9/cassandra/developing/cql/index.html)
