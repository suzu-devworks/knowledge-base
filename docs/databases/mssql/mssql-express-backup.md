# SQL Server Express Backup

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Backup Script Overview](#backup-script-overview)
- [Backup Script (db-backup.bat)](#backup-script-db-backupbat)
- [SQL backup Script (db-backup.sql)](#sql-backup-script-db-backupsql)

## Overview

SQL Server Express では SQL Server Agent サービスが利用できない（起動できない）制約があるため、SQL Server 標準の定期ジョブ機能を使用した自動バックアップを実行することができません。
しかし、Windows のタスクスケジューラ等とバッチファイル（`SQLCMD` コマンド）を組み合わせることで、定期的なデータベースバックアップおよび世代管理を実現します。

## Environments

- ???: Microsoft SQL Server 2012 Express (11.0) on Windows Server / Windows PC

## Backup Script Overview

バックアップ処理は、バッチファイル（`db-backup.bat`）から `SQLCMD` ユーティリティを介して SQL スクリプト（`db-backup.sql`）を呼び出すことで実行します。

主な機能・処理の流れは以下の通りです：

1. **日付・時刻の取得とログ初期化**: 実行時のタイムスタンプを取得し、バックアップログファイルを作成します。
2. **データベースバックアップの実行**: `db-backup.sql` を呼び出し、メインDBおよび履歴DBのバックアップファイル（`.bak`）を出力します。
3. **世代管理とファイルの整理**:
   - 直近のバックアップを保持し、古いバックアップファイルをアーカイブフォルダ（`OLD`）へ移動します。
   - 指定した世代数（例: 5世代）を超えた古いバックアップファイルを自動削除します。
4. **関連フォルダのバックアップ（`xcopy`）**: バックアップで指定したフォルダをバックアップ先へ同期コピーします。

## Backup Script (db-backup.bat)

<!-- spell-checker: disable -->
```bat
:: -------------------------------------------------------------------
:: SQLSERVER Backup Script
:: -------------------------------------------------------------------
@ECHO off
SETLOCAL
SET COPYOPT=/S /E /V /R /Y

:: -----------------
:: System Date
:: -----------------
SET _DATE=%DATE:/=%
SET _TIME=%TIME: =0%
SET _TIME=%_TIME:~0,2%%_TIME:~3,2%%_TIME:~6,2%
SET DATETIME=%_DATE%%_TIME%

:: -----------------
:: SQL Server definition
:: -----------------
SET SQLCMD="C:\Program Files\Microsoft SQL Server\110\Tools\Binn\SQLCMD.EXE"
SET SQLSERVER=localhost
SET SQLLOGIN=-U sa -P *****
SET SQLDBNAME=examples_db

:: -----------------
:: Backup source path
:: -----------------
SET SOURCE_DEPS="<replaced here>"

:: -----------------
:: Backup destination path
:: -----------------
FOR /f %%i IN ('cd') DO SET DIR=%%i
SET SQLFILE=%DIR%\db-backup.sql
SET LOGFILE=%DIR%\db-backup-log.txt
SET BACKUP_DBDIR=%DIR%\db
SET BACKUP_DBDIR_OLD=%BACKUP_DBDIR%\OLD
SET BACKUP_DEPSDIR=%DIR%\deps

:: -----------------
:: Clear Log file
:: -----------------
ECHO. > %LOGFILE%
ECHO. >> %LOGFILE%
ECHO --- %DATE% %TIME% --- >> %LOGFILE%

:: -----------------
:: Backup database
:: -----------------
ECHO. >> %LOGFILE%
ECHO DBバックアップ >> %LOGFILE%
ECHO DBバックアップ...
SET DBFILEPATH=%BACKUP_DBDIR%\%SQLDBNAME%_%DATETIME%.bak
ECHO ファイル名=%DBFILEPATH% >> %LOGFILE%

%SQLCMD% -S %SQLSERVER% %SQLLOGIN% -i %SQLFILE% -v Param1="%DBFILEPATH%" -b >> %LOGFILE%
IF ERRORLEVEL 1 GOTO ABORT

:: -----------------
:: Backup File arrangement
:: -----------------
ECHO. >> %LOGFILE%
ECHO DBバックアップの整理 >> %LOGFILE%
ECHO DBバックアップの整理...
for /f "delims= skip=1" %%F in ('dir /B /O-D %BACKUP_DBDIR%\%SQLDBNAME%_??????????????.bak') do (
    ECHO 移動："%BACKUP_DBDIR%\%%F" >> %LOGFILE%
    MOVE "%BACKUP_DBDIR%\%%F" "%BACKUP_DBDIR_OLD%" > NUL
)
for /f "delims= skip=1" %%F in ('dir /B /O-D %BACKUP_DBDIR%\%SQLDBHIST%_??????????????.bak') do (
    ECHO 移動："%BACKUP_DBDIR%\%%F" >> %LOGFILE%
    MOVE "%BACKUP_DBDIR%\%%F" "%BACKUP_DBDIR_OLD%" > NUL
)

SET GENERATION=5
ECHO 世代数=%GENERATION% >> %LOGFILE%
SET /A SKIP=%GENERATION%-1
for /f "delims= skip=%SKIP%" %%F in ('dir /B /O-D %BACKUP_DBDIR_OLD%\%SQLDBNAME%_??????????????.bak') do (
    ECHO 削除："%BACKUP_DBDIR_OLD%\%%F" >> %LOGFILE%
    DEL /Q "%BACKUP_DBDIR_OLD%\%%F" > NUL
)
for /f "delims= skip=%SKIP%" %%F in ('dir /B /O-D %BACKUP_DBDIR_OLD%\%SQLDBHIST%_??????????????.bak') do (
    ECHO 削除："%BACKUP_DBDIR_OLD%\%%F" >> %LOGFILE%
    DEL /Q "%BACKUP_DBDIR_OLD%\%%F" > NUL
)

:: -----------------
:: Copy DEPS Folder
:: -----------------
ECHO. >> %LOGFILE%
ECHO DEPS フォルダのコピー >> %LOGFILE%
ECHO DEPS フォルダのコピー...
RMDIR /S /Q %BACKUP_DEPSDIR%
XCOPY %COPYOPT% %SOURCE_DEPS% %BACKUP_DEPSDIR%\ >> %LOGFILE%
IF ERRORLEVEL 1 GOTO ABORT

:NORMAL
ECHO. >> %LOGFILE%
ECHO --- %DATE% %TIME% --- >> %LOGFILE%
ECHO 完了しました。
GOTO END

:ABORT
COLOR C
ECHO. >> %LOGFILE%
ECHO --- %DATE% %TIME% --- >> %LOGFILE%
ECHO エラー発生！ >> %LOGFILE%
ECHO エラー発生！
PAUSE

:END
ENDLOCAL
EXIT /B 0
```
<!-- spell-checker: enable -->

## SQL backup Script (db-backup.sql)

```sql
USE [master]
GO

-- 引数からバックアップファイルパスを取得してバックアップ実行
DECLARE @filepath nvarchar(256) = N'$(Param1)'
BACKUP DATABASE [CHIBA_EMERGENCY] TO DISK = @filepath 
    WITH INIT, NAME = N'Batch Backup'
GO

-- 定期バックアップ用サンプルスクリプト
DECLARE @now datetime = GETDATE()
DECLARE @dirpath nvarchar(128) = N'D:\Program Files\Microsoft SQL Server\MSSQL11.SQLEXPRESS\MSSQL\Backup\'
DECLARE @filetime nvarchar(128) = REPLACE(REPLACE(REPLACE(CONVERT(varchar(20), @now, 120),'-',''),':',''),' ','_') 
DECLARE @filename nvarchar(128) = N'EXAMPLES_' + @filetime + N'.bak'
DECLARE @filepath nvarchar(128) = @dirpath + @filename

PRINT '実行日=' + CONVERT(varchar(20), @now, 120)
PRINT 'ファイル名=' + @filename

BACKUP DATABASE [EXAMPLES] TO DISK = @filepath WITH INIT, 
    NAME = N'EXAMPLES - Regular Backup'
GO
```
