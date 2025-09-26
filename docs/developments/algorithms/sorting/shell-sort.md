# Shell Sort

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Implementation in C](#implementation-in-c)
- [References](#references)

## Overview

Shell Sort sorts sub arrays of elements at a certain interval. The interval is then reduced, and the process is repeated until the interval is 1\.

**Choosing a gap:**

A common choice for the gap h is the sequence ${h_{n+1}=3h_{n}+1}$, starting with h=1,4,13,40,121,.... Applying the gaps in descending order results in an average time complexity of around ${\displaystyle O(n^{1.25})}$.

**Data locality:**

Array index references can be scattered, leading to poor memory access efficiency. Accessing contiguous memory is more efficient, especially for large datasets.

## Implementation in C

```c
#include <stdio.h>

#define SWAP(a, b) (a ^= b, b = a ^ b, a ^= b)

void sort(int *data, int length) {
    int gap;
    int i, j;

    for (gap = 1; gap < length / 3; gap = 3 * gap + 1) {
    }

    while (gap > 0) {
        for (i = gap; i < length; i++) {
            for (j = i - gap; j >= 0; j = j - gap) {
                if (data[j] > data[j + gap]) {
                    SWAP(data[j], data[j + gap]);
                }
                else {
                    break;
                }
            }
        }
        gap = gap / 3;
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

- [Shellsort - Wikipedia](https://en.wikipedia.org/wiki/Shellsort)

<!-- spell-checker: words Shellsort -->