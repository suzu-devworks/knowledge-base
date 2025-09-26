# Shaker Sort

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Algorithm and example](#algorithm-and-example)
- [Implementation in C](#implementation-in-c)
- [References](#references)

## Overview

Shaker Sort is an improved version of Bubble Sort. While Bubble Sort scans in only one direction, Shaker Sort scans back and forth, making it more efficient.

## Algorithm and example

| # | | |
| -- | -- | -- |
| 0 | 8 4 3 7 6 5 2 1   | Original |
| 1 | 4 [8] 3 7 6 5 2 1 | Forward: Swap since 8 > 4 |
| 2 | 4 3 [8] 7 6 5 2 1 | Swap since 8 > 3 |
| 3 | 4 3 7 [8] 6 5 2 1 | Swap since 8 > 7 |
| 4 | 4 3 7 6 [8] 5 2 1 | Swap since 8 > 6 |
| 5 | 4 3 7 6 5 [8] 2 1 | Swap since 8 > 5 |
| 6 | 4 3 7 6 5 2 [8] 1 | Swap since 8 > 2 |
| 7 | 4 3 7 6 5 2 1 [8] | Swap since 8 > 1 |
| 8 | 4 3 7 6 5 (1) 2 [8] | Reverse: Swap since 2 > 1 |
| 9 | 4 3 7 6 (1) 5 2 [8] | Swap since 5 > 1 |
| 10 | 4 3 7 (1) 6 5 2 [8] | Swap since 6 > 1 |
| 11 | 4 3 (1) 7 6 5 2 [8] | Swap since 7 > 1 |
| 12 | 4 (1) 3 7 6 5 2 [8] | Swap since 3 > 1 |
| 13 | (1) 4 3 7 6 5 2 [8] | Swap since 4 > 1 |
| | ... | |
| 18 | (1) 3 4  6  5 2 [7] [8] | Forward: Number of swaps: 4 |
| | ... | |
| 22 | (1) (2) 3 4 6 5 [7] [8] | Reverse: Number of swaps: 4 |
| | ... | |
| 25 | (1) (2) 3 4 5 [6] [7] [8] | Forward: Number of swaps: 1 |
| | ... | |
| 28 | (1) (2) (3) 4 5 [6] [7] [8] | Reverse: Number of swaps: 0 |
| | ... | |
| 30 | (1) (2) (3) 4 [5] [6] [7] [8] | Forward: Number of swaps: 0 |
| 31 | (1) (2) (3) (4) [5] [6] [7] [8] | Reverse: Number of swaps: 0 |

- Number of comparisons: 31
- Number of swaps: 22

## Implementation in C

```c
#include <stdio.h>

#define SWAP(a, b) (a ^= b, b = a ^ b, a ^= b)

void sort(int *data, int length) {
    int left = 0;
    int right = length - 1;

    while (left < right) {
        int last_swap;

        for (int i = left; i < right; i++) {
            if (data[i] > data[i + 1]) {
                SWAP(data[i], data[i + 1]);
                last_swap = i;
            }
        }
        right = last_swap;

        for (int i = right; left < i; i--) {
            if (data[i - 1] > data[i]) {
                SWAP(data[i - 1], data[i]);
                last_swap = i;
            }
        }
        left = last_swap;
    }
}

int main(void) {
    int num[] = { 25, 91, 14, 78, 5, 42, 63, 87, 30, 55 };
    int len = sizeof(num) / sizeof(num[0]);

    sort(num, len);

    for (int i = 0; i < len; i++) {
        printf("%d = %3d\n", i, num[i]);
    }
}
```

```console
0 =   5
1 =  14
2 =  25
3 =  30
4 =  42
5 =  55
6 =  63
7 =  78
8 =  87
9 =  91
```

## References

- [Cocktail shaker sort - Wikipedia](https://en.wikipedia.org/wiki/Cocktail_shaker_sort)
