# Algorithm for Compression

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Lossless and Lossy compression](#lossless-and-lossy-compression)
- [Shannon–Fano coding](#shannonfano-coding)
- [Huffman coding](#huffman-coding)
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

<!-- spell-checker: words LZSS -->

## Shannon–Fano coding

<!-- spell-checker: words Fano -->

Shannon–Fano coding is not widely used, so we will only cover the theory.

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

## Huffman coding

Huffman coding is a method of generating prefix codes based on the occurrence frequency of characters in a string. It is one of the most famous data compression algorithms.

1. Original string
   > WEB+DB\_PRESS

2. Create a list of nodes for each character with its frequency. Sort the nodes in ascending order of frequency.

    | Symbol | W | \+ | D | \_ | P | R | E | B | S |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | frequency | 1 | 1 | 1 | 1 | 1 | 1 | 2 | 2 | 2 |

3. Combine the two nodes with the lowest frequencies into a single new node. The new node's frequency is the sum of the two combined nodes' frequencies. Place the new node back into the list while maintaining the sorted order.

    | Symbol | D | \_ | P | R | (W\+) | E | B | S |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | frequency | 1 | 1 | 1 | 1 | 2 | 2 | 2 | 2 |

4. Repeat the process until all nodes are combined into a single tree structure

    | Symbol | P | R | (D\_) | (W\+) | E | B | S |
    | -- |  -- | -- | -- | -- | -- | -- | -- |
    | frequency | 1 | 1 | 2 | 2 | 2 | 2 | 2 |

    | Symbol | (PR) | (D\_) | (W\+) | E | B | S |
    | -- |  -- | -- | -- | -- | -- | -- |
    | frequency | 2 | 2 | 2 | 2 | 2 | 2 |

    | Symbol | (W\+) | E | B | S | ((PR)(D\_)) |
    | -- |  -- | -- | -- | -- | -- |
    | frequency | 2 | 2 | 2 | 2 | 4 |

    | Symbol | B | S | ((W\+)E) | ((PR)(D\_)) |
    | -- |  -- | -- | -- | -- |
    | frequency | 2 | 2 | 4 | 4 |

    | Symbol | (BS) | ((W\+)E) | ((PR)(D\_)) |
    | -- |  -- | -- | -- |
    | frequency | 4 | 4 | 4 |

    | Symbol | ((PR)(D\_)) | ((BS)((W\+)E)) |
    | -- |  -- | -- |
    | frequency | 4 | 8 |

    | Symbol | (((PR)(D\_))((BS)((W\+)E))) |
    | -- |  -- |
    | frequency | 12 |

5. Assign binary codes to each character by traversing the tree from the root to the leaf nodes.

    Huffman tree:

    ```console
              0┌─────────────────────┐1
         0┌────┴────┐1       0┌──────┴──────┐1
      0┌──┴──┐1 0┌──┴──┐1 0┌──┴──┐1     0┌──┴───┐1
       P     R   D     _   B     S   0┌──┴──┐1  E
                                      W     +
    ```

    | Symbol | P | R | D | \_ | B | S | W | \+ | E |
    | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
    | Code | 000 | 001 | 010 | 011 | 100 | 101 | 1100 | 1101 | 111 |

## References

- [Data compression - Wikipedia](https://en.wikipedia.org/wiki/Data_compression)
