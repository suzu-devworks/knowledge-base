# Chrony NTP Client and Server Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation](#installation)
  - [On Fedora, CentOS 8](#on-fedora-centos-8)
  - [On CentOS 7](#on-centos-7)
- [`chronyc` Commands](#chronyc-commands)
  - [Commands Without Authentication](#commands-without-authentication)
  - [Commands Requiring Authentication](#commands-requiring-authentication)
- [Client Configuration](#client-configuration)
  - [`chrony.conf`](#chronyconf)
- [Server Configuration](#server-configuration)
  - [Firewall settings](#firewall-settings)
  - [`chrony.conf` and others](#chronyconf-and-others)
- [Start Debugging](#start-debugging)
- [Start Service](#start-service)
- [Configurations](#configurations)
  - [Directive Reference](#directive-reference)
- [Troubleshooting](#troubleshooting)
  - [(SOLVED) Unable to sync time with NICT](#solved-unable-to-sync-time-with-nict)
  - [(SOLVED) 501 Not authorized](#solved-501-not-authorized)
  - [(SOLVED) SELinux prevents chronyd from write access to chronyd.pid](#solved-selinux-prevents-chronyd-from-write-access-to-chronydpid)
- [References](#references)

## Overview

`chrony` は、Network Time Protocol (NTP) の実装であり、従来のリファレンス実装である `ntpd` を置き換えるものです。
Linux や macOS を含む Unix 系オペレーティングシステムで動作し、GNU GPL v2 の下で公開されています。
Fedora 16 から採用され、Red Hat Enterprise Linux 8 や SUSE Linux Enterprise Server 15 では
デフォルトの NTP クライアントおよびサーバーとして標準採用されています。

- [Chrony Official Project Homepage](https://chrony-project.org/)

## Environments

動作検証を行った環境およびパッケージの履歴は以下の通りです。

- 2011-11-14: Fedora Core 16
- 2018-08-05: `chrony-3.3-1` on Fedora 28
- 2018-08-07: CentOS 7
- 2019-06-08: `chrony version 3.5` on Fedora 30
- 2020-01-06: `chrony-3.5-4.fc31.x86_64` on Fedora 31
- 2020-01-30: CentOS 8

## Installation

### On Fedora, CentOS 8

デフォルトでインストールされています。手動でインストールする場合は以下を実行します。

```bash
sudo dnf install chrony
```

### On CentOS 7

デフォルトでインストールされていますが、`ntp` パッケージも一緒にインストールされている場合があり分かりづらいため確認が必要です。
通常は `chrony` が標準で動作します。

```bash
yum list chrony ntp ntpdate
```

## `chronyc` Commands

### Commands Without Authentication

一般ユーザー権限でも実行可能なステータス確認コマンドです。

`activity` : オンライン中のサーバー数を表示:

```bash
chronyc activity
```

```console
200 OK
5 sources online
0 sources offline
0 sources doing burst (return to online)
0 sources doing burst (return to offline)
0 sources with unknown address
```

`sources` : 現在接続中の時刻ソースを表示:

```bash
chronyc sources
```

```console
210 Number of sources = 5
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ ntp-a3.nict.go.jp             1   6   377    28   +547us[ +547us] +/-   13ms
^+ ntp-a2.nict.go.jp             1   6   377    30   -726us[ -726us] +/-   12ms
^+ ntp-b3.nict.go.jp             1   6   377    28   +573us[ +573us] +/-   11ms
^* ntp-a3.nict.go.jp             1   6   377    32   +723us[ +682us] +/-   13ms
^- ntp3.jst.mfeed.ad.jp          2   6   377    29   -415us[ -415us] +/-  126ms
```

> [ `+` または `*` が同期中のソースを示す ]

`sourcestats` : ドリフト速度やオフセットなどの統計情報を表示

```bash
chronyc sourcestats
```

```console
210 Number of sources = 5
Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
==============================================================================

ntp-a3.nict.go.jp          18  10   913     -0.281      2.573   -357us   811us
ntp-a2.nict.go.jp          16   9   969     +0.554      2.554    +35us   695us
ntp-b3.nict.go.jp          18  13   915     -0.155      1.483   +787us   495us
ntp-a3.nict.go.jp          19  11   975     +0.009      1.478   -505us   523us
ntp3.jst.mfeed.ad.jp       18  11   913     -0.419      1.139   -230us   362us
```

`tracking` : システムクロックの性能や精度に関する詳細パラメータを表示

```bash
chronyc tracking
```

```console
Reference ID    : 85F3EEF4 (ntp-a3.nict.go.jp)
Stratum         : 2
Ref time (UTC)  : Sun Aug 05 10:39:45 2018
System time     : 0.000027244 seconds slow of NTP time
Last offset     : -0.000082145 seconds
RMS offset      : 0.000279789 seconds
Frequency       : 3.631 ppm fast
Residual freq   : -0.022 ppm
Skew            : 1.002 ppm
Root delay      : 0.023950603 seconds
Root dispersion : 0.000274640 seconds
Update interval : 64.4 seconds
Leap status     : Normal
```

### Commands Requiring Authentication

従来の `chrony.conf` 内の `commandKey` や `chrony.key` ファイルによるパスワード認証は非推奨・非サポートとなりました。
管理コマンドの実行には `root` または `chrony` ユーザー権限（`sudo`）が必要です。

`password` : パスワード認証の確認

```bash
chronyc password
```

```console
Authentication is no longer supported
```

`clients` : NTPクライアント一覧を表示

```bash
sudo chronyc clients
```

```console
Hostname                      NTP   Drop Int IntL Last     Cmd   Drop Int  Last
===============================================================================
localhost                       0      0   -   -     -       1      0   -    18
```

クライアントログ無効化時の表示例:

```bash
sudo /usr/libexec/chrony-helper command clients
```

```console
Hostname                      NTP   Drop Int IntL Last     Cmd   Drop Int  Last
===============================================================================

519 Client logging is not active in the daemon
```

`accheck` : アクセス制御（許可/拒否）の判定をチェック

```bash
sudo chronyc accheck 192.168.100.1
```

```console
208 Access allowed
209 Access denied
...
```

`makestep`: 強制的に時刻ステップ同期を実行

```bash
sudo chronyc -d makestep]
```

```console
Resolved 127.0.0.1 to 127.0.0.1
Resolved ::1 to ::1
Opening connection to /var/run/chrony/chronyd.sock
Sent 28 bytes
Timeout 1.000000 seconds
Received 28 bytes
Reply cmd=43 reply=1 stat=0
200 OK
```

## Client Configuration

### `chrony.conf`

標準のクライアント向け設定ファイル (`/etc/chrony.conf`) です。

<!-- spell-checker: disable -->
```text
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#pool 2.fedora.pool.ntp.org iburst

## [NICT] -- stratum 1
pool ntp.nict.jp iburst

## [NTP POOL PROJECT] -- stratum 2
pool jp.pool.ntp.org iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
#local stratum 10

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

# Get TAI-UTC offset and leap seconds from the system tz database.
leapsectz right/UTC

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
log measurements statistics tracking

## Send a message to syslog if a clock adjustment is larger than 0.5 seconds.
logchange 0.5
```

## Server Configuration

### Firewall settings

内部ネットワークのクライアントに時刻を提供するため、NTPサービスポート (`123/udp`) を解放します。

```bash
sudo firewall-cmd --list
```

```console
FedoraWorkstation (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s31f6
  sources: 
  services: dhcpv6-client ssh samba-client mdns
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

NTPサービスを一時的に追加:

```bash
sudo firewall-cmd --add-service=ntp
```

NTPサービスを恒久的に追加:

```bash
sudo firewall-cmd --add-service=ntp --permanent
```

```bash
sudo firewall-cmd --list-services
```

```console
dhcpv6-client ssh samba-client mdns ntp
```

### `chrony.conf` and others

サーバー設定に関連する主なファイル：

- `/etc/chrony.conf`
- `/etc/chrony.keys`
- `/etc/logrotate.d/chrony`

サーバーモード用の設定例 (`/etc/chrony.conf`)：

<!-- spell-checker: disable -->
```text
# Use public servers from the pool.ntp.org project.
#pool 2.fedora.pool.ntp.org iburst

## [NICT] -- stratum 1
server ntp.nict.jp iburst maxpoll 16
server ntp.nict.jp iburst maxpoll 16
server ntp.nict.jp iburst maxpoll 16
server ntp.nict.jp iburst maxpoll 16

## [mfeed] -- stratum 2
server ntp.jst.mfeed.ad.jp iburst maxpoll 12   

## Ignore stratum in source selection.
stratumweight 0

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Allow NTP client access from local subnets.
allow 192.168.100.0/24
allow 172.17.0.0/24

## Listen for commands only on localhost.
bindcmdaddress 127.0.0.1
bindcmdaddress ::1

# Serve time even if not synchronized to a time source.
local stratum 10

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

## Disable logging of client accesses.
noclientlog

## Send a message to syslog if a clock adjustment is larger than 0.5 seconds.
logchange 0.5

# Get TAI-UTC offset and leap seconds from the system tz database.
leapsectz right/UTC

# Specify directory for log files.
logdir /var/log/chrony
```
<!-- spell-checker: enable -->

## Start Debugging

手動で `chronyd` をデバッグモードで起動する場合のコマンドです。

```bash
sudo chronyd -u chrony -n -d
```

<!-- spell-checker: disable -->
```console
2018-08-05T10:17:57Z chronyd version 3.3 starting (+CMDMON +NTP +REFCLOCK +RTC +PRIVDROP +SCFILTER +SIGND +ASYNCDNS +SECHASH +IPV6 +DEBUG)
2018-08-05T10:17:57Z commandkey directive is no longer supported
2018-08-05T10:17:57Z Frequency 3.717 +/- 0.433 ppm read from /var/lib/chrony/drift
2018-08-05T10:17:57Z Using right/UTC timezone to obtain leap second data
2018-08-05T10:18:03Z Selected source 2001:df0:232:eea0::fff3
2018-08-05T10:18:03Z System clock TAI offset set to 37 seconds
[CTRL+C]
2018-08-05T10:18:06Z chronyd exiting
```
<!-- spell-checker: enable -->

## Start Service

`systemd` を使用したサービスの起動および状態確認です。

```bash
sudo systemctl start chronyd
```

```bash
sudo systemctl status chronyd
```

<!-- spell-checker: disable -->
```console
● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2018-08-05 19:23:30 JST; 5s ago
     Docs: man:chronyd(8)
           man:chrony.conf(5)
  Process: 11169 ExecStartPost=/usr/libexec/chrony-helper update-daemon (code=exited, status=0/SUCCESS)
  Process: 11165 ExecStart=/usr/sbin/chronyd $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 11167 (chronyd)
    Tasks: 1 (limit: 4915)
   Memory: 1.3M
   CGroup: /system.slice/chronyd.service
           └─11167 /usr/sbin/chronyd
8月 05 19:23:30 ntp.server.local systemd[1]: Starting NTP client/server...
8月 05 19:23:30 ntp.server.local chronyd[11167]: chronyd version 3.3 starting
8月 05 19:23:30 ntp.server.local chronyd[11167]: Frequency 3.717 +/- 0.433 ppm read from /var/lib/chrony/drift
8月 05 19:23:30 ntp.server.local systemd[1]: Started NTP client/server.
8月 05 19:23:35 ntp.server.local chronyd[11167]: Selected source 2001:df0:232:eea0::fff4
```
<!-- spell-checker: enable -->

## Configurations

### Directive Reference

<!-- spell-checker: disable -->
| Directive | Description |
| :- | :- |
| `pool` | NTP サーバープールを使用する場合に指定します。DNS ラウンドロビン等で複数の IP アドレスを返すサーバーをまとめて指定できます。`pool pool.ntp.org iburst` |
| `server` | 個別の参照先 NTP サーバーを指定します。<br>**主要オプション:** <br>`iburst`: 起動直後に 4 回連続で問い合わせを行い、初期の時刻同期処理を高速化します。<br> `maxpoll`: 最大ポーリング間隔を $2^n$ 秒で指定します（デフォルトは $10 \rightarrow 2^{10} = 1024\text{秒} \approx 17\text{分}$、$16 \rightarrow 2^{16} = 65536\text{秒} \approx 18\text{時間}$）。※ NICT の推奨規則に従い、`maxpoll` は 8 以上（$2^8 = 256\text{秒}$）に設定することが推奨されます。 |
| `stratumweight` | 同期先 NTP サーバーを選択する際、Stratum ごとの距離（秒）を指定します。`0` を指定することでサーバー選択時の Stratum による優先度計算を無視します。 |
| `driftfile` | システムクロックの進行速度のズレ（ドリフト率）を記録保存するファイルパスを指定します。 |
| `rtcsync` | 有効にすると、カーネルモードにより 11 分ごとにシステム時刻がリアルタイムクロック (RTC) へ自動コピーされます。 |
| `makestep` | 大きな時刻のズレが生じた際、徐々に補正するのではなくステップモードで即時補正します。`makestep 1.0 3`: 起動後の最初の 3 回の更新時において、ズレが 1.0 秒を超えている場合にステップ補正します。 |
| `allow` / `deny` | NTP クライアントから `chronyd` へのアクセス制御ルールを設定します（IPv4/IPv6 CIDR 表記に対応）。デフォルトは全拒否です。 |
| `cmdallow` / `cmddeny` | `chronyc` コマンドによる管理アクセス制御ルールを設定します。 |
| `local` | 外部の NTP ソースが利用できない場合でも、自サーバーの RTC をマスターとして動作させる際の Stratum を指定します（例: `local stratum 10`）。 |
| `keyfile` | NTP パケット認証および `chronyc` 管理コマンド認証で使用する鍵ペア定義ファイルのパスを指定します。 |
| `cmdport` | `chronyc` コマンド通信用の UDP ポート番号を指定します（デフォルト: `323/udp`）。 |
| `noclientlog` | クライアントからのアクセスログ記録を無効化します（デバッグ時はコメントアウトして有効化）。 |
| `logchange` | システムクロックの調整量が指定値（秒）を超えた場合、syslog へ警告メッセージを出力します。 |
| `logdir` | ログファイルの出力先ディレクトリを指定します。 |
| `log` | ログとして記録するカテゴリを指定します (`measurements`, `statistics`, `tracking`, `rtc`)。 |
| `manual` | `chronyc` 経由での手動時刻設定コマンド (`settime`) の受け入れを有効化します。デフォルトは無効です。 |
<!-- spell-checker: enable -->

## Troubleshooting

### (SOLVED) Unable to sync time with NICT

> 2011-11-14: FC16

- 現象:
  - ネットワーク環境により、IPv6 通信で NICT サーバーと時刻同期ができない場合があります。
- 対策:
  - `/etc/sysconfig/chronyd` ファイルを作成または編集し、IPv4 通信を強制するオプションを追加します。

```ini
# /etc/sysconfig/chronyd
OPTIONS=-4
```

### (SOLVED) 501 Not authorized

> 2018-08-05: FC28

- 現象:
  - `chronyc -a makestep` 等を実行した際、`501 Not authorized` エラーが発生する。  
- 原因:
  - バージョン 2.2 以降、パスワード認証コマンドが廃止されました。  
- 対策:
  - `sudo` 権限を付与するか、`root` または `chrony` ユーザーでローカル実行します。

NG:

```console
$ chronyc -a makestep

501 Not authorized
```

OK:

```console
$ sudo chronyc makestep

200 OK
```

### (SOLVED) SELinux prevents chronyd from write access to chronyd.pid

> 2018-08-05: FC28

- 現象:**
  - SELinux のセキュリティポリシーにより、`chronyd` が `/var/run/chrony/chronyd.pid` に書き込みを行えず起動失敗する。  
- 対策:
  - ファイルのセキュリティコンテキストを再設定または復元します。

```bash
sudo restorecon -v /var/run/chrony/chronyd.pid
```

## References

- [RFC 1305 - Network Time Protocol (Version 3) Specification](https://datatracker.ietf.org/doc/html/rfc1305)
- [RFC 2030 - Simple Network Time Protocol (SNTP) Version 4](https://datatracker.ietf.org/doc/html/rfc2030)
