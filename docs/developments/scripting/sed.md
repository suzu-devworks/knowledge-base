# Sed

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Command Line](#command-line)
- [Syntax](#syntax)
  - [Sed commands](#sed-commands)
  - [Addressing](#addressing)
  - [Processing Multiple Commands](#processing-multiple-commands)
  - [Using Shell Variables](#using-shell-variables)
  - [Grouping](#grouping)
  - [Labels, Branching, and Loops](#labels-branching-and-loops)

## Overview

Sed is a program used for text transformations and other data processing on an input stream.

## Command Line

`-e`: Process a script (can be omitted)

```shell
sed -e 'script1' -e 'script2' -e 'script3' input.txt > output.txt
```

`-f`: Process a script file

```shell
sed -f script-file.sed input.txt > output.txt
```

`-n`: Only print output based on explicit print commands in the script

```shell
sed -n 'script' input.txt > output.txt
```

## Syntax

### Sed commands

```sed
[address[,address]] command [ arguments ]
```

| command | Description |
| :---- | :---- |
| `s/regexp/replacement/flag` | Replaces the pattern matched by the regular expression. |
| `y/string1/string2/` | Replaces single characters; multiple characters can be replaced simultaneously. |
| `d` | Deletes the line. |
| `N` | Appends the next line to the pattern space. |
| `p` | Prints the line. |
| `w \[filename\]` | Writes the line to a file. |

| flag | Description |
| :---- | :---- |
| `g` | Global: Processes all matches on a line. |
| `p` | Print: Prints the matched and transformed line. |
| `w` | Write: Writes the matched and transformed line to a file. |

`input.txt`:

<!-- spell-checker: disable -->
```txt
aaabbbcccddd aaabbbcccddd
AAABBBCCCDDD aaabbbcccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Replace "bbb" with "eee":

<!-- spell-checker: disable -->
```console
$ sed -e 's/bbb/eee/' input.txt

aaaeeecccddd aaabbbcccddd
AAABBBCCCDDD aaaeeecccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Use global:

<!-- spell-checker: disable -->
```console
$ sed -e 's/bbb/eee/g' input.txt

aaaeeecccddd aaaeeecccddd
AAABBBCCCDDD aaaeeecccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

When using 'p', output from other lines is suppressed:

<!-- spell-checker: disable -->
```console
$ sed -e 's/AAA/sss/p' input.txt

sssBBBCCCDDD aaabbbcccddd
```
<!-- spell-checker: enable -->

When using 'w', specify a filename (only transformed lines are written to the file):

<!-- spell-checker: disable -->
```console
$ sed -e 's/bbb/eee/gw output.txt' input.txt

aaaeeecccddd aaaeeecccddd
AAABBBCCCDDD aaaeeecccddd
1234567890!? !"#$%&'()?/_

$ cat output.txt

aaaeeecccddd aaaeeecccddd
AAABBBCCCDDD aaaeeecccddd
```
<!-- spell-checker: enable -->

To output all lines:

<!-- spell-checker: disable -->
```console
$ sed -e 's/bbb/eee/g' -e 'w output.txt' input.txt

aaaeeecccddd aaaeeecccddd
AAABBBCCCDDD aaaeeecccddd
1234567890!? !"#$%&'()?/_

$ cat output.txt

aaaeeecccddd aaaeeecccddd
AAABBBCCCDDD aaaeeecccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Replace characters a->x, b->y, c->z:

<!-- spell-checker: disable -->
```console
$ sed -e 'y/abc/xyz/' input.txt

xxxyyyzzzddd xxxyyyzzzddd
AAABBBCCCDDD xxxyyyzzzddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Delete the last line of `input.txt`:

<!-- spell-checker: disable -->
```console
$ sed -e '$ d' input.txt

aaabbbcccddd aaabbbcccddd
AAABBBCCCDDD aaabbbcccddd
```
<!-- spell-checker: enable -->

Match and replace \[/\]:

```console
$ echo $PATH

/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:/opt/X11/bin

$ echo $PATH | sed -e 's\!/usr/local\!/use\!'

/usr/bin:/bin:/usr/sbin:/sbin:/use/bin:/opt/X11/bin
```

Replace with \[&\]:

<!-- spell-checker: disable -->
```console
$ sed -e 's/b/¥&/g' input.txt

aaa&&&cccddd aaa&&&cccddd
AAABBBCCCDDD aaa&&&cccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Replace with \[/\]:

<!-- spell-checker: disable -->
```console
$ sed -e 's/b/\//g' input.txt

aaa///cccddd aaa///cccddd
AAABBBCCCDDD aaa///cccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Replace with \[\\\]:

<!-- spell-checker: disable -->
```console
$ sed -e 's/b/\\/g' input.txt

aaa\\\cccddd aaa\\\cccddd
AAABBBCCCDDD aaa\\\cccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Reference a regular expression using '&':

<!-- spell-checker: disable -->
```console
$ sed -e 's/aaa/+&+/g' input.txt

+aaa+bbbcccddd +aaa+bbbcccddd
AAABBBCCCDDD +aaa+bbbcccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

### Addressing

| address | Description |
| :---- | :---- |
| `\[omitted\]` | Executes the command on all input lines. |
| line number | The line with the specified number. |
| line1,line2 | The range from line1 to line2. |
| line1,line2\! | All lines except for the range from line1 to line2. |
| `/regexp/` | The lines that match the regular expression. |
| `$` | The last line. |

Lines 1-2:

<!-- spell-checker: disable -->
```console
$ sed -e '1,2p' input.txt

aaabbbcccddd aaabbbcccddd
AAABBBCCCDDD aaabbbcccddd
```
<!-- spell-checker: enable -->

Lines 2 to the end ($):

<!-- spell-checker: disable -->
```console
$ sed -e '2,$p' input.txt

AAABBBCCCDDD aaabbbcccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

Lines starting with 'aaa':

<!-- spell-checker: disable -->
```console
$ sed -e '/^aaa/p' input.txt

aaabbbcccddd aaabbbcccddd
```
<!-- spell-checker: enable -->

Lines from one that starts with 'aaa' to one that ends with 'ddd':

<!-- spell-checker: disable -->
```console
$ sed -e '/^aaa/,/ddd$/p' input.txt

aaabbbcccddd aaabbbcccddd
AAABBBCCCDDD aaabbbcccddd
```
<!-- spell-checker: enable -->

### Processing Multiple Commands

You can specify multiple commands using ';'.

Executes 'd'->'s' commands sequentially, one line at a time:

<!-- spell-checker: disable -->
```console
$ sed -e '2d;s/aaa/eee/g' input.txt

eeebbbcccddd eeebbbcccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

You can also specify multiple '-e' commands:

<!-- spell-checker: disable -->
```console
$ sed -e '2d' -e 's/aaa/eee/g' input.txt

eeebbbcccddd eeebbbcccddd
1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

### Using Shell Variables

Normally, sed commands are enclosed in single quotes to prevent the shell from interpreting them unexpectedly. However, to use shell variables, you must use double quotes.

Using `$HOSTNAME`:

<!-- spell-checker: disable -->
```console
$ sed \-e "s/.\*/$HOSTNAME:&/g" input.txt

MacBookAir.local:aaabbbcccddd aaabbbcccddd
MacBookAir.local:AAABBBCCCDDD aaabbbcccddd
MacBookAir.local:1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->

### Grouping

In a script file, you can group multiple commands with {...} to execute them on a single address.

`script.sed`

```sed
1,3 {
    s/aaa/eee/g
    y/abc/xyz/
    p
}
```

### Labels, Branching, and Loops

| Method | Description |
| :---- | :---- |
| `:label` | Specifies a label. The label name must be unique within the script. |
| `b label` | Branches unconditionally to the command specified by the label name. |
| `conditional b label` | Branches to the command specified by the label name if the input line meets the condition. For example, /pattern/b label name branches if the input line matches the pattern. |
| `conditional \!b label` | Branches to the command specified by the label name if the input line does not meet the condition. For example, /pattern/\!b label name branches if the input line does not match the pattern. |
| `t label` | Branches to the command specified by the label name only if a substitution (s) command was successful after the previous input line was read or a conditional branch was performed. |
| `T label` | Branches to the command specified by the label name only if a substitution (s) command was not successful after the previous input line was read or a conditional branch was performed. |

`sample.sed`:

```sed
# A loop to concatenate all input data into a single line, then remove newlines.
:loop
N
$!b loop
s/\n//g
```

<!-- spell-checker: disable -->
```console
$ sed -f sample.sed input.txt

aaabbbcccddd aaabbbcccdddAAABBBCCCDDD aaabbbcccddd1234567890!? !"#$%&'()?/_
```
<!-- spell-checker: enable -->
