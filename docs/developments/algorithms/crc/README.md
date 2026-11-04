# Algorithm for CRC

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [How to calculate CRC](#how-to-calculate-crc)
  - [modulo 2](#modulo-2)
  - [Left shift/ Right shift](#left-shift-right-shift)
  - [Initializer](#initializer)
  - [Specify](#specify)
- [Optimize](#optimize)
  - [Omit the most significant bit of the generator polynomial](#omit-the-most-significant-bit-of-the-generator-polynomial)
- [Standard CRC (Polynomial)](#standard-crc-polynomial)
- [CRC-5-USB in USB](#crc-5-usb-in-usb)
- [CRC-16 in USB](#crc-16-in-usb)
- [CRC-15-CAN in CAN2.0](#crc-15-can-in-can20)
- [CRC-16-CCITT in Bluetooth](#crc-16-ccitt-in-bluetooth)
- [CRC-32 in PNG](#crc-32-in-png)
- [Reference](#reference)

## Overview

巡回冗長検査（じゅんかいじょうちょうけんさ、Cyclic Redundancy Check, CRC）は、誤り検出符号の一種で、主にデータ転送などに伴う偶発的な誤りの検出によく使われている。

## How to calculate CRC

CRCは大きなデータ（数値）をある素数で割った時のあまりを送信するデータの末尾につけ、同じ計算を受信側で行うことで誤りを検出する方法であり、$n$ビットのCRCは$n$ビット未満のバースト誤りを検出することができる。

### modulo 2

データを素数で割る場合に問題となるのが減算による桁下がりの問題で、通常の減算の場合数桁のデータを見て初めて割り算ができる。論理回路やコンピュータでこの複雑さは大きなボトルネックになるため modulo 2 という効率の良い演算方法を使用する。

通常の2進数の演算とは異なり、加算による桁上がり（繰り上がり）、減算による桁下がり（繰り下がり）を無視する。

modulo 2 の加算:

$$
\begin{aligned}
&0 \oplus 0 = 0\\
&0 \oplus 1 = 1\\
&1 \oplus 0 = 1\\
&1 \oplus 1 = 0 （桁上がりを無視）
\end{aligned}
$$

modulo 2 の減算:

$$
\begin{aligned}
&0 - 0 = 0\\
&0 - 1 = (0 - (-1)) = (0 \oplus 1) = 1\\
&1 - 0 = 1\\
&1 - 1 = 0
\end{aligned}
$$

modulo 2 の乗算:

$$
\begin{aligned}
&0 \cdot 0 = 0\\
&0 \cdot 1 = 0\\
&1 \cdot 0 = 0\\
&1 \cdot 1 = 1
\end{aligned}
$$

modulo 2の減算については「1と-1は法2に関して合同」が成立する。

$$1 \equiv -1 \pmod{2}$$

結果CRCの計算は以下に収束する。（左シフトの場合）

- MSBが1の場合、1ビット左シフトして生成多項式とXORする
- MSBが0の場合、1ビット左シフトする。

### Left shift/ Right shift

生成多項式の表現は

$$
x^{16} + x^{15} + \dots + x^2 + 1
$$

とかなっているが、次数の小さいものがCRCをFIFOとしてみたときの入り口より。
ただしどちらをCRCのMSBにもってくるかは方式に依存。$+1$がLSBのときは「左送り」、$x^n$がLSBのときは「右送り」。

### Initializer

上図の各レジスタの初期値は全1です。これは0が先行し、それ以外が同一値のビット列において、先行連続0の数に拘らず同一CRC値となるのを防止します。

### Specify

初期値、出力XOR、バイト反転、ビット反転
CRC計算の初期値として、固定ビットパターン（全ビット0 or 1）を使用する実装がある。
これは同期がずれた際にCRCの対象となる部分を明らかにするための実装である。つまり、ある時点でメッセージが受信されるはずだという場合、同期がずれると先頭に0がずっと並んだメッセージを受信したようになる。固定パターンがメッセージの先頭に必ず存在するなら、同期がずれてもメッセージの範囲がすぐにわかる。

## Optimize

### Omit the most significant bit of the generator polynomial

> 生成多項式の最上位ビットを省略

$n$ビットCRCは $(n+1)$ ビットの除数で定義されるもので、最上位ビットは常に1である。すると$n$ビットのレジスタではオーバフローする。最上位ビットは省略して計算しても結果が同じになる。
最上位ビット（MSB）を確認し、最終的にMSBがクリアされ、MSBをもたらすもの、されていた場合に残りの部分から（XOR）生成多項式を引きます。
そして、LSBゼロを残して、1ビット位置だけ左に残りをシフトします。
最後に、LSBの位置に（XOR）次のペイロードビットを養います。

理論上の除算：

```text
11010011101 00000 <--- 入力
100101            <--- 除数 (6 Bits)
------                 ※最上位ビットを揃えてXOR
0100011
>100101
 ------
 000110110
 >>>100101
    ------
    0100111
    >100101
     ------
     000010 0000
     >>>>10 0101
         -- ----
         00 01010
         >0 00000
----------- ----- 
11010011101 01010 <--- 結果 
```

最上位ビットを省略した除算:

```text
1 1010011101 00000 <--- 入力
> 00101            <--- 除数 (5 Bits)
- -----                 ※1ビットcarryしたらXOR
  100011
  >00101
   -----
   00110110
   >>>00101
      -----
      100111
      >00101
       -----
       00010 0000
       >>>>0 0101
           - ----
           0 01010
           > 00000 
- ---------- -----
1 1010011101 01010 <--- 結果
```

## Standard CRC (Polynomial)

通常記載している生成多項式は最上位ビットを省略している。

| Name | Uses / Polynomial | Normal | Reversed | Reversed Reciprocal |
| :--- | :--- | :--- | :--- | :--- |
| CRC-1 | Parity bit | 0x1 | 0x1 | 0x1 |
| CRC-5-USB | USB Token | 0x05 | 0x14 | 0x12 |
| CRC-16-CCITT | X.25, V.41, XMODEM, Bluetooth | 0x1021 | 0x8408 | 0x8810 |
| CRC-16-IBM | USB Data | 0x8005 | 0xA001 | 0xC002 |
| CRC-32 | Mpeg-2, zlib, PNG, Ethernet, GZip, PKZIP | 0x04C11DB7 | 0xEDB88320 | 0x82608EDB |
| CRC-32C | iSCSI, Btrfs, ext4 | 0x1EDC6F41 | 0x82F63B78 | 0x8F6E37A0 |

## CRC-5-USB in USB

USBの仕様では、PID以外の全てのフィールドでCRCを計算し誤り検知を行う。

- [CRC Description (USB.org)](http://www.usb.org/developers/docs/whitepapers/crcdes.pdf)
- [Blog Reference](http://www.indigresso.com/_blog/?p=83)

USBトークンパケットのフォーマットを以下に示す。

| Fields | PID | ADDR | ENDP | CRC |
| :--- | :--- | :--- | :--- | :--- |
| Bits | 8(LSB) | 7 | 4 | 5(MSB) |

CRCはPID以外の11ビットから計算する。

| 仕様書に記載された内容 | CRC計算の仕様 |
| :--- | :--- |
| | |

```text
width=5 poly=0x05 init=0x1f refin=true refout=true xorout=0x1f check=0x19 name="CRC-5/USB"
```

## CRC-16 in USB

| Fields | PID | DATA | CRC16 |
| :--- | :--- | :--- | :--- |
| Bits | 8(LSB) | 0-8192 | 16(MSB) |

```text
width=16 poly=0x8005 init=0xffff refin=true refout=true xorout=0xffff check=0xb4c8 name="CRC-16/USB"
```

---

## CRC-15-CAN in CAN2.0

---

## CRC-16-CCITT in Bluetooth

```text
11111111111111111
01000100100101001
10111011011010110 <= 反転している。
```

---

## CRC-32 in PNG

```text
width=32 poly=0x04c11db7 init=0xffffffff refin=true refout=true xorout=0xffffffff check=0xcbf43926 name="CRC-32"
```

- [PNG Specification: CRC Algorithm](https://www.w3.org/TR/PNG/#5CRC-algorithm)

| 仕様書に記載された内容 | CRC計算の仕様 |
| :--- | :--- |
| The CRC polynomial employed is $x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1$ | 生成多項式は次を使用する。生成多項式は 0x04C11DB7 / 0xEDB88320 |
| the 32-bit CRC is initialized to all 1's | CRCは all 1 で初期化する。 初期値は 0xFFFFFFFF |
| the data from each byte is processed from the least significant bit (1) to the most significant bit (128). | 1バイトはLSB(1)からMSB(128)へ処理します。ビットシフトは右送り。 |
| After all the data bytes are processed, the CRC is inverted (its ones complement is taken). | 全てのデータを処理した後、CRCのビットを反転します。出力XOR (0xFFFFFFFF) |
| This value is transmitted (stored in the datastream) MSB first. | 値の送信順はMSB first(ビッグエンディアン)とする。 |
| For the purpose of separating into bytes and ordering, the least significant bit of the 32-bit CRC is defined to be the coefficient of the $x^{31}$ term. | バイト順序に分離するため、CRCの最下位ビットは $x^{31}$ とする。 |

## Reference

- [巡回冗長検査 (Wikipedia)](https://ja.wikipedia.org/wiki/%E5%B7%A1%E5%9B%9E%E5%86%97%E9%95%B7%E6%A4%9C%E6%9F%BB)
- [Computation of cyclic redundancy checks (Wikipedia)](https://en.wikipedia.org/wiki/Computation_of_cyclic_redundancy_checks)
- [CRC Catalogue](http://reveng.sourceforge.net/crc-catalogue/)
