# Selection Sort

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Algorithm and example](#algorithm-and-example)
- [Implementation in C](#implementation-in-c)
- [References](#references)

## Overview

Selection Sort finds the minimum or maximum value in the unsorted portion of the array and swaps it with the first element of that unsorted portion.

## Algorithm and example

## Implementation in C

```c
#include <stdio.h>

#define SWAP(a, b) (a ^= b, b = a ^ b, a ^= b)

void sort(int *data, int length) {
    for (int i = 0; i < length - 1; i++) {
        int min = data[i];
        int minindex = i;

        for (int j = i + 1; j < length; j++) {
            if (data[j] < min) {
                min = data[j];
                minindex = j;
            }
        }
        
        if (i != minindex) {
            SWAP(data[i], data[minindex]);
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
<!-- spell-checker: words minindex -->

## References

- [Selection sort - Wikipedia](httphttps://en.wikipedia.org/wiki/Selection_sort)
