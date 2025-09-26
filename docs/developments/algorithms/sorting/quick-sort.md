# Quick Sort

## Table of Contents <!-- omit in toc -->

## Overview

Quick Sort selects a pivot value and partitions the array into two sub-arrays: one containing values less than the pivot, and one containing values greater than the pivot. This process is then repeated recursively on the sub-arrays.

Choosing a pivot: Selecting a pivot from the middle of the array is more efficient than choosing the first or last element. Even better is to sample a few values (around three) from the array and use their median as the pivot.

## Algorithm and example

## Implementation in C

```c
#include <stdio.h>

#define SWAP(a, b) (a ^= b, b = a ^ b, a ^= b)

void quick(int *data, int left, int right) {
    int pivot;
    int i, j;

    if (left >= right) {
        return;
    }

    pivot = data[(left + right) / 2];
    i = left - 1;
    j = right + 1;
    while (1) {
        while (data[++i] < pivot) {}
        while (data[--j] > pivot) {}
        if (i >= j) {
            break;
        }

        SWAP(data[i], data[j]);
    }

    quick(data, left, i - 1);
    quick(data, j + 1, right);
}

void sort(int *data, int length) {
    quick(data, 0, length - 1);
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

- [Quicksort - Wikipedia](https://en.wikipedia.org/wiki/Quicksort)
