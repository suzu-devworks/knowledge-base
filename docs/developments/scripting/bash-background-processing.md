# Bash - Background Processing

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Running and Stopping Background Processes](#running-and-stopping-background-processes)
- [Stopping Background Processes from a Shell Script](#stopping-background-processes-from-a-shell-script)
- [Waiting for a Background Process to Finish](#waiting-for-a-background-process-to-finish)
- [Terminating a Background Script with CTRL+C](#terminating-a-background-script-with-ctrlc)

## Overview

This document explains how to perform asynchronous processing using background tasks in shell scripts.

## Running and Stopping Background Processes

By adding an ampersand (&) after a command, you can run it as a background process and immediately enter the next command. In a multi-CPU environment, you can see that each command uses its own CPU when viewed with top. However, this does not guarantee faster execution, especially if there are other bottlenecks, such as a single HDD.

Run a process in the background:

```console
$ sleep 1000 &

[1] 24472
```

Stop a background process:

```console
$ sleep 1000 &

[1] 31485

$ jobs

[1]+ Running      sleep 1000 &

$ kill %1

[1]+ Terminated   sleep 1000 &

$ jobs

```

## Stopping Background Processes from a Shell Script

You can terminate a foreground process with CTRL+C, but you cannot terminate a background process in the same way.

`backup_sample.sh`:

```bash
#!/bin/bash

tar zcvf vm.tar.gz \~/VM/ &\> /dev/null &
tar zcvf old.tar.gz \~OLD/ &\> /dev/null &
```
<!-- spell-checker:words zcvf -->

```console
# Execute the script
$ ./backup_sample.sh

# The jobs command does not show the background processes
$ jobs

# Use ps and kill to terminate them
$ ps
  PID TTY          TIME CMD
 4364 pts/1    00:00:00 bash
 9725 pts/1    00:00:00 tar
 9726 pts/1    00:00:00 tar
 9727 pts/1    00:00:00 gzip
 9728 pts/1    00:00:00 gzip
 9729 pts/1    00:00:00 ps

$ kill 9725
```

A one-liner to terminate them:

```shell
ps | grep tar | awk '{print $1}' | xargs kill
```

## Waiting for a Background Process to Finish

You can group commands using { } and place & at the end. The commands inside the braces will be processed sequentially as a single background process.

This is a method to wait for background processes to finish using semaphore files.

```bash
#!/bin/bash

rm -f ./sem.{1,2}
{
    tar zcvf vm.tar.gz \~/VM/ &\> /dev/null
    touch ./sem.1
} &

{
    tar zcvf old.tar.gz \~OLD/ &\> /dev/null
    touch ./sem.2
} &

while sleep 3 ; do
    if \[ \-e ./sem.1 \-a \-e ./sem.2 \]; then
        rm ./sem.{1,2}
        exit 0
    fi
done
```

## Terminating a Background Script with CTRL+C

This script uses trap to catch the SIGINT signal (2) and terminate the background processes and clean up.

```bash
#!/bin/bash

EXIT(){
    ps | grep tar | self 1 | xargs kill
    rm \-f ./{vm,old}.tar.gz
    # Wait for the next touch to finish after terminating tar
    while \! rm ./sem.1 ; do sleep 1 ; done
    while \! rm ./sem.2 ; do sleep 1 ; done
    exit 1
}

# Trap signal 2 (SIGINT)
trap EXIT 2

rm \-f ./sem.{1,2}
{
    tar zcvf vm.tar.gz \~/VM/ &\> /dev/null
    touch ./sem.1
} &

{
    tar zcvf old.tar.gz \~OLD/ &\> /dev/null
    touch ./sem.2
} &

while sleep 3 ; do
    if \[ \-e ./sem.1 \-a \-e ./sem.2 \]; then
        rm ./sem.{1,2}
        exit 0
    fi
done
```
