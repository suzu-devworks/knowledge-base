# Docker Tips of Using Systemd

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Systemd on CentOS Container](#systemd-on-centos-container)
  - [Error: Failed to connect to bus: Host is down](#error-failed-to-connect-to-bus-host-is-down)
  - [Error: Failed to get D-Bus connection: Operation not permitted](#error-failed-to-get-d-bus-connection-operation-not-permitted)
  - [Use privileged option](#use-privileged-option)
  - [Use CAP\_SYS\_ADMIN](#use-cap_sys_admin)
- [CentOS Formal Method](#centos-formal-method)
  - [Docker build](#docker-build)
- [Troubleshooting](#troubleshooting)
  - [Mount failed for selinuxfs on /sys/fs/selinux: Operation not permitted (on Fedora 32)](#mount-failed-for-selinuxfs-on-sysfsselinux-operation-not-permitted-on-fedora-32)
- [Reference](#reference)

## Overview

Docker ではあまりそういう使い方をしないのかもしれないが、Docker で Systemd から起動したい場合がある。
基本的には Docker の起動プログラムで init (systemd) を呼ぶだけのはずなのだが、いろいろ問題があるようだ。
CentOS 公式 image に起動方法の説明があるのだが...

## Environments

- 2020-10-11: CentOS:8 (8.2.2004) image on Docker 19.03.13 on Fedora 32
- 2019-07-07: CentOS:7 (7.6.1810) image on Docker Desktop Community Version 2.0.0.3 (31250) on macOS 10.14
- 2017-09-26: CentOS:7 (7.4.1708)

## Systemd on CentOS Container

### Error: Failed to connect to bus: Host is down

CentOS 8 コンテナは systemd が PID 1 じゃない的なエラーが表示される。

```bash
docker run -d --name systemd-test-8 centos:8 /usr/sbin/init
docker exec -it systemd-test-8 bash
```

```console
# In docker
[root@bc7fd3c5a406 /]# systemctl status systemd-journald

System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down

[root@bc7fd3c5a406 /]# ps -ef

UID PID PPID C STIME TTY TIME CMD
root 1 0 0 09:25 ? 00:00:00 /usr/sbin/init
root 44 0 0 09:52 pts/0 00:00:00 bash
root 57 44 0 09:52 pts/0 00:00:00 ps -ef

# ログも存在しない
[root@f5d22ac9e759 /]# journalctl -e --no-pager
No journal files were found.
-- No entries --
```

### Error: Failed to get D-Bus connection: Operation not permitted

CentOS 7 コンテナでは D-Bus 接続系のエラーになる。

```bash
docker run -d --name systemd-test-7 centos:7 /usr/sbin/init
docker exec -it systemd-test-7 bash
```

```console
# In docker
[root@e527f28c7101 /]# systemctl status systemd-journald

Failed to get D-Bus connection: Operation not permitted

[root@e527f28c7101 /]# ps -e

PID TTY TIME CMD
1 ? 00:00:00 systemd
7 pts/0 00:00:00 bash
31 pts/0 00:00:00 ps
```

### Use privileged option

特権モード (`--privileged`) で起動する。SELinux が有効な場合この方法しか無いようだ。

```bash
docker run -d --privileged --name systemd-test-8 centos:8 /usr/sbin/init
docker exec -it systemd-test-8 bash
```

```console
# In docker
[root@f8fb666d6253 /]# systemctl status systemd-journald

● systemd-journald.service - Journal Service
Loaded: loaded (/usr/lib/systemd/system/systemd-journald.service; static; vendor preset: disabled)
Active: active (running) since Sun 2020-10-11 08:59:08 UTC; 19min ago
Docs: man:systemd-journald.service(8)
man:journald.conf(5)
Main PID: 31 (systemd-journal)
Status: "Processing requests..."
CGroup: /docker/f8fb666d6253acc459e5757982198510458bad1075d51ca00a7c9e272a3ec7c4/system.slice/systemd-journald.service
└─31 /usr/lib/systemd/systemd-journald

Oct 11 08:59:08 f8fb666d6253 systemd-journald[31]: Journal started
Oct 11 08:59:08 f8fb666d6253 systemd-journald[31]: Runtime journal (/run/log/journal/b525b23b573a435b82f9d785573892e7) is 8.0M, max>
Oct 11 08:59:08 f8fb666d6253 systemd-journald[31]: Runtime journal (/run/log/journal/b525b23b573a435b82f9d785573892e7) is 8.0M, max>
Warning: Journal has been rotated since unit was started. Log output is incomplete or unavailable.

# systemd が PID 1 にいないけどね。
[root@f8fb666d6253 /]# ps -ef

UID PID PPID C STIME TTY TIME CMD
root 1 0 0 08:59 ? 00:00:00 /usr/sbin/init
root 31 1 0 08:59 ? 00:00:00 /usr/lib/systemd/systemd-journald
root 34 1 0 08:59 ? 00:00:00 /usr/lib/systemd/systemd-udevd
dbus 62 1 0 08:59 ? 00:00:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-
root 113 0 0 09:00 pts/0 00:00:00 bash
root 160 113 0 09:16 pts/0 00:00:00 ps -ef

# ログ表示された。
[root@f8fb666d6253 /]# journalctl -e --no-pager

...
Oct 11 08:59:11 f8fb666d6253 dnf[67]: CentOS-8 - AppStream 2.2 MB/s | 5.8 MB 00:02
Oct 11 08:59:13 f8fb666d6253 dnf[67]: CentOS-8 - Base 1.3 MB/s | 2.2 MB 00:01
Oct 11 08:59:15 f8fb666d6253 dnf[67]: CentOS-8 - Extras 9.0 kB/s | 8.1 kB 00:00
Oct 11 08:59:15 f8fb666d6253 dnf[67]: Metadata cache created.
Oct 11 08:59:15 f8fb666d6253 dnf[67]: Failed to set locale, defaulting to C.UTF-8
...
```

特権モード (`--privileged`) はホストに対して特権っぽいので怖い。
コンテナ内の時計を変更すると、ホストの時計が変更されたことがある。

### Use CAP_SYS_ADMIN

ホストの SELinux が無効で、Docker 1.2.0 以降であれば `CAP_SYS_ADMIN` ケーパビリティの指定で実現できるらしいが、ホストの SELinux を無効にするのがハードル高い。
やってみたけどダメだった。

```bash
docker run -d --cap-add=SYS_ADMIN -v /sys/fs/cgroup:/sys/fs/cgroup:ro --name systemd-test2-8 centos:8 /usr/sbin/init
```

## CentOS Formal Method

CentOS 公式の方法は Dockerfile を作ることである。
この方法もホストの SELinux が無効であることが前提のようだ。

### Docker build

```dockerfile
FROM centos:7
ENV container docker
RUN (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == \
systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*;
VOLUME [ "/sys/fs/cgroup" ]
CMD ["/usr/sbin/init"]
```

```console
$ docker build --rm --tag local/centos-systemd:7 .

...
Successfully tagged local/centos-systemd:7

$ docker image ls

REPOSITORY TAG IMAGE ID CREATED SIZE
local/centos-systemd 7 f52a9e2520be 8 seconds ago 197MB
centos 7 196e0ce0c9fb 11 days ago 197MB

$ docker run -d -p "22:22222" local/centos-systemd:7

403bfa5e711e7b1bc98b684ed40aca7a642f4a01a0401731cf2543e6e8c2bb2e

$ docker exec -it heuristic_hypatia /usr/bin/bash
```

```console
# in container
[root@403bfa5e711e /]# ps -e

PID TTY TIME CMD
1 ? 00:00:00 systemd
7 pts/0 00:00:00 bash
21 pts/0 00:00:00 ps
```

起動しない。
結局特権が必要なのだろうか...。

```bash
docker run --privileged -d -p "22222:22" local/centos-systemd:7 
docker exec -it git-server /usr/bin/bash
```

```console
# in container
[root@bd8692a13b96 /]# ps -e

PID TTY TIME CMD
1 ? 00:00:00 systemd
19 ? 00:00:00 systemd-journal
21 pts/0 00:00:00 bash
36 pts/0 00:00:00 ps
```

## Troubleshooting

### Mount failed for selinuxfs on /sys/fs/selinux: Operation not permitted (on Fedora 32)

SELinux / Systemd 関連と思われる。

```console
$ docker logs suspicious_mendeleev 

Mount failed for selinuxfs on /sys/fs/selinux: Operation not permitted

$ sudo sestatus

SELinux status: enabled
SELinuxfs mount: /sys/fs/selinux
SELinux root directory: /etc/selinux
Loaded policy name: targeted
Current mode: enforcing
Mode from config file: enforcing
Policy MLS status: enabled
Policy deny_unknown status: allowed
Memory protection checking: actual (secure)
Max kernel policy version: 33
```

上記を特権で起動したら別のエラーとなった。
`/etc/selinux/targeted/policy/policy.32` はあるが `33` は存在しない...。

```console
$ docker logs systemd-test-8 
SELinux: Could not open policy file <= /etc/selinux/targeted/policy/policy.33: No such file or directory
```

## Reference

- [centos - Docker Hub](https://hub.docker.com/_/centos)
