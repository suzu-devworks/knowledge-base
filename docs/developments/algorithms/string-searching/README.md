# Algorithm for String Searching

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Brute-force (Simple-search)](#brute-force-simple-search)
- [Boyer-Moore](#boyer-moore)
- [Reference](#reference)

## Overview

文字列探索とは、ある文字列の中から別のある文字列を探索することである。

| 名称 | 手法 | 平均計算量 | 最悪計算量 |
| :--- | :--- | :--- | :--- |
| Brute-force | 理論的にありうるパターン全てを探索する。 | ー | $O(nm)$ |
| Boyer-Moore | 検索対象文字列（キー）の前処理を行い、検索対象テキストの前処理は行わない。 | $O(n)$ | $O(nm)$ |
| Knuth-Morris-Pratt | テキスト（文字列）$S$から単語$W$を探すにあたり、不一致となった位置と単語自身の情報から次に照合を試すべき位置を決定することで検索を効率化する。 | $O(k + n)$ ($k$は$S$の長さ、$n$は$W$の長さ) | 線形 |
| Aho-Corasick | 入力テキスト($S$)について有限の文字列群（辞書）($W$)の各要素を探す辞書式マッチングアルゴリズムの一種である。全パターンのマッチングを一斉に探索する。 | $O(m + n)$ ($m$は$S$の長さ、$n$は$W$の長さ) | 線形 |
| Rabin-Karp | ハッシュ関数を利用してテキストからパターン（サブ文字列）を探す文字列検索アルゴリズムの一種。1つのパターンの検索にはあまり用いられないが、理論的には重要であり、複数パターンの検索には効果的である。 | $O(n)$ | $O(nm)$ |

## Brute-force (Simple-search)

力任せ、総当たりと訳される方式。簡単に実装できるが効率が悪い。
入力文字列の先頭から検索文字列を照合し、照合の合否に関わらず、入力文字の照合開始位置を1つすすめる。あとはこれをひたすら繰り返す。

```text
TEXT|  w  e  b  w  e  p  d  b  w  e  b  d  b  w  w  e  b  d  b  w  e  b  b  b
  1 | [w][e][b] d  b
  2 |    w  e  b  d  b
  3 |       w  e  b  d  b
  4 |         [w][e] b  d  b
  5 |             w  e  b  d  b
  6 |                w  e  b  d  b
  7 |                   w  e  b  d  b
  8 |                      w  e  b  d  b
  9 |                        [w][e][b][d][b]
 10 |                            w  e  b  d  b
 11 |                               w  e  b  d  b
 12 |                                  w  e  b  d  b
 13 |                                     w  e  b  d  b
 14 |                                       [w] e  b  d  b
 15 |                                          [w][e][b][d][b]
 16 |                                              w  e  b  d  b
 17 |                                                 w  e  b  d  b
 18 |                                                    w  e  b  d  b
 19 |                                                       w  e  b  d  b  
 20 |                                                         [w][e][b] d  b
```

sample with C code:

```c
int search(const char *text, int textlen, const char *pattern, int patternlen)
{
    int i;
    int match_count = 0;
    for (i = 0; i < textlen - patternlen + 1; i++)
    {
        if (strncmp(&text[i], pattern, patternlen) == 0)
        {
            ++match_count;
        }
    }
    return match_count;
}
```

## Boyer-Moore

ボイヤー-ムーア法は文字列の照合を末尾から行い、あらかじめ検索文字列を操作して作成したskipテーブルをもとに次に照合する位置までずらす。
この方法には特殊なケースがあり、照合に失敗した位置からskip分ずらした場合、照合開始位置が前に戻ってしまう場合がある。この場合にはしかたが無いので1文字分だけをずらすようにする。

| w | e | b | d |
| :-: | :-: | :-: | :-: |
| 4 | 3 | 2 | 1 |

```text
 TEXT|  w  e  b  w  e  p  d  b  w  e  b  d  b  w  w  e  b  d  b  w  e  b  b  b
   1 |  w  e  b  d  b
   2 |           w  e  b [d][b]
   3 |                    w  e  b  d [b]
   4 |                         [w][e][b][d][b]
   5 |                                w  e  b  d  b
   6 |                                           [w][e][b][d][b]
   7 |                                                  w  e  b  d  b
   8 |                                                           w  e  b  d  b
```

sample with C code:

```c
int search(const char *text, const char *pattern)
{
    int match_count = 0;
    int textlen = (int)strlen(text);
    int patternlen = (int)strlen(pattern);
    int skip[UCHAR_MAX];
    bool is_match;
    int start;
    int pos;
    int dx;
    int i;
    char c;

    for (i = 0; i < UCHAR_MAX; i++) {
        skip[i] = patternlen;
    }
    for (i = 0; i < patternlen - 1; i++) {
        c = pattern[i];
        skip[(int)c] = patternlen - 1 - i;
    }

    start = 0;
    while (start + patternlen <= textlen) {
        is_match = true;
        pos = -1;
        for (i = patternlen - 1; i >= 0; i--) {
            if (text[start + i] != pattern[i]) {
                is_match = false;
                pos = i;
                break;
            }
        }
        if (is_match) {
            match_count++;
            c  = pattern[patternlen - 1];
            dx = skip[(int)c];
        }
        else {
            c  = text[start + pos];
            dx = skip[(int)c];
            dx -= (patternlen - 1 - pos);
        }
        dx = (dx <= 0) ? 1 : dx;
        start += dx;
    }
    return match_count;
}
```

## Reference

- [文字列探索 (Wikipedia)](http://ja.wikipedia.org/wiki/%E6%96%87%E5%AD%97%E5%88%97%E6%8E%A2%E7%B4%A2)
