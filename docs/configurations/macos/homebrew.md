# Homebrew

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Environments](#environments)
- [Installation](#installation)
  - [Require dependencies](#require-dependencies)
  - [Homebrew Install](#homebrew-install)
  - [Homebrew Update / Upgrade](#homebrew-update--upgrade)
  - [UnInstall](#uninstall)
- [Homebrew　cask](#homebrewcask)
- [I have installed things like this before](#i-have-installed-things-like-this-before)
- [References](#references)

## Overview

Homebrew（ホームブルー）は、macOSオペレーティングシステム（およびLinux）上のパッケージ管理システムのひとつである。同じくmacOSのためのMacPortsやFinkと同様の目的と機能を備え、利用が広がりつつある。
LinuxのDebianのAPTに似た使用感である。Max Howellによって開発された。

- [🍺 Homebrew](https://brew.sh/ja/)

「ホームブリュー」と読む人もいるが、「ホームブルー」のほうが正式な発音により近い。
ブルワリーの「ぶるー」、自家醸造ビールの意味。

## Environments

- 2017-08-14: Homebrew 1.3.1 on OSX 10.11
- 2020-01-27: Homebrew 2.2.5 on macOS 10.15.2
- 2020-05-07: Homebrew 2.2.15 on macOS 10.15.4
- 2026-11-06: Homebrew 7.0.12 on macOS 26.6.2

## Installation

### Require dependencies

「 Command Line developer Tools 」を必要とします。

```zsh
% xcode-select --install

xcode-select: note: install requested for command line developer tools
```

### Homebrew Install

公式サイトのTopに書かれたコマンドでインストール

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

以前は ruby でインストールしていた・・・やめたんだね。

```text
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Homebrew Update / Upgrade

update は Homebrew のアップデートのみ行う。

```zsh
% brew update
```

upgrade では Homebrew のアップデートに加えて Homebrew でインストールしたパッケージのアップデートも行う。

```zsh
% brew upgrade
```

### UnInstall

ここを見ろという指示が公式サイトにあるの。

- [Uninstall Homebrew](https://github.com/homebrew/install#uninstall-homebrew)

```console
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

## Homebrew　cask

GUI用のHomebrewです。
「きゃすく」は「樽」とか「桶」の意味。

Homebrew 2.7で仕様変更があり、`brew cask` コマンドは `--cask` オプションに変更された。

わかりやすいところで、Firefoxをインストールする例を示します。

```zsh
% brew install --cask firefox
```

```zsh
% brew list --cask           

firefox xquartz
```

## I have installed things like this before

```zsh
% brew install ca-certificates 
% brew install ffmpeg 
% brew install volta
% brew install tmux
% brew install openssl@3
```

<!-- spell-checker: disable -->
```zsh
% brew list          

==> Formulae
ca-certificates jemalloc        libvmaf         ncurses         opus            svt-av1         volta           xz
dav1d           lame            libvpx          openssl@3       sdl2-compat     tmux            x264
ffmpeg          libevent        mpg123          openssl@4       sdl3            utf8proc        x265

==> Casks
firefox xquartz
```
<!-- spell-checker: enable -->

## References

- [Homebrew - GitHub](https://github.com/Homebrew)
