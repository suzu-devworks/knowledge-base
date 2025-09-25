# Shannon–Fano coding

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Encoding principles and examples](#encoding-principles-and-examples)
- [References](#references)

## Overview

Shannon-Fano coding is a lossless data compression method invented by Claude Shannon and Roberto Fano in 1948.

<!-- spell-checker: words Fano -->

Neither Shannon–Fano algorithm is guaranteed to generate an optimal code. For this reason, Shannon–Fano codes are almost never used.

Huffman coding is almost as computationally simple and produces prefix codes that always achieve the lowest possible expected code word length, under the constraints that each symbol is represented by a code formed of an integral number of bits.

## Encoding principles and examples

When splitting data into two, the data size can vary, and analysis becomes difficult depending on how the data at an indivisible position is encoded.

1. Original string
    > WEB+DB\_PRESS

2. Find the number of occurrences of each symbol in the data to be compressed, and sort the symbols in descending order of their count. If the counts are the same, the order is arbitrary.

    | Symbol | E | B | S | W | \+ | D | \_ | P | R |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | Count | 2 | 2 | 2 | 1 | 1 | 1 | 1 | 1 | 1 |
    | Probabilities | 0.22 | 0.22 | 0.22 | 0.11 | 0.11 | 0.11 | 0.11 | 0.11 | 0.11 |

3. Split the sorted data into two parts with as equal a total count as possible. Assign a '0' to one part and a '1' to the other.

    | Symbol | E | B | S | W | \+ | D | \_ | P | R |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | #1 | 0 | 0 | 0 | 1 | 1 | 1 | 1 | 1 | 1 |
    | #2 | 00 | 01 | 01 | 10 | 10 | 10 | 11 | 11 | 11 |
    | #3 | - | 010 | 011 | 100 | 101 | 101 | 110 | 111 | 111 |
    | #4 | - | - | - | - | 1010 | 1011 | - | 1110 | 1111 |
    | Code | 00 | 010 | 011 | 100 | 1010 | 1011 | 110 | 1110 | 1111 |

## References

- [Shannon–Fano coding - Wikipedia](https://en.wikipedia.org/wiki/Shannon%E2%80%93Fano_coding)
