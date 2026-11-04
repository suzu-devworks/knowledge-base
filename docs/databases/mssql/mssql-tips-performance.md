# SQL Server Tips for Performance

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Performance Monitor](#performance-monitor)
  - [Performance Monitor Counters](#performance-monitor-counters)
  - [SQL Server Counters](#sql-server-counters)
- [Case Analysis](#case-analysis)
  - [High CPU Usage (User Time 100%, Low CPU Queue)](#high-cpu-usage-user-time-100-low-cpu-queue)
  - [High CPU Usage (User Time 100%, High CPU Queue)](#high-cpu-usage-user-time-100-high-cpu-queue)
  - [Moderate CPU Usage (User Time 50%, Low CPU Queue)](#moderate-cpu-usage-user-time-50-low-cpu-queue)
  - [Low CPU Usage with High Disk Queue](#low-cpu-usage-with-high-disk-queue)
  - [Idle CPU and Zero Disk Queue](#idle-cpu-and-zero-disk-queue)
  - [Fixed 50% CPU Usage (Dual Core / Hyper-Threading)](#fixed-50-cpu-usage-dual-core--hyper-threading)
  - [High Privileged Time CPU Usage](#high-privileged-time-cpu-usage)

## Overview

SQL Server におけるパフォーマンスモニタリングの監視項目（OS標準およびSQL Server固有）および、各リソース状態における主な発生原因とトラブルシューティングのパターンをまとめたドキュメント。

## Performance Monitor

### Performance Monitor Counters

| Counter | Performance Object | Description |
| --- | --- | --- |
| Memory | Available MBytes | プロセスへの割り当てまたはプロセスの使用にすぐに利用可能な物理メモリ |
|  | Cache Bytes | 現在、物理メモリ内にあってアクティブになっているシステム ファイル キャッシュサイズ |
|  | Committed Bytes | コミットされた仮想メモリ。コミットメモリは領域がディスクページングファイルに予約されている |
|  | Page/sec | ハードページフォールト（ディスクアクセスが必要）を解決するためにディスクから読み取られた、またはディスクへ書き込まれたページの数。 |
|  | Page Faults/sec | 1秒間のページフォールト数の平均、ページフォールト操作数とも一致する。 |
| PhysicalDisk | % Disk Time | ディスクドライブが読み取り、または書き込み要求を処理していてビジー状態にあった経過時間の比率 |
|  | Avg. Disk Queue Length | サンプリング間隔中に選択したディスクキューに入った読み取りおよび書き込み要求数の平均値 |
|  | Current Disk Queue Length | 処理中の要求も含むディスクに対する要求の数 |
|  | Disk Bytes/sec | 秒ごとの書き込みまたは読み取り操作により、ディスク間でバイトが転送された速度 |
| Process | Working Set | ワーキングセット（プロセスのスレッドが最後に参照したメモリページのセット）の現在のバイト数 |
| Processor | % Processor Time | プロセッサがスレッドを実行するために使用した経過時間の比率 |
|  | % Privileged Time | プロセスのスレッドが特権モードでコード実行に費やした経過時間の割合 |
|  | % User Time | ユーザーモードで経過した非アイドルプロセッサ時間の割合。 |
| System | Processor Queue Length | 処理中の要求は含まないプロセッサーキューにあるスレッドの数 |
| Network Adapter | Bytes Total/sec | 秒ごとのネットワークアダプター上で送受信されるバイト数 |

### SQL Server Counters

| Counter | Performance Object | Description |
| --- | --- | --- |
| SQLServer:Buffer Manager |  | SQL Server のメモリ使用量やバッファーキャッシュヒット率など、SQL Server で使用するメモリバッファーに関する情報 |
|  | Buffer Cache Hit Ratio | ディスクから読み取らずにバッファキャッシュ内で見つかったページの割合 |
|  | Total Pages | バッファキャッシュ内での総ページ数 |
|  | Database pages | データバッファキャッシュに使用しているページ数 |
|  | Procedure cache pages | プロシージャキャッシュに使用しているページ数 |
| SQLServer:Memory Manager |  | 現在割り当てられているロック構造の総数など、SQL Server のメモリの利用状況に関する情報 |
|  | Total Server Memory | サーバーが使用している動的メモリ総量 |
|  | Connection Memory | ユーザー接続用に使用しているメモリ量 |
|  | Lock Memory | ロックに使用しているメモリ量 |
| SQLServer:Plan Cache |  | ストアドプロシージャ、トリガー、クエリプランを格納するプロシージャキャッシュに関する情報 |
| SQLServer:Databases |  | トランザクションログの空きディスク容量データベース内のアクティブなトランザクションの数など、SQL Serverに関する情報 |
|  | Active Transactions | 実行中のトランザクション数 |
|  | Transactions/sec | 1秒あたりの実行トランザクション数 |
| SQLServer:Locks |  | ロックタイムアウトやデッドロックなど、SQL Server による各ロック要求についての情報 |
|  | Lock Waits/sec | ロック平均待ち時間 |
|  | Lock Timeout/sec | 1秒あたりのロックタイムアウト数 |
|  | Number of Deadlocks/sec | 1秒あたりのデッドロック数 |
| SQLServer:SQL Statistics |  | クエリのコンパイルと再コンパイルの回数、およびSQL Serverのインスタンスが受信するバッチの数に関する情報 |
| SQLServer:General Statistics |  | SQL Server インスタンスに接続しているユーザーの数など、サーバー全体の一般的な利用状況に関する情報 |
|  | User Connections | 接続中ログインユーザー数 |
|  | Logins/sec | 1秒あたりのログインユーザー数 |
| SQLServer:Exec Statistics |  | 分散クエリ、拡張ストアドプロシージャなどの実行数と実行時間 |
| SQLServer:Access Methods |  | データベース内のデータへのアクセス方法を監視するためのカウンター |
|  | Table Lock Escalations/sec | 1秒あたりのロックエスカレーション数 |
|  | Page Splits/sec | 1秒あたりのページ分割数 |

## Case Analysis

### High CPU Usage (User Time 100%, Low CPU Queue)

- Status
  - CPU使用率 User Time が 100%
  - CPU待ち Processor Queue Length が比較的少ない
- Cause
  - 一つの処理だけ遅くなっている可能性がある。
    - 1つ2つのバッチがひたすら実行している。
    - 重いSQLが少数実行されている。

### High CPU Usage (User Time 100%, High CPU Queue)

- Status
  - CPU使用率 User Time が 100%
  - CPU待ち Processor Queue Length が多発している
- Cause
  - サーバーの処理が遅くなっている。
    - CPUの内 User Time が高い場合、該当のプロセスを特定し対処する必要がある。
    - データベースチューニングが必要となる可能性がある。
    - CPU待ちが多いのはOLAPのように、多数のプロセスがサーバー上で稼働している可能性がある。

### Moderate CPU Usage (User Time 50%, Low CPU Queue)

- Status
  - CPU使用率 User Time が 50%程度
  - CPU待ち Processor Queue Length が少ない
- Cause
  - OSから見ると処理は遅くない。
    - データベースでデータのロック待ちなどが発生している場合がある。サーバーの処理が遅くなっている。

### Low CPU Usage with High Disk Queue

- Status
  - CPU使用率 User Time が 30%程度
  - CPU待ち Processor Queue Length はない
  - ディスクキュー Avg. Disk Queue Length が10〜20
- Cause
  - サーバーの処理が遅くなっている。
    - ページングの可能性がある。

### Idle CPU and Zero Disk Queue

- Status
  - CPU使用率 User Time が 0%
  - CPU待ち Processor Queue Length は少数
  - ディスクキュー Avg. Disk Queue Length も 0
- Cause
  - OSから見ると処理は遅くない。
    - サーバーに処理が届いていない可能性がある。

### Fixed 50% CPU Usage (Dual Core / Hyper-Threading)

- Status
  - CPU使用率 User Time が 50%
  - CPU待ち Processor Queue Length は 0
  - ディスクキュー Avg. Disk Queue Length も 0
- Cause
  - OS全体でみると処理は遅くない。
    - サーバーがデュアルコア、もしくはハイパースレッドの場合には、1つの処理だけ遅くなっている可能性がある。
    - 性能やリソースが限界に達し、CPU利用率が50%以上に上がらない可能性がある。

### High Privileged Time CPU Usage

- Status
  - CPU使用率 Privileged Time が通常より高い
  - CPU待ち Processor Queue Length が高い
- Cause
  - サーバーの処理が遅くなっている。
    - OS内部で何らかの問題が発生している。
    - スピンロック (mutex) により他のプロセスのロックを待っている可能性がある。
