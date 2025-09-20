# XML tags for C# documentation comments

- [XML tags for C# documentation comments](#xml-tags-for-c-documentation-comments)
  - [Microsoft words](#microsoft-words)
  - [Tags](#tags)
    - [see vs seealso](#see-vs-seealso)
  - [References](#references)

## Microsoft words

- class comment
  - Provides a ...
    - ...を提供する。
  - Supports ...
    - ...をサポートする。
  - Represents a ...
    - ...を表す。
  - Indicates that ...
    - ...を示す。
  - Indicates whether ...
    - ...かどうかを示す。
  - Extension methods for `<see cref="..." />`
    - Extensionsのパターン

- methods comment
  - Gets a value ...
  - Returns a value ...

- return comment
  - (bool) `<c>true</c>` if ...; otherwise,  `<c>false</c>`.

## Tags

### see vs seealso

- `<see >`
  - テキスト内のリンクに使用
- `<seealso >`
  - 「関連項目」セクションに表示するテキストに使用

## References

- [Recommended XML tags for C# documentation comments](https://learn.microsoft.com/ja-jp/dotnet/csharp/language-reference/xmldoc/recommended-tags)
