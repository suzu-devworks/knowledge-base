# Git 設定

## Initial Setup

とりあえずは、ユーザー名とEMailをグローバル設定へ設定する。

```shell
git config --global user.name "XXXXXX XXXXXX"
git config --global user.email "XXXX@kogenet.local"
```

## Custom favorite setup

```shell
# merge 時に --no-ff をデフォルトにする。
git config --global --add merge.ff false

# pull -> merge 時に --ff only にする。
git config --global --add pull.ff only

# push 先は省略しない
git config --global --add push.default nothing

# 日本語ファイル名の文字化け対応
git config --global core.quotepath false

```

## Windows の改行コード自動変更を無効化

```shell
git config --global core.autocrlf input
```

実は core.autocrlf false の場合のほうが問題があるようです。

## 認証をキャッシュする

Gitlabなどのパスワード認証があるリポジトリで毎回認証を入力するのがめんどくさいので。

```shell
# old
git config --global credential.helper "cache --timeout 900"

# only macos
git config --global credential.helper osxkeychain

# only windows
git config --global credential.helper wincred

```

timeout のデフォルトは 900 (15分)らしいっす。

## chmodのパーミッション変更を無視する

```shell
git config --global core.filemode false
```

## 内部に Proxy がおる

通常は http_proxy 、 https_proxy 、および all_proxy 環境変数を使用してコンフィグレーションされたHTTPプロキシをオーバーライドするらしいです。リモート単位でも設定できるらしい。

```shell
# 構文は [protocol：//] [user [：password] @] proxyhost [：port]
git config --global http.proxy http://proxy.example.com:8080
git config --global https.proxy http://proxy.example.com:8080

# remote 単位 
git config --global remote.<name>.proxy http://proxy.example.com:8080
```

公式のマニュアルに https.proxyって無いんだよナー。いらんのかなー。

## 自己証明書を許容する

オレオレ証明書を使っているところの対応

```shell
git config --global http.sslVerify false
```
