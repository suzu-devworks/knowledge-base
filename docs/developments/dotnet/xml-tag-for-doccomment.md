# XML tags for C# documentation comments

## Table of Contents <!-- omit in toc -->

- [Microsoft Words](#microsoft-words)
- [Tags](#tags)
  - [see vs seealso](#see-vs-seealso)
- [References](#references)

## Microsoft Words

**Class comments:**

- Provides a ...  
  ...を提供する。
- Supports ...  
  ...をサポートする。
- Represents a ...  
  ...を表す。
- Indicates that ...  
  ...を示す。
- Indicates whether ...  
  ...かどうかを示す。
- Extension methods for `<see cref="..." />`  
  Extensionsのパターン

**Method comments:**

- Gets a value ...
- Returns a value ...

**Return comments:**

- (bool) `<c>true</c>` if ...; otherwise, `<c>false</c>`.

## Tags

### see vs seealso

- `<see>`  
  Used for inline links within text.
- `<seealso>`  
  Used for "See Also" section references.

## References

- [Recommended XML tags for C# documentation comments](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/xmldoc/recommended-tags)
