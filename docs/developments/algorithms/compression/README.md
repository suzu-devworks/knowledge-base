# Algorithm for Compression

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Lossless and Lossy compression](#lossless-and-lossy-compression)
- [References](#references)

## Overview

Data compression is the process of converting data into another data set with a smaller data size while maintaining its essential properties. This is also called high-efficiency coding, and in information theory, it is known as source coding.

## Lossless and Lossy compression

Lossless compression is a compression method where the data, when restored, is completely identical to the original.  

Lossy compression is a compression method where the data, when restored, is not completely identical to the original. In many lossy compression methods, components that are not strongly recognized by humans are removed to compress the data.

| Name | Type | Theory | Description |
| :---- | :---- | :---- | :---- |
| Shannon–Fano coding | Lossless | Entropy coding | Not always the shortest code length; not widely used. |
| Huffman coding | Lossless | Entropy coding | Used in compression formats like JPEG and ZIP (Deflate). |
| Slide-Dictionary coding (LZ77/LZSS/LZW) | Lossless | Dictionary coding | This method uses a dictionary to find and replace repeated sequences of data. |
| Byte Pair Encoding | Lossless | Dictionary coding | This algorithm replaces the most frequent pairs of consecutive bytes with a single new byte that does not appear in the data. |
| Run-length encoding | Lossless | Dictionary coding | This compression algorithm replaces sequences of identical data with a single data value and the number of times it appears in the sequence. |
| Deflate | Lossless | Hybrid | This is a lossless data compression algorithm that combines Huffman coding with the LZ77 algorithm. |

<!-- spell-checker: words Fano -->
<!-- spell-checker: words LZSS -->

## References

- [Data compression - Wikipedia](https://en.wikipedia.org/wiki/Data_compression)
