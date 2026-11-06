
# Cassandra TTL Tips

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Basic usage(CQL)](#basic-usagecql)
  - [When specifying TTL when inserting](#when-specifying-ttl-when-inserting)
  - [Default TTL settings for the entire table](#default-ttl-settings-for-the-entire-table)
- [Key Mechanisms and Points to Note](#key-mechanisms-and-points-to-note)
- [Estimating data retention capacity and TTL](#estimating-data-retention-capacity-and-ttl)

## Overview

Apache Cassandraにおける TTL（Time-To-Live：有効期限） は、データやカラムに自動的な有効期限（秒数）を設定し、期限経過後に自動で削除・期限切れ（Tombstone化）させる機能です。

Cassandra では、TTL を使うことでデータの自動削除を簡単に実現できます。
つまり、たくさん入れて勝手に消えるデータを扱うことが　Cassandra の得意分野です。

## Basic usage(CQL)

### When specifying TTL when inserting

挿入時にTTLを指定する場合：USING TTL 句を使います（単位は秒）。

```sql
INSERT INTO users (id, name) VALUES ('1', 'Alice') USING TTL 86400;
```

> この例では24時間（86400秒）後に自動削除されます

データが消える前の残り時間の確認を行えます。

```sql
cqlsh:example> SELECT id, name, TTL(name) FROM users WHERE id = '1';

 id | name  | ttl(name)
----+-------+-----------
  1 | Alice |     86366

```

> Primary key は `TTL()` 関数に指定できません

### Default TTL settings for the entire table

テーブル全体にデフォルトの TTL を設定する場合は、`default_time_to_live` オプションを使用します。

```sql
CREATE TABLE users (
    id text PRIMARY KEY,
    name text,
    created_at timestamp
) WITH default_time_to_live = 86400;
```

> この例では、テーブル全体のデフォルト TTL が 24 時間（86400 秒）に設定されます

```sql
DESC users

CREATE TABLE example.users (
    id text PRIMARY KEY,
    created_at timestamp,
    name text
) WITH additional_write_policy = '99p'
    AND allow_auto_snapshot = true
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND cdc = false
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '16', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND memtable = 'default'
    AND crc_check_chance = 1.0
    AND default_time_to_live = 86400
    AND extensions = {}
    AND gc_grace_seconds = 864000
    AND incremental_backups = true
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair = 'BLOCKING'
    AND speculative_retry = '99p';
```

TTLの指定なしでデータを挿入すると、テーブルのデフォルト TTL が適用されます。

```sql
INSERT INTO users (id, name, created_at) VALUES ('2', 'Bob', toTimestamp(now()));
```

残り時間を確認するには、`TTL()` 関数を使用します。

```sql
cqlsh:example> SELECT id, name, created_at, TTL(name) FROM users WHERE id = '2';

 id | name | created_at                      | ttl(name)
----+------+---------------------------------+-----------
  2 |  Bob | 2026-11-06 13:01:34.007000+0000 |     86285
```

> ELECT句の中で timestamp 型のカラムと ttl() 関数を直接足し引きして計算することはCQLの仕様上できません。

テーブルのデフォルト TTL を変更、無効化する場合は、`ALTER TABLE` 文を使用します。

```sql
-- デフォルト TTL を変更する場合
ALTER TABLE users WITH default_time_to_live = 43200;

-- デフォルト TTL を無効化する場合（0に設定）
ALTER TABLE users WITH default_time_to_live = 0;
```

デフォルト TTL を設定したテーブルに　TTL を指定してデータを挿入すると、指定した TTL が優先されます。

```sql
INSERT INTO users (id, name, created_at) VALUES ('3', 'Charlie', toTimestamp(now())) USING TTL 3600;
```

残り時間を確認すると、指定した TTL が適用されていることがわかります。

```sql
cqlsh:example> SELECT id, name, created_at, TTL(name) FROM users WHERE id = '3';

 id | name    | created_at                      | ttl(name)
----+---------+---------------------------------+-----------
  3 | Charlie | 2026-11-06 13:01:34.007000+0000 |      3590
```

## Key Mechanisms and Points to Note

- Tombstone（墓石）の発生：TTLによる期限切れが発生すると、そのデータはすぐ物理的に消えるわけではなく「Tombstone」としてマークされます。その後、コンパクション（Compaction）処理によってディスク上から完全に削除されます。
- 大規模利用時の注意点：
  - 極端に短いTTL（1秒など）：クラスター内のノード間伝搬にタイムラグが生じ、データ不整合や予期せぬ挙動の原因になることがあります。
  - コンパクション戦略の影響：デフォルトのサイズ層圧縮戦略（STCS）でTTLを大量に使うと、読み書きの負荷やスペース増大を招くケースがあります。
- TTLの上限（20年問題）：CassandraのTTLには最大値の制限（約20年）があり、内部のタイムスタンプやオーバーフローに関連する仕様上の注意点があります。

## Estimating data retention capacity and TTL

1. 純粋なデータサイズ（純データ量）の計算

    まずは、テーブルの定義と想定されるレコード数から基本のサイズを算出します。

    > ※正確な公式（セルごとのオーバーヘッド計算など）もありますが、概算設計フェーズでは以下の簡略式が実用的です。

    $$純データ量 = 1レコードあたりの平均バイト数 \times 想定レコード数$$

    - レコードサイズの見積もり例：
      - id (UUID: 16バイト) + created_at (Timestamp: 8バイト) + status (Text: 約10バイト) ＝ 約34バイト
    - TTL利用時の注意：
      - TTLを設定している場合、データは「一定期間で蓄積が止まり、プラトー（横ばい）に達する」挙動になります。
      - レコード数は、1日あたりのインサート件数 \times TTLの保持日数 で計算してください。

2. クラスター全体の「必要物理容量」の計算

    Cassandraはデータを複数のノードに複製するため、設定した Replication Factor（RF: レプリケーション係数）の分だけ掛け算になります。
    さらに、インデックスやメタデータのオーバーヘッドとして一般的に 20%（1.2） のマージンを上乗せします。

    $$必要物理容量 = 純データ量 \times RF \times 1.2$$

    例： 純データ量が 100 GB、RF=3 の場合：

    $$\text{必要物理容量} = 100 \text{ GB} \times 3 \times 1.2 = 360 \text{ GB} \quad (\text{クラスター全体で最低限必要なディスク容量})$$

3. ディスク安全稼働率（Compactionマージン）の考慮【最重要】

    Cassandraで最も注意すべきなのが、「ディスクいっぱいにデータを貯めてはいけない」 という点です。背後で動くコンパクション（SSTableの結合・整理）処理の際、一時的に古いデータと新しいデータがディスクに共存するため、大きな空きスペースが必要になります。

    選択するコンパクション戦略（Compaction Strategy）によって、ディスクの最大使用率の推奨値が変わります：

    | コンパクション戦略 | 推奨されるディスク最大使用率 | ディスクに必要な余裕（倍率） | 主なユースケース |
    |------------------|----------------------------|----------------------------|----------------|
    | STCS (Size-Tiered) | 50% まで | 2.0倍 | 書き込みが非常に多いテーブル |
    | LCS (Leveled) | 80% まで | 1.25倍 | 読み込み・更新が多いテーブル |
    | TWCS (Time-Window) | 70%〜80% まで | 1.3〜1.4倍 | TTLを多用する時系列データ |

    TTLを利用する場合、時間軸でデータを区切って効率よく破棄できる TWCS を採用するのが一般的です。
    そのため、本番環境ではディスク容量の約70%を上限として計算します。

4. 最終的なトータルディスク見積もり式

    これまでの要素をすべて組み合わせると、調達すべきインフラの総ディスク容量が割り出せます。

    $$\text{総ディスク容量（クラスター全体）} = \frac{\text{純データ量} \times RF \times 1.2}{\text{ディスク安全稼働率（0.5 ～ 0.8）}}$$

5. ノード1台あたりのディスクサイズへの切り出し

    最後に、これをクラスターのノード数で割ることで、サーバー1台あたりに積むべきSSDの容量が決まります。

    $$\text{ノードあたりのディスク容量} = \frac{\text{総ディスク容量（クラスター全体）}}{\text{ノード数}}$$

    例： 総ディスク容量が 360 GB、ノード数が 3 の場合：

    $$\text{ノードあたりのディスク容量} = \frac{360 \text{ GB}}{3} = 120 \text{ GB}$$
  
   - ノードあたりのデータ量限界：
     - Cassandraの運用ベストプラクティスとして、運用の安定性（障害時のデータ復旧・ストリーミング時間）の観点から、1ノードあたりの実データ保持量は1TB〜2TB未満に抑えることが強く推奨されています。
     - もし計算結果が1台あたり2TBを超えてしまう場合は、1台のディスクを大きくするのではなく、ノード数を増やして（スケールアウト）負荷と容量を分散させてください。
