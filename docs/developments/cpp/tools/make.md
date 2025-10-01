# Make

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Rules](#rules)
  - [Target](#target)
  - [Tasks (Phony Targets)](#tasks-phony-targets)
  - [Automatic Variables](#automatic-variables)
  - [Rule verification](#rule-verification)
- [Implicit Rules](#implicit-rules)
  - [Show Implicit Rules](#show-implicit-rules)
  - [Defined Variables](#defined-variables)
- [Old-Fashioned Suffix Rules](#old-fashioned-suffix-rules)
- [Macro (Variable) Assignment](#macro-variable-assignment)
- [Builtin Functions](#builtin-functions)
  - [Functions That Control Make](#functions-that-control-make)
  - [Functions for String Substitution and Analysis](#functions-for-string-substitution-and-analysis)
  - [Functions for File Names](#functions-for-file-names)
  - [Functions for Conditionals](#functions-for-conditionals)
  - [Other Functions](#other-functions)
- [Nested](#nested)
- [References](#references)

## Overview

Make is a command-line interface software tool that performs actions ordered by configured dependencies as defined in a configuration file called a makefile.

## Rules

A makefile is composed of blocks of text called rules.

### Target

To create a target, make checks the dependencies files and executes the commands. The commands must be preceded by a Tab character.

```make
# target rule.  
target: dependencies ...  
   commands  
   ...
```

If no target is specified with the make command, the first target in the makefile is executed. Because of this, it is conventional to place an all target at the beginning.

### Tasks (Phony Targets)

You can specify a file name that does not actually exist as a target. These are called tasks or phony targets. Common conventional targets include `clean` and `install`.

```make
# task rule.  
.PHONY: task  
task: dependency-targets ...  
   commands  
   ...
```

The .PHONY (phony: false, sham) target is a special target used to declare a task target.  
If this is not specified, the rule will not function correctly if a file with the same name exists.  

```console
$ touch clean  
$ make clean  
make: 'clean' is up to date.
```

### Automatic Variables

Automatic variables calculate their values each time a rule is executed, based on the rule's target and dependencies.

| Variable | Description |
| :---- | :---- |
| $$@ | The target file name |
| $$% | The target member name when the target is an archive member |
| $$< | The name of the first prerequisite (dependency) |
| $$? | The names of all prerequisites newer than the target |
| $$^ | The names of all prerequisites |
| $$+ | The names of all prerequisites in the order they appeared in the makefile |
| $$* | The target name with the suffix removed |

```make
# pattern rule.  
example.o: depend-1.c depend-2.c depend-3.c  
    @echo $@    # example.o  
    @echo $%    #
    @echo $<    # depend-1.c  
    @echo $?    # depend-1.c depend-2.c depend-3.c  
    @echo $^    # depend-1.c depend-2.c depend-3.c  
    @echo $+    # depend-1.c depend-2.c depend-3.c  
    @echo $*    # example

# archive rule.  
archive(target1.o): depend-1.o depend-2.o depend-3.o  
    @echo $@    # archive
    @echo $%    # target1.o
    @echo $<    # depend-1.o
    @echo $?    # depend-1.o depend-2.o depend-3.o
    @echo $^    # depend-1.o depend-2.o depend-3.o
    @echo $+    # depend-1.o depend-2.o depend-3.o
    @echo $*    # archive
```

To use an internal macro like `$@` as a literal string, you must escape the dollar sign with another `$`. Escaping `$@` results in `$$@`.

### Rule verification

If make isn't behaving as expected, you can try a dry run to see what commands would be executed.

<!-- spell-checker: disable -->
```console
$make -n$ make --dry-run  
...
mkdir -p ./build  
clang -std=c11 -g3 -Wall -Wextra -pthread -I/usr/include/gtk-3.0 -I/usr/include/at-spi2-atk/2.0 -I/usr/include/at-spi-2.0 -I/usr/include/dbus-1.0 -I/usr/lib/x86_64-linux-gnu/dbus-1.0/include -I/usr/include/gtk-3.0 -I/usr/include/gio-unix-2.0/ -I/usr/include/cairo -I/usr/include/pango-1.0 -I/usr/include/harfbuzz -I/usr/include/pango-1.0 -I/usr/include/atk-1.0 -I/usr/include/cairo -I/usr/include/pixman-1 -I/usr/include/freetype2 -I/usr/include/libpng16 -I/usr/include/gdk-pixbuf-2.0 -I/usr/include/libpng16 -I/usr/include/glib-2.0 -I/usr/lib/x86_64-linux-gnu/glib-2.0/include   -c -MMD -MP -o build/example-0.o -c src/example-0.c  
mkdir -p ./build  
clang   build/example-0.o  -lgtk-3 -lgdk-3 -lpangocairo-1.0 -lpango-1.0 -latk-1.0 -lcairo-gobject -lcairo -lgdk_pixbuf-2.0 -lgio-2.0 -lgobject-2.0 -lglib-2.0 -o build/example-0
```
<!-- spell-checker: enable -->

## Implicit Rules

Implicit rules tell make how to use customary techniques so that you do not have to specify them in detail when you want to use them

### Show Implicit Rules

You can check the "Implicit Rules" with the following commands:

```console
$ make --question --print-data-base

# GNU Make 4.1,  
# Built for x86_64-pc-linux-gnu,  
# Copyright (C) 1988-2014 Free Software Foundation, Inc.,  
# License GPLv3+: GNU GPL version 3 or later [<http://gnu.org/licenses/gpl.html>](<http://gnu.org/licenses/gpl.html>),  
# This is free software: you are free to change and redistribute it.,  
# There is NO WARRANTY, to the extent permitted by law.,  
...

## This displays a list of available target options.
$ make -qp | awk -F':' '/^[a-zA-Z0-9][^$#\/\t=]*:([^=]|$)/ {split($1,A,/ /);for(i in A)print A[i]}'
Makefile
all
clean
...
```

### Defined Variables

| Variable | Description |
| :---- | :---- |
| CC = cc | C compile command |
| CXX = g++ | C++ compile command |
| CFLAGS | C compile options |
| CXXFLAGS | C++ compile options |
| CPPFLAGS | Preprocessor options |
| LDFLAGS | Linker options |
| LDLIBS | (No description given) |
| TARGET_ARCH | Target options for cross-compilation |

<!-- spell-checker: words LDLIBS CFLAGS CPPFLAGS CXXFLAGS LDFLAGS -->

## Old-Fashioned Suffix Rules

Suffix rules are an older method. Although they are used in some implicit rules, pattern rules are more general and clearer, so suffix rules have become obsolete.

```make
# double suffix rule  
.c.o:  
    $(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<

# to pattern rule  
%.o: %.c  
    $(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<
```

## Macro (Variable) Assignment

| Assignment Operator | Description |
| :---- | :---- |
| `VAR = value` | Deferred assignment. The value is computed each time the variable is used. If you assign the result of a shell command, the command will be executed every time the variable is read. |
| `VAR := value` | Immediate assignment. The variable's value is processed once and stored. This concise and powerful type of assignment should be the default choice. |
| `VAR ::= value` | Immediate assignment (same as :=). |
| `VAR ?= value` | Acts like := only if the variable is not yet defined. |
| `VAR += value` | Append assignment operator. If the variable was previously set with := or ::=, the right-hand side is treated as an immediate value. Otherwise, it is treated as a deferred value. |
| `VAR != value` | Shell assignment operator. The right-hand side is immediately evaluated, passed to the shell, and the result is stored in the variable on the left-hand side. |

## Builtin Functions

Builtin functions include those for messages and string manipulation.

### Functions That Control Make

- `$(error text…)`  
  Generates a fatal error where the message is text.
- `$(warning text…)`
  This function works similarly to the error function, above, except that make doesn’t exit.
- `$(info text…)`  
  This function does nothing more than print its (expanded) argument(s) to standard output.

```make
ifdef ERROR1
$(error error is $(ERROR1))
endif
```
<!-- spell-checker: words ifdef -->

### Functions for String Substitution and Analysis

- `$(subst from,to,text)`  
  Substitution operation.
- `$(patsubst pattern,replacement,text)`  
  Pattern substitution operation, includes wildcards.
- `$(strip string)`  
  Removes whitespace.
- `$(findstring find,in)`  
  Finds a string.
- `$(filter pattern…,text)`  
  Deletes words that match a pattern.
- `$(filter-out pattern…,text)`  
  Deletes words that *do not* match a pattern.
- `$(sort list)`  
  Sorts words.
- `$(word n,text)`  
  Returns the n-th word.
- `$(wordlist s,e,text)`  
  Returns the list of words in text starting with word s and ending with word e (inclusive).
- `$(words text)`  
  Returns the number of words.
- `$(firstword names…)`  
  Returns the first name.
- `$(lastword names…)`  
  Returns the last name.

<!-- spell-checker: words patsubst wordlist findstring firstword lastword -->

### Functions for File Names

- `$(dir names…)`  
  Extracts the directory part.
- `$(notdir names…)`  
  Extracts the file part.
- `$(suffix names…)`  
  Extracts the suffix (extension) part.
- `$(basename names…)`  
  Extracts the part without the suffix.
- `$(addsuffix suffix,names…)`  
  Adds a suffix.
- `$(addprefix prefix,names…)`  
  Adds a prefix.
- `$(join list1,list2)`  
  Joins words.
- `$(wildcard pattern)`  
  Returns a list of file names matching a wildcard.

<!-- spell-checker: words addprefix addsuffix notdir -->

### Functions for Conditionals

- `$(if condition,then-part[,else-part])`  
  Conditional expansion in a functional context.
- `$(or condition1[,condition2[,condition3…]])`  
  A “short-circuiting” OR operation.
- `$(and condition1[,condition2[,condition3…]])`  
  A “short-circuiting” AND operation.
- `$(intcmp lhs,rhs[,lt-part[,eq-part[,gt-part]]])`  
  Numerical comparison of integers.

<!-- spell-checker: words intcmp -->

### Other Functions

- `$(foreach var,list,text)`  
  Expands arguments multiple times

## Nested

To handle nested subdirectories:

```make
# List the subdirectories here, separated by spaces  
subdirs := \
    aaa \
    bbb \
    ccc

.PHONY: all 
all: $(subdirs)

.PHONY: clean
clean: $(subdirs)

.PHONY: $(subdirs)
$(subdirs): 
   $(MAKE) $(MAKECMDGOALS) -C $@ 
```
<!-- spell-checker: words subdirs MAKECMDGOALS -->

## References

- [Make (software) - Wikipedia](https://en.wikipedia.org/wiki/Make_(software))
- [GNU make - Makefile Conventions](https://www.ecoop.net/coop/translated/GNUMake3.77/make_14.jp.html)
- [Functions (GNU make)](https://www.gnu.org/software/make/manual/html_node/Functions.html)
