# Heap Sort

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Algorithm and example](#algorithm-and-example)
- [Implementation in C](#implementation-in-c)
- [References](#references)

## Overview

Heap Sort first transforms the unsorted portion of the data into a heap (a binary tree with a specific ordering property). It then extracts the maximum (or minimum) value from the heap and places it in the sorted portion, detaching the sorted part from the heap.

## Algorithm and example

## Implementation in C

```c
#include <stdio.h>

#define SWAP(a, b) (a ^= b, b = a ^ b, a ^= b)

void shift_down(int *data, int start, int length) {
    int parent = start;
    int child = parent * 2 + 1;

    while (child <= length - 1) {
        if (child < length - 1 && data[child] < data[child + 1]) {
            child++;
        }

        if (data[child] <= data[parent]) {
            break;
        }

        SWAP(data[parent], data[child]);
        parent = child;
        child = parent * 2 + 1;
    }
}

void sort(int *data, int length) {
    // Build the heap
    for (int i = length / 2 - 1; i >= 0; i--) {
        shift_down(data, i, length);
    }

    // Extract elements from the heap  
    for (int i = length - 1; i > 0; i--) {
        SWAP(data[0], data[i]);
        shift_down(data, 0, i);
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

- [Heapsort - Wikipedia](https://en.wikipedia.org/wiki/Heapsort)

<!-- spell-checker: words Heapsort -->