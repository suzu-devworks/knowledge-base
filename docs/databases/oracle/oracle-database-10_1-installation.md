# Oracle Database 10.1 Installation

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation on Fedora Core 4](#installation-on-fedora-core-4)
  - [1. Documentation Accessibility](#1-documentation-accessibility)
  - [2. Review Information About this Guide](#2-review-information-about-this-guide)
  - [3. Log In to the System as root](#3-log-in-to-the-system-as-root)
  - [4. Check the Hardware Requirements](#4-check-the-hardware-requirements)
  - [5. Check the Software Requirements](#5-check-the-software-requirements)
  - [6. Create Required UNIX Groups and User](#6-create-required-unix-groups-and-user)
  - [7. Create Required Directories](#7-create-required-directories)
  - [8. Configure Kernel Parameters](#8-configure-kernel-parameters)
  - [9. Mount the Product Disc](#9-mount-the-product-disc)
  - [10. Log In as the oracle User and Configure the oracle User's Environment](#10-log-in-as-the-oracle-user-and-configure-the-oracle-users-environment)
  - [11. Install Oracle Database 10g](#11-install-oracle-database-10g)
  - [12. Install Products from the Oracle Database 10g Companion CD](#12-install-products-from-the-oracle-database-10g-companion-cd)
  - [13. What to Do Next](#13-what-to-do-next)
  - [14. Additional Information](#14-additional-information)
- [Create Database](#create-database)
  - [Start Oracle Database Configuration Assistant](#start-oracle-database-configuration-assistant)
  - [ORA-07202: sltln: invalid parameter to sltln](#ora-07202-sltln-invalid-parameter-to-sltln)
- [Configurations](#configurations)
  - [Oracle JRE の文字化け対策](#oracle-jre-の文字化け対策)
  - [自動起動スクリプトの作成](#自動起動スクリプトの作成)

## Overview

Fedora Core 4 上での Oracle Database 10.1 のインストール手順および各種設定ガイド。
見出しの名称は「Oracle Database クイック・インストレーション・ガイド(10.1)」に準拠。

## Environments

- 2006-04-14: Oracle Database 10.1.0.2 on Fedora Core 4

## Installation on Fedora Core 4

### 1. Documentation Accessibility

省略。

### 2. Review Information About this Guide

省略。

### 3. Log In to the System as root

省略。

### 4. Check the Hardware Requirements

ハードウェア要件。標準構成は以下の通り。

| 要件 | 最小値 |
| :--- | :--- |
| 物理メモリ（RAM） | 512MB（524288KB） |
| スワップ領域 | 1GB（1048576KB）または RAM のサイズの 2 倍<br>RAM が 2GB 以上あるシステムでは、スワップ領域は RAM のサイズの 1 ～ 2 倍 |
| /tmp ディスク領域 | 400MB（409600KB） |
| ソフトウェア・ファイルのディスク領域 | 2.5GB（2621440KB） |
| データベース・ファイルのディスク領域 | 1.2GB（1258290KB） |

```bash
grep MemTotal /proc/meminfo
grep SwapTotal /proc/meminfo
df -h /tmp
df -h /u01
df -h /u02
```

### 5. Check the Software Requirements

ソフトウェア要件の確認。
OUIが /etc/redhat-releaseファイルの内容を確認してインストーラを中止させてしまうので、ファイルの内容を偽装してインストールする必要がある。

```bash
cp /etc/fedora-release /etc/fedora-release.org
echo redhat-3 > /etc/fedora-release
```

標準のリポジトリから、不足しているパッケージを追加する。

<!-- spell-checker: disable -->
```bash
# Oracle指定 （2006/4/14時点）
yum install gcc-4.0.2-8
yum install gcc-c++-4.0.2-8
yum install cpp-4.0.2-8
yum install glibc-2.3.6-95.20
yum install glibc-devel-2.3.6-10.i386
yum install glibc-headers-2.3.6-10.i386
yum install glibc-kernheaders-2.4-9.1.94
yum install make-3.80-7
yum install openmotif-2.2.3-10
yum install openmotif21-2.1.30-14
yum install setarch-1*
yum install compat-db-4*
yum install compat-gcc-32*
yum install compat-gcc-32-c++-33*
yum install compat-libstdc++-33*
yum install compat-libstdc++-33-3.2.3-47

# そのほか 
yum install xorg-x11-deprecated-libs-6.8.2-31
yum install libaio-0.3.104-2
yum install selinux-policy-targeted-1.25.2-4
```
<!-- spell-checker: enable -->

### 6. Create Required UNIX Groups and User

必須のUNIXグループおよびユーザーの作成。

- Oracleインベントリ・グループ（通常は`oinstall`）
- OSDBAグループ（通常は`dba`）
- OSOPERグループ（オプション。通常は`oper`）
- Oracleソフトウェア所有者（通常は`oracle`）

```bash
groupadd oinstall
groupadd dba
groupadd oper
useradd -g oinstall -G dba,oper oracle
passwd oracle
```

### 7. Create Required Directories

必要なディレクトリの作成。

- /u01/app/oracle（Oracle ベース・ディレクトリ）
- /u02/oradata（オプションの Oracle データファイル・ディレクトリ）

```bash
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01
chmod -R 775 /u01
mkdir -p /u02/oradata
chown -R oracle:oinstall /u02/oradata
chmod -R 775 /u02/oradata
```

### 8. Configure Kernel Parameters

カーネル・パラメータの構成。

| パラメータ | 最小値 | ファイル |
| :--- | :--- | :--- |
| `semmsl`<br>`semmns`<br>`semopm`<br>`semmni` | 250<br>32000<br>100<br>128 | `/proc/sys/kernel/sem` |
| `shmall` | 2097152 | `/proc/sys/kernel/shmall` |
| shmmax | 物理メモリーのサイズの半分（バイト） | `/proc/sys/kernel/shmmax` |
| `shmmni` | 4096 | `/proc/sys/kernel/shmmni` |
| `file-max` | 65536 | `/proc/sys/fs/file-max` |
| `ip_local_port_range` | 1024 65000 | `/proc/sys/net/ipv4/ip_local_port_range` |

```bash
echo "semmsl semmns semopm semmni:" 
/sbin/sysctl -a | grep sem

echo "shmall:"
/sbin/sysctl -a | grep shmall

echo "shmmax:"
/sbin/sysctl -a | grep shmmax

echo "shmmni:"
/sbin/sysctl -a | grep shmmni

echo "file-max:"
/sbin/sysctl -a | grep file-max

echo "ip_local_port_range:"
/sbin/sysctl -a | grep ip_local_port_range
```

要件より小さい値があるばあいにのみ記述を追加する。

`/etc/sysctl.conf`:

```ini
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 2097152
kernel.shmmax = 2147483648
fs.file-max = 65536
net.ipv4.ip_local_port_range = 1024 65000
```

下記コマンドでsysctl.confを読み込む。

```bash
/sbin/sysctl -p
```

Linux システム上のソフトウェアのパフォーマンスを向上するには、次に示す、oracle ユーザーのシェルの制限を増加する必要がある。

| シェルの制限 | limits.confの項目 | 強い制限 |
| :--- | :--- | :--- |
| オープンなファイル記述子の最大 | nofile | 65536 |
| シングル・ユーザーに対して使用可能なプロセスの最大数 | nproc | 16384 |

次の行を /etc/security/limits.conf ファイルに追加する。

`/etc/security/limits.conf`:

```text
oracle          soft    nproc   2047
oracle          hard    nproc   16384
oracle          soft    nofile  1024
oracle          hard    nofile  65536
```

次の行が /etc/pam.d/login ファイルに存在しない場合は追加する。

`/etc/pam.d/login`:

```text
・・・
session required /lib/security/pam_limits.so
```

デフォルトのシェル起動ファイルに次の変更を加える。

`/etc/profile.d/oracle.sh`:

```sh
if [ "$USER" = "oracle" ];then
  if [ "$SHELL" = "/bin/ksh" ];then
    ulimit -p 16384
    ulimit -n 65536
  else
    ulimit -u 16384 -n 65536
  fi
fi
```

### 9. Mount the Product Disc

製品ディスクのマウント。
マウントしたディスクに「NOEXEC」属性がついていないことだけ確認する。

### 10. Log In as the oracle User and Configure the oracle User's Environment

oracle ユーザーとしてのログインと oracle ユーザーの環境の構成。
インストーラは、oracle アカウントから実行する。ただし、インストーラを起動する前に、oracle ユーザーの環境を構成する必要がある。

- デフォルトのファイル・モード作成マスク（umask）を 022 に設定する
- DISPLAY、ORACLE_BASE および ORACLE_SID 環境変数を設定する

`/etc/oraclerc`:

```sh
# /etc/oraclerc
# ORACLE DATABASE 10.1.0.2 Setting
# Create xxxx/xx/xx
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/10.1.0/db_1
export ORACLE_TERM=xterm
export ORACLE_SID=ora10g
export NLS_LANG=japanese_Japan.AL32UTF8
# export LD_ASSUME_KERNEL=2.4.1
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/jre:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
PATH=/usr/sbin:$PATH
PATH=$ORACLE_HOME/bin:$PATH
export PATH
```

`$HOME/.bash_profile`:

```sh
・・・
umask 022
# export DISPLAY=xxxxx:0.0
export TEMP=/tmp
export TMPDIR=$TEMP
# Oracle 10g Setting
if [ -f /etc/oraclerc ]; then
  . /etc/oraclerc
fi
```

※DISPLAY環境変数は、ローカルでインストールするため不要。
oracleユーザで再ログイン、もしくは別端末で環境設定が正しく行えていることを確認する。

```console
# 環境設定の確認
$ env | more

# ファイル作成マスクを確認する
$ umask
0022
```

### 11. Install Oracle Database 10g

OUIを起動したときにロケールが日本語だと、文字化け（豆腐化）し正しく表示されない。これはOUIが起動するときのJREでのフォント定義が実環境と合っていないためである。ただし、インストーラは自動設定するJREを参照するため、インストール時には修正できないので我慢して英語でインストールする。

```console
$ echo $LANG
ja_JP.UTF-8

# このままインストーラを起動しても豆腐（□□□□□□）である
$ export LANG=en_US.UTF-8
```

```console
cd /tmp
/media/cdrecorder/db/runInstall.sh
```

| 画面 | アクション |
| :--- | :--- |
| Welcome | [ Next ] |
| Specify Inventory Directory and Credentials | Inventory directory: /u01/app/oracle/oraInventory （確認）<br>Specify Operating System group Name: oinstall （確認）<br>[ Next ] |
| Run orainstRoot.sh | ※指定通りrootユーザでコマンドを実行<br>#/u01/app/oracle/oraInventory/orainstRoot.sh<br>[ Continue ] |
| Specify File Locations | Source Path: /media/cdrecorder/db/stage/products.xml<br>Destination Name: oraDB_home1<br>Destination Path: /u01/app/oracle/product/10.1.0/db_1<br>[ Next ] |
| Select Installation Type | Installation Type: Enterprise Edition<br>***重要***<br>Product Languages: Japaneseを必ず追加すること<br>[ Next ] |
| Product-Specific Prerequisite Checks | 「checking recommend operating system packages」でwarningにになるが、許可するチェックを付け、次にすすめる。<br>[ Next ] |
| Select Database Configuration | ***重要***<br>「Do Not Create a Starter database」を選択<br>・・・初期データベースは作成しない（どうせエラーになるので）<br>[ Next ] |
| Summary | [ Install ] |
| Install | 時間がかかる・・・ |
| Configuration Assistants | [ OK ] |
| Setup Privileges | ※指定通りrootユーザでコマンドを実行<br># /u01/app/oracle/product/10.1.0/db_1/root.sh<br>[ OK ] |
| End of Installation | [ Exit ] |

### 12. Install Products from the Oracle Database 10g Companion CD

省略。

### 13. What to Do Next

インストール後の作業。

1. OSの偽装をもとに戻す

```console
# mv /etc/fedora-release /etc/fedora-release.redhat
# cp /etc/fedora-release.org /etc/fedora-release
```

OUIでのインストール後、inittabに自動的に下記が記載されるが、RAC以外の場合には不要であるので削除する。

`/etc/inittab`:

```text
・・・
#h1:35:respawn:/etc/init.d/init.cssd run >/dev/null 2>&1 </dev/null
```

また、下記はOUIが作ったファイルなので削除可能。

```console
# rm -f /etc/init.cssd
# rm -f /etc/init.no_cssd
# rm -f /etc/init.orig
```

そのままだと、せっかくJREでフォントを設定しても英語のままであるのと、dbcaを起動したときにキャラクタセットで英語しか選択できなくなるので、LANGを戻しておく。

```console
export LANG=ja_JP.UTF-8
```

### 14. Additional Information

省略。

## Create Database

### Start Oracle Database Configuration Assistant

いろいろ問題有・・・とりあえず端末から起動する。

```console
$ dbca
Warning: Cannot convert string "-kochi-mincho-medium-r-normal--*-140-*-*-p-*-iso10646-1" to type FontStruct ← 気にしない
```

| 画面 | アクション |
| :--- | :--- |
| ようこそ | 「次へ」 |
| ステップ 1/12： 操作 | 実行する操作： 「データベースの作成」を選択<br>「次へ」 |
| ステップ 2/12： データベース・テンプレート | 使用するテンプレート： 「カスタム・データベース」を選択したいが、ORA-03113が発生する。 がまんして「汎用」を選択<br>「次へ」 |
| ステップ 3/12： データベース識別情報 | グローバル・データベース名： ora10g<br>SID： ora10g<br>※RACではないので同じにする<br>「次へ」 |
| ステップ 4/12： 管理オプション | 「Enterprise Managerを使用してデータベースを構成」にチェックがあること<br>「データベース管理にDatabase Control を使用します」が選択されていること<br>「次へ」 |
| ステップ 5/12： データベース資格証明 | 本当は別々のパスワードが望ましいんだけど、面倒くさいので１つにする。<br>入力して<br>「次へ」 |
| ステップ 6/12： 記憶域オプション | 記憶域： 「ファイル・システム 」を選択<br>「次へ」 |
| ステップ 7/12： データベース・ファイルの位置 | ファイルの位置: 「Oracle Management Filesの使用」を選択<br>データベース領域： /u02/oradata/<br>※カスタムデータベースの場合、OMFを選択するとそのあとのデータベースの作成でORA-07202が発生する<br>「次へ」 |
| ステップ 8/12： リカバリ構成 | 「フラッシュ・リカバリ領域の指定」にチェックがあること<br>フラッシュリカバリ領域： デフォルト<br>フラッシュリカバリ領域サイズ： デフォルト<br>「次へ」 |
| ステップ 9/12： データベース・コンテンツ | 「サンプル・スキーマ」にチェックつける<br>「次へ」 |
| ステップ 10/12： 初期化パラメータ | 【メモリー】<br> メモリー管理： 標準を選択<br> 割合: 30%<br>【サイズ指定】<br> デフォルトのまま<br>【キャラクタセット】<br> データベース・キャラクタ・セット： JA16SJISを選択<br> ※JA16SJISTILDEを選択しておくと〜'が文字化けしないが今回はわざと。<br> 各国語キャラクタ・セット： AL16UTF-16 (デフォルトのまま)<br> デフォルト言語： 日本語 (デフォルトのまま)<br> デフォルト地域： 日本 (デフォルトのまま)<br>【接続モード】<br> 接続モード： 共有サーバー・モード を選択<br> 共有サーバー： 1<br>「次へ」 |
| ステップ 11/12： データベース記憶域 | デフォルトのまま<br>※あとでどうとでもなるので無視<br>「次へ」 |
| ステップ 12/12： 作成オプション | 「データベースの作成」にチェックがあること<br>「完了」 |
| Database Configuration Assistantウィンドウ 作成中画面 | データベースの作成開始中・・・ |
| Database Configuration Assistantウィンドウ 完了画面 | ※この画面とっても大事なのでよく読むように・・・<br>「終了」 |

### ORA-07202: sltln: invalid parameter to sltln

DBCAから以下の条件でデータベースを作成すると発生する。

- カスタムデータベースで作成
- ファイル配置をOMFで指定する
- データベースの作成、およびスクリプトの実行でも発生する

自動生成したinit.oraの以下の行に問題があるよう。

`init.ora`:

```ini
・・・
control_file=""
・・・
```

CREATE DATABASE文は、init.ora に db_create_file_dist を記述した場合にOMFとして処理する。
しかし、上記の記述が同時にある場合には異常終了してしまう。
もともとこの行はテンプレートに存在し、OMF以外の場合には新たなcontrol_fileを自動出力しているため削除して問題無いようだ。

## Configurations

### Oracle JRE の文字化け対策

インストール直後の状態では、OUIと同様に dbca, netca などのJavaアプリケーションが文字化けする。これは参照しているJREのfont.property.ja が実環境に合っていないためであり、修正する必要がある。
実際には、「$ORACLE_HOME/jdk/jre/lib/font.property.ja」 を編集するが、同じディレクトリにサンプルファイルがあるので、それをコピーして修正するのが楽である。

```console
# mv font.property.ja font.property.ja.org
# cp font.properties.ja_JP_UTF8.Sun font.property.ja
```

コピーしたファイルの最後のほうを、実環境に合わせて変更する。

`$ORACLE_HOME/jdk/jre/lib/font.property.ja`:

```ini
・・・
# Del
#filename.-kochi-mincho-medium-r-normal--*-%d-*-*-c-*-jisx0201.1976-0=/usr/X11R6/lib/X11/fonts/truetype/kochi-mincho.ttf
#filename.-kochi-mincho-medium-r-normal--*-%d-*-*-c-*-jisx0208.1983-0=/usr/X11R6/lib/X11/fonts/truetype/kochi-mincho.ttf
#filename.-kochi-gothic-medium-r-normal--*-%d-*-*-c-*-jisx0201.1976-0=/usr/X11R6/lib/X11/fonts/truetype/kochi-gothic.ttf
#filename.-kochi-gothic-medium-r-normal--*-%d-*-*-c-*-jisx0208.1983-0=/usr/X11R6/lib/X11/fonts/truetype/kochi-gothic.ttf

# Add
filename.-kochi-mincho-medium-r-normal--*-%d-*-*-c-*-jisx0201.1976-0=/usr/share/fonts/japanese/TrueType/kochi-mincho-subst.ttf
filename.-kochi-mincho-medium-r-normal--*-%d-*-*-c-*-jisx0208.1983-0=/usr/share/fonts/japanese/TrueType/kochi-gothic-subst.ttf
filename.-kochi-gothic-medium-r-normal--*-%d-*-*-c-*-jisx0201.1976-0=/usr/share/fonts/japanese/TrueType/kochi-gothic-subst.ttf
filename.-kochi-gothic-medium-r-normal--*-%d-*-*-c-*-jisx0208.1983-0=/usr/share/fonts/japanese/TrueType/kochi-gothic-subst.ttf
```

netcaなどjavaアプリケーションを起動して確認する。

### 自動起動スクリプトの作成

サーバーの起動でデータベースを自動起動するようにスクリプトを作成する。
oratabファイルで、初期起動するORACLE_SIDを設定する。

`/etc/oratab`:

```text
・・・
# Entries are of the form:
#   $ORACLE_SID:$ORACLE_HOME:gt;:
・・・
ora10g:/u01/app/oracle/product/10.1.0/db_1:Y
```

initスクリプトを自作する。

`/etc/init.d/oracle-database`:

<!-- spell-checker: disable -->
```sh
#!/bin/sh
#
# chkconfig: 345 99 10
# description: Oracle auto start-stop script.
#
# Change the value of ORACLE_HOME to specify the correct Oracle home
# directory for your installation.

ORACLE_HOME=/u01/app/oracle/product/10.1.0/db_1
#
# Change the value of ORACLE to the login name of the
# oracle owner at your site.
#
ORACLE=oracle

PATH=${PATH}:$ORACLE_HOME/bin

#set shell limits for ORACLE DATABASE
ulimit -c unlimited
ulimit -d unlimited
ulimit -s 65536

RETVAL=0

start() {
    if [ -f $ORACLE_HOME/bin/tnslsnr ]; then
        KIND="Oracle NET"
        echo -n "Starting $KIND services: "
        su $ORACLE -c "$ORACLE_HOME/bin/lsnrctl start"
        echo
    fi
    KIND="Oracle Database"
    echo -n "Starting $KIND services: "
    su $ORACLE -c "$ORACLE_HOME/bin/dbstart"
    RETVAL=$?
    echo
    return $RETVAL
}

stop() {
    KIND="Oracle Database"
    echo -n "Shutting down $KIND services: "
    su $ORACLE -c "$ORACLE_HOME/bin/dbshut"
    RETVAL=$?
    echo
    if [ -f $ORACLE_HOME/bin/tnslsnr ]; then
        KIND="Oracle NET"
        echo -n "Shutting down $KIND services: "
        su $ORACLE -c "$ORACLE_HOME/bin/lsnrctl stop"
        echo
    fi
    return $RETVAL
}

restart() {
    stop
    start
}

case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    restart)
        restart
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
esac

exit $?
```
<!-- spell-checker: enable -->

スクリプトを作成したら、chkconfigで登録する。

```bash
chkconfig --add /etc/init.d/oracle-database
```
