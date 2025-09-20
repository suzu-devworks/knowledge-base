# Regular Expressions: Backtracking and Grouping

## Table of Contents <!-- omit in toc -->

- [Backtracking](#backtracking)
- [Grouping](#grouping)
  - [Matched Subexpressions (Capturing)](#matched-subexpressions-capturing)
  - [Named Matched Subexpressions (Capturing)](#named-matched-subexpressions-capturing)
  - [Balancing Group Definitions (Capturing)](#balancing-group-definitions-capturing)
  - [Non-Capturing Groups](#non-capturing-groups)
  - [Group Options](#group-options)
  - [Zero-width Positive Lookahead Assertions](#zero-width-positive-lookahead-assertions)
  - [Zero-width Negative Lookahead Assertions](#zero-width-negative-lookahead-assertions)
  - [Zero-width Positive Lookbehind Assertions](#zero-width-positive-lookbehind-assertions)
  - [Zero-width Negative Lookbehind Assertions](#zero-width-negative-lookbehind-assertions)
  - [Atomic Groups](#atomic-groups)
- [References](#references)

## Backtracking

Backtracking occurs in regular expressions when the pattern contains optional quantifiers (`*`, `+`, `{n,m}`) or alternations (`|`, `(?(name)yes|no)`).  
It is a core feature that enables powerful and flexible pattern matching, but can also significantly affect the performance of the regex engine.

## Grouping

### Matched Subexpressions (Capturing)

```regexp
(subexpressions)
```

Parentheses are used for capturing groups.  
Groups are numbered from left to right, starting at 1.  
Group 0 refers to the entire match.  
If group names are duplicated, the value of the `Group` object is determined by the last successful capture in the input string.

### Named Matched Subexpressions (Capturing)

```regexp
(?<name>subexpressions)
(?'name'subexpressions)
```

Assigns a name to the capturing group.  
You can reference the matched subexpression within the same regex using `\k<name>`.

<!-- spell-checker: words subexpression -->

### Balancing Group Definitions (Capturing)

```regexp
(?<name1-name2>subexpressions)
(?'name1-name2'subexpressions)
```

Defines a balancing group, mainly used for matching nested constructs.

### Non-Capturing Groups

```regexp
(?:subexpressions)
```

Groups subexpressions without capturing them.

### Group Options

```regexp
(?imnsx-imnsx:subexpressions)
```

<!-- spell-checker: words imnsx -->

Set options for a specific group:

| Option | Name                | Description                                                                 |
|--------|---------------------|-----------------------------------------------------------------------------|
| i      | IgnoreCase          | Case-insensitive matching.                                                  |
| m      | Multiline           | `^` and `$` match the start/end of each line, not just the whole string.    |
| s      | Singleline          | `.` matches any character, including `\n`.                                  |
| n      | ExplicitCapture     | Only named groups are captured.                                             |
| x      | IgnorePatternWhitespace | Ignores unescaped whitespace and enables comments after `#`.           |

### Zero-width Positive Lookahead Assertions

```regexp
(?=subexpressions)
```

Matches a position where the subexpression follows.  
For example, `X(?=Y)` matches `X` only if it is followed by `Y`.

**Zero-width** means the assertion matches a position, not actual characters (similar to `^`, `$`, `\b`, etc.).

### Zero-width Negative Lookahead Assertions

```regexp
(?!subexpressions)
```

Matches a position where the subexpression does **not** follow.  
For example, `X(?!Y)` matches `X` only if it is **not** followed by `Y`.

### Zero-width Positive Lookbehind Assertions

```regexp
(?<=subexpressions)
```

Matches a position where the subexpression precedes.  
For example, `(?<=Y)X` matches `X` only if it is preceded by `Y`.

### Zero-width Negative Lookbehind Assertions

```regexp
(?<!subexpressions)
```

Matches a position where the subexpression does **not** precede.  
For example, `(?<!Y)X` matches `X` only if it is **not** preceded by `Y`.

### Atomic Groups

```regexp
(?>subexpressions)
```

Represents an atomic group (also known as non-backtracking or once-only subexpression).  
Once an atomic group matches, the regex engine will not backtrack into it.

## References

- [Regular Expression Language - Quick Reference - .NET | Microsoft Learn](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/regular-expression-language-quick-reference)
- [Backtracking in .NET regular expressions - .NET | Microsoft Learn](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/backtracking-in-regular-expressions)
- [Grouping Constructs in Regular Expressions - .NET | Microsoft Learn](https://docs.microsoft.com/ja-jp/dotnet/standard/base-types/grouping-constructs-in-regular-expressions)
