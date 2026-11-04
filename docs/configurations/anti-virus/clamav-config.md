# Clam Antivirus Configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation](#installation)
  - [Installing ClamAV on Fedora](#installing-clamav-on-fedora)
- [Freshclam Configuration](#freshclam-configuration)
  - [Configuring freshclam Command](#configuring-freshclam-command)
  - [Automating Updates via Cron](#automating-updates-via-cron)
  - [Logging Setup](#logging-setup)
  - [Integration with clamd Daemon](#integration-with-clamd-daemon)
  - [Running freshclam](#running-freshclam)
- [Clamd Configuration](#clamd-configuration)
  - [Configuring clamdscan Command](#configuring-clamdscan-command)
  - [On-Access Scan Setup](#on-access-scan-setup)
  - [Desktop Notifications](#desktop-notifications)
  - [Logging Setup for clamd](#logging-setup-for-clamd)
  - [SELinux Configuration](#selinux-configuration)
  - [Starting the Service](#starting-the-service)
  - [Permissions for Executable Users](#permissions-for-executable-users)
- [Running clamdscan](#running-clamdscan)
  - [Options](#options)
  - [Return Codes](#return-codes)
- [Testing Scan with Sample Malware](#testing-scan-with-sample-malware)
  - [ANTI-MALWARE TESTFILE](#anti-malware-testfile)
- [Obsolete: clamscan Command](#obsolete-clamscan-command)
  - [Running clamscan](#running-clamscan)
  - [Main options](#main-options)

## Overview

Clam AntiVirus（ClamAV）とは、オープンソースで提供されているクロスプラットフォームのアンチウイルスソフトウェアである。Clam AntiVirus の開発プロジェクトでは、メールゲートウェイ等で電子メールやファイルのウイルススキャンを行うことを開発目標の主眼としている。

- [ClamAVNet](https://www.clamav.net/)

## Environments

- 2015-10-29: `ClamAV 0.98.7` on Fedora 22
- 2017-07-17: `ClamAV 0.99.2` on Fedora 26
- 2018-08-04: `clamav-0.100.1-1` on Fedora 28
- 2019-06-06: `ClamAV 0.101.2` on Fedora 30
- 2020-01-07: `clamav-0.101.5-1.fc31.x86_64` on Fedora 31

## Installation

### Installing ClamAV on Fedora

DNF パッケージマネージャーを使用して関連パッケージを検索します。

```bash
sudo dnf search clam
```

<!-- spell-checker: disable -->
```console
メタデータの期限切れの確認は、1:00:25 時間前の 2018年08月04日 15時19分12秒 に実施しました。
================================= 概要 & 名前 の一致: clam ==================================
clamd.x86_64 : The Clam AntiVirus Daemon
clamav-filesystem.noarch : Filesystem structure for clamav
clamav.x86_64 : End-user tools for the Clam Antivirus scanner
clamav-milter.x86_64 : Milter module for the Clam Antivirus scanner
clamav-lib.x86_64 : Dynamic libraries for the Clam Antivirus scanner
clamav-lib.i686 : Dynamic libraries for the Clam Antivirus scanner
clamav-server-systemd.x86_64 : Systemd initscripts for clamav server
clamav-data.noarch : Virus signature data for the Clam Antivirus scanner
clamtk.noarch : Easy to use graphical user interface for Clam anti virus
clamav-data-empty.noarch : Empty data package for the Clam Antivirus scanner
clamav-update.x86_64 : Auto-updater for the Clam Antivirus scanner data-files
clamav-scanner-systemd.x86_64 : systemd initscripts for clamav scanner daemon
clamav-devel.i686 : Header files and libraries for the Clam Antivirus scanner
clamav-devel.x86_64 : Header files and libraries for the Clam Antivirus scanner
clamav-unofficial-sigs.noarch : Scripts to download unoffical clamav signatures
clamav-milter-systemd.x86_64 : Systemd initscripts for the clamav sendmail-milter
exim-clamav.x86_64 : Clam Antivirus scanner daemon configuration for use with Exim
perl-ClamAV-Client.noarch : Client class for the ClamAV clamd virus scanner daemon
claws-mail-plugins-clamd.x86_64 : Use Clam AntiVirus to scan messages in Claws Mail
===================================== 名前 の一致: clam =====================================
clamz.x86_64 : Amazon Downloader
clamsmtp.x86_64 : A SMTP virus scanning system
thunar-sendto-clamtk.noarch : Simple virus scanning extension for Thunar
===================================== 概要 の一致: clam =====================================
klamav.x86_64 : Clam Anti-Virus on the KDE Desktop
```
<!-- spell-checker: enable -->

ClamAV 本体、データベース更新ツール、systemd スキャナーモジュールをインストールします。

```bash
sudo dnf install clamav clamav-update clamav-scanner-systemd
```

## Freshclam Configuration

### Configuring freshclam Command

*※ Fedora 30 以降では初期状態でコメントアウト済みです。*

`/etc/freshclam.conf` 内の `Example` 行を確認します。

```console
$ sudo grep -e Example /etc/freshclam.conf

## Example config file for freshclam
#Example ★ コメントアウト済みであることを確認
```

実行時に以下のエラーが表示された場合：

```console
$ sudo freshclam

ERROR: Please edit the example config file /etc/freshclam.conf
ERROR: Can't open/parse the config file /etc/freshclam.conf
```

`sed` コマンドで `Example` 行をコメントアウトします。

```bash
sudo sed -e 's/^Example/\#Example/g' -i /etc/freshclam.conf
```

`/etc/freshclam.conf` の設定内容：

```text
# Comment or remove the line below.
#Example ← コメントアウト
```

### Automating Updates via Cron

cron で `freshclam-sleep`（または `freshclam`）を実行してパターンファイルを自動更新します。
*※ Fedora 30 以降ではデフォルトでコメントアウト済みです。*

デフォルトの `/etc/cron.d/clamav-update` 設定：

```text
## Adjust this line...
MAILTO=root
## It is ok to execute it as root; freshclam drops privileges and becomes
## user 'clamupdate' as soon as possible
0 */3 * * * root /usr/share/clamav/freshclam-sleep
```

`/etc/sysconfig/freshclam` の確認：

```console
$ sudo grep -e FRESHCLAM_DELAY /etc/sysconfig/freshclam

# FRESHCLAM_DELAY= ★ コメントアウト済み
```

起動しない場合は、コメントアウト処理を実施します。

```bash
sudo sed -e 's/^FRESHCLAM_DELAY/\#FRESHCLAM_DELAY/g' -i /etc/sysconfig/freshclam
```

`/etc/sysconfig/freshclam` の設定内容：

```text
### REMOVE ME: By default, the freshclam update is disabled to avoid
### REMOVE ME: network access without prior activation
#FRESHCLAM_DELAY=disabled-warn         # コメントアウト
```

### Logging Setup

初期状態では `freshclam` のログは `syslog` にのみ出力されるため、ログローテーションが効果的に機能しません。ウイルスデータベース更新ログを分離してファイルに出力するよう設定します。

`/etc/logrotate.d/clamav-update` (デフォルト)：

```text
/var/log/freshclam.log {
    monthly
    notifempty
}
```

`/etc/freshclam.conf` の編集：

```text
# Path to the log file (make sure it has proper permissions)
# Default: disabled
# コメントを外す
UpdateLogFile /var/log/freshclam.log

# Log time with each message.
# Default: no
# コメントを外す (UpdateLogFile に日時を記録)
LogTime yes

# Enable verbose logging.
# Default: no
# コメントを外す
LogVerbose yes

# Use system logger (can work together with UpdateLogFile).
# Default: no
# コメントアウト (syslog には出力しない)
# LogSyslog yes
```

### Integration with clamd Daemon

`freshclam` で定義ファイルを更新した際、`clamd` にデータベースを再読み込み（RELOAD）させる連携設定を行います。

```bash
sudo sed -e 's/\#\(NotifyClamd\).*/\1 \/etc\/clamd.d\/scan.conf/' -i /etc/freshclam.conf
```

`/etc/freshclam.conf` の設定内容：

```text
# Send the RELOAD command to clamd.
# Default: no
NotifyClamd /etc/clamd.d/scan.conf
```

### Running freshclam

手動でパターン更新コマンドを実行します。

```bash
sudo freshclam --quiet
```

<!-- spell-checker: disable -->
```console
$ ls -la /var/lib/clamav/

total 170448
drwxr-xr-x.  2 clamupdate clamupdate      4096 Jan  7 22:52 .
drwxr-xr-x. 63 root       root            4096 Jan  7 22:09 ..
-rw-r--r--.  1 clamupdate clamupdate    296388 Jan  7 22:52 bytecode.cvd
-rw-r--r--.  1 clamupdate clamupdate  56363436 Jan  7 22:52 daily.cvd
-rw-r--r--.  1 clamupdate clamupdate 117859675 Jan  7 22:52 main.cvd
-rw-------.  1 clamupdate clamupdate        64 Jan  7 22:52 mirrors.dat

$ tail /var/log/freshclam.log

Tue Jan  7 22:52:08 2020 -> Downloading main.cvd [100%]
Tue Jan  7 22:52:14 2020 -> main.cvd updated (version: 59, sigs: 4564902, f-level: 60, builder: sigmgr)
Can'query main.59.106.1.0.2606470000000000000000006810da54.ping.clamav.net
Tue Jan  7 22:52:19 2020 -> Downloading daily.cvd [100%]
Tue Jan  7 22:52:24 2020 -> daily.cvd updated (version: 25687, sigs: 2095762, f-level: 63, builder: raynman)
Can't query daily.25687.106.1.0.2606470000000000000000006810da54.ping.clamav.net
Tue Jan  7 22:52:27 2020 -> Downloading bytecode.cvd [100%]
Tue Jan  7 22:52:27 2020 -> bytecode.cvd updated (version: 331, sigs: 94, f-level: 63, builder: anvilleg)
Can't query bytecode.331.106.1.0.2606470000000000000000006810da54.ping.clamav.net
Tue Jan  7 22:52:32 2020 -> Database updated (6660758 signatures) from database.clamav.net (IP: 2606:4700::6810:da54)
```
<!-- spell-checker: enable -->

## Clamd Configuration

`clamd` はサーバー（デーモン）として動作し、TCP または Unix ソケットを通じてクライアントからの要求を受け取りウイルスを検査します。頻繁にスキャンを実施する環境では `clamscan` よりも `clamd` を利用する方が効率的です。

### Configuring clamdscan Command

未設定状態で実行するとエラーになります。

```console
$ clamdscan

ERROR: Please edit the example config file /etc/clamd.d/scan.conf
ERROR: Can't parse clamd configuration file /etc/clamd.d/scan.conf
```

`sed` コマンドで設定ファイル `/etc/clamd.d/scan.conf` を一括調整します。

```bash
sudo sed -e 's/^Example/\#Example/g' \
         -e 's/^\#LocalSocket /LocalSocket /g' \
         -e 's/^\#LocalSocketGroup /LocalSocketGroup /g' \
         -e 's/^\#LocalSocketMode /LocalSocketMode /g' \
         -e 's/^\#FixStaleSocket /FixStaleSocket /g' \
         -e 's/^TCPSocket /\#TCPSocket /g' \
         -e 's/^\#ExcludePath /ExcludePath /g' \
         -e '/ExcludePath ^\/sys\//a ExcludePath ^\/dev\/' \
         -e 's/^User /\#User /g' \
         -i /etc/clamd.d/scan.conf
```

`/etc/clamd.d/scan.conf` の設定内容：

```text
# Example をコメントアウト
#Example

# デーモンが受信するローカルソケットのパス
LocalSocket /var/run/clamd.scan/clamd.sock

# ソケットファイルの所有グループ設定
LocalSocketGroup virusgroup

# ソケットファイルのパーミッション
LocalSocketMode 660

# 不正終了時の古いソケットファイルを自動削除
FixStaleSocket yes

# スキャン対象から除外する仮想ディレクトリ
ExcludePath ^/proc/
ExcludePath ^/sys/
ExcludePath ^/dev/
```

### On-Access Scan Setup

`ScanOnAccess` を使用してオンアクセススキャン（リアルタイム保護）を有効化します。

> **注意点:**
>
> - `OnAccessIncludePath` にはルートディレクトリ `"/"` を指定できません。
> - `OnAccessMountPath` には `"/"` を指定可能ですが、以下のオプションと組み合わせることはできません。
>   - `OnAccessExtraScanning`
>   - `OnAccessExcludePath`（※ `OnAccessIncludePath` 専用のため）
>   - `OnAccessPrevention`（※ `"/"` がロックされるため）

システム全体を対象とする場合：

```bash
sudo sed -e 's/^[\#]*\(ScanOnAccess\).*$/\1 yes/' \
         -e 's/^[\#]*\(OnAccessMountPath \/$\)/\1/' \
         -e 's/^\(OnAccessPrevention.*\)$/\#\1/' \
         -e 's/^[\#]*\(OnAccessExcludeRootUID\).*$/\1 yes/' \
         -i /etc/clamd.d/scan.conf
```

特定のディレクトリのみを対象とする場合：

```bash
sudo sed -e 's/^[\#]*\(ScanOnAccess\).*$/\1 yes/' \
         -e 's/^[\#]*\(OnAccessIncludePath \/$\)/\1/' \
         -e '/^\#OnAccessIncludePath \/students$/a OnAccessIncludePath \/home' \
         -e '/^\#OnAccessExcludePath.*$/a OnAccessExcludePath \/home\/storage\/ISOs' \
         -i /etc/clamd.d/scan.conf
```

`/etc/clamd.d/scan.conf` の設定内容：

```text
## On-access Scan Settings
ScanOnAccess yes

# スキャン対象のマウントポイント設定
OnAccessMountPath /

# root ユーザー (UID 0) のアクセスを除外
OnAccessExcludeRootUID yes
```

### Desktop Notifications

GNOME の `notify-send` を使用してデスクトップ通知を設定します。

通知スクリプト `/usr/local/bin/clamd-detected.sh` を作成します。

<!-- spell-checker: disable -->
```bash
#!/bin/bash
PATH=/usr/bin
alert="Signature detected: $CLAM_VIRUSEVENT_VIRUSNAME in $CLAM_VIRUSEVENT_FILENAME"

# systemd ログ出力コマンドの存在確認を行いログに記録
if [[ -z $(command -v systemd-cat) ]]; then 
    echo "$(date) - $alert" >> /var/log/clamav/infected.log
else 
    echo "$alert" | /usr/bin/systemd-cat -t clamav -p emerg
fi

# ログイン中のすべてのグラフィカルユーザーセッションにアラートを送信
XUSERS=($(who | awk '{print $1$NF}' | sort -u))
for XUSER in "${XUSERS[@]}"; do
    NAME=(${XUSER/(/ })
    DISPLAY=${NAME[1]/)/}
    DBUS_ADDRESS=unix:path=/run/user/$(id -u "${NAME[0]}")/bus
    echo "run $NAME - $DISPLAY - $DBUS_ADDRESS -" >> /tmp/testlog 
    /usr/bin/sudo -u "${NAME[0]}" DISPLAY="${DISPLAY}" \
                      DBUS_SESSION_BUS_ADDRESS="${DBUS_ADDRESS}" \
                      PATH="${PATH}" \
                      /usr/bin/notify-send -i dialog-warning "ClamAV Alert" "$alert"
done
```
<!-- spell-checker: enable -->

`/etc/clamd.d/scan.conf` にスクリプト呼び出し設定を追加します。

```bash
sudo sed -e '/^\#VirusEvent /a VirusEvent \/usr\/local\/bin\/clamd-detected\.sh' \
         -i /etc/clamd.d/scan.conf
```

`/etc/clamd.d/scan.conf` の設定内容：

```text
VirusEvent /usr/local/bin/clamd-detected.sh
```

### Logging Setup for clamd

`clamd` のログを専用ファイルに出力する設定を行います。

```bash
sudo sed -e 's/^\#LogFile /LogFile /' \
         -e 's/^\#LogTime/LogTime/' \
         -e 's/^LogSyslog/\#LogSyslog/' \
         -i /etc/clamd.d/scan.conf
```

`/etc/clamd.d/scan.conf` の設定内容：

```text
LogFile /var/log/clamd.scan
LogTime yes
# LogSyslog yes
```

新しいログローテーション設定 `/etc/logrotate.d/clamd` を作成します。

```text
/var/log/clamd.scan {
    monthly
    notifempty
    missingok
    postrotate
        killall -HUP clamd 2>/dev/null || :
    endscript
}
```

ログローテーション設定のテストを実行します。

```bash
sudo logrotate -d /etc/logrotate.d/clamd
```

```console
reading config file /etc/logrotate.d/clamd
Allocating hash table for state file, size 64 entries
Creating new state
...
Handling 1 logs
rotating pattern: /var/log/clamd.scan  monthly (no old logs will be kept)
empty log files are not rotated, old logs are removed
considering log /var/log/clamd.scan
Creating new state
  Now: 2018-08-04 16:51
  Last rotated at 2018-08-04 16:00
  log does not need rotating (log has been already rotated)
```

### SELinux Configuration

SELinux の Boolean 値を確認します。

```console
% sudo getsebool -a | grep -E "(clam|virus)"

antivirus_can_scan_system --> on
antivirus_use_jit --> on
```

`on` になっていない場合は永続設定を有効化します。

```bash
sudo setsebool -P antivirus_use_jit on 
sudo setsebool -P antivirus_can_scan_system on
```

### Starting the Service

1. パターンファイルを最新化します。

    ```bash
    sudo freshclam
    ```

2. サービスを有効化して起動します。

    ```bash
    sudo systemctl enable clamd@scan
    ```

    ```console
    Created symlink /etc/systemd/system/multi-user.target.wants/clamd@scan.service → /usr/lib/systemd/system/clamd@scan.service.
    ```

    ```bash
    sudo systemctl start clamd@scan
    ```

3. サービスの状態を確認します。

    ```bash
    sudo systemctl status clamd@scan
    ```

    ```console
    ● clamd@scan.service - Generic clamav scanner daemon
        Loaded: loaded (/usr/lib/systemd/system/clamd@scan.service; enabled; vendor preset: disabled)
        Active: active (running) since Sat 2018-08-04 17:10:14 JST; 30s ago
      Main PID: 14745 (clamd)
        Tasks: 2 (limit: 4915)
        Memory: 527.1M
        CGroup: /system.slice/system-clamd.slice/clamd@scan.service
                └─14745 /usr/sbin/clamd -c /etc/clamd.d/scan.conf
    ```

### Permissions for Executable Users

一般ユーザーで `clamdscan` を実行した際に権限エラーとなる場合、ソケットファイル `/run/clamd.scan/clamd.sock` へのアクセス権を与えるため `virusgroup` ユーザーグループに追加します。

```console
$ clamdscan

ERROR: Could not connect to clamd on LocalSocket /run/clamd.scan/clamd.sock: Permission denied
```

```bash
sudo usermod -aG virusgroup $USER
```

> ※ 設定適用にはシステムの再起動または再ログインが必要です。

## Running clamdscan

カレントディレクトリをスキャンします。

```console
$ clamdscan

home/user/: OK

----------- SCAN SUMMARY -----------
Infected files: 0
Time: 135.766 sec (2 m 15 s)
```

`/home` 配下全体をスキャンします。

```bash
clamdscan /home
```

### Options

| オプション | 説明 |
| --- | --- |
| `--quiet` | エラーメッセージのみを出力 |
| `--stdout` | 標準エラー（stderr）ではなく標準出力（stdout）に出力 |
| `--remove` | 感染しているファイルを削除（※取扱い注意） |
| `--move=DIRECTORY` | 感染しているファイルを指定ディレクトリへ移動 |
| `--copy=DIRECTORY` | 感染しているファイルを指定ディレクトリへコピー |

### Return Codes

- `0`: ウイルスは検出されませんでした。
- `1`: ウイルスが検出されました。
- `2`: エラーが発生しました。

## Testing Scan with Sample Malware

### ANTI-MALWARE TESTFILE

EICAR が提供している無害なテスト用サンプルウイルスを取得し、正常に検知・駆除できるか確認します。

- [Download Anti Malware Testfile - EICAR](https://www.eicar.org/download-anti-malware-testfile/)

サンプルウイルスの取得とスキャン・削除の実行：

```console
# サンプルウイルス（無害）を取得
$ wget http://www.eicar.org/download/eicar.com \
       http://www.eicar.org/download/eicar.com.txt \
       http://www.eicar.org/download/eicar_com.zip \
       http://www.eicar.org/download/eicarcom2.zip

# サンプルウイルスをスキャンして自動駆除（削除）
$ clamdscan --remove

/home/user/working/eicar.com: Eicar-Test-Signature FOUND
/home/user/working/eicar.com: Removed.
/home/user/working/eicar.com.txt: Eicar-Test-Signature FOUND
/home/user/working/eicar.com.txt: Removed.
/home/user/working/eicar_com.zip: Eicar-Test-Signature FOUND
/home/user/working/eicar_com.zip: Removed.
/home/user/working/eicarcom2.zip: Eicar-Test-Signature FOUND
/home/user/working/eicarcom2.zip: Removed.

----------- SCAN SUMMARY -----------
Infected files: 4 (ウイルスが発見され駆除された）
Time: 16.540 sec (0 m 16 s)
```

## Obsolete: clamscan Command

スタンドアロンコマンドの `clamscan` は、実行するたびにパターンファイルをメモリにロードするためスキャンに時間がかかります（非推奨）。

### Running clamscan

ホームディレクトリ配下を再帰的にスキャンします。

```console
$ clamscan -i -r ~

----------- SCAN SUMMARY -----------
Known viruses: 726063
Engine version: 0.95.3
Scanned directories: 278
Scanned files: 1431
Infected files: 0 (ウイルスは検出されなかった）
Data scanned: 81.36 MB
Data read: 71.17 MB (ratio 1.14:1)
Time: 10.560 sec (0 m 10 s)
```

### Main options

| オプション | 説明 |
| --- | --- |
| `-l ファイル名` (`--log=ファイル名`) | 指定したファイルに結果ログを保存 |
| `-r` (`--recursive`) | ディレクトリを再帰的に処理 |
| `-i` (`--infected`) | ウイルスに感染しているファイルのみを表示 |
| `--remove` | 発見したウイルスファイルを削除 |
| `--bell` | 感染ファイル発見時にターミナルベルを鳴らす |
