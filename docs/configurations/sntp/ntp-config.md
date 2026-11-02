# NTP Server Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Time Synchronization Methods](#time-synchronization-methods)
- [Installation](#installation)
  - [Fedora and CentOS](#fedora-and-centos)
- [Firewall Configuration](#firewall-configuration)
- [ntpd Configuration](#ntpd-configuration)
- [Starting the Service](#starting-the-service)
  - [Using Systemd](#using-systemd)
  - [Using SystemV Service](#using-systemv-service)
- [ntpdate](#ntpdate)
  - [Hardware Clock Synchronization](#hardware-clock-synchronization)
- [Other Date Commands](#other-date-commands)
- [Troubleshooting](#troubleshooting)
  - [(SOLVED) Unable to Synchronize with ntpd in Local Network](#solved-unable-to-synchronize-with-ntpd-in-local-network)
- [References](#references)

## Overview

NTP (Network Time Protocol) は、ネットワークを経由してコンピュータの時刻を同期するためのプロトコルです。
通信には UDP プロトコルの 123 番ポートを相互に使用します。

> [!NOTE]
> 2006年（平成18年）6月12日より、[NICT](http://www2.nict.go.jp/w/w114/tsp/PubNtp/index.html) にて
> 毎秒100万リクエストの処理能力を持つStratum 1 の公開NTPサービスが開始されました。

<!-- spell-checker: words NICT -->

## Environments

動作確認および検証を実施した環境は以下の通りです。

- CentOS 7 (ntpd)
- Fedora 各バージョン (FC14, Fedora XX)

## Time Synchronization Methods

| 同期モード | 説明 |
| ----- | ----- |
| slew | 徐々に時刻を合わせていくモード。デフォルトでNTPサーバーとの時刻のずれが 128 ms 以内の場合は slew モードとなるため、時刻の急激な変化を防ぎつつ 128 ms 以内に収めます。 |
| step | 1回の処理で強制的に時刻を合わせるモード。システム起動時の特別な場合や、ネットワーク状態の悪化などが原因で 128 ms 以上の大幅なずれが生じている場合に実施されます。 |

## Installation

### Fedora and CentOS

```bash
sudo yum install ntp
```

## Firewall Configuration

```bash
sudo iptables -A INPUT -m state --state NEW -m udp -p udp --dport 123 -j ACCEPT
```

<!-- spell-checker: words dport -->

## ntpd Configuration

設定ファイルパス: `/etc/ntp.conf`

<!-- spell-checker: disable -->
```text
# For more information about this file, see the man pages
# ntp.conf(5), ntp_acc(5), ntp_auth(5), ntp_clock(5), ntp_misc(5), ntp_mon(5).

# 変動情報記録ファイルの指定 (起動前にファイルが存在することを確認し、なければ作成しておく)
driftfile /var/lib/ntp/drift

# デフォルトですべてのNTPアクセスを拒否
restrict default ignore
restrict -6 default ignore

# ループバックアドレスからのNTPアクセスを許可
restrict 127.0.0.1
restrict -6 ::1

# 内部ネットワークからの同期および参照を許可
restrict 192.168.0.0 mask 255.255.255.0 nomodify notrap noquery

# pool.ntp.org プロジェクトの公開サーバーを使用
# (プロジェクトへの参加検討はこちら: http://www.pool.ntp.org/join.html)
# maxpoll はデフォルトで 10 ($2^{10} = 1024\text{秒}$) ですが、16 ($2^{16} = 65536\text{秒} \approx 18.2\text{時間}$) 程度で問題ありません

# NICT (Stratum 1)
restrict ntp.nict.jp nomodify notrap noquery
server ntp.nict.jp iburst maxpoll 16
server ntp.nict.jp iburst maxpoll 16
server ntp.nict.jp iburst maxpoll 16

# MFEED (Stratum 2)
restrict ntp.jst.mfeed.ad.jp nomodify notrap noquery
server ntp.jst.mfeed.ad.jp maxpoll 16

# ローカルクロック (上記サーバーにすべて参照できない場合の予備)
server 127.127.1.0 # local clock
fudge 127.127.1.0 stratum 10

# 以下はデフォルト設定を維持
# Enable public key cryptography.
#crypto
includefile /etc/ntp/crypto/pw

# Key file containing the keys and key identifiers used when operating
# with symmetric key cryptography.
keys /etc/ntp/keys

# Specify the key identifiers which are trusted.
#trustedkey 4 8 42

# Specify the key identifier to use with the ntpdc utility.
#requestkey 8

# Specify the key identifier to use with the ntpq utility.
#controlkey 8

# Enable writing of statistics records.
#statistics clockstats cryptostats loopstats peerstats
```
<!-- spell-checker: enable -->

> [!WARNING]
> `server` や `restrict` ディレクティブに `-4` オプションを指定すると各種エラーの原因となるため、指定しないでください。

## Starting the Service

### Using Systemd

サービス起動:

```bash
sudo systemctl start ntpd
```

サービス状態確認:

```bash
sudo systemctl status ntpd
```

```console
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled)
   Active: active (running) since Thu 2014-11-27 23:56:12 JST; 4min 0s ago
 Main PID: 10061 (ntpd)
   CGroup: /system.slice/ntpd.service
           └─10061 /usr/sbin/ntpd -u ntp:ntp -g
...
Hint: Some lines were ellipsized, use -l to show in full.
```
<!-- spell-checker: words ellipsized -->

ntpd の自動起動を有効化:

```bash
sudo systemctl enable ntpd
```

同期状態および参照先サーバーの確認:

```bash
ntpq -p
```

<!-- spell-checker: words ntpq -->

```console
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 ntp-a2.nict.go. .INIT.          16 u    -   64    0    0.000    0.000   0.000
 ntp-a3.nict.go. .INIT.          16 u    -   64    0    0.000    0.000   0.000
 ntp-b3.nict.go. .INIT.          16 u    -   64    0    0.000    0.000   0.000
 ntp3.jst.mfeed. .INIT.          16 u    -   64    0    0.000    0.000   0.000
*LOCAL(0)        .LOCL.          10 l   49   64    1    0.000    0.000   0.000
```
<!-- spell-checker: words refid mfeed LOCL -->

### Using SystemV Service

大幅に時間がずれていると ntpd が起動できないため、事前に手動で同期する:

```bash
sudo ntpdate ntp.nict.jp
```

```console
ntp.nict.jp 8 Mar 16:32:07 ntpdate[3289]: adjust time server 133.243.238.243 offset -0.026441 sec
```

NTPD 起動:

```bash
sudo service ntpd restart
```

```console
ntpd を停止中:                                     [  OK  ]
ntpd を起動中:                                     [  OK  ]
```

参照先サーバーの確認:

```bash
ntpq -p
```

```console
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 ntp-a2.nict.go. .NICT.           1 u    6   64    1   35.880   54.561   0.000
 ntp3.jst.mfeed. 210.173.160.86   2 u    5   64    1   30.302   57.442   0.000
 LOCAL(0)        .LOCL.          10 l    4   64    1    0.000    0.000   0.000
```

chkconfig で ntpd の自動起動を設定:

<!-- spell-checker: words chkconfig -->

```bash
sudo chkconfig ntpd on
```

## ntpdate

マシン自体の時刻を一時的に調整するだけであれば NTP パッケージに含まれる `ntpdate` コマンドも使用できますが、
公式ドキュメントによると `ntpdate` はエラーハンドリングが不十分なため、`ntpd -q` オプションによる時刻合わせを行うことが推奨されています。

> [!WARNING]
> `ntpd` が起動している状態で `ntpdate` を実行すると "socket is in use" エラーになります。事前に `ntpd` を停止してください。

```bash
sudo service ntpd stop
```

```console
ntpd を停止中:                                     [  OK  ]
```

```bash
sudo ntpdate ntp.nict.jp
```

```console  
ntp.nict.jp 8 Mar 16:32:07 ntpdate[3289]: adjust time server 133.243.238.243 offset -0.026441 sec
```

ntpdate サービス起動

```bash
sudo service ntpdate start
```

```console
ntpdate: 時間サーバーと同期中:                    [  OK  ]
```

### Hardware Clock Synchronization

`ntpdate` の起動時に時刻合わせを行った後、同時にハードウェアクロック (`hwclock`) も動的に合わせる場合は以下のファイルを設定します。

設定ファイルパス: `/etc/sysconfig/ntpdate`

<!-- spell-checker: words hwclock sysconfig -->

```ini
# Options for ntpdate
OPTIONS="-p 2"

# Number of retries before giving up
RETRIES=2

# Set to 'yes' to sync hw clock after successful ntpdate
SYNC_HWCLOCK=yes
```

## Other Date Commands

システムクロックの確認:

```bash
date
```

```console
2010年 6月 24日 木曜日 18:22:26 JST
```

ハードウェアクロックの確認:

```bash
hwclock
```

```console
2010年06月24日 18時23分12秒 -0.390909 秒
```

NTPで合わせたシステムクロックをハードウェアクロックへ反映:

```bash
sudo hwclock --systohc
```
<!-- spell-checker: words systohc -->

## Troubleshooting

### (SOLVED) Unable to Synchronize with ntpd in Local Network

> FC14

内部ネットワークから、構築した NTPD サーバーに対して時刻同期を要求した際に以下のエラーが発生しました。

```bash
sudo ntpdate -d <localTimeServer>
```

```console
transmit(xxx.xxx.xxx.xxx)
...
xxx.xxx.xxx.xxx: Server dropped: no data
...
server xxx.xxx.xxx.xxx, port 123
...
no server suitable for synchronization found
```

原因と対処手順:

1. 最初にファイアウォール（iptables / Port 123 UDP）の設定を確認・開放しました。
2. `ntp.conf` の `restrict` 設定を調整する際、直接個別IPアドレスを指定すると同期できますが、サブネットマスク (`mask`) で複数アドレスを一括指定した場合に正常に同期できていないことが判明しました。
3. 検証として `restrict` 設定から `-4` オプションを外したところ、正常に同期が完了しました。

## References

- [RFC 1305 - Network Time Protocol (Version 3) Specification](https://datatracker.ietf.org/doc/html/rfc1305)
- [RFC 2030 - Simple Network Time Protocol (SNTP) Version 4](https://datatracker.ietf.org/doc/html/rfc2030)
