# memcached Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Terminology](#terminology)
- [Installation](#installation)
- [Memcached DDoS attacks](#memcached-ddos-attacks)

## Overview

memcached（memory cache daemon）は、分散メモリキャッシュシステムを構築するためのオープンソースなキャッシュサーバーです。サーバーのメインメモリ上にデータベースのクエリ結果やファイル内容などを一時保管することで、データアクセスの高速化を実現します。

オンメモリで動作するKVS（Key-Value Store）であり、KeyとValueを1対1で対応づけて管理します。割り当てられたメモリ容量の上限に達した場合、LRU（Least Recently Used：最終アクセス日時が最も古いデータを削除）アルゴリズムに基づいて自動的に不要データを破棄し、新規データ用の領域を確保します。

- [memcached - a distributed memory object caching system](https://memcached.org/)
- [memcached GitHub repository](https://github.com/memcached/memcached)

## Environments

- 2020-09-28: memcached 1.6.7 on Docker

## Terminology

| Words | Description |
| :--- | :--- |
| Page | Slabに割り当てられるメモリ領域の単位（デフォルト1MB）。 |
| Chunk | キャッシュデータ（レコード）を格納するための最小単位領域。 |
| Slab Class | 特定のサイズ単位に切り分けられたChunkの集合体クラス。 |

## Installation

Dockerを利用して単体の memcached サーバーを起動します。

```bash
docker run --rm --name my-memcache -d -p 11211:11211 memcached:1.6-alpine memcached -m 64　-U 0
```

| Option | Description |
|--|--|
| `-p 11211:11211` | ホストとコンテナ間のポートマッピング（標準TCPポート: 11211）|
| `-m 64` | アイテム割り当て用最大メモリサイズ（メガバイト単位、デフォルト64MB）|
| `-U 0` | UDP プロトコルを無効化(最近はデフォルトで無効化されている) |

## Memcached DDoS attacks

Memcached DDoS攻撃（メンキャッシュド・ディードス・こうげき）とは、高速なメモリキャッシュシステムである「Memcached」の脆弱性や設定不備を悪用し、標的のサーバーに対して爆発的な量のトラフィックを送りつけてダウンさせる「リフレクション（反射）型DDoS攻撃」の一種です。

Memcached DDoS 攻撃は、次の手順で進行します。

1. 攻撃者は、インターネット上に公開されている脆弱な Memcached サーバーをスキャンして特定します。
2. 攻撃者は、標的サーバーのIPアドレスを送信元として偽装し、Memcached サーバーに小さなリクエストを送信します。
3. 脆弱な Memcached サーバーは、標的サーバーに対して大量のレスポンスを返します。
4. 標的サーバーは、膨大な量のトラフィックを処理できず、サービスが停止します。

主な対策

- Memcached サーバーをインターネットに公開しない
- ファイアウォールで Memcached の UDP ポート（デフォルト11211）への外部アクセスを遮断する
- Memcached サーバーのアクセス制御を適切に設定し、信頼できるクライアントからのみアクセス可能にする
- 不要な場合は UDP プロトコルを無効化する
- 定期的に Memcached サーバーのログを監視し、異常なアクセスを検知する。
