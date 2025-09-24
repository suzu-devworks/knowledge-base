# Bash

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Syntax](#syntax)
  - [Quotation](#quotation)
  - [Redirection and File Descriptors](#redirection-and-file-descriptors)
  - [Here Documents](#here-documents)
- [Variables](#variables)
  - [Shell Variables](#shell-variables)
  - [String Manipulation (Bash v4 and later)](#string-manipulation-bash-v4-and-later)
  - [Environment Variables](#environment-variables)
  - [Arrays](#arrays)
  - [Positional Parameters](#positional-parameters)
  - [Special Shell Variables](#special-shell-variables)
- [Operators](#operators)
  - [Operation Priority](#operation-priority)
  - [Arithmetic Operation](#arithmetic-operation)
  - [Test](#test)
- [Control Flow](#control-flow)
  - [`if` ... `else`](#if--else)
  - [`case`](#case)
  - [`for`](#for)
  - [`while`](#while)
  - [`until`](#until)
- [Subroutines](#subroutines)
  - [Processing STDIN a Subroutine](#processing-stdin-a-subroutine)
  - [Returning a String from a Subroutine](#returning-a-string-from-a-subroutine)
- [Built-in Commands](#built-in-commands)
- [Login Shell](#login-shell)

## Overview

Bash (Bourne-Again Shell) is one of the shells used in Unix, written for the GNU Project. Bash is standard on many Linux systems and Mac OS X (10.3 and later) and runs on many Unix-like operating systems. It has also been ported to Microsoft Windows by the Cygwin project.

## Syntax

### Quotation

The shell provides three types of quotation marks to control the expansion and substitution of special characters like meta-characters: the single quote ', the double quote ", and the backtick \`. Each has a different function.

| Symbol | Description |
| :---- | :---- |
| 'text' | Single quotes: The string is treated literally. |
| "text" | Double quotes: The string is treated literally, except for $ (variable expansion), \` (command substitution), and \\ (escape character). |
| \`command\` | Backticks: The command is executed and its output replaces the backticks. |

### Redirection and File Descriptors

The shell has the ability to switch the input and output destinations of a command (I/O redirection).

```shell
# Redirect standard output to a file
command > file_name

# Redirect standard error to a file (append)
command 2>> file_name

# Redirect both to a single file (merge)
command > file_name 2>&1

# Redirect standard input from a file
command < file_name
```

| File Descriptor | Number | Description |
| :---- | :---- | :---- |
| `STDIN` | 0 | Standard input, usually from the keyboard. |
| `STDOUT` | 1 | Standard output, usually to the command prompt screen. |
| `STDERR` | 2 | Standard error, usually for error messages on the command prompt screen. |

### Here Documents

The \<\< redirection operator provides a special mechanism called a "here document," which allows you to remove the need for external files.

```shell
grep $1 << EOD
sample1
sample2
sample3
sample4
sample5
EOD
```

## Variables

### Shell Variables

Shell variables have no concept of data types, and all variables are treated as strings.

```shell
# Setting a shell variable
variable="value"

# Unsetting a shell variable
unset variable

# Displaying shell variables
set
```

| Syntax | Description |
| :---- | :---- |
| `$variable` | Reference the value set in the variable. |
| `${variable}` | Reference the value set in the variable. The curly braces {} are used to separate the variable name from surrounding characters. |
| `${variable-string}` | If the variable is unset or null, use String as a default value. |
| `${variable=string}` | If the variable is unset or null, set its value to String and then expand it. |
| `${variable?message}` | If the variable is unset or null, print message to standard error and exit. |
| `${variable#word}` | Remove the shortest matching word prefix from the variable's value. |
| `${variable%word}`| Remove the shortest matching word suffix from the variable's value. |
| `${variable//pattern/word}` | Replace all occurrences of pattern with word in the variable's value. |

### String Manipulation (Bash v4 and later)

You can convert the case of strings within shell variables.

If Bash v4 is not available, use `tr "A-Z" "a-z"`.

| Syntax | Description |
| :---- | :---- |
| `${variable^}, ${variable^^}` | Convert the string to uppercase (only the first letter for ^). |
| `${variable,}, ${variable,,}` | Convert the string to lowercase (only the first letter for ,). |
| `${variable~}, ${variable~~}` | Invert the case of the string (only the first letter for \~). |

### Environment Variables

Environment variables are a type of shell variable used to configure the user's environment. They are usually set with export to affect all commands executed during the login session.

```shell
# Setting an environment variable
variable="value"; export variable

# Unsetting an environment variable
unset variable

# Displaying environment variables
printenv (or env)
```

### Arrays

Bash also supports one-dimensional arrays for shell variables.

```shell
# Setting an array
array=(val1 val2 val3 val4...)
array[5]="value"

# Referencing an element (empty if not initialized)
${array[0]}

# Referencing all elements
"${array[*]}"

# Referencing the number of elements
"${#array[*]}"

# Unsetting the array
unset array
```

### Positional Parameters

These are variables for receiving command-line arguments. The shift command can be used to shift them, which allows you to reference arguments beyond the 9th. The shift command does not affect $0.

| Parameter | Description |
| :---- | :---- |
| `$0` | The name of the shell script file. |
| `$1`, `$2`, ..., `$9` | The arguments. |
| `$#` | The number of parameters. |
| `$*` | All parameters except $0. |
| `$@` | Same as $\*, but when enclosed in double quotes, each parameter is treated as a separate argument. |

### Special Shell Variables

| Parameter | Description |
| :---- | :---- |
| `$?` | The exit status of the last executed command. |
| `$$` | The process ID of the current shell. |
| `$-` | The current options set for the shell. |
| `$!` | The process ID of the most recently executed background process. |

## Operators

### Operation Priority

| Priority | Operator | Operation |
| :---- | :---- | :---- |
| 1 | () | Grouping |
| 2 | str:regexp | Compares string str with regular expression regexp. |
| 3 | \* / % \+ \- | Arithmetic operations. |
| 4 | \<, \<=, \=, \!=, \>=, \> | Comparison operators. |
| 5 | & | Bitwise logical AND. |
| 6 | ^ | Bitwise exclusive OR. |
| 7 | \| | Bitwise logical OR. |

### Arithmetic Operation

To perform arithmetic operations, use the expr command or enclose the expression in $(()).

```shell
expr val1 operator val2

# Assign result to a variable
variable=`expr val1 operator val2`
variable=$((val1+val2))
```

### Test

Used in if statements.

```shell
$ test EXPRESSION
# or
$ [ (OPTION) ]
```

| Type | Option | Description |
| :---- | :---- | :---- |
| File Type | \-d \[file\] | True if \[file\] exists and is a directory. |
|  | \-f \[file\] | True if \[file\] exists and is a regular file. |
|  | \-L \[file\] | True if \[file\] exists and is a symbolic link. |
| File Permissions | \-r \[file\] | True if \[file\] exists and is readable. |
|  | \-w \[file\] | True if \[file\] exists and is writable. |
|  | \-x \[file\] | True if \[file\] exists and is executable. |
| File Attributes | \-e \[file\] | True if \[file\] exists. |
|  | \-s \[file\] | True if \[file\] exists and has a size greater than zero. |
|  | file1 \-nt file2 | True if file1 is newer than file2 based on modification date. |
|  | file1 \-ot file2 | True if file1 is older than file2 based on modification date. |
| Strings | \-n string | True if the length of the string is greater than zero. |
|  | \-z string | True if the length of the string is zero. |
|  | string1 \= string2 | True if the two strings are equal. |
|  | string1 \!= string2 | True if the two strings are not equal. |
| Numbers | num1 \-eq num2 | True if num1 \== num2. |
|  | num1 \-ne num2 | True if num1 \!= num2. |
|  | num1 \-lt num2 | True if num1 \< num2. |
|  | num1 \-le num2 | True if num1 \<= num2. |
|  | num1 \-gt num2 | True if num1 \> num2. |
|  | num1 \-ge num2 | True if num1 \>= num2. |
| Logical Operators | expr1 \-a expr2 | AND |
|  | expr1 \-o expr2 | OR |

## Control Flow

### `if` ... `else`

Checks the exit status of a command in the conditional expression. If it is true (0), the then block is executed. Conditional expressions are often written in shorthand, but the test command is also commonly used.

```shell
if condition1; then
  command1
  ...
elif condition2; then
  command2
  ...
else
  command3
  ...
fi
```
<!-- spell-checker: words elif -->

Simple conditions can sometimes be replaced with && and ||.

```shell
# Equivalent to: if command1; then command2; fi
command1 && command2

# Equivalent to: if \!command1; then command2; fi
command1 || command2
```

### `case`

Evaluates a single shell variable and executes one or more commands if a matching pattern is found. The ;; can be replaced with ; to fall through to the next pattern. The final \*) is the default case.

```shell
case $variable in
  pattern1)
    command1
    ...
    ;;
  pattern2)
    command2
    ...
    ;;
  ...
  \*)
    command
    ...
    ;;
esac
```
<!-- spell-checker: words esac -->

### `for`

Used to execute a set of commands a specified number of times. If in and the following values are omitted, the positional parameters ($\*) are used. For simple loops of consecutive numbers, you can use the seq command.

```shell
for variable [in String1, String2....]
do
  command
  ...
done

# Example: Lists numbers from 0 to 9
for i in $(seq 0 9); do echo $i; done
```

### `while`

Executes a loop as long as a certain condition is met. The condition is the same as in if, judging whether the command's exit status is true (0).

```shell
while condition
do
  command
  ...
done
```

### `until`

Executes a loop as long as the exit status is non-zero (false), which is the opposite of while.

```shell
until condition
do
  command
  ...
done
```

## Subroutines

### Processing STDIN a Subroutine

To process STDIN a subroutine, you can use the read command with a while loop.

```shell
sub_sample () {
  local line
  while read line
  do
    echo $line
  done
}

echo "abc" | sub_sample
```

### Returning a String from a Subroutine

Since return can only return a number, you can use echo to return a string, as shown in the example below.

```shell
sub_sample () {
  echo "return!!"
}

returnValue=`sub_sample`
```

## Built-in Commands

| Command | Description |
| :---- | :---- |
| : | NOP (no operation). |
| break \[Level No\] | Exits a for or while loop. |
| continue \[Level No\] | Skips to the next iteration of a for or while loop. |
| exit \[n\] | Exits the shell with an optional status code n. |
| exec \[arguments...\] | Replaces the shell with the command specified by the arguments. |
| eval \[arguments...\] | Parses the arguments as input to the shell and then executes them as a command. |
| export \[variable\] | Sets an environment variable. |
| read \[variable\] | Reads a single line from standard input. |
| set | Displays and sets shell options and variables. \-v shows input lines as they are read, and \-x prints commands and their arguments as they are executed. |
| unset | Unsets variables or functions. |
| return | Returns from a subroutine. |
| shift \[n\] | Shifts the positional parameters. |
| test | Evaluates a conditional expression. |
| times | Displays process execution times. |
| trap \[Command\]\[n\] | Executes Command when signal n is received. |
| wait \[n\] | Waits for the process with PID n to finish. |
<!-- spell-checker: words Unsets -->

## Login Shell

| Shell | Description |
| :---- | :---- |
| `.bash_profile` | This script is executed only when you log in. It is used to set environment variables (using export) that should be available to all commands executed during the session. |
| `.bashrc` | This script is executed every time you start an interactive Bash session. It is used to set non-exported variables, define aliases, define shell functions, and configure command-line completion. |
