# Streaming Protocol

## Table of Contents <!-- omit in toc -->

- [HLS (HTTP Live Streaming)](#hls-http-live-streaming)
- [MPEG-DASH (MPEG - Dynamic Adaptive Streaming over HTTP)](#mpeg-dash-mpeg---dynamic-adaptive-streaming-over-http)
- [WebRTC　(Web Real-Time Communication)](#webrtcweb-real-time-communication)
  - [関連技術](#関連技術)
- [SRT](#srt)
- [RTMP](#rtmp)
- [RSTP](#rstp)

<!-- spell-checker: words RTMP -->
<!-- spell-checker: words RSTP -->

## HLS (HTTP Live Streaming)

Apple が iPhone から Flash を削除する取り組みの一環として 2009 年にリリースしました。

IETF から 2017 年 8 月に、プロトコルのバージョン 7 を説明する RFC 8216 が公開されました。

HLS の特徴として以下のようなポイントが挙げられます。

- HTTP アダプティブビットレートストリーミング(ネットワークの状態が変化したときに、ストリームの途中で動画の画質を調整する機能)をサポートしている
- VOD （オンデマンド配信）とライブ配信の双方に対応している
- HTTPS による暗号化とユーザー認証に対応している
- 専用の設備が不要であり、Apache などの Web サーバーでも配信可能

HLS はインデックスファイルと、セグメントファイルとに分かれて構成されているのが特徴。

|                      |                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------ |
| インデックスファイル | m3u8 プレイリストと呼ばれるセグメントファイルの場所や再生時間、再生順序などを定義したメタデータです。  |
| セグメントファイル   | ts ファイルと呼ばれており、MPEG2 Transport Stream 形式で細かく分割された複数の動画データファイルです。 |

## MPEG-DASH (MPEG - Dynamic Adaptive Streaming over HTTP)

HLS 標準の代替として、Moving Pictures Expert Group (MPEG) によって開発された最新のストリーミング プロトコルの 1 つです。

2012 年 4 月に ISO 国際標準規格 (ISO/IEC 23001-6) としてリリースしています。

HLS と DASH の主な違いは。

- MPEG-DASHでは、任意のエンコード標準を使用できます。一方、HLSではH.264またはH.265の使用が必須です。

- HLSは、Appleデバイスでサポートされている唯一の形式です。iPhone、MacBook、およびその他のApple製品は、MPEG-DASHで配信された動画を再生できません。

- 現在、HLSのデフォルトの長さは6秒ですが、デフォルトを調節することも可能です。MPEG-DASHセグメントの長さは通常 2 ～ 10 秒ですが、最適な長さは 2 ～ 4 秒です。

## WebRTC　(Web Real-Time Communication)

ウェブブラウザやモバイルアプリケーションにシンプルなAPI経由でリアルタイム通信（RTC）を提供する自由かつオープンソースのプロジェクトです。直接ピアツーピア通信を可能にすることで、Web ページ内でのオーディオおよびビデオ通信が可能になり、プラグインのインストールやネイティブ アプリのダウンロードが不要になります。

Apple、Google、Microsoft、Mozilla、Operaによってサポートされており、
仕様は、 World Wide Web Consortium (W3C) およびInternet Engineering Task Force (IETF)によって公開されています。

- [RFC 8825 - Overview: Real-Time Protocols for Browser-Based Applications](https://tex2e.github.io/rfc-translater/html/rfc8825.html)
- [RFC 8830 - WebRTC MediaStream Identification in the Session Description Protocol](https://tex2e.github.io/rfc-translater/html/rfc8830.html)
- [RFC 8834 - Media Transport and Use of RTP in WebRTC](https://tex2e.github.io/rfc-translater/html/rfc8834.html)

### 関連技術

- **ICE ( Interactive Connectivity Establishment )**  
WebRTC自体には、シグナリング（ピア同士の検索と接続確立のためのプロセス）のためのAPIは含まれていません。代わりに、アプリケーションでは（ICE)を用います。なお、ICEのプロセスではシグナリングやNATとラバーサルのためにサーバーを介した通信が必要な場合もあります。
プライベート環境であれば　HTTPベースの WebAPI や Chrome　の各拡張機能があります。  
[RFC 8445](https://tex2e.github.io/rfc-translater/html/rfc8445.html) で標準化されています。

- **STUN (Session Traversal Utilities for NATs)**  
NAT traversal (NAT通過)の方法の１つとして使われる標準化された通信プロトコルです。STUNサーバーは外部ネットワークから見た際の自身のIPアドレスを教えてくれます。そのアドレスと自身のPCのアドレスを比較してNAT越えが必要かを判断します。  
[RFC 8489](https://tex2e.github.io/rfc-translater/html/rfc8489.html) で標準化されています。

- **TURN (Traversal Using Relay around NAT)**  
NAT や Firewall を超えて通過することを補助するための通信プロトコルです。  
[RFC 8656](https://tex2e.github.io/rfc-translater/html/rfc8656.html) で標準化されています。

- **SDP (Session Description Protocol)**
ストリーミングメディアの初期化パラメータを記述する形式。  
[RFC 8866](https://tex2e.github.io/rfc-translater/html/rfc8866.html) で標準化されています。

- **P2P**
サーバーを介さず、端末同士が直接通信すること。通信相手を探すシグナリング用のICEサーバーやNATを超えるためのTRUNサーバーは別途必要になります。

- **SFU (Selective Forwarding Unit)**  
P2Pではなくサーバー経由で行う技術です。
配信者が直接相手に通信するのではなく、サーバーを間に挟むことで、動画の視聴者増加などによる、端末への負荷軽減を可能にします。これにより、リアルタイムで多くの視聴者に音声や動画を配信できます。

## SRT

## RTMP

## RSTP
