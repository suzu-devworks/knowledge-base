# Perl

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Basic Syntax](#basic-syntax)
  - [Strict Syntax Check](#strict-syntax-check)
  - [Exception Handling](#exception-handling)
- [Variables](#variables)
  - [Scope](#scope)
  - [Scalar Variables](#scalar-variables)
  - [Array Variables](#array-variables)
  - [Hash Variables](#hash-variables)
- [Operators](#operators)
  - [String Operators](#string-operators)
  - [Range Operators](#range-operators)
- [Control Flow](#control-flow)
  - [`if`, `elsif`, `else`](#if-elsif-else)
  - [`while`](#while)
  - [`for`](#for)
  - [`foreach`](#foreach)
  - [`next`, `last`](#next-last)
- [Subroutines](#subroutines)
  - [Subroutine Syntax](#subroutine-syntax)
  - [Arguments and Return Values](#arguments-and-return-values)
- [Regular Expressions](#regular-expressions)
  - [Pattern Match Operator](#pattern-match-operator)
  - [Substitution Operator](#substitution-operator)
  - [Transliteration Operator](#transliteration-operator)
- [Reference](#reference)

## Overview

Perlはテキストを整形したり出力したりするのに最適化された言語。テキスト操作にかけてはその右に出るものはない、というぐらいの機能が揃っている。

## Environments

## Basic Syntax

### Strict Syntax Check

Perlでスクリプトを記述する上でプログラムの先頭には必ず次の2行を記述する。

```perl
#!/usr/bin/perl <option> /* Unix用コマンド宣言、Option指定も記述できる */    #             /* コメント */
use strict;              /* 変数宣言を強制したりする */
use warnings;            /* 初期化されない変数を参照したときなど、詳細なエラーを表示する */
```

### Exception Handling

Perl形式の例外処理はtry-catch形式ではない。

```perl
eval {
    die "例外が発生しました。";
};
if ($@){
    print "Error: $@\n";  # $@にメッセージが入っている。
}
```

## Variables

### Scope

変数は初期化の仕方によって、スコープが異なったものになる。

- 変数宣言をせずに初期化された変数: 大域変数
- `my` 宣言をしてから初期化された変数: 静的スコープで、ブロック内でのみ有効な局所変数

### Scalar Variables

スカラー変数とは、数値や文字列のような単純なデータを格納する変数で、1つの変数で1つの値を保持する。

```perl
my $value = 2 + 3;
my $text = "abc";
```

### Array Variables

配列はスカラ変数のリスト。個々のスカラには、0から始まる数字のインデックスを使ってアクセスすることができる。

```perl
my @array = (0, 1, 2);
```

### Hash Variables

ハッシュは、複数のスカラをキーと値のペアによって格納する変数。配列とは違って順番がなく、キーを指定して個々のスカラにアクセスする。

```perl
%hash = (english=>100, math=>80);
```

## Operators

### String Operators

文字列演算子は、値を文字列リテラルと解釈して演算処理を行う。

```perl
# 文字列連結演算子
my $x = "a" . "b";
print $x;     # "ab"を出力

# 文字列繰り返し演算子
my $y = "-" x 10;
print $y;     # "----------"を出力
```

### Range Operators

リストコンテキストでは、左の値から右の値まで数えあげた値からなる配列を返す。

```perl
# 大文字すべての配列を取得
my $alpha = ("A" .. "Z");

# 16進数の数値が得られる
my $hex = (0 .. 9, "a" .. "f")[$num & 15];
```

## Control Flow

### `if`, `elsif`, `else`

条件分岐を行うには `if` ～ `elsif` ～ `else` を利用する。条件分岐後の処理は1文だけでも必ずブロックで囲まなければならない。

```perl
if ( 条件式 ) {
    # 処理....
} elsif ( 条件式 ) {
    # 処理....
} else {
    # 処理....
}
```

`if` は修飾子として後置することもできる。
この場合は逆にブロックで囲むことはできず、1文しか指定できない。

```perl
処理 if 条件;
```

### `while`

`while` では条件が満たされる間、処理が繰り返される。

```perl
while ( 条件式 ) {
    ...
}
```

### `for`

forは配列を要素番号を使って繰り返すときに利用される。

```perl
for ( 式1; 式2; 式3 ) {
    ...
}
```

### `foreach`

`foreach` はすべての配列の要素を順番に処理したい場合に利用される。

```perl
foreach $変数 ( @リスト ){
    ...
}
```

### `next`, `last`

ループ制御は `continue`, `break` ではなく `next`, `last` なので注意。

```perl
while (<>){
    next if /^\d+/;  # 行頭に数値がある場合にはスキップ
    ...
}
```

`continue` は別に構文があり、ループの最後でそれが実行される。
`next` で飛ばされた場合も処理されるが、`last` で終了する場合には処理されない。

```perl
while (<>){
    ...
} continue {
    ...
}
```

## Subroutines

### Subroutine Syntax

`sub` はサブルーチンを定義するキーワードです。

```perl
# サブルーチン定義
sub サブルーチン名 {
    ...
}

# 呼び出し
サブルーチン名();
&サブルーチン名;
```

### Arguments and Return Values

引数は `@_` で渡される。戻り値はreturnで戻す。

```perl
($total, $mean) = &answer(80, 55, 75);

sub answer {
    ($x, $y, $z) = @_;
    my $total = $x + $y + $z;
    my $mean = $total / 3;
    return ($total, $mean);
}
```

## Regular Expressions

### Pattern Match Operator

パターンマッチ演算子は `m//` を使用する。区切り文字がスラッシュ  `/` の場合には `m` を省略できる。

```perl
if ( $str =~ /正規表現/ ) {
    ... # マッチした場合の処理
}

if ( $str !~ /正規表現/ ) {
    ... # マッチしなかった場合の処理
}
```

特殊変数 `$_` を使用している場合には左辺を省略できる。

```perl
if ( /正規表現/ ) {
    ... # マッチした場合の処理
}
```

パターンのなかに `/` が多い場合には他の文字に置き換えることができる。

```perl
if ( m|http://| ) {
    ... # マッチした場合の処理
}
```

Perlには評価コンテキストというのがあり、構文上何が要求されているのかの違いにより、評価値が変化する。特にパターンマッチの場合には意味が大きく変わる。

```perl
# リストコンテキストでのパターンマッチ
@match = ("123 a 56 x" =~ m/\d+/g);
# @match => ("123", "56")

# スカラーコンテキストでのパターンマッチ
$match = ("123 a 56 x" =~ m/\d+/g);
# $match => "1"

# ハッシュコンテキスト
# 変数 $_ から VARIABLE=VALUE という対を全て取り出してハッシュに代入
%hash = /(\b\w+)=(\S*)/g;
```

| モード修飾子 | 説明 |
| --- | --- |
| g | 繰り返しマッチする |
| i | 大文字と小文字の区別をしない |
| o | 変数展開を1度だけ行う |
| m | 文字列を複数行としてあつかう |
| s | 文字列を単一行としてあつかう |
| x | 拡張正規表現を行う |

### Substitution Operator

置き換え演算子は `s///` を使用する。

```perl
$str = "This is a pen.";
$str =~ s/pen/book/;
print $str;
# This is a book.
```

置き換えした値を他の変数に代入する。

<!-- spell-checker: disable -->
```perl
$str = "if you aint runnin game.\n";
($str2 = $str) =~ s/aint/are not/;
print $str, $str2;
# (str)  if you aint runnin game.
# (str2) if you are not runnin game.
```
<!-- spell-checker: enable -->

キャプチャ変数を使用する。

```perl
$str = "back Looking";
$str =~ s/([^ ]*) *([^ ]*)/$2 $1/;
print $str, "\n";
# Looking back
```

置き換え文字列で関数を使用する。

```perl
s/\d+/sprintf("%02d",$&)/e;
```

| モード修飾子 | 説明 |
| --- | --- |
| e | 置き換え文字列を式として評価する |

### Transliteration Operator

変換演算子`tr///`は、検索文字列に含まれる各文字を、置き換え文字列に1文字ずつ変換する。

```perl
$str =~ tr/a-z/A-Z/; # to Upper
```

| モード修飾子 | 説明 |
| --- | --- |
| c | 検索リストに指定された文字以外を検索 |
| d | 置換されなかった文字を削除 |
| s | 置換後の文字が重複している場合は削除して1つにつめる |

## Reference

- [Perl講座(Smart)](http://rfs.jp/sb/perl/)
