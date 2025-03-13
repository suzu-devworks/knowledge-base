# FFmpeg - Media Convertor

FFmpeg は動画と音声を変換することのできるUNIX系OS生まれのフリーソフトウェアであり、下記のライブラリなどを含む。ライセンスはコンパイル時のオプションにより、LGPLかGPLとなる。

## Reference

* [FFmpeg.org](http://ffmpeg.org/index.html)

## Overview

### Libraries

| Library     | Description                     |
| :---------- | :------------------------------ |
| libavcodec  | 動画/音声のコーデックライブラリ |
| libavformat | 動画/音声のコンテナライブラリ   |
| libswscale  | 色空間・サイズ変換ライブラリ    |
| libavfilter | 動画のフィルタリングライブラリ  |

### Decoders

| type          | decoder                                                           |
| :------------ | :---------------------------------------------------------------- |
| .avi          | MPEG-1 Layer 3 (MP3) デコーダー Microsoft-4 4.2 デコーダー        |
| .wmv          | Windows Media Video 9 デコーダー Windows Media Audio 8 デコーダー |
| .flv          | On2 VP6/Flash デコーダー MPEG-1 Layer 3 (MP3) デコーダー          |
| .mp4          | H.264 デコーダー MPEG-4 AAC デコーダ                              |
| .VOR (DVD-VR) | MPEG-2 Video デコーダー DVD AC-3 (ATSC A/52) デコーダー           |

## Installation

### Install from RPMFusion

RPMFusion の ffmpeg は、ライセンスの問題上 faac 対応ではないので aac へ変換できないらしい。  
つまりmp4を作成できない。・・・そのうちやろう。（[参考](http://www.hyde-tech.com/~hyde/fedora_12_installation_notes.html#ffmpeg)）

```shell
yum --enablerepo=rpmfusion-free,rpmfusion-free-update install ffmpeg
```

## Usage and command line options

```shell
ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}... 
```

| Category   | Option   | Description                                                                                      |
| :--------- | :------- | :----------------------------------------------------------------------------------------------- |
| \<common\> | \-i      | 入力ファイル名を指定する。                                                                       |
|            | \-y      | 出力するファイル名と同じ名前のファイルが出力先にある場合に上書する。                             |
| \<video\>  | \-b      | 動画部分のビットレートを、Kbit/s単位で設定（初期設定：200Kbit/s)                                 |
|            | \-r      | フレームレートの設定(初期設定:25)                                                                |
|            | \-s      | 動画のサイズを横×縦で設定                                                                        |
|            | \-aspect | アスペクト比の設定                                                                               |
|            | \-vcodec | ビデオコーデックを設定 設定しない場合は入力ファイルと同じコーデックを使用する。                  |
| \<audio\>  | \-ab     | ビットレートを設定する。mp3(MPEG-1 Audio Layer-3)の場合32k〜320k                                 |
|            | \-ar     | サンプリング周波数を設定する。(規定値48000)                                                      |
|            | \-ac     | 音声のチャンネル数を設定する。                                                                   |
|            | \-vol    | 通常の音量を256として音量を設定する。(2倍の音量にしたい時は512を指定する。)                      |
|            | \-acodec | 音声コーデックを設定する。設定しない場合は動画同様入力されたファイルと同じコーデックを使用する。 |

| \# 動画ファイルを mp3 へ変換する  $ ffmpeg \-i src.flv \-ab 128k out.mp3 \# mp4も同じ $ ffmpeg \-i in-file.mp4 \-ab 128k out-file.mp3 \# 動画ではないがaacも同じ (faad2-libsが必要) $ ffmpeg \-i in-file.m4a \-ab 128k out-file.mp3 |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
