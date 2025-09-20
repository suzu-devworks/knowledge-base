# Regular Expressions Backtracking and Grouping

- [Regular Expressions Backtracking and Grouping](#regular-expressions-backtracking-and-grouping)
  - [Backtracking](#backtracking)
  - [Grouping](#grouping)
    - [Matched subexpressions (Capturing)](#matched-subexpressions-capturing)
    - [Named matched subexpressions (Capturing)](#named-matched-subexpressions-capturing)
    - [Balancing group definitions (Capturing)](#balancing-group-definitions-capturing)
    - [Non-Capturing groups (Non-Capturing)](#non-capturing-groups-non-capturing)
    - [Group options (Non-Capturing)](#group-options-non-capturing)
    - [Zero-width positive lookahead assertions (Non-Capturing)](#zero-width-positive-lookahead-assertions-non-capturing)
    - [Zero-width negative lookahead assertions (Non-Capturing)](#zero-width-negative-lookahead-assertions-non-capturing)
    - [Zero-width positive lookbehind assertions (Non-Capturing)](#zero-width-positive-lookbehind-assertions-non-capturing)
    - [Zero-width negative lookbehind assertions (Non-Capturing)](#zero-width-negative-lookbehind-assertions-non-capturing)
    - [Atomic groups (Non-Capturing)](#atomic-groups-non-capturing)
  - [References](#references)

## Backtracking

バックトラッキングは、正規表現パターンに省略可能な量指定子（*, +, {n,m}）、または代替構成体（|, (?(name)yes|no)）が含まれている場合に発生します。

バックトラッキングは、正規表現を強力にするための中心的な機能で、これにより、非常に複雑なパターンを照合できる強力かつ柔軟な正規表現を作成できるようになります。 その一方で、多くの場合、正規表現エンジンのパフォーマンスを左右する最大の要因になります。

## Grouping

### Matched subexpressions (Capturing)

```regexp
( subexpressions )
```

 かっこを使用するキャプチャ。
 左かっこの順番に基づいて、左から右に自動的に 1 から始まる番号が付けられます。
 番号 0 は、正規表現パターン全体と一致するテキストです。
 名前が重複する場合、 Group オブジェクトの値は、入力文字列の最後の正常なキャプチャによって決定されます。

### Named matched subexpressions (Capturing)

```regexp
(?<name>subexpressions)
(?'name'subexpressions)
```

キャプチャに名前をつけてアクセスする方法。
```\k<name>```で一致した部分式が同じ正規表現内で参照されます。

### Balancing group definitions (Capturing)

```regexp
(?<name1-name2>subexpressions)
(?'name1-name2'subexpressions)
```

### Non-Capturing groups (Non-Capturing)

```regexp
(?:subexpressions)
```

### Group options (Non-Capturing)

```regexp
(?imnsx-imnsx:subexpressions)
```
<!-- spell-checker: words imnsx -->

||||
|--|--|--|
| i | IgnoreCase | 大文字と小文字を区別しない一致を使用します。|
| m | Multiline  | 複数行モードを使用します。 ^ と $ は、(入力文字列の先頭および末尾ではなく) 各行の先頭および末尾と一致します。|
| s | Singleline | 単一行モードを使用します。このモードでは、ピリオド (.) は任意の 1 文字と一致します (\n を除くすべての文字の代用)。|
| n | ExplicitCapture | 名前のないグループをキャプチャしません。|
| x | IgnorePatternWhitespace | エスケープされていない空白をパターンから除外し、シャープ記号 (#) の後ろのコメントを有効にします。|

### Zero-width positive lookahead assertions (Non-Capturing)

```regexp
(?=subexpressions)
```

先読み（lookahead）は、指定した subexpressions が後ろに続くパターンを検索します。
```X(?=Y)```で記述した場合、Y が続く場合の X にマッチします。
通常、ゼロ幅の肯定先読みアサーションは正規表現パターンの末尾にあります。

**Zero-width** は「文字列では無く、文字列内の任意の位置とマッチする正規表現」のことらしく
```^```, ```$``` ```¥b```(単語境界)などの仲間のことです。
バックトラッキングしない仕様を利用して使用することが多いようです。

### Zero-width negative lookahead assertions (Non-Capturing)

```regexp
(?!subexpressions)
```

否定(negative)なので、```X(?Y!)```で記述した場合、Y が続かない場合の X にマッチします。
通常、ゼロ幅の否定先読みアサーションは正規表現の先頭または末尾で使用されます。

### Zero-width positive lookbehind assertions (Non-Capturing)

```regexp
(?<=subexpressions)
```

後読み（lookbehind）なので、手前を探します。
```(?<=Y)X``` は X の前に Y がある場合にのみマッチします。
通常、ゼロ幅の肯定後読みアサーションは正規表現の先頭で使用されます。

### Zero-width negative lookbehind assertions (Non-Capturing)

```regexp
(?<!subexpressions)
```

後読み否定（lookbehind）となるので、
```(?<!Y)X``` は X の前に Y がない場合にのみマッチします。
通常、ゼロ幅の否定後読みアサーションは正規表現の先頭で使用されます。

### Atomic groups (Non-Capturing)

```regexp
(?>subexpressions)
```

アトミック グループを表します (他の正規表現エンジンでは、非バックトラッキング部分式、アトミック部分式、または 1 回のみの部分式として知られています)。

## References

- [Regular Expression Language - Quick Reference - .NET | Microsoft Learn](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/regular-expression-language-quick-reference)
- [Backtracking in .NET regular expressions - .NET | Microsoft Learn](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/backtracking-in-regular-expressions)
- [Grouping Constructs in Regular Expressions - .NET | Microsoft Learn](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/grouping-constructs-in-regular-expressions)
