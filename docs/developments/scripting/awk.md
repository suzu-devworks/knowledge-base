# AWK

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Command Line](#command-line)
- [Syntax](#syntax)
  - [Basic Structure](#basic-structure)
  - [Patterns](#patterns)
  - [Range](#range)
  - [Built-in Variables](#built-in-variables)
  - [Arrays](#arrays)
  - [Basic Operators](#basic-operators)
  - [Control Flow](#control-flow)
  - [Built-in Commands](#built-in-commands)
- [Examples](#examples)
  - [Remove blank lines](#remove-blank-lines)
  - [Remove consecutive duplicate lines](#remove-consecutive-duplicate-lines)
  - [Combine multiple consecutive blank lines into one](#combine-multiple-consecutive-blank-lines-into-one)
  - [Print the number of lines for each unique value in the first field](#print-the-number-of-lines-for-each-unique-value-in-the-first-field)
  - [Replace tabs with spaces](#replace-tabs-with-spaces)
  - [List of Groups per User](#list-of-groups-per-user)
  - [Data Aggregation](#data-aggregation)
  - [Calendar Display](#calendar-display)
- [References](#references)

## Overview

AWK (/ɔːk/) is a scripting language designed for text processing and typically used as a data extraction and reporting tool.

## Command Line

```console
usage: awk [-F fs] [-v var=value] [-f progfile | 'prog'] [file ...]
```
<!-- spell-checker: words progfile -->

Execute 'program' is:

```shell
awk 'program' input.txt
```

Execute script file is:

```shell
awk -f script.awk input.txt
```

## Syntax

### Basic Structure

After performing the initial processing, it reads each line from the stream and processes each line according to the regular expression pattern. If there are multiple patterns that match, they are processed from the top. Finally, after processing all lines, it performs the final processing. All variables used in AWK are initialized before starting.

```awk
BEGIN { initial processing } 
( pattern 1 ) { action 1 }
( pattern 2 ) { action 2 }
... 
END { final processing }
```

### Patterns

| pattern | Description |
| :---- | :---- |
| Empty pattern | All lines match. |
| `/regexp/` | Lines that match the regular expression are targeted. |
| `$1 == 10` | Lines where the value of the first field is 10 are targeted. |
| `$1 >= 10` | Lines where the value of the first field is 10 or greater are targeted. |
| `$1 < 100 && $2 == "ABC"` | Lines where the value of the first field is less than 100 and the second field is "ABC" are targeted. |
| `$1 ~ /regexp/` | Lines where the first field matches the regular expression are targeted. |
| `length($1) == 0` | Lines where the length of the first field is 0 |

Lines longer than 30 characters:

```shell
awk 'length > 30' input.txt
```

Lines with 5 to 10 fields:

```shell
awk 'NF >= 5 && NF <= 10' input.txt
```

First 5 lines:

```shell
awk 'NR <= 5' input.txt
```

Even-numbered lines:

```shell
awk 'NR%2 == 0' input.txt
```

Lines containing "Question" or "Answer":

```shell
awk '/Question|Answer/' input.txt
```

Lines starts with '#' (comment):

```shell
awk '^\#' input.txt
```

### Range

Specifying two values ​​separated by a comma defines a range.

From the line containing "begin" to the line containing "end":

```shell
awk '/^begin$/,/^end$/' input.txt
```

`input.txt`

```txt
begin
This line is selected
This line is also selected 
end
This line is not selected
begin
This line is selected
```

### Built-in Variables

| Variable | Description |
| :---- | :---- |
| `$0` | Represents the input line |
| `$n` | Represents the nth field |
| `ARGC`, `ARGV` | Represents command-line arguments when awk is started |
| `FILENAME` | Represents the input file name |
| `FNR` | Represents the record number of the current file |
| `FS` | Defines the regular expression for the input field separator. Default value is space and tab |
| `NF` | Represents the number of fields in the current record |
| `NR` | Represents the number of records read. The difference from FNR is that when multiple files are read, it includes the total number of records. |
| `OFS` | Defines the output field separator. Default is a space. |
| `ORS` | Defines the output record separator. Default is a newline (\n). |
| `RS` | Defines the input record separator. Default is a newline (\n). |

Change the field separator to a comma:

```shell
awk 'BEGIN{ FS=",";} { print $1, $3, $5 }' input.txt
awk -F, '{ print $1, $3, $5 }' input.txt
```

 Display the 1st, 3rd, and 5th fields:

```shell
awk '{ print $1, $3, $5 }' input.txt
```

Display with formatting:

```shell
awk '{ printf("%s %-9s %s\n",$1,$3,$5)}' input.txt
```

 Search for "member":

```shell
awk '/member/{ print $1, $5, $9}' input.txt
```

Add a date prefix:

```shell
awk '{print strftime("%Y/%m/%d-%H:%M%S : ",systime()), $0}' input.txt
```
<!-- spell-checker: words systime -->

### Arrays

AWK arrays are associative arrays, meaning they can use strings as indices, not just numbers.

```awk
# Set an array 
a[n] = 10
b["India"] = "インド"

# Set a multi-dimensional array 
c[x,y] = 0

# Delete an array when no longer needed
delete a[n]
```

### Basic Operators

| Priority | Operator | Operation |
| :---- | :---- | :---- |
| 1 | `(...)` | Grouping |
| 2 | `$` | Field reference |
| 3 | `++` `--` | Increment and decrement. Both prefix and postfix versions are available. |
| 4 | `^` | Exponentiation (You can also use `**`. There is also an assignment operator `**=`, but these are not defined in the POSIX standard.) |
| 5 | `+` `-` `!` | Unary plus, unary minus, logical negation. |
| 6 | `*` `/` `%` | Multiplication, division, modulus |
| 7 | `+` `-` | Addition and subtraction |
| 8 | space | String concatenation (variable1 variable2) |
| 9 | `<` `<=` `>` `>=` `!=` `==` | Relational operators |
| 10 | `~` `!~` | Regular expression match, non-match |
| 11 | `in` | Array membership |
| 12 | `&&` | AND condition |
| 13 | `\|\|` | OR condition |
| 14 | expr1 `?` expr2 `:` expr3 | Ternary operator |
| 15 | `=` `+=` `-=` `*=` `/=` `%=` `^=` | Assignment. Both the standard assignment statement (var=value) and other assignment operators are supported. |

Numeric Conversion:

In AWK, which does not have data types, addition with 0 is used to perform arithmetic operations on values ​​as numbers.

```awk
variable + 0
```

### Control Flow

```awk
if (condition) statement [else statement]
if (condition) statement [else if (condition) statement ][else statement]

while (condition) statement
do statement while (condition)
for (expr1; expr2; expr3) statement
for (var in array) statement
break
continue
delete array[index]
delete array
exit [expression]
{ statements }
```

| Command | Description |
| :---- | :---- |
| `break` | The break statement exits the innermost for, while, or do-while loop. |
| `continue` | The `continue` statement is similar to `break` and can only be used within `for`, `while`, and `do-while` loops. `continue` skips the remaining part of the loop body and immediately starts the next iteration of the loop. |
| `next` | The `next` statement forces awk to immediately stop processing the current record and move on to the next record. This means that any subsequent rules will not be applied to the current record. Similarly, any remaining actions within the currently executing rule will not be executed. |
| `exit` | The `exit` statement immediately stops awk from executing the current rule and stops processing the input. Any remaining input is ignored. |

### Built-in Commands

| Command | Description |
| :---- | :---- |
| `print item1, item2, ...` | Outputs to standard output in a standard format. |
| `printf format, item1, item2, ...` | Outputs to standard output in a more complex format. |
| `length ([string])` | Returns the number of characters. |
| `gsub (regexp, replacement [, target])` | Replaces occurrences of the first argument (regexp) in the third argument (target) with the second argument (replacement). The third argument is modified in place. |
<!-- spell-checker: words gsub -->

## Examples

### Remove blank lines

```shell
awk 'length != 0' input.txt
awk 'NF != 0' input.txt
```

### Remove consecutive duplicate lines

```shell
awk 'before != $0 {print; before = $0}' input.txt
```

### Combine multiple consecutive blank lines into one

```shell
awk 'NF != 0 || bf != 0 { print; bf=NF}' input.txt
```

### Print the number of lines for each unique value in the first field

```shell
   awk '{ id[$1] = id[$1] " " NR } END { for (elm in id) print elm " " id[elm]}' input.txt
```

### Replace tabs with spaces

```shell
# assigning to $1, which seems pointless, redefines $0
awk '{$1 = $1; print $0}' input.txt
```

### List of Groups per User

This script reads the `/etc/passwd` file and outputs the groups that each user belongs to.

```awk
BEGIN {
   FS=":";
   while (getline <"/etc/passwd"){
       printf("%s: ",$1);
       system("id -Gn " $1);
   }
}
```
<!-- spell-checker: words getline -->

### Data Aggregation

This script processes financial data from note.dat to calculate the total balance, maximum withdrawal, maximum deposit, and average for different categories.

`summary.awk`:

```awk
BEGIN { FS="\t+"; n = 3 }

{
    gsub(",","",$n)
    gsub(/\/[0-9]+/,"月", $1)
    summary("総計")
    summary($1)
}

$2 != "給料" { summary("給与外")}

function summary(c) {
    if (MAX[c]+0 < $n+0 ) MAX[c] = $n
    if (MIN[c]+0 > $n+0 ) MIN[c] = $n
    COUNT[c]++
    TOTAL[c] += $n
}

END {
    for (c in TOTAL){
        printf "残高(%s) = %d\n", c, TOTAL[c]
        printf "最大出金額(%s) = %d\n", c, MIN[c]
        printf "最大入金額(%s) = %d\n", c, MAX[c]
        printf "平均(%s) = %f\n", c, TOTAL[c]/COUNT[c]
        print "---"
    }
}
```

`note.dat`:

```txt
10/1 水道  -20
10/2 電気  -6,231
10/3 ガス  -300
10/3 電話  -6,503
10/20 パチンコ -20,200
10/23 競馬  3,000
10/25 給料  223,000
10/26 飲食店のつけ -134,523
11/1 水道  -30
11/2 電気  -6,231
11/3 ガス  -700
11/3 電話  -10，300
11/5 競馬  -164,000
11/10 麻雀  40,000
11/25 給与  234，450
11/26 飲食店のつけ -122,459
```

```console
$ awk -f summary.awk note.dat

残高(給与外) = -417973
最大出金額(給与外) = -164000
最大入金額(給与外) = 40000
平均(給与外) = -27864.866667
---
残高(総計) = -194973
最大出金額(総計) = -164000
最大入金額(総計) = 223000
平均(総計) = -12185.812500
---
残高(10月) = 58223
最大出金額(10月) = -134523
最大入金額(10月) = 223000
平均(10月) = 7277.875000
---
残高(11月) = -253196
最大出金額(11月) = -164000
最大入金額(11月) = 40000
平均(11月) = -31649.500000
---
```

### Calendar Display

`calender.awk`:

```awk
BEGIN {
    M[1]=31; M[2]=28; M[3]=31; M[4]=30; M[5]=31; M[6]=30
    M[7]=31; M[8]=31; M[9]=30; M[10]=31; M[11]=30; M[12]=31
    W[0]="日"; W[1]="月"; W[2]="火"; W[3]="水"; W[4]="木"; W[5]="金"; W[6]="土"
    
    split(option(), date, ".")
    n = week1(date[1]+0, date[2]+0, date[3]+0)
    if ( date[3] != "" ) print W[n] "曜日"
    else if ( date[2] != "" ) {
        calender(date[1], date[2], M[date[2]], n)
    }
    else {
        for (i = 1; i <= 12; i++ ){
            calender(date[1], i, M[i], week1(date[1], i, 0))
            print ""
        }
    }
}

function week1(y, m, d, s) {
    s = 5
    y %= 400
    if ( y == 0 || (y%4 == 0 && y%100 != 0 )) if ( M[2] == 28 ) M[2]++
    if ( y > 0 ) s += --y+int(y/4)-int(y/100)+int(y/400)+2
    while ( m > 1 ) s += M[--m]
    return (s+d)%7
}

function calender(y, m, d, s, i) {
    printf("     %d%s %d%s\n", y, "年", m, "月")
    print  "日 月 火 水 木 金 土 "
    printf("%s", repeat(" ", ((s+1)%7)*3))
    for (i = 1; i < d; i++){
        printf("%2d ", i)
        if ((s+i)%7 == 6 ) print ""
    }
    if ((s+i)%7 != 0 ) print ""
}

function repeat(str, time, i, ret ){
    for (i = 1; i <= time; i++) ret = str ret
    return ret
}

function option(opt) {
    if(ARGC > 1 && ARGV[1] ~ /^\+.*$/){
        #print ARGC, ARGV[1]
        opt = substr(ARGV[1], 2)
        delete ARGV[1]
    }
    return opt
}

```

```console
$ awk -f calender.awk +2013.10.4
金曜日

$ awk -f calender.awk +2013.10
     2013年 10月
日 月 火 水 木 金 土 
       1  2  3  4  5 
 6  7  8  9 10 11 12 
13 14 15 16 17 18 19 
20 21 22 23 24 25 26 
27 28 29 30 

$ awk -f calender.awk +2013
... 
```

## References

- [Gawk: Effective AWK Programming - GNU Project - Free Software Foundation](https://www.gnu.org/software/gawk/manual/)
