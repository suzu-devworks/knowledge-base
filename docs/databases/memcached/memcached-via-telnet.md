# Interacting with Memcached via Telnet

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Connect/Disconnect](#connectdisconnect)
- [Basic Text Protocol](#basic-text-protocol)
  - [Storage commands (`set`, `add`, `replace`, `append`, `prepend` and `cas`)](#storage-commands-set-add-replace-append-prepend-and-cas)
    - [cas (Check-And-Set)](#cas-check-and-set)
  - [Retrieval commands (`get`, `gets`, `gat`, and `gats`)](#retrieval-commands-get-gets-gat-and-gats)
  - [Delete command](#delete-command)
  - [incr, decr](#incr-decr)
  - [Statistics commands (`stats` and `version`)](#statistics-commands-stats-and-version)
- [Data Expiration Behavior](#data-expiration-behavior)
- [Meta Text Protocol](#meta-text-protocol)

## Overview

memcachedはデフォルトでポート11211でTCP/UDPリクエストを受け付けます。
コマンドラインから接続するには telnet または nc（netcat） を使います。

ただし、UDPでの接続は最近のバージョンではデフォルトで無効化されているため、通常はTCPで接続します。

## Connect/Disconnect

How to connect:

```bash
telnet localhost 11211
```

```console
Trying ::1...
Connected to localhost.
Escape character is '^]'.
```

How to disconnect:

```bash
quit
```

```console
Connection closed by foreign host.
```

## Basic Text Protocol

### Storage commands (`set`, `add`, `replace`, `append`, `prepend` and `cas`)

構文:

```text
<command> <key> <flag> <expire> <size>
<value>
```

- `<flag>`: データの圧縮またはアプリケーション側のメタデータ（0: 非圧縮など）
- `<expire>`: 有効期限（秒数、またはUNIXタイムスタンプ）。`2592000`（30日）を超える場合はタイムスタンプとみなされる。`0`を指定した場合は無期限
- `<size>`: 保存するデータのバイト長

`set` キーと値を設定:

```text
set key1 0 0 3
foo
```

```console
STORED
```

`replace` キーが存在する場合に値を更新:

```text
replace key1 0 0 3
bar
```

```console
STORED
```

`append` キーの値の末尾にデータを追加:

```text
append key1 0 0 6
#post#
```

```console
STORED
```

`prepend` キーの値の先頭にデータを追加:

```text
prepend key1 0 0 5
#pre#
```

```console
STORED
```

#### cas (Check-And-Set)

排他制御（楽観的ロック）を行うために使用します。`gets` で取得したCAS IDを指定して更新を試み、その間に他者がデータを更新していた場合（CAS IDが変更されていた場合）は `EXISTS` が返り更新が失敗します。

構文:

```text
cas <key> <flag> <expire> <size> <cas ID>
```

実行例:

```text
set key3 0 0 10
{a, b, c}.
```

```console
STORED
```

```text
gets key3
```

```console
VALUE key3 0 10 11
{a, b, c}.
END
```

> CAS ID = 11

別プロセスで値が書き換えられた場合:

```text
replace key3 0 0 4
ABCD
```

```console
STORED
```

> 別プロセスで値が書き換えられたことにより、古いCAS IDでの更新が失敗することを示しています。

古いCAS ID (11) での更新試行:

```text
cas key3 0 0 5 11
12345
```

```console
EXISTS
```

現在のデータと新しいCAS IDの確認:

```text
gets key3
```

```console
VALUE key3 0 4 12
ABCD
END
```

> 現在のデータが更新され、新しいCAS IDが 12 に更新されていることを示しています。

### Retrieval commands (`get`, `gets`, `gat`, and `gats`)

構文:

```text
<command> <key>
```

`get` データ取得:

```text
get key1
```

```console
VALUE key1 0 14
#pre#bar#post#
END
```

`gets` データ取得:

```text
gets key1
```

```console
VALUE key1 0 19 4
#pre#bar#post#
END
```

> `gets` コマンドを実行すると、値の末尾にCAS ID（上記例では `4`）が返却されます。

### Delete command

構文:

```text
delete <key>
```

`delete` キーの削除:  

```text
delete key1
```

```console
STORED
```

キーが存在しない場合:

```text
delete key2
```

```console
NOT_FOUND
```

### incr, decr

構文:

```text
<command> <key> <value>
```

実行例:

```text
set num1 0 0 4
1000
```

```console
STORED
```

`incr` データの加算:

```text
incr num1 100
```

```console
1100
```

```text
incr num1 10000
```

```console
11100
```

`decr` データの減算:

```text
decr num1 1
```

```console
11099
```

```text
decr num1 15000
```

```console  
0
```

> `decr` の結果がマイナスになる場合は `0` で止まります。

### Statistics commands (`stats` and `version`)

ステータス確認:

```text
stats
```

```console
STAT pid 1
STAT uptime 564
STAT time 1601299301
STAT version 1.6.7
 ... 
END
```

Items の確認:

```text
stats items
```

```console
STAT items:1:number 3
STAT items:1:number_hot 0
 ...
END
```

> itemsの後の数値は slab class 番号

slabs の確認:

```text
stats slabs
```

```console
STAT 1:chunk_size 96
STAT 1:chunks_per_page 10922
STAT 1:total_pages 1
STAT 1:total_chunks 10922
STAT 1:used_chunks 3
 ...
END
```

キャッシュキーの一覧表示:

```text
stats cachedump <slab_id> <limit>
stats cachedump 1 100
```

```console
ITEM key3 [4 b; 0 s]
ITEM num1 [5 b; 0 s]
ITEM key1 [19 b; 0 s]
END
```

バージョン:

```text
version
```

```console
VERSION 1.6.7
```

## Data Expiration Behavior

有効期限を設定すると、時間が経過したアイテムは参照不能となり自動破棄されます。

`key10` の有効期限を 10 秒に設定します:

```text
set key10 0 10 20
10 sec after delete.
```

```console
STORED
```

有効期限が切れる前に取得する場合:

```text
get key10
```

```console  
VALUE key10 0 20
10 sec after delete.
END
```

10秒経過後に取得する場合:

```text
get key10
```

```console
END
```

## Meta Text Protocol

> [!NOTE]
> Meta Text Protocol は、Memcached が使用するテキストベースの新しい通信プロトコルです。従来の Telnet を用いたプロトコルに比べ、より効率的な通信が可能です。
