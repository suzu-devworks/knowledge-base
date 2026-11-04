# Docker Tips of Using NetworkManager

## Table of Contents <!-- omit in toc -->

## Overview

Docker に nmcli がいません。というか NetworkManager がインストールされていません。
本ドキュメントでは、CentOS コンテナ上で NetworkManager を動かす手順について解説します。

## Environments

* 2020-10-19: NetworkManager-1.22.8-5.el8_2.x86_64.rpm, Docker-ce 19.03.13 on Fedora 32
* 2019-07-07: NetworkManager.x86_64 1:1.12.0-10.el7_6, Docker Desktop Community Version 2.0.0.3 on macOS 10.14

## NetworkManager on CentOS Container

### Start Container

NetworkManager には Docker と相性の悪い systemd が必要になるため、コンテナ起動時に `--privileged` オプションを指定する必要があります。

```bash
docker run -d --privileged --name systemd-test-8 centos:8 /usr/sbin/init
```

### Install NetworkManager

コンテナ内部に入り、dnf パッケージマネージャーを使用して NetworkManager をインストールします。

```bash
docker exec -it systemd-test-8 bash
```

```console
# in container.
[root@eaf7bce03212 /]# dnf makecache

Failed to set locale, defaulting to C.UTF-8
CentOS-8 - AppStream                                                 6.4 kB/s | 4.3 kB     00:00        
CentOS-8 - Base                                                      8.3 kB/s | 3.9 kB     00:00        
CentOS-8 - Extras                                                    3.0 kB/s | 1.5 kB     00:00        
Metadata cache created.

[root@eaf7bce03212 /]# dnf install NetworkManager

Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:00:21 ago on Mon Oct 19 13:49:32 2020.
Dependencies resolved.
===================================================================================================
 Package                  Architecture      Version                Repository              Size
===================================================================================================
Installing:
 NetworkManager           x86_64            1:1.22.8-5.el8_2       BaseOS                 2.3 M
Installing dependencies:
 NetworkManager-libnm     x86_64            1:1.22.8-5.el8_2       BaseOS                 1.7 M
 libndp                   x86_64            1.7-3.el8              BaseOS                  40 k
 polkit-libs              x86_64            0.115-11.el8           BaseOS                  76 k

Transaction Summary
===================================================================================================
Install  4 Packages

Total download size: 4.1 M
Installed size: 15 M
Is this ok [y/N]: y
...
Installed:
  NetworkManager-1:1.22.8-5.el8_2.x86_64        NetworkManager-libnm-1:1.22.8-5.el8_2.x86_64
  libndp-1.7-3.el8.x86_64             polkit-libs-0.115-11.el8.x86_64                  

Complete!
```

### Start NetworkManager Service

systemctl コマンドでサービスの状態を確認し、起動します。

<!-- spell-checker: disable -->
```console
# in container.
[root@eaf7bce03212 /]# systemctl list-unit-files *[Nn]et*

UNIT FILE                          STATE      
NetworkManager-dispatcher.service  enabled    
NetworkManager-wait-online.service enabled    
NetworkManager.service             enabled    
network-online.target              static     
network-pre.target                 static     
network.target                     static     

6 unit files listed.

[root@eaf7bce03212 /]# systemctl start NetworkManager

[root@eaf7bce03212 /]# systemctl status NetworkManager

● NetworkManager.service - Network Manager
   Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2020-10-19 13:50:45 UTC; 5s ago
     Docs: man:NetworkManager(8)
 Main PID: 201 (NetworkManager)
   CGroup: /docker/eaf7bce032126a60e011b4f738b36736303e76b64d1fb8f855f7b19f3197d634/system.slice/NetworkManager.service
           └─201 /usr/sbin/NetworkManager --no-daemon

Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6899] manager: rfkill: WWAN enabled by radio killswitch; enab>
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6901] manager: Networking is enabled by state file
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6902] dhcp-init: Using DHCP client 'internal'
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6908] settings: Loaded settings plugin: ifcfg-rh ("/usr/lib64>
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6909] settings: Loaded settings plugin: keyfile (internal)
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6933] device (lo): carrier: link connected
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6935] manager: (lo): new Generic device (/org/freedesktop/Net>
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6942] device (eth0): carrier: link connected
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6945] manager: (eth0): new Veth device (/org/freedesktop/Netw>
Oct 19 13:50:45 eaf7bce03212 NetworkManager[201]: <info>  [1603115445.6959] manager: startup complete
```
<!-- spell-checker: enable -->

### Check NetworkManager

`nmcli` コマンドを実行し、ネットワークデバイスの認識状態を確認します。

```console
# in container.
[root@eaf7bce03212 /]# nmcli

eth0: unmanaged
        "eth0"
        ethernet (veth), 02:42:AC:11:00:02, sw, mtu 1500

lo: unmanaged
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

Use "nmcli device show" to get complete information about known devices and
"nmcli connection show" to get an overview on active connection profiles.

Consult nmcli(1) and nmcli-examples(7) manual pages for complete usage details.
```
