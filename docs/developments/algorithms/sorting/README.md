# Algorithm for Sorting

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Stability](#stability)
- [References](#references)

## Overview

Sorting is the process of arranging a collection of data into a specific order.

## Stability

A sorting algorithm is considered stable if the relative order of equal elements is preserved after the sort.

| Name | Method | Average Case Complexity | Worst Case Complexity | Stable |
| :---- | :---- | :---- | :---- | :---- |
| [Bubble Sort](./bubble-sort.md) | Exchange | ${\displaystyle n^{2}}$ | ${\displaystyle n^{2}}$ | Yes |
| [Shaker Sort](./shaker-sort.md) | Exchange | ${\displaystyle n^{2}}$ | ${\displaystyle n^{2}}$ | Yes |
| [Shell Sort](./shell-sort.md) | Insertion | ${\displaystyle n\log n}$ | ${\displaystyle ≧ n^{1.5}}$ | No |
| [Selection Sort](./selection-sort.md) | Selection | ${\displaystyle n^{2}}$ | ${\displaystyle n^{2}}$ | No |
| [Heap Sort](./heap-sort.md) | Selection | ${\displaystyle n\log n}$ | ${\displaystyle n\log n}$ | No |
| [Merge Sort](./merge-sort.md) | Merge | ${\displaystyle n\log n}$ | ${\displaystyle n\log n}$ | Yes |
| [Quick Sort](./quick-sort.md) | Partitioning | ${\displaystyle n\log n}$ | ${\displaystyle n^{2}}$ | No |

## References

- [SoSorting algorithm - Wikipedia](https://en.wikipedia.org/wiki/Sorting_algorithm)
