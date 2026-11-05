# Windows Batch Script Examples

## Table of Contents <!-- omit in toc -->

- [Interactive User Input](#interactive-user-input)
- [Dynamic Timestamped Log Filenames](#dynamic-timestamped-log-filenames)
- [Check If Folder Is Empty](#check-if-folder-is-empty)
- [Keep N Latest Files (Backup Generation Management)](#keep-n-latest-files-backup-generation-management)
- [Bulk Rename Files](#bulk-rename-files)

## Interactive User Input

`set /p` と `if /i` を組み合わせてインタラクティブな入力受け取り処理を構築します。

`interactive-input.cmd`:

```bat
@echo off
:TYPEIN
set /p IN=メッセージ[y/n]?: 
if /i "%IN%"=="y" goto :YES
if /i "%IN%"=="n" goto :NO
goto :TYPEIN

:YES
echo YESで回答！！
goto :EOF

:NO
echo NOで回答！！
goto :EOF
```

## Dynamic Timestamped Log Filenames

環境変数の部分文字列抽出を利用して、`YYYYMMDDHHMMSS` 形式のログファイル名を動的に生成します。

`timestamped-log.cmd`:

```bat
@echo off
set _DATE=%DATE:~0,4%%DATE:~5,2%%DATE:~8,2%
set _TIME=%TIME:~0,2%%TIME:~3,2%%TIME:~6,2%
set TIMESTAMP=%_DATE%%_TIME: =0%
set LOGFILENAME=%TIMESTAMP%TEST.log
echo ファイル名: %LOGFILENAME%
```

## Check If Folder Is Empty

指定フォルダ内にファイルが存在するかどうかを `dir /b` の結果で判定します。

`check-empty-folder.cmd`:

```bat
@echo off
call :IS_EMPTY_FOLDER "%~f1"
if errorlevel 2 goto :EMPTY
if errorlevel 1 goto :ABORT
if errorlevel 0 goto :DONE
goto :ABORT

:IS_EMPTY_FOLDER
if "%~1" == "" exit /b 1
echo PATH=%1
for /f "delims=" %%a in ('dir /b %1') do (
    if "%%a" neq "" exit /b 0
)
exit /b 2

:ABORT
echo 異常終了しました
goto :EOF

:EMPTY
echo フォルダは空です。
goto :EOF

:DONE
echo フォルダは空ではありません
goto :EOF
```

## Keep N Latest Files (Backup Generation Management)

作成日時の新しいファイルを指定個数（例: 3個）残し、古いファイルを自動削除するバックアップ世代管理のパターンです。

`keep-latest-files.cmd`:

```bat
@echo off
set GENERATION=3
for /f "delims= skip=%GENERATION%" %%f in ('dir /b /s /O-D') do (
    echo 削除対象 => "%%f"
    del /q "%%f"
)
```

## Bulk Rename Files

特定の文字を含むファイル名を一括で置換・リネームする例です。ループ内での文字列置換をサブルーチン呼び出しで実現します。

`bulk-rename.cmd`:

```bat
@echo off
for /f %%f in ('dir /b') do call :rename_sub %%f
exit /b

:rename_sub
set FILENAME=%1
set NEWNAME=%FILENAME:xxx=yyy%
if not "%FILENAME%"=="%NEWNAME%" (
    rename %FILENAME% %NEWNAME%
)
exit /b
```
