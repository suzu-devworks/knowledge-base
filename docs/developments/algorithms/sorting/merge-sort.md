# Merge Sort

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Algorithm and example](#algorithm-and-example)
- [Implementation in C](#implementation-in-c)
- [References](#references)

## Overview

Merge Sort merges two sorted arrays into a single, sorted array. It is well-suited for parallel processing.

## Algorithm and example

## Implementation in C

```c
#include <stdio.h>
#include <stdlib.h> // for malloc and free

// Helper function to merge two sorted arrays  
void merge(int *data1, int *data2, int count1, int count2) {
    int *merged = (int *)malloc(sizeof(int) * (count1 + count2));
    int current = 0;
    int index1 = 0;
    int index2 = 0;

    while (index1 < count1 && index2 < count2) {
        if (data1[index1] <= data2[index2]) {
            merged[current++] = data1[index1++];
        }
        else {
            merged[current++] = data2[index2++];
        }
    }

    while (index1 < count1) {
        merged[current++] = data1[index1++];
    }

    while (index2 < count2) {
        merged[current++] = data2[index2++];
    }

    // Copy back to original arrays
    current = 0;
    for (index1 = 0; index1 < count1; index1++) {
        data1[index1] = merged[current++];
    }

    for (index2 = 0; index2 < count2; index2++) {
        data2[index2] = merged[current++];
    }

    free(merged);
}

void sort(int *data, int length) {
    int mid;
    if (length == 1) return;

    mid = length / 2;
    sort(data, mid);
    sort(data + mid, length - mid);
    merge(data, data + mid, mid, length - mid);
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
<!-- spell-checker: words malloc stdlib -->

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

- [Sorting (Wikipedia)](https://en.wikipedia.org/wiki/Sorting_algorithm)
