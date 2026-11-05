# Cassandra GUI client (Kindrat/cassandra-client)

## Table of Contents <!-- omit in toc -->

## Overview

Cassandra 3 用の Java 製 GUI クライアント。
相当古い。

- [Kindrat/cassandra-client GitHub](https://github.com/Kindrat/cassandra-client)

## Environments

- 2020-09-27:
  - Kindrat/cassandra-client (commit fc981884ec8253f0fc2ba688def110f08f7ba80d)
  - AdoptOpenJDK (build 11.0.7+10)
  - Fedora 32

## Getting Started

```console
$ git clone https://github.com/Kindrat/cassandra-client.git
$ cd cassandra-client

# Java バージョン確認 (Java 11)
$ java --version
openjdk 11.0.7 2020-04-14
OpenJDK Runtime Environment AdoptOpenJDK (build 11.0.7+10)

# ビルド & 起動
$ ./gradlew build
$ java -jar build/libs/cassandra-client-1.0.2-exec.jar
# または gradle から起動
$ ./gradlew bootRun
```

### [SOLVED] cgroup mount　point does not exist

Fedora 環境において cgroup v2 が有効なため `./gradlew build` 時にエラーが発生する場合がある。cgroup v1 のマウントポイントを偽装して対応する。

```console
# cgroup v1 のマウントポイントを作成
$ sudo mkdir -p /sys/fs/cgroup/systemd
$ sudo sh -c "cat /proc/mounts | grep \/sys\/fs\/cgroup\/systemd > /dev/null || mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd"

# 再度ビルド実行
$ ./gradlew build
```

### [SOLVED] assertion 'GDK_IS_X11_DISPLAY (display)' failed

Wayland/GTK3 環境等で起動直後に SIGSEGV エラーが発生する場合の対応。`-Djdk.gtk.version=2.` オプションを指定して起動する。

```console
./gradlew bootRun -Djdk.gtk.version=2.
```
