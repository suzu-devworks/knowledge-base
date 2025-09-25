# Huffman coding

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Encoding principles and examples](#encoding-principles-and-examples)
- [References](#references)

## Overview

Huffman coding is a method of generating prefix codes based on the occurrence frequency of characters in a string. It is one of the most famous data compression algorithms.

## Encoding principles and examples

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

- [Huffman coding - Wikipedia](https://en.wikipedia.org/wiki/Huffman_coding)
