# memcached Cluster Tips

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Cluster Nodes Configuration](#cluster-nodes-configuration)
- [Distribution of registered data across multiple locations](#distribution-of-registered-data-across-multiple-locations)
  - [1. Client Application (Python)](#1-client-application-python)
  - [2. Running the Client Script](#2-running-the-client-script)
  - [3. Verifying Distribution per Node](#3-verifying-distribution-per-node)
- [Network Failure Behavior Verification](#network-failure-behavior-verification)
  - [1. Simulate Network Failure](#1-simulate-network-failure)
  - [2. Write New Keys during Failure](#2-write-new-keys-during-failure)
  - [3. Running the Update Script](#3-running-the-update-script)
  - [4. Verify Updated Keys on Node 1](#4-verify-updated-keys-on-node-1)
  - [5. Recover Network and Read Keys](#5-recover-network-and-read-keys)
  - [6. Running the Get Script](#6-running-the-get-script)
- [Conclusion (Problems and risks that arise)](#conclusion-problems-and-risks-that-arise)

## Overview

memcached の スケーリングを試すとともに、どうやら memcached クラスタに問題があるらしいので確認します。

## Environments

- 2026-11-01: memcached 1.6.45 on Docker

## Cluster Nodes Configuration

クラスタを構成する各ノードの設定を確認します。今回は Docker を利用して、異なるポートで memcached を起動します。

Node 1 (Port: 11211):

```bash
docker run --rm --name my-memcache1 -d -p 11211:11211 memcached:1.6-alpine memcached
```

Node 2 (Port: 11212):

```bash
docker run --rm --name my-memcache2 -d -p 11212:11211 memcached:1.6-alpine memcached
```

```bash
docker ps
```

```console
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                                             NAMES
8cc9787ebc4d   memcached:1.6-alpine   "docker-entrypoint.s…"   6 seconds ago    Up 5 seconds    0.0.0.0:11212->11211/tcp, [::]:11212->11211/tcp   my-memcache2
003f8500ddce   memcached:1.6-alpine   "docker-entrypoint.s…"   28 seconds ago   Up 27 seconds   0.0.0.0:11211->11211/tcp, [::]:11211->11211/tcp   my-memcache1
```

## Distribution of registered data across multiple locations

複数ノードのクラスタを指定してデータを投入し、自動的にノード間で分散されることを確認します。

### 1. Client Application (Python)

Python環境を作成し、`python-memcached` ライブラリをインストールします。

```bash
mkdir memcached-test
cd memcached-test/

pyenv install 3.14
pyenv global 3.14
python -m venv .venv
source .venv/bin/activate

pip install -U pip setuptools
pip install python-memcached
```

<!-- spell-checker: words pyenv venv setuptools -->

データの登録（`registered-multiple.py`）:

```python
import memcache

# 2つのノードを指定
client = memcache.Client(["127.0.0.1:11211", "127.0.0.1:11212"], cache_cas=True)

# キーの登録
client.set("key01", "value01")
client.set("key02", "value02")
client.set("key03", "value03")
client.set("key04", "value04")
client.set("key05", "value05")

# 読み出し確認
print("key01.value :", client.get("key01"))
print("key02.value :", client.get("key02"))
print("key03.value :", client.get("key03"))
print("key04.value :", client.get("key04"))
print("key05.value :", client.get("key05"))
```

### 2. Running the Client Script

作成した `registered-multiple.py` を実行して、データが正しく分散されていることを確認します。

```bash
python registered-multiple.py
```

```console
key01.value : value01
key02.value : value02
key03.value : value03
key04.value : value04
key05.value : value05
```

### 3. Verifying Distribution per Node

Telnetで各ノードに接続し、キーが分散して格納されていることを確認します。

Node 1 (11211) の内容確認:

```bash
telnet 127.0.0.1 11211
```

```text
stats cachedump 1 100
```
<!-- spell-checker: words cachedump -->

```console
ITEM key04 [7 b; 0 s]
ITEM key03 [7 b; 0 s]
ITEM key01 [7 b; 0 s]
END
```

Node 2 (11212) の内容確認:

```bash
telnet 127.0.0.1 11212
```

```text
stats cachedump 1 100
```
<!-- spell-checker: words cachedump -->

```console
ITEM key05 [7 b; 0 s]
ITEM key02 [7 b; 0 s]
END
```

## Network Failure Behavior Verification

分散クラスタにおいて特定のノードへネットワーク障害が発生した場合、クライアントライブラリはダウンしたノードをバイパスし、生存しているノードにキーを読み書きしに行く挙動をとることがあります。これにより、障害復旧時にデータ不整合が発生するリスクがあります。

### 1. Simulate Network Failure

Dockerの仮想インターフェースを無効化し、 Node 2 への接続を遮断します。

使用している Docker ネットワークを確認します。

```bash
docker inspect --format '{{json .NetworkSettings.Networks}}' my-memcache2
```

```console
{"bridge":{"IPAMConfig":... }}
```

Dockerネットワークからコンテナを切り離します。

```bash
docker network disconnect bridge my-memcache2
```

繋がらないことを確認します。

```bash
telnet 127.0.0.1 11212
```

```console
Trying 127.0.0.1...
telnet: Unable to connect to remote host: Connection refused
```

### 2. Write New Keys during Failure

一部のノードが遮断された状態でクライアントからキーの更新・追加を実行します。

データ更新（`update-multiple.py`）:

```python
import memcache

client = memcache.Client(["127.0.0.1:11211", "127.0.0.1:11212"], cache_cas=True)

client.set("key01", "updated01")
client.set("key02", "updated02")
client.set("key03", "updated03")
client.set("key04", "updated04")
client.set("key05", "updated05")
```

### 3. Running the Update Script

作成したスクリプトを実行します。

```bash
python update-data.py
```

### 4. Verify Updated Keys on Node 1

Node 1 (11211) のステータス確認:

```text
stats cachedump 1 100
```
<!-- spell-checker: words cachedump -->

```console
ITEM key05 [9 b; 0 s]
ITEM key04 [9 b; 0 s]
ITEM key03 [9 b; 0 s]
ITEM key02 [9 b; 0 s]
ITEM key01 [9 b; 0 s]
END
```

```text
lru_crawler metadump all
```
<!-- spell-checker: words metadump -->

```console
OK
key=key01 exp=-1 la=1793668833 cas=5 fetch=no cls=1 size=77 flags=16 
key=key02 exp=-1 la=1793668833 cas=6 fetch=no cls=1 size=77 flags=16 
key=key03 exp=-1 la=1793668833 cas=7 fetch=no cls=1 size=77 flags=16 
key=key04 exp=-1 la=1793668833 cas=8 fetch=no cls=1 size=77 flags=16 
key=key05 exp=-1 la=1793668833 cas=9 fetch=no cls=1 size=77 flags=16 
END
```

> Node 2 に元々格納されていたキーの一部（例: `key05`）が Node 1 側に書き込まれていることがわかります。

### 5. Recover Network and Read Keys

ネットワーク障害を復旧させ、再度クライアントからデータを取り出します。

```bash
docker network connect bridge my-memcache2 
```

復旧後のデータ取得（`get-multiple.py`）:

```python
import memcache

client = memcache.Client(["127.0.0.1:11211", "127.0.0.1:11212"], cache_cas=True)

print("key01.value :", client.get("key01"))
print("key02.value :", client.get("key02"))
print("key03.value :", client.get("key03"))
print("key04.value :", client.get("key04"))
print("key05.value :", client.get("key05"))
```

### 6. Running the Get Script

作成したスクリプトを実行します。

```bash
python update-data.py
```

```console
key01.value : updated01
key02.value : value02
key03.value : updated03
key04.value : updated04
key05.value : value05
```

> Node 2 に元々格納されていたキーの一部（例: `key02` や `key05`）が古い値のまま取得されていることがわかります。

## Conclusion (Problems and risks that arise)

**復旧後、元々 Node 2 に存在した古くなったキャッシュ値と、障害中に他ノードへ書き込まれた最新キャッシュが混在・逆転し、期待しないデータが取得される問題が発生します。**
**ノード障害時のフェイルオーバー設定や無効化戦略（一致ハッシュ法・Consistent Hashingなど）の導入検討が必要です。**
