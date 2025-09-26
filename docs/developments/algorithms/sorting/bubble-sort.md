# Bubble Sort

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Algorithm and example](#algorithm-and-example)
- [Implementation in C](#implementation-in-c)
- [References](#references)

## Overview

Bubble Sort compares adjacent elements and swaps them if they are in the wrong order. Despite its slow worst-case time, its simple algorithm and easy implementation make it a common choice. It is a stable in-place sort, also known as sinking sort. The following code sorts by comparing two elements from the end of the array and pushing the smallest value to the front.

## Algorithm and example

| # | | |
| -- | -- | -- |
| 0 | 8 4 3 7 6 5 2 1   | Original |
| 1 | 4 [8] 3 7 6 5 2 1 | Swap since 8 > 4 |
| 2 | 4 3 [8] 7 6 5 2 1 | Swap since 8 > 3 |
| 3 | 4 3 7 [8] 6 5 2 1 | Swap since 8 > 7 |
| 4 | 4 3 7 6 [8] 5 2 1 | Swap since 8 > 6 |
| 5 | 4 3 7 6 5 [8] 2 1 | Swap since 8 > 5 |
| 6 | 4 3 7 6 5 2 [8] 1 | Swap since 8 > 2 |
| 7 | 4 3 7 6 5 2 1 [8] | Swap since 8 > 1 |
| | ... | |
| 13 | 3 4 6 5 2 1 [7] [8] | Number of swaps: 5 |
| | ... | |
| 18 | 3 4 5 2 1 [6] [7] [8] | Number of swaps: 3 |
| | ... | |
| 22 | 3 4 2 1 [5] [6] [7] [8] | Number of swaps: 2 |
| | ... | |
| 25 | 3 2 1 [4] [5] [6] [7] [8] | Number of swaps: 2 |
| | ... | |
| 27 | 2 1 [3] [4] [5] [6] [7] [8] | Number of swaps: 2 |
| 28 | 1 [2] [3] [4] [5] [6] [7] [8] | Swap since 2 > 1  |
| 29 | [1] [2] [3] [4] [5] [6] [7] [8] | Sorted  |

- Number of comparisons: 29
- Number of swaps: 22

## Implementation in C

```c
#include <stdio.h>

#define SWAP(a, b) (a ^= b, b = a ^ b, a ^= b)

void sort(int *data, int length) {
    for (int i = 0; i < length - 1; i++) {
        for (int j = length - 1; j > i; j--) {
            if (data[j - 1] > data[j]) {
                SWAP(data[j - 1], data[j]);
            }
        }
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

- [SorBubble sort - Wikipedia](htthttps://en.wikipedia.org/wiki/Bubble_sort)
