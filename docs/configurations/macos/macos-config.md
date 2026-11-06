# macOS configuration

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Basic settings](#basic-settings)
  - [Computer name](#computer-name)
  - [User Display Name](#user-display-name)
  - [Dock](#dock)
  - [Keyboard](#keyboard)
  - [Mouse](#mouse)
  - [Lock screen](#lock-screen)
  - [Finder display order and arrangement](#finder-display-order-and-arrangement)
- [Application Configuration](#application-configuration)
  - [Safari Development menu](#safari-development-menu)
  - [Google Chrome](#google-chrome)
  - [ESET Cyber Security Pro](#eset-cyber-security-pro)
  - [Canon Utility](#canon-utility)
  - [Office 365](#office-365)
  - [KeePass XC](#keepass-xc)
  - [VSCode](#vscode)
  - [Docker Desktop for macOS](#docker-desktop-for-macos)
  - [Github Desktop](#github-desktop)
  - [Command Line Tools (CLT) for Xcode](#command-line-tools-clt-for-xcode)
  - [Homebrew](#homebrew)
  - [Git for macOS](#git-for-macos)
- [Tips](#tips)
  - [Do not create a ".DS\_Store" file in a network folder](#do-not-create-a-ds_store-file-in-a-network-folder)

## Overview

Configuration macOS Ventura (13.1) for Apple silicon

## Environments

- 2015-09-30: macOS El Capitan 10.11
- 2020-01-25: macOS Catalina 10.15.2
- 2023-06-01: macOS Ventura 13.1
- 2026-11-06: macOS Tahoe 26.6.2

## Basic settings

### Computer name

インストールプロセスにコンピュータ名の変更がないので真っ先に変更する

- 🍎 > システム設定 > 一般 > 共有 > ローカルホスト名[編集]

### User Display Name

- 🍎 > システム設定 > ユーザとグループ
  - ユーザーの一覧で、`control` を押しながら変更するユーザーを選択して表示する「詳細オプション」を押す
  - フルネームを変更すると表示名が変わります

### Dock

大きすぎるので小さくする

- 🍎 > システム設定 > デスクトップと Dock > Dock サイズ

### Keyboard

英数字の連続入力（長押し）が効かない (15.1)

```shell
defaults write -g ApplePressAndHoldEnabled -bool false
```

実行後再起動すること

~~うざいのでライブ変換はOFF~~

- ~~システム環境設定 > キーボード > テキスト入力 入力ソース 編集 > 日本語 > ライブ変換~~

### Mouse

気持ち悪いのでOFF

- 🍎 > システム設定 > マウス > ナチュラルスクロール

### Lock screen

- 🍎 > システム設定 > ロック画面
  - 使用していない場合にはスクリーンセーバを開始： 5分後
  - バッテリー駆動時に使用していない場合はディスプレイをオフ： 5分後
  - 電源アダプター接続時に使用していない場合はディスプレイをオフ： 5分後
  - スクリーンセイバー開始後またはディスプレイがオフになったあとにパスワードを要求: 5秒後

### Finder display order and arrangement

- Finder > 表示 > 表示オプションを表示
  - グループ分け： 種類
  - 表示順序：　名前

## Application Configuration

### Safari Development menu

- Safari > 設定 > 詳細
  - 「Webデベロッパーメニューバ用のメニューを表示」をON

### Google Chrome

1. ダウンロードされないので直接 URL を叩く
   - <https://dl.google.com/chrome/mac/stable/GGRO/googlechrome.dmg>
2. インストール直後は「更新できない」とか「古いバージョン」とか言われるが、落ち着くと最新になる
3. Google アカウントで同期する

### ESET Cyber Security Pro

1. CLUB ESET でログイン
   - <https://canon-its.jp/club-eset/>
2. ダウンロードしてインストール
3. 拡張機能がブロック云々言ってくるので設定する
   - システム環境設定 > プライバシーとセキュリティ > 一部のシステムソフトウェアでは、・・・で「詳細」 > 「ESET ...」を３つ許可する
4. 一部のフォルダーにアクセスできませんうんぬん言ってくるので設定する
   - システム環境設定 > プライバシーとセキュリティ > フルディスクアクセス > 「ESET ...」を 2 つ許可する
5. アクティベート
6. 再起動
7. モジュールやパターンを更新

### Canon Utility

1. Canon Inkjet Smart Connect をダウンロードして起動すれば全部やってくれそう。
   - <https://canon.jp/support/software/os/select?pr=4807&os=165>

### Office 365

1. Microsoft アカウントからダウンロードしてインストール
   - <https://account.microsoft.com/account>
2. One Drive も一緒にインストールされる（と思う）

### KeePass XC

1. ネットからダウンロードしてインストール
   - <https://keepassxc.org/>

### VSCode

1. ネットからダウンロードしてインストール
   - <https://code.visualstudio.com/download>

### Docker Desktop for macOS

1. ネットからダウンロードしてインストール
   - <https://docs.docker.jp/desktop/mac/apple-silicon.html>

### Github Desktop

1. ネットからダウンロードしてインストール
   - <https://desktop.github.com/>

### Command Line Tools (CLT) for Xcode

```shell
xcode-select --install
```

### Homebrew

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

after install

```shell
echo '# Set PATH, MANPATH, etc., for Homebrew.' >> /Users/akira/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/akira/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### Git for macOS

```shell
brew install git
```

## Tips

### Do not create a ".DS_Store" file in a network folder

```zsh
 defaults write com.apple.desktopservices DSDontWriteNetworkStores true
```
