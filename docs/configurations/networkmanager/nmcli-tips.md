# NetworkManager nmcli Tips

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Show Objects](#show-objects)
  - [General](#general)
  - [Connection](#connection)
  - [Device](#device)
- [Operations](#operations)
  - [Networking off / on](#networking-off--on)
  - [Radio wifi off / on](#radio-wifi-off--on)
  - [Connection down / up](#connection-down--up)
  - [Error: Not authorized to control networking](#error-not-authorized-to-control-networking)
- [Configuration](#configuration)
  - [Hostname](#hostname)
  - [Connect automatically](#connect-automatically)
  - [Static IPv4 address](#static-ipv4-address)
  - [Dynamic IPv4 address](#dynamic-ipv4-address)
  - [Static IPv6 address](#static-ipv6-address)
  - [Dynamic IPv6 address](#dynamic-ipv6-address)
  - [Command Completion](#command-completion)
- [References](#references)

## Overview

NetworkManager (ネットワークマネージャー) は、libudevやその他のLinuxカーネルインタフェース上で動作し、高水準のネットワーク設定用のインタフェースの提供も行うデーモンである。
nmcli は NetworkManager のコマンドラインツール。

## Environments

- 2019-07-07: NetworkManager 1.12.0-10.el7_6 on CentOS 7

## Show Objects

### General

```console
$ nmcli general status

STATE      CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN        
connected  full          enabled  enabled  enabled  enabled     
```

### Connection

NetworkManagerはすべてのネットワーク設定を「接続」として保存します。
複数の設定を保存しておき、デバイスに対して1つだけ「アクティブ」にすることで、さまざまなネットワークと構成をすばやく切り替えることができます。

```console
$ nmcli connection show

NAME  UUID                                  TYPE      DEVICE     
eth0  b0194673-8a93-46e2-a455-d87ba58080b6  ethernet  eth0       
eth1  68186f56-1f05-448d-b9ab-11800f16eae5  ethernet  --         
eth2  a1456aaf-5da5-4593-94fa-34441d96e343  ethernet  --      
```

```console
$ nmcli c show eth0

connection.id:                          eth0
connection.uuid:                        b0194673-8a93-46e2-a455-d87ba58080b6
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              eth0
connection.autoconnect:                 no
connection.autoconnect-priority:        0
connection.autoconnect-retries:         -1 (default)
connection.auth-retries:                -1
connection.timestamp:                   1562579566
connection.read-only:                   no
connection.permissions:                 --
connection.zone:                        --
connection.master:                      --
connection.slave-type:                  --
connection.autoconnect-slaves:          -1 (default)
connection.secondaries:                 --
connection.gateway-ping-timeout:        0
connection.metered:                     unknown
connection.lldp:                        default
connection.mdns:                        -1 (default)
802-3-ethernet.port:                    --
802-3-ethernet.speed:                   0
...
```

### Device

物理ネットワークインターフェイスを管理します。

```console
$ nmcli device status

DEVICE  TYPE      STATE         CONNECTION     
eth0    ethernet  connected     eth0           
eth1    ethernet  disconnected  --             
eth2    ethernet  disconnected  --             
lo      loopback  unmanaged     --        
```

```console
$ nmcli d show eth0

GENERAL.DEVICE:                         eth0
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         52:54:00:85:B1:A5
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     eth0
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveCo
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         192.168.100.228/24
IP4.GATEWAY:                            192.168.100.1
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 192.168.100.1, mt 
IP4.ROUTE[2]:                           dst = 192.168.100.0/24, nh = 0.0.0.0, mt
IP4.DNS[1]:                             192.168.100.1
IP4.DOMAIN[1]:                          net1.local
IP6.ADDRESS[1]:                         fe80::42d:ba0e:6392:d19d/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 100
IP6.ROUTE[2]:                           dst = ff00::/8, nh = ::, mt = 256, table
```

## Operations

### Networking off / on

機内モードのようなものです。

**Off (disable)**:

```console
$ sudo nmcli networking off
$ nmcli g

STATE   CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN        
asleep  none          enabled  enabled  enabled  enabled     

$ nmcli d

DEVICE  TYPE      STATE      CONNECTION     
eth0    ethernet  unmanaged  --             
eth1    ethernet  unmanaged  --             
eth2    ethernet  unmanaged  --             
lo      loopback  unmanaged  --             
```

**On (enable)**:

```console
$ sudo nmcli networking on

[  132.878974] IPv6: ADDRCONF(NETDEV_UP): eth0: link is not ready
[  132.888245] IPv6: ADDRCONF(NETDEV_UP): eth1: link is not ready
[  132.895191] IPv6: ADDRCONF(NETDEV_UP): eth2: link is not ready

$ nmcli g

STATE     CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN        
connected full          enabled  enabled  enabled  enabled     

$ nmcli d

DEVICE  TYPE      STATE         CONNECTION     
eth0    ethernet  connected     eth0           
eth1    ethernet  disconnected  --             
eth2    ethernet  disconnected  --             
lo      loopback  unmanaged     --     
```

### Radio wifi off / on

「WI-FI」など無線接続の off/on スイッチです。

**Off (disable)**:

```console
$ sudo nmcli radio wifi off
$ nmcli radio

WIFI-HW  WIFI      WWAN-HW  WWAN        
enabled  disabled  enabled  enabled     
```

**On (enable)**:

```console
$ sudo nmcli radio wifi on
$ nmcli r

WIFI-HW  WIFI     WWAN-HW  WWAN        
enabled  enabled  enabled  enabled  
```

### Connection down / up

ネットワーク接続のリンクアップです。

```console
$ sudo sh -c "nmcli c down eth0 ; nmcli c up eth0"

Connection 'eth0' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

$ nmcli c

NAME  UUID                                  TYPE      DEVICE     
eth0  b0194673-8a93-46e2-a455-d87ba58080b6  ethernet  eth0       
eth1  68186f56-1f05-448d-b9ab-11800f16eae5  ethernet  --         
eth2  a1456aaf-5da5-4593-94fa-34441d96e343  ethernet  --   
```

### Error: Not authorized to control networking

Connection up したらエラーになった。sudo ではできるので権限っぽい。

```console
$ nmcli c up eth0

Error: Connection activation failed: Not authorized to control networking.

$ nmcli general permissions

PERMISSION                                                        VALUE     
org.freedesktop.NetworkManager.enable-disable-network             no        
org.freedesktop.NetworkManager.enable-disable-wifi                no        
org.freedesktop.NetworkManager.enable-disable-wwan                no        
org.freedesktop.NetworkManager.enable-disable-wimax               no        
org.freedesktop.NetworkManager.sleep-wake                         no        
org.freedesktop.NetworkManager.network-control                    auth      
org.freedesktop.NetworkManager.wifi.share.protected               no        
org.freedesktop.NetworkManager.wifi.share.open                    no        
org.freedesktop.NetworkManager.settings.modify.system             auth      
org.freedesktop.NetworkManager.settings.modify.own                auth      
org.freedesktop.NetworkManager.settings.modify.hostname           auth      
org.freedesktop.NetworkManager.settings.modify.global-dns         auth      
org.freedesktop.NetworkManager.reload                             auth      
org.freedesktop.NetworkManager.checkpoint-rollback                auth      
org.freedesktop.NetworkManager.enable-disable-statistics          no        
org.freedesktop.NetworkManager.enable-disable-connectivity-check  no     
```

ちなみにGNOMEからだとユーザー権限で設定できるなぜだ？？

```console
$ nmcli g p

PERMISSION                                                        VALUE       
org.freedesktop.NetworkManager.enable-disable-network             yes         
org.freedesktop.NetworkManager.enable-disable-wifi                yes         
org.freedesktop.NetworkManager.enable-disable-wwan                yes         
org.freedesktop.NetworkManager.enable-disable-wimax               yes         
org.freedesktop.NetworkManager.sleep-wake                         no          
org.freedesktop.NetworkManager.network-control                    yes         
org.freedesktop.NetworkManager.wifi.share.protected               yes         
org.freedesktop.NetworkManager.wifi.share.open                    yes         
org.freedesktop.NetworkManager.settings.modify.system             yes         
org.freedesktop.NetworkManager.settings.modify.own                yes         
org.freedesktop.NetworkManager.settings.modify.hostname           auth        
org.freedesktop.NetworkManager.settings.modify.global-dns         auth        
org.freedesktop.NetworkManager.reload                             auth        
org.freedesktop.NetworkManager.checkpoint-rollback                auth        
org.freedesktop.NetworkManager.enable-disable-statistics          yes         
org.freedesktop.NetworkManager.enable-disable-connectivity-check  yes         
org.freedesktop.NetworkManager.wifi.scan                          unknown  
```

## Configuration

### Hostname

```bash
sudo nmcli general hostname "centos7.local"
```

### Connect automatically

```bash
sudo nmcli connection modify eth0 connection.autoconnect yes
```

### Static IPv4 address

ipv4 static:

```bash
$ sudo nmcli connection modify eth2 \
    ipv4.addresses "192.168.9.100/24" \
    ipv4.gateway "192.168.9.1" \
    ipv4.method manual
```

ipv4 DNS static:

```bash
sudo nmcli connection modify eth0 ipv4.dns "8.8.8.8"
sudo nmcli connection modify eth0 +ipv4.dns "8.8.4.4"
```

link up:

```bash
sudo nmcli connection down eth0
sudo nmcli connection up eth0
```

Verify the connection:

```console
$ nmcli
eth0: connected to eth0
    "Red Hat Virtio"
    ethernet (virtio_net), 52:54:00:85:B1:A5, hw, mtu 1500
    inet4 192.168.9.100/24
    route4 192.168.9.0/24
    route4 0.0.0.0/0
    inet6 fe80::42d:ba0e:6392:d19d/64
    route6 fe80::/64
    route6 ff00::/8
```

### Dynamic IPv4 address

```bash
sudo nmcli connection modify eth0 ipv4.method auto
sudo nmcli connection modify eth0 ipv4.address ""
sudo nmcli connection modify eth0 ipv4.dns ""
```

### Static IPv6 address

ipv6 static:

```bash
$ sudo nmcli connection modify eth0 \
    ipv6.addresses "2001:db8::100/64" \
    ipv6.gateway "2001:db8:0::1" \
    ipv6.method manual
```

ipv6 DNS static:

```bash
sudo nmcli connection modify eth0 ipv6.dns "2001:4860:4860::8888"
sudo nmcli connection modify eth0 +ipv6.dns "2001:4860:4860::8844"
```

link up:

```bash
sudo nmcli connection down eth0
sudo nmcli connection up eth0
```

Verify the connection:

```console
eth0: connected to eth0
    "Red Hat Virtio"
    ethernet (virtio_net), 52:54:00:85:B1:A5, hw, mtu 1500
    ip6 default
    inet6 2001:db8::100/64
    inet6 fe80::84a5:191a:e9d8:9c13/64
    route6 fe80::/64
    route6 2001:db8::/64
    route6 ff00::/8
    route6 ::/0
```

### Dynamic IPv6 address

```bash
sudo nmcli connection modify eth0 ipv6.method auto
sudo nmcli connection modify eth0 ipv6.address ""
sudo nmcli connection modify eth0 ipv6.dns ""
```

### Command Completion

コマンド補完は bash-completion に入っています。

```bash
sudo yum install bash-completion
. /etc/profile.d/bash_completion.sh
```

## References

- [NetworkManager: NetworkManager Reference Manual - GNOME](https://developer.gnome.org/NetworkManager/stable/)
- [nm-settings: NetworkManager Reference Manual - GNOME](https://developer.gnome.org/NetworkManager/stable/nm-settings.html)
- <https://ja.wikipedia.org/wiki/NetworkManager>
