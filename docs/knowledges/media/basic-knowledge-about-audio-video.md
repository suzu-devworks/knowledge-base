# Audio/Video まわりの基礎知識

- [Audio/Video まわりの基礎知識](#audiovideo-まわりの基礎知識)
  - [コンテナフォーマット （container format）](#コンテナフォーマット-container-format)
  - [コーデック (Codec)](#コーデック-codec)

## コンテナフォーマット （container format）

> デジタル分野におけるコンテナフォーマットは複数の個別データを単一ファイル（コンテナ）へ格納する規格（フォーマット）である。
> [wikipedia ...](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%83%95%E3%82%A9%E3%83%BC%E3%83%9E%E3%83%83%E3%83%88)

*Video*:

|          | extension                          |                                   | developed by        |
| -------- | ---------------------------------- | --------------------------------- | ------------------- |
| AVI      | `.avi`                             | Audio Video Interleave (AVI)      | Micorosft           |
| MOV      | `.mov`,`.qt`                       | QuickTime file format (MOV)       | Apple               |
| MPEG2-PS | `.mpg`,`.m2p`,`.m2ps`              | MPEG Program Stream               | MPEG                |
| MPEG2-TS | `.ts`,`.mts`,`.m2t`,`.m2ts`        | MPEG transport stream             | MPEG                |
| MP4      | `.mp4`,`.m4a`                      | MPEG-4 Part 14 - ISO/IEC 14496-14 | ISO/IEC             |
| Ogg      | `.ogv`,`.oga`,`.ogx`,`.ogg`,`.spx` | Ogg - RFC 3533                    | Xiph.Org Foundation |
| ASF      | `.asf`,`.wma`,`.wmv`               | Advanced Systems Format (ASF)     | Microsoft           |
| FLV      | `.flv`,`.f4v`,`.f4p`,`.f4a`,`.f4b` | Flash Video                       | Adobe               |
| WebM     | `.webm`                            | WebM                              | Google              |

*Audio*:

|     | extension           |                                 | developed by            |
| --- | ------------------- | ------------------------------- | ----------------------- |
| WAV | `.wav`              | RIFF waveform Audio Format(WAV) | Microsoft, IBM          |
| MP3 | `.mp3`              | MPEG-1 Audio Layer-3            | Fraunhofer-Gesellschaft |
| AAC | `.aac`,`.m4a`,`mp4` | Advanced Audio Coding           | ISO/IEC                 |

## コーデック (Codec)

> コーデック (Codec) は、符号化方式を使ってデータのエンコード（符号化）とデコード（復号）を双方向にできる装置やソフトウェアなどのこと。 また、そのためのアルゴリズムを指す用語としても使われている。
> [wikipedia ...](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%BC%E3%83%87%E3%83%83%E3%82%AF)

*ITC-T*:

| codec            | fourcc     | description                                                                                   |
| ---------------- | ---------- | --------------------------------------------------------------------------------------------- |
| H.264/MPEG-4 AVC | H264, AVC1 | MPEG-4 Part 10、ブロック指向の動き補償コーディングに基づくビデオ圧縮規格。                    |
| H.265/HEVC       | HVC1, HEV1 | 広く使用されているAVCの後継としてMPEG-Hプロジェクトの一部として設計されたビデオ圧縮規格です。 |
|                  |            |                                                                                               |

*MPEG*:

| codec  | fourcc | description                                                                                    |
| ------ | ------ | ---------------------------------------------------------------------------------------------- |
| MPEG-2 |        | 放送やHDTVなどを想定した規格であり、デジタル放送や記録メディアなど様々な用途に利用されている。 |
| MPEG-4 |        | 動画・音声全般を扱う多様なマルチメディア符号化フォーマットを規定している。                     |
|        |        |                                                                                                |

*Google/On2 Technologies*:

| codec | fourcc | description                                                                                    |
| ----- | ------ | ---------------------------------------------------------------------------------------------- |
| VP8   | VP80   | On2 Technologiesによってリリースされたオープンでロイヤリティフリーのビデオ圧縮形式。 - RFC6386 |
| VP9   | VP90   | Googleによって開発されたオープンでロイヤリティフリーな動画圧縮コーデック、VP8の後継。          |
|       |        |                                                                                                |

*Alliance for Open Media*:

| codec | fourcc | description                            |
| ----- | ------ | -------------------------------------- |
| AV1   | AV01   | AOMedia Video 1、Google VP9/VP10の後継 |
|       |        |                                        |
