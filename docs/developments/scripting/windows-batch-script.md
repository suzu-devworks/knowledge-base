# Windows Batch Script

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Syntax](#syntax)
  - [Suppress Command Echo](#suppress-command-echo)
  - [Comments](#comments)
  - [Multi-line Commands](#multi-line-commands)
- [Standard I/O](#standard-io)
  - [Redirection](#redirection)
  - [Pipeline](#pipeline)
  - [Null Device](#null-device)
- [Variables](#variables)
  - [Environment Variables](#environment-variables)
    - [Operators for Arithmetics (`set /a`)](#operators-for-arithmetics-set-a)
  - [System Environment Variables](#system-environment-variables)
  - [String Manipulation](#string-manipulation)
  - [Special Variables](#special-variables)
  - [Modifiers for Special Variables](#modifiers-for-special-variables)
  - [Scope Management](#scope-management)
  - [String Literals and Escaping](#string-literals-and-escaping)
- [Control Flow](#control-flow)
  - [Conditional Branching (if ... else ...)](#conditional-branching-if--else-)
    - [Exit Code Check](#exit-code-check)
    - [String Comparison](#string-comparison)
    - [File Existence Check](#file-existence-check)
    - [Extended Comparison Operators](#extended-comparison-operators)
    - [Advanced Conditions](#advanced-conditions)
  - [Looping (for ... in ... do ...)](#looping-for--in--do-)
    - [Range Enumeration (/L)](#range-enumeration-l)
    - [Set Enumeration](#set-enumeration)
    - [File / Directory Enumeration](#file--directory-enumeration)
    - [Parsing File Content / Command Output (/F)](#parsing-file-content--command-output-f)
  - [Jump (goto)](#jump-goto)
- [Subroutines](#subroutines)
  - [Call Subroutine in Same File](#call-subroutine-in-same-file)
  - [Call External Script](#call-external-script)
  - [Asynchronous Execution](#asynchronous-execution)
  - [Exit Code Handling](#exit-code-handling)

## Overview

バッチファイル（Batch File）とは、MS-DOS のOSシェル（COMMAND.COM）で実行するスクリプトが由来で、
Windows のコマンドプロンプト（cmd.exe）で実行したい命令列をテキストファイルに記述したスクリプトである。
正直何と呼べば良いのか迷うところではあるが、一般的には「バッチファイル」や「バッチスクリプト」と呼ばれることが多い。

実行するとシェルプログラムが一行ずつコマンドを評価・実行する。
UNIX 系のシェルスクリプトに相当し、拡張子には `.bat` や `.cmd` が使用される。

## Environments

- 2026-11-05: Windows Batch Script (cmd.exe) on Windows 10 / Windows 11

## Syntax

### Suppress Command Echo

実行コマンドのエコーバック（画面表示）を抑制するには `@echo off` をスクリプトの先頭に記述します。単一のコマンドのみエコーバックを抑えたい場合は先頭に `@` を付与します。

`sample.cmd`:

```bat
@echo off
echo Hello World
```

### Comments

コメント行を作成するには `rem` コマンドを使用するか、コロン 2 つ `::` を記述します。

`sample.cmd`:

```bat
rem これがコメントです
:: これもコメントになります
:JUMP_LABEL
```

### Multi-line Commands

長いコマンドを複数行に渡って記述する場合はキャレット `^` を末尾に付与します。ただし、ダブルクォートで括られた文字列内ではキャレットによる行継続が無効化されるため注意が必要です。

`sample.cmd`:

```bat
echo ^
  "abc" ^
  "def"
```

## Standard I/O

### Redirection

標準出力や標準エラー出力をファイルに保存したり、ファイルから標準入力を読み込んだりするためのリダイレクト記法です。

`sample.cmd`:

```bat
:: 標準出力をファイルへリダイレクト
echo "normal-out" > out.txt

:: 標準エラーをファイルへ追記リダイレクト
error-command 2>> out.txt

:: 標準出力と標準エラーの両方をファイルへ追記リダイレクト
prog1.exe >> out.txt 2>&1

:: 標準入力をファイルからリダイレクト
sort < out.txt

:: ファイルからの入力結果をソートして別ファイルへ出力
sort < out2.txt > out_sort.txt
```

| Handle Name | Number | Description |
| --- | --- | --- |
| STDIN | 0 | 標準入力（通常はキーボード） |
| STDOUT | 1 | 標準出力（通常はプロンプト画面） |
| STDERR | 2 | 標準エラー出力（通常はプロンプト画面） |
| UNDEFINED | 3-9 | アプリケーションごとに定義 |

### Pipeline

あるコマンドの標準出力を、別のコマンドの標準入力へ引き渡します。

`sample.cmd`:

```bat
dir /b | find ".txt"
```

### Null Device

出力を捨てる（画面表示を抑止する）場合は Null デバイス (`NUL`) へリダイレクトします。

`sample.cmd`:

```bat
echo "error-out" > NUL
```

## Variables

### Environment Variables

変数の設定には `set` を使用し、参照時は `%VARIABLE%` のようにパーセント記号で囲みます。算術演算を行う場合は `/a` オプション、ユーザー入力を受け取る場合は `/p` オプションを使用します。

`sample.cmd`:

```bat
set ALPHA_UPPER=ABCDEFGHIJKLMNOPQRSTUVWXYZ
set ALPHA_COUNT=10

:: 算術演算による設定
set /a TOTAL=%ALPHA_COUNT% * 10

:: 画面からの入力受取
set /p INPUT_VAL=値を入力してください:

:: 変数の消去
set ALPHA_UPPER=
```

#### Operators for Arithmetics (`set /a`)

| Priority | Operator | Description |
| --- | --- | --- |
| 1 | ( ) | グループ化 |
| 2 | * / % + - | 算術演算 |
| 3 | << >> | 論理シフト |
| 4 | & | ビット単位の論理積 (AND) |
| 5 | ^ | ビット単位の排他的論理和 (XOR) |
| 6 | \| | ビット単位の論理和 (OR) |
| 7 | = *= /= += -= &= ^= \|= <<= >>= | 代入演算 |
| 8 | , | 式の区切り記号 |

### System Environment Variables

システムによって自動的に定義・更新される主な特殊環境変数です。

| Variable | Description |
| --- | --- |
| %PATH% | 実行可能ファイルを検索するパス |
| %CD% | カレントディレクトリの絶対パス |
| %DATE% | 現在の日付 |
| %TIME% | 現在の時刻 |
| %RANDOM% | 0 から 32767 の範囲のランダムな整数 |
| %ERRORLEVEL% | 直前に実行されたコマンドの終了コード |
| %USERNAME% | ログイン中のユーザー名 |
| %COMPUTERNAME% | コンピュータ名 |

### String Manipulation

環境変数に対して部分文字列の抽出（Substring）や置換（Replace）を行えます。

| Syntax | Description |
| --- | --- |
| %I:~O,L% | 変数 %I% のオフセット O から L 文字分を抽出し返します。O や L に負の値を指定した場合は末尾からの長さになります。 |
| %I:AAA=BBB% | 変数 %I% に含まれる文字列 AAA を BBB に置換します。 |

### Special Variables

バッチスクリプトの実行時引数や `for` 文で利用する変数です。

| Variable | Description |
| --- | --- |
| %0 | 実行中のバッチファイル名 |
| %1, %2, ... %9 | バッチファイルまたはサブルーチンへの第 1〜9 引数 |
| %* | バッチファイルまたはサブルーチンに引き渡されたすべての引数 |
| %I | `for` 文で使用する繰り返し変数（アルファベット 1 文字）。バッチファイル内で記述する場合は `%%I` とします。大文字小文字は区別されます。 |

### Modifiers for Special Variables

引数や `for` 変数（例: `%1`, `%%i`）に対してファイルパスや属性を展開・変換する修飾子です。

| Syntax | Description |
| --- | --- |
| %~I | 引用符 (`"`) を除去します。 |
| %~fI | 完全修飾パス名に展開します。 |
| %~dI | ドライブ文字のみに展開します。 |
| %~pI | パス名のみ（ファイル名なし）に展開します。 |
| %~nI | ファイル名のみ（拡張子なし）に展開します。 |
| %~xI | 拡張子のみに展開します。 |
| %~sI | 短い形式のパス名（8.3形式）に展開します。 |
| %~aI | ファイルの属性に展開します。 |
| %~tI | ファイルの最終更新日時に展開します。 |
| %~zI | ファイルのサイズに展開します。 |
| %~$PATH:I | 環境変数 PATH に指定されたディレクトリを検索し、最初に見つかったファイルの完全修飾パスに展開します。 |
| %~dpI | ドライブ文字とパス名のみに展開します。 |
| %~nxI | ファイル名と拡張子のみに展開します。 |
| %~fsI | 短い形式の完全修飾パス名に展開します。 |
| %~dp$PATH:I | PATH 上から検索し、最初に見つかった場所のドライブ文字とパスに展開します。 |
| %~ftzaI | `dir` コマンドの出力行のように詳細情報をまとめて展開します。 |

### Scope Management

`setlocal` と `endlocal` で挟まれた区間内での変数変更はローカルスコープとなり、外部の環境変数に影響を与えません。

`sample.cmd`:

```bat
set XXX="Hello global"
echo %XXX%

setlocal
set XXX="Hello local"
echo %XXX%
endlocal

echo %XXX%
```

### String Literals and Escaping

特殊文字のエスケープにはキャレット `^` やダブルクォートを使用します。

| Literal | Description |
| --- | --- |
| `^`char | キャレットエスケープ。特定の制御文字（例: `^M` で CR、`^J` で LF）に変換されます。 |
| \<CR\> | キャリッジリターン |
| \<LF\> | ラインフィード |
| `\"` | ダブルクォート |
| `\^` | キャレット記号 |
| `\<` | 左山括弧 |
| `\\` | 円記号 / バックスラッシュ |
| `%%` | パーセント記号 |

## Control Flow

### Conditional Branching (if ... else ...)

条件分岐を行います。複数行の処理を実行する場合は括弧 `()` でブロック化します。

`sample.cmd`:

```bat
if expression (
    command1
    command2
) else (
    command3
)
```

#### Exit Code Check

```bat
if [not] errorlevel number command1 [else command2]
```

#### String Comparison

```bat
if [not] string1==string2 command1 [else command2]
```

#### File Existence Check

```bat
if [not] exist FileName command1 [else command2]
```

#### Extended Comparison Operators

コマンド拡張機能（Windows 2000 以降デフォルトで有効）が適用されている場合に利用可能な演算子です。

`sample.cmd`:

```bat
if [/i] string1 operator string2 command1 [else command2]
```

| Operator | Mark Equivalent | Description |
| --- | --- | --- |
| EQU | == | 等しい |
| NEQ | != | 等しくない |
| LSS | < | より小さい (未満) |
| LEQ | <= | より小さいか等しい (以下) |
| GTR | > | より大きい |
| GEQ | >= | より大きいか等しい (以上) |

| Option | Description |
| --- | --- |
| /i | 大文字と小文字を区別しない (Ignore case) |

#### Advanced Conditions

```bat
:: CMD 拡張機能の内部バージョン確認
if cmdextversion number command1 [else command2]

:: 変数が定義済みかどうかのチェック
if defined variable command1 [else command2]
```

### Looping (for ... in ... do ...)

指定したファイル、数値範囲、文字列、またはコマンド実行結果に対してループ処理を行います。

#### Range Enumeration (/L)

```bat
for /L %%variable in (start, step, end) do command

:: 例: 1 から 5 まで列挙
for /L %%i in (1, 1, 5) do @echo %%i
```

#### Set Enumeration

```bat
:: リテラル文字列の列挙
for %%i in (a, b, c) do @echo %%i

:: 環境変数（区切り文字区切り）の列挙
for %%i in (%PATH%) do @echo %%i
```

#### File / Directory Enumeration

```bat
:: ワイルドカードに一致するファイルを列挙
for %%i in (*.doc) do @echo %%i

:: ディレクトリのみを列挙 (/D)
for /D %%i in ("*work") do @echo %%i

:: ディレクトリツリーを再帰的に検索 (/R)
for /R C:\ %%i in (*.cs) do @echo %%i
```

#### Parsing File Content / Command Output (/F)

ファイルの内容やコマンドの実行結果を解析して繰り返し処理を行います。

```bat
:: ファイルの内容を解析
for /F "tokens=*" %%i in (out.txt) do @echo %%i

:: コマンド実行結果を解析
for /F "usebackq tokens=2,4" %%i in (`dir`) do echo %%j=%%i
```

| ParsingKeywords | Description |
| --- | --- |
| eol=c | 行末・行頭のコメント記号（1文字）を指定し、該当行をスキップします。 |
| skip=n | ファイルの先頭から指定した行数ぶんスキップします。 |
| delims=xxx | トークン分離に使用する区切り文字セットを指定します。 |
| tokens=x,y,m-n | 各行から取得して変数（%%i, %%j...）に割り当てるトークン番号を指定します。末尾に `*` を付けると残りの全文字列が格納されます。 |
| usebackq | コマンド実行に逆引用符 (`` ` ``)、文字列リテラルに単一引用符 (`'`)、ファイル名に二重引用符 (`"`) を使用するオプションです。 |

### Jump (goto)

指定したラベル行へ移動します。スクリプトの末尾へ移動して終了する場合は特殊なラベル `:EOF` を使用します。

`sample.cmd`:

```bat
:JUMP1
goto JUMP2

:JUMP2
goto :EOF
```

## Subroutines

### Call Subroutine in Same File

`call :label` 形式で呼び出すことで、同一ファイル内でサブルーチンを実行できます。呼び出し先からは `exit /b` で復帰します。

`sample.cmd`:

```bat
call :sub abc %*
echo 戻り値: %errorlevel%
goto :EOF

:sub
echo サブルーチン開始
exit /b 99
```

### Call External Script

別ファイルのスクリプトを呼び出す際、単にスクリプト名のみを指定すると制御が移ったまま復帰しません。呼び出し元へ処理を戻したい場合は `call` コマンドまたは `cmd /c` を使用します。

`sample.cmd`:

```bat
:: call を使ったスクリプト呼び出し
echo 別スクリプト開始
call sample_sub.cmd
echo 別スクリプト終了

:: cmd /c を使ったスクリプト呼び出し
cmd /c sample_sub.cmd
```

`sample_sub.cmd`:

```bat
exit /b 1
```

### Asynchronous Execution

`start` コマンドを使用することで、別プロセスを非同期で立ち上げて後続処理を進めることができます。

`sample.cmd`:

```bat
start notepad
echo 実行中...
```

### Exit Code Handling

呼び出したサブルーチンや外部コマンドの終了コードは `%ERRORLEVEL%` で評価します。

`sample.cmd`:

```bat
call sample_sub.bat %*
if errorlevel 1 goto ABORT
if errorlevel 0 goto DONE
goto ABORT

:ABORT
echo 異常終了しました
goto :EOF

:DONE
echo 正常終了しました
goto :EOF
```

`sample_sub.cmd`:

```bat
exit /b 1
```
