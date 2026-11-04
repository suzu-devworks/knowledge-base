# Oracle Database 9i Tips for Performance Tuning

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Source](#source)
- [Oracle Tuning Overview](#oracle-tuning-overview)
  - [Tuning Objectives](#tuning-objectives)
  - [Tuning Procedure](#tuning-procedure)
  - [OLAP System Tuning](#olap-system-tuning)
  - [DSS System Tuning](#dss-system-tuning)
  - [Common Issues](#common-issues)
  - [Trade-offs](#trade-offs)
- [Diagnostic and Tuning Utilities](#diagnostic-and-tuning-utilities)
  - [Trace Files](#trace-files)
  - [Dynamic Performance Views](#dynamic-performance-views)
  - [Dynamic Performance Views (System Statistics)](#dynamic-performance-views-system-statistics)
  - [Dynamic Performance Views (Wait Events)](#dynamic-performance-views-wait-events)
  - [STATSPACK Utility](#statspack-utility)
  - [STATSPACK Diagnostics](#statspack-diagnostics)
- [Shared Pool Tuning](#shared-pool-tuning)
  - [Shared Pool Architecture](#shared-pool-architecture)
  - [Dynamic SGA](#dynamic-sga)
  - [Library Cache Tuning Goals](#library-cache-tuning-goals)
  - [SQL Sharing](#sql-sharing)
  - [Preventing SQL Re-parsing](#preventing-sql-re-parsing)
  - [Library Cache Diagnostics](#library-cache-diagnostics)
  - [Required Library Cache Size](#required-library-cache-size)
  - [Library Cache Parameters](#library-cache-parameters)
  - [Shared Pool Reserved Area](#shared-pool-reserved-area)
  - [Pinning Objects](#pinning-objects)
  - [Data Dictionary Cache Tuning](#data-dictionary-cache-tuning)
  - [Data Dictionary Cache Diagnostics](#data-dictionary-cache-diagnostics)
  - [User Global Area](#user-global-area)
  - [Large Pool](#large-pool)
- [Database Buffer Cache Tuning](#database-buffer-cache-tuning)
  - [Database Buffer Cache Overview](#database-buffer-cache-overview)
  - [Dynamic SGA](#dynamic-sga-1)
  - [Database Buffer Cache Tuning Goals](#database-buffer-cache-tuning-goals)
  - [Database Buffer Cache Diagnostics](#database-buffer-cache-diagnostics)
  - [Database Buffer Cache Advisor](#database-buffer-cache-advisor)
  - [Database Buffer Cache Tuning](#database-buffer-cache-tuning-1)
  - [Table Caching](#table-caching)
  - [Multiple Buffer Pools](#multiple-buffer-pools)
  - [Free Lists](#free-lists)
  - [Multiple DBWR Processes](#multiple-dbwr-processes)
- [Redo Log Buffer Tuning](#redo-log-buffer-tuning)
  - [Redo Log Buffer Overview](#redo-log-buffer-overview)
  - [Redo Log Buffer Tuning Goals](#redo-log-buffer-tuning-goals)
  - [Redo Log Buffer Diagnostics](#redo-log-buffer-diagnostics)
  - [NOLOGGING](#nologging)
- [Other SGA Tuning](#other-sga-tuning)
- [Database File Configuration and I/O Diagnostics](#database-file-configuration-and-io-diagnostics)
- [Sort Area Tuning](#sort-area-tuning)
- [Rollback Segment Tuning](#rollback-segment-tuning)
- [Lock Contention Diagnostics](#lock-contention-diagnostics)
- [Shared Server Configuration Tuning](#shared-server-configuration-tuning)
- [SQL Tuning](#sql-tuning)
- [Application Tuning](#application-tuning)
- [Efficient Management of Database Resources](#efficient-management-of-database-resources)

## Overview

Oracle Database 9i のパフォーマンスチューニングに関するメモ・ノウハウをまとめたドキュメント。

## Source

- Oracle 9i Database Gold 1Z0-033J

## Oracle Tuning Overview

### Tuning Objectives

| 評価軸 | 内容 |
| --- | --- |
| 応答時間 | リクエストに対するデータ受信（問い合わせの結果など）<br>表の更新やレポートの生成に要する時間 |
| 可用性 | バックアップやリカバリにかかる時間<br>パラメータ調整に要するインスタンスの停止、起動の時間 |
| ヒット率 | パフォーマンスが時間の経過に従って向上しているか低下しているか |
| 待機イベント | 待機時間が時間の経過に従って増加しているか減少しているか<br>ボトルネックがそこにあるのか |

### Tuning Procedure

1. 設計のチューニング（設計者）
2. アプリケーションのチューニング（開発者）
3. メモリのチューニング（管理者）
4. I/Oのチューニング（管理者）
5. 競合のチューニング（管理者）
6. オペレーティングシステムのチューニング（OS管理者）

### OLAP System Tuning

- 一定時間内にどれだけの数のトランザクションを処理できるか（スループット）がパフォーマンスの目安になる。
- データの冗長性がなくなるように正規化する。
- 必要最低限の索引を作成する。

### DSS System Tuning

- 個々の処理をどのくらい短時間で処理できるか（ターンアラウンド）がパフォーマンスの目安になる。
- 結合処理を最低限にするために非正規化する。

### Common Issues

- 不適切なセッション管理：ログオン／ログオフを繰り返すようなアプリケーションはパフォーマンスを低下させる。
- 不適切なカーソル管理：バインド変数を使用せずにリテラルで似たようなSQLを発行すると、個別に実行計画を解析するため多くのCPUを消費する。

### Trade-offs

- 制御ファイルやREDOログファイルを多重化させると安全性は向上しますが、ディスクI/Oは増加します。
- リカバリにかかる時間を減少させるためには、定期的なバックアップが必要になりますが、バックアップをデータベース起動中に行うとパフォーマンスに影響します。
- インスタンスリカバリをできるだけ早く終了させるには、チェックポイントを頻繁に行う必要があり、ディスクI/Oが増加します。
- 同時にアクセスするユーザー数や1秒あたりのトランザクション数が多くなれば、それだけリソース競合がおきるためパフォーマンスに影響します。

## Diagnostic and Tuning Utilities

### Trace Files

| ファイル種別 | 内容 |
| --- | --- |
| アラートファイル | エラー情報やデフォルトではない初期化パラメータ、データベース構造を変更させるSQL文が含まれる。<br>`alert_<SID>.log` |
| バックグラウンドトレースファイル | `バックグラウンドプロセスで検出されたエラーが格納されるファイルです。BACKGROUND_DUMP_DEST` 初期化パラメータで指定したディレクトリに出力されます。<br>`<SID>_<プロセス名>_<PID>.trc` |
| ユーザートレースファイル | サーバープロセスによって生成されるファイルです。<br>USER_DUMP_DEST 初期化パラメータで指定したディレクトリに出力されます。<br>`<SID>_ora_<PID>.trc` |

### Dynamic Performance Views

動的パフォーマンスビューは、ほとんどが `X$` で始まる表へのビューです。
これらの名前は `V$FIXED_TABLE` ビューにてリストアップすることができます。

### Dynamic Performance Views (System Statistics)

システム統計情報は、インスタンスやロック、アクセス情報など、さまざまな統計をさしています。

| ビュー名 | 内容 |
| --- | --- |
| `V$STATNAME` | すべての統計の名前と統計番号、クラスが確認できます。 |
| `V$SYSSTAT` | インスタンスが起動してから収集されたシステム統計の累積情報です。 |
| `V$SESSTAT` | ユーザーがOracle Server に接続してから収集されたシステム統計の累積情報が、ログイン中のセッションごとに確認できます。 |
| `V$MYSTAT` | 現在のセッションが開始してから収集されたシステム統計情報です。 |

### Dynamic Performance Views (Wait Events)

待機イベントは、さまざまな理由で待機する必要があった情報を提供します。

| ビュー名 | 内容 |
| --- | --- |
| `V$EVENT_NAME` | すべての待機イベントを説明します。 |
| `V$SYSTEM_EVENT` | インスタンスが起動してから収集された待機イベントの累積情報です。 |
| `V$SESSION_EVENT` | ユーザーがOracle Server に接続してから収集された待機イベントの累積情報が、ログイン中のセッションごとに確認できます。 |
| `V$SESSION_WAIT` | 現在待機しているセッションの待機イベントの情報です。 |

### STATSPACK Utility

STATSPACKを使用すると、任意のタイミングで統計を収集してデータベースに格納できます。またピーク時間や1日、1週間といった任意の期間でもパフォーマンス数値をレポート出力することができます。

Create:

```bash
sqlplus / as sysdba
```

```sql
SQL> @$ORACLE_HOME/rdbms/admin/spcreate.sql
...
Enter value for perfstat_password: *****
...
Enter value for default_tablespace: TOOLS
...
Enter value for temporary_tablespace: TEMP
...
SQL>
```

Collect:

```bash
sqlplus perfstat/*****
```

```sql
SQL> alter session set timed_statistics = true;
SQL> EXECUTE statspack.snap;
```

Report:

```bash
sqlplus perfstat/*****
```

```sql
SQL> @$ORACLE_HOME/rdbms/admin/spreport.sql
```

### STATSPACK Diagnostics

- サマリー部
  - データベースとインスタンスの名前、バージョン情報
  - スナップショットが作成された時刻、セッション数など
  - キャッシュの現在のサイズ（Cache Sized (end))
    - 終了時点でのデータベースバッファキャッシュ、共有プール、REDOログバッファサイズとデータブロックサイズが出力されます。
  - 負荷のプロファイル（Load Profile）
    - 秒単位、トランザクション単位で期間内に生成されたREDOサイズや論理読み取り、物理読み取り、書き込みなど。
  - インスタンスの効率（Instance Efficiency Percentages (Target 100%)）
    - データベースバッファキャッシュのヒット率やメモリソートで完了した比率など、いづれの比率も100%近くが理想です。
  - 上位５つの待機イベント（Tip 5 Wait Events）
    - すべての待機イベントのうち、待機合計時間が多い上位５つの待機イベントを出力します。
- メイン情報
  - 期間内の待機イベント
    - V$SYSTEM_EVENTからの情報です。
  - 共有プールに存在するSQL文の情報
    - 開始時と終了時の統計収集した時点の共有プールに存在していたSQL文が表示されます。
    - 実行回数、解析コール回数、バッファ取得回数、バッファ読み取り回数などでそれぞれソートした結果を出力します。
  - 期間内のシステム統計情報
    - V$SYSSTATからの情報です。
  - 表領域とデータベースファイルへのI/O
    - V$FILESTATからの情報です。
  - データベースバッファプール統計
    - V$BUFFER_POOL_STATISTICS、V$DB_CACHE_ADVICEからの情報です。
  - ロールバックセグメント統計
    - V$ROLL_STAT、V$UNDO_STATからの情報です。
  - 共有プール統計
    - V$ROWCACHE, V$LIBRARYCACHEからの情報です。
  - 初期化パラメータの値
    - デフォルト以外の初期化パラメータが出力されます。

## Shared Pool Tuning

### Shared Pool Architecture

| 領域 | 内容 |
| --- | --- |
| ライブラリキャッシュ | 文や解析コード、実行計画を含むSQL領域、PL/SQL領域が格納されます。ライブラリキャッシュ上に存在する文をユーザーが発行すると、文を解析することなくキャッシュ上の情報が使用されます。 |
| データディクショナリキャッシュ | 表や列定義、権限を含むデータディクショナリが格納されます。 |
| ユーザーグローバル領域 | 共有サーバー接続時のセッション情報、カーソル情報が格納されます。<br>ラージプールが構成されている場合にはラージプールに格納されます。 |

### Dynamic SGA

Oracle9iから共有プールやデータベースバッファキャッシュのサイズを、インスタンス停止せずに `ALTER SYSTEM` コマンドで動的に変更できるようになりました。
これに伴いSGAの全体サイズを `SGA_MAX_SIZE` 初期化パラメータで決定され、「グラニュル」と呼ばれるサイズで割り当てることになりました。`SGA_MAX_SIZE` 初期化パラメータは動的に変更できません。

- `SGA_MAX_SIZE` の値が128MB以下の場合、4MB
- `SGA_MAX_SIZE` の値が128MBを超える場合、16MB(Windowsは8MB)

### Library Cache Tuning Goals

Oracle Server は、データディクショナリキャッシュを優先して共有プール上に残そうとするため、ライブラリキャッシュのヒット率を向上させれば、自然とデータディクショナリキャッシュのヒット率も向上します。

- 再解析を最小限に抑える
  - ユーザー間でSQLを共有する
  - 共有SQL領域で文が無効になるのを防止する
  - 十分な領域を割り当てて、文の割当が解除されるのを防止する
- 断片化を抑える
  - 予約領域で大量のメモリーが必要になったときに備える。
  - 頻繁に使用する大きなオブジェクトを共有プールに固定する
  - 無名PL/SQLブロックではなく、ストアドプログラムを使用する。
  - 共有サーバー接続用にラージプールを構成する。

### SQL Sharing

実行計画が共有されているかどうかは次のビューを使用します。

| ビュー名 | 内容 |
| --- | --- |
| `V$LIBRARYCACHE` | ライブラリキャッシュ全体<br>GETHITRATIO（GETHITS/GETS）の目安は0.9（90%）以上です。 |
| `V$SQLAREA` | 個々のSQL<br>LOADS列が2以上の場合には再解析されています。<br>VERSION_COUNT列が2以上の場合には、文が同一の異なる実行計画が作成されるためLOADS列が増加します。<br>INVALIDATIONS列が2以上の場合には、文が無効になったためLOADS列が増加します。<br>少ないLOADSに対して、多くのPARSE_CALLS, EXECUTIONが行われることが目安です。 |
| `V$SQLTEXT` | V$SQLAREAでは1000バイトまでしかSQLを表示できないため長いSQLを確認するために使用する。 |

共有プールの目標は、カーソル共有であれば、V$LIBRARYCACHE の GETHITRATIO（GETHITS/GETS）が0.9以上です。

```sql
SQL> select namespace, gethitratio from v$librarycache;
NAMESPACE        GETHITRATIO
---------------- -----------
SQL AREA         .546815667
TABLE/PROCEDURE  .69372862
BODY             .712609971
TRIGGER          .707317073
INDEX            .330578512
...

SQL> select sql_text,version_count, loads, parse_calls from v$sqlarea where sql_text like '%v$session%' AND sql_text not like '%sql_text%';
SQL_TEXT                                                                         VERSION_COUNT      LOADS PARSE_CALLS
-------------------------------------------------------------------------------- ------------- ---------- -----------
select * from v$session                                                                      1          1           1

SQL> select address, hash_value, piece, sql_text from v$sqltext where sql_text like '%v$session%' AND sql_text not like '%sql_text%' order by address, hash_value, piece;
ADDRESS          HASH_VALUE      PIECE SQL_TEXT
---------------- ---------- ---------- ----------------------------------------------------------------
0000000062364D28 3881741610          0 select * from v$session
0000000062382A10 1603532425         19 sions axs,       sys.v$session db
```

### Preventing SQL Re-parsing

SQL文を共有してもまだ再解析をしているようであれば、次の２つが考えられます。

- 共有SQL領域で文が無効になっている
- 共有プールの領域不足のために文の割当が解除されている

共有SQL領域の文が無効になるのは次のDDLで変更された後にSQL文を実行した場合です。

- `ALTER TABLE` による列の追加、変更、削除
- `ALTER` コマンドによる表、ビュー、シノニム、順序の変更
- `ANALYZE` コマンドの実行
- 索引の作成、変更、削除
- `ALTER` コマンドによるストアドプログラムの再コンパイル

```sql
-- 共有SQLの無効数
SQL> select SUM(invalidations) from v$librarycache;
SUM(INVALIDATIONS)
------------------
               220

-- SQL単位の無効化情報
SQL> select sql_text, loads, invalidations from v$sqlarea where sql_text like '%v$session%' AND sql_text not like '%sql_text%';
SQL_TEXT                                                                              LOADS INVALIDATIONS
-------------------------------------------------------------------------------- ---------- -------------
select * from v$session                                                                   1             0

-- 解析コール数（システム統計より）
SQL> select name, value from v$sysstat WHERE name like 'parse count%';
NAME                                                                                VALUE
-------------------------------------------------------------------------------- ----------
parse count (total)                                                                   13259
parse count (hard)                                                                     5840
parse count (failures)                                                                   13
parse count (describe)                                                                    4
```

### Library Cache Diagnostics

ライブラリキャッシュは `V$LIBRARYCACHE` でミス率を算出します。

```sql
SQL> SELECT
        sum(pins) AS "Executions",
        sum(reloads) AS "Cache Misss",
        sum(reloads) / sum(pins) AS "Miss(%)"
    FROM
        v$librarycache;

Executions Cache Misss   Miss(%)
---------- ----------- ----------
    124188        3243 .026113634
```

ミス率が1％(0.01)以上の場合には、`SHARED_POOL_SIZE` を大きくして共有プールに十分な領域を割り当てます。

### Required Library Cache Size

```sql
-- ストアドプログラムの使用メモリ
SQL> SELECT sum(sharable_mem) FROM v$db_object_cache 
     WHERE type in ('PROCEDURE', 'FUNCTION', 'PACKAGE', 'PACKAGE BODY') ;
SUM(SHARABLE_MEM)
-----------------
          5303008

-- SQL文の使用メモリ
SQL> SELECT sum(sharable_mem) FROM v$sqlarea WHERE executions > 5;
SUM(SHARABLE_MEM)
-----------------
          6840943

-- ピーク時間帯でのオープンカーソルの使用メモリ
SQL> SELECT sum(users_opening * 250) from v$sqlarea;
SUM(USERS_OPENING*250)
----------------------
                 28000
```

### Library Cache Parameters

| パラメータ名 | 内容 |
| --- | --- |
| `OPEN_CURSORS` | セッションでオープンできるカーソルの最大数<br>default: 50 |
| `CURSOR_SPACE_FOR_TIME` | カーソルがクローズするまで破棄しないようにする<br>default: FALSE |
| `SESSION_CACHED_CURSORS` | キャッシュされるカーソル数<br>default: 0 |
| `CURSOR_SHARING` | 同じカーソルを共有するSQL文を「同じ」と判断するルールを設定します。 |

### Shared Pool Reserved Area

共有プールで空き領域が断片化を起こさないよう予約領域を使用することで、大きなオブジェクトに備えメモリを用意しておけます。

| パラメータ名 | 内容 |
| --- | --- |
| `SHARED_POOL_RESERVED_SIZE` | 予約領域のサイズ<br>default: SHARED_POOL_SIZEの5% |

```sql
-- 予約領域のサイズ確認
SQL> show parameters shared_pool
NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
shared_pool_reserved_size            big integer 15938355
shared_pool_size                     big integer 0

-- 予約領域の診断
SQL> SELECT free_space, request_misses, request_failures FROM v$shared_pool_reserved;
FREE_SPACE REQUEST_MISSES REQUEST_FAILURES
---------- -------------- ----------------
   8554696              0                1

-- 共有プールの空領域
SQL> SELECT * FROM v$sgastat WHERE pool ='shared pool' AND name = 'free memory';
POOL        NAME                            BYTES     CON_ID
----------- -------------------------- ---------- ----------
shared pool free memory                  13866048          0
```

`request_failures` が1以上であれば、`SHARED_POOL_RESERVED_SIZE` パラメータを調整します。

### Pinning Objects

オブジェクトをロードする場合、他のオブジェクトを破棄する可能性がありパフォーマンスに影響を与えます。オブジェクトを固定すると `ALTER SYSTEM FLUSH SHARED POOL` コマンドでも破棄されないため非常に強力ですが、できるだけ断片化がおきないようデータベースの起動直後に固定します。

```sql
-- on SYSDBA
SQL> @$ORACLE_HOME/rdbms/admin/dbmspool.sql

-- on Other user
SQL> EXECUTE dbms_shared_pool.keep('オブジェクト名')
SQL> EXECUTE dbms_shared_pool.keep('スキーマ名.オブジェクト名')
SQL> EXECUTE dbms_shared_pool.keep('スキーマ名.パッケージ、プロシージャ、ファンクション', 'P')
SQL> EXECUTE dbms_shared_pool.keep('スキーマ名.順序', 'Q')
SQL> EXECUTE dbms_shared_pool.keep('スキーマ名.トリガー', 'R')
SQL> EXECUTE dbms_shared_pool.keep('スキーマ名.タイプ', 'T')
```

### Data Dictionary Cache Tuning

Oracle Server はライブラリキャッシュよりデータディクショナリキャッシュを優先して格納していきます。しかしできるだけデータディクショナリキャッシュのキャッシュミスが無いよう調整します。

### Data Dictionary Cache Diagnostics

データディクショナリキャッシュは `V$ROWCACHE` でミス率を算出します。

```sql
SQL> SELECT
        sum(gets) AS "Gets",
        sum(getmisses) AS "Cache Misss",
        sum(getmisses) / sum(gets) AS "Miss(%)"
    FROM
        v$rowcache;

      Gets Cache Misss   Miss(%)
---------- ----------- ----------
    309419       55760 .180208714
```

ミス率が15％以上の場合には、`SHARED_POOL_SIZE` を大きくして共有プールに十分な領域を割り当ててます。

### User Global Area

セッション情報や、セッションで発行しているSQL文に関するメモリは次のものがあります。

- カーソル情報（取り出された行数などのSQL文のランタイム情報）
- セッションデータ（セキュリティ情報やリソース使用量）
- スタック領域（プロセス固有の変数等）

このうちカーソル情報とセッションデータを合わせてUGAといいます。
UGAは専用サーバー接続の場合にはPGAに、共有サーバー接続の場合にはSGAに格納されます。

```sql
-- 現セッション(V$mystat)のugaサイズ
SQL> SELECT sum(m.value) FROM v$mystat m, v$statname n WHERE m.statistic# = n.statistic# AND n.name ='session uga memory';
SUM(M.VALUE)
------------
     1330144

-- 全ユーザー(V$sesstat)のugaサイズ
SQL> SELECT sum(s.value) FROM v$sesstat s, v$statname n WHERE s.statistic# = n.statistic# AND n.name ='session uga memory';
SUM(S.VALUE)
------------
    29368416

-- 全ユーザー(V$sesstat)の最大ugaサイズ
SQL> SELECT sum(s.value) FROM v$sesstat s, v$statname n WHERE s.statistic# = n.statistic# AND n.name ='session uga memory max';
SUM(S.VALUE)
------------
    66291856
```

### Large Pool

複数のプロセス通信のために共有プールを使用するため、本来のライブラリキャッシュやデータディクショナリキャッシュの領域が圧迫されてしまうことを目的でラージプールを構成することができます。
ラージプールを構成するには `LARGE_POOL_SIZE` 初期化パラメータを設定します。

ラージプールを使用する処理を次に示します。

- I/Oスレーブプロセス
  - DBWn、LGWn、ARCHにおいてディスクへの書き込み並列化プロセス用のメモリ
- バックアップとリストア操作
  - Recovery Manager（RMAN)を使用する場合に、I/Oスレーブが有効になっていると使用するメモリをラージプールから割り当てます。
- 共有サーバー
  - 共有サーバー接続でのセッションデータとカーソル情報を含むUGAとして使用するメモリをラージプールで割り当てます。
- パラレル処理
  - `PARALLEL_AUTOMATIC_TUNING` 初期化パラメータが TRUE の場合、パラレルクエリーで処理するメモリをラージプールで割り当てます。

## Database Buffer Cache Tuning

### Database Buffer Cache Overview

### Dynamic SGA

### Database Buffer Cache Tuning Goals

### Database Buffer Cache Diagnostics

### Database Buffer Cache Advisor

### Database Buffer Cache Tuning

### Table Caching

### Multiple Buffer Pools

### Free Lists

### Multiple DBWR Processes

## Redo Log Buffer Tuning

### Redo Log Buffer Overview

### Redo Log Buffer Tuning Goals

### Redo Log Buffer Diagnostics

### NOLOGGING

## Other SGA Tuning

## Database File Configuration and I/O Diagnostics

## Sort Area Tuning

## Rollback Segment Tuning

## Lock Contention Diagnostics

## Shared Server Configuration Tuning

## SQL Tuning

## Application Tuning

## Efficient Management of Database Resources
