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

Fedora 環境において cgroup v2 が有効なため `./gradlew build` 時にエラーが発生する場合がある。

```console
# build
$ ./gradlew build

...
    インタフェース Comparableで宣言されているT extends Object
/home/akira/workspaces/refs/cassandra-client/src/main/java/com/github/kindrat/cassandra/client/filter/fields/LePredicate.java:19: 警告:[unchecked] raw型ComparableのメンバーとしてのcompareTo(T)への無検査呼出しです
            return Comparable.class.cast(actual).compareTo(expected) <= 0;
                                                          ^
  Tが型変数の場合:
    インタフェース Comparableで宣言されているT extends Object
警告6個

> Task :envComposeUp
Creating network "cassandra-env_default" with the default driver
Creating cassandra-env_cassandra_1 ... 
Creating cassandra-env_cassandra_1 ... error

ERROR: for cassandra-env_cassandra_1  Cannot start service cassandra: cgroups: cgroup mountpoint does not exist: unknown

ERROR: for cassandra  Cannot start service cassandra: cgroups: cgroup mountpoint does not exist: unknown
Encountered errors while bringing up the project.
Removing cassandra-env_cassandra_1 ... 
Removing cassandra-env_cassandra_1 ... done
Removing network cassandra-env_default

> Task :envComposeUp FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':envComposeUp'.
> Exit-code 1 when calling /home/akira/workspaces/refs/cassandra-client/.gradle/docker-compose/1.22.0/docker-compose.sh, stdout: N/A

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output. Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 29s
13 actionable tasks: 7 executed, 6 up-to-date
```

cgroup v1 のマウントポイントを偽装して対応する。

```console
# cgroup v1 のマウントポイントを作成
$ sudo mkdir -p /sys/fs/cgroup/systemd
$ sudo sh -c "cat /proc/mounts | grep \/sys\/fs\/cgroup\/systemd > /dev/null || mount -t cgroup -o none,name=systemd cgroup /sys/fs/cgroup/systemd"
```

再度ビルド実行

<!-- spell-checker: disable -->
```console
$ ./gradlew build

> Task :envComposeUp
...

> Task :compileTestJava
...

> Task :test
...
org.springframework.boot.logging.logback.SpringBootJoranConfigurator@61fe0e72 - Registering current configuration as safe fallback point

      .   ____          _            __ _ _
     /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
    ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
     \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
      '  |____| .__|_| |_|_| |_\__, | / / / /
     =========|_|==============|___/=/_/_/_/
     :: Spring Boot ::        (v2.1.3.RELEASE)
...

> Task :envComposeDown
Removing cassandra-env_cassandra_1 ... 
Removing cassandra-env_cassandra_1 ... done
Removing network cassandra-env_default

Deprecated Gradle features were used in this build, making it incompatible with Gradle 5.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/4.10.2/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 46s
18 actionable tasks: 5 executed, 13 up-to-date
```
<!-- spell-checker: enable -->

### [SOLVED] assertion 'GDK_IS_X11_DISPLAY (display)' failed

Wayland/GTK3 環境等で起動直後に SIGSEGV エラーが発生する場合がある

```console
./gradlew bootRun 
...

(java:45117): Gdk-CRITICAL **: 23:27:18.165: gdk_x11_display_set_window_scale: assertion 'GDK_IS_X11_DISPLAY (display)' failed
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x00007faafe0dd710, pid=45117, tid=45158
#
...

BUILD FAILED in 2s
3 actionable tasks: 1 executed, 2 up-to-date
```

`-Djdk.gtk.version=2.` オプションを指定して起動する。

```console
./gradlew bootRun -Djdk.gtk.version=2.

> Task :bootRun
...

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.3.RELEASE)

...

BUILD SUCCESSFUL in 14s
3 actionable tasks: 1 executed, 2 up-to-date

```
