# Algorithm for Recommend

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
  - [Bridging the gap between ideal and reality](#bridging-the-gap-between-ideal-and-reality)
  - [Process for making a recommendation](#process-for-making-a-recommendation)
  - [Indicators (moments) that indicate the actual situation](#indicators-moments-that-indicate-the-actual-situation)
  - [Predict against the actual situation (linear model)](#predict-against-the-actual-situation-linear-model)
  - [Linear and nonlinear models](#linear-and-nonlinear-models)
- [Recommendations focusing on items](#recommendations-focusing-on-items)
  - [Correlation coefficient](#correlation-coefficient)
  - [Preprocessing](#preprocessing)
  - [Analysis](#analysis)
  - [Postprocessing](#postprocessing)
- [Recommendations focusing on content](#recommendations-focusing-on-content)
  - [Preprocessing](#preprocessing-1)
  - [Analysis](#analysis-1)
  - [Postprocessing](#postprocessing-1)

## Overview

コンピュータによって蓄積したデータに基づいて「おすすめ」を提示する仕組みをレコメンドシステムと呼ぶ。

### Bridging the gap between ideal and reality

> 理想と現実のギャップを埋める

レコメンドするうえで、最も理想的な情報はすべての商品に対する顧客の評価である。その中で一番良いものこそ、最良のレコメンドとなる。しかし現実問題として、すべての顧客が商品を評価することはない。このギャップが縮まるほど、商品の評価が理想的な状況に近づく。

- 「予測値」
  - まだわからない部分を補う
  - 何かしらの仮定を行って補われた評価
- 「欠損値」
  - データが得られない部分
  - 本来存在するはずなのに得られていない情報

### Process for making a recommendation

> レコメンドするまでの流れ

レコメンドシステムが情報のギャップを埋めるアルゴリズムを中心に全体の流れを追ったのが次のフローである。

| # | Phase | 説明 |
| :-: | :--- | :--- |
| 1 | 構築 | システム構築時にレコメンドの判断材料（アイテムとコンテンツ）として何を利用するのかを決める。 |
| 2 | 蓄積 | 閲覧した商品や購入・評価など判断材料をデータとして蓄積する。 |
| 3 | 前処理 | データをもとに分析アルゴリズムへの入力を生成する。 |
| 4 | 分析 | 欠損値を補完した入力値を分析する。分析結果から仮定に基づいた予想値を決める。 |
| 5 | 後処理 | 求まった予想値をもとにレコメンド商品を決定する。 |

### Indicators (moments) that indicate the actual situation

> 実態を示す指標（モーメント）

統計的な分布から実態が推測できる。これがモーメント（分布に対する積率母関数）と呼ばれる。

- [モーメント（数学）Wikipedia](http://ja.wikipedia.org/wiki/%E3%83%A2%E3%83%BC%E3%83%A1%E3%83%B3%E3%83%88_\(%E6%95%B0%E5%AD%A6\))

| モーメント | 定義 | 意味 |
| :--- | :--- | :--- |
| 1次モーメント（平均） | $\mu = \frac{1}{n}\sum_{i=1}^{n}{x}_{i}$ | 山の頂点 |
| 2次モーメント（分散） | $\sigma^2 = \frac{1}{n-1}\sum_{i=1}^{n}({x}_{i}-{\mu})^2$ | 山の広がり |
| 3次モーメント（歪度） | $s_{k} = \frac{n}{(n-1)(n-2){\sigma}^3}\sum_{i=1}^{n}({x}_{i}-\mu)^3$ | 山の傾き |
| 4次モーメント（尖度） | $k_{w} = \frac{n(n+1)}{(n-1)(n-2)(n-3){\sigma}^4}\sum_{i=1}^{n}({x}_{i}-\mu)^4$ | 山の尖り |
| … | | |

### Predict against the actual situation (linear model)

> 実態に対して予測する（線形モデル）

実態を表現しているモーメントを保つようにギャップを補えば、それは観測結果と整合するはずである。

### Linear and nonlinear models

> 線形モデルと非線形モデル

厳密には、線形性を有した関数で行う予測を線形モデルいう。
線形性を有した関数とは直線に限らず、乗算と加算で合成できるものを指し、たとえば三角関数なども含まれる。個々の成分を乗じたり加減算で分解できるのが特徴。
これらのモデルは2次モーメントまでで表現できる分布を仮定しており、そうでない分布では誤差が目立ってくる。3次以上のモーメントが働く分布では、非線形モデルを採用する必要がある。

---

## Recommendations focusing on items

> アイテムに注目したレコメンド

商品間の類似性に着目したレコメンド。同時に購入されるという現象から、商品の類似性をつかむのが狙い。そのためには商品の類似性を比較できる値が必要となる。

### Correlation coefficient

> 相関係数

任意の2つの対象の類似性や因果関係を図る指標。
対象$X$と対象$Y$について$N$個の観測結果 $\{x_i, y_i\}$ があるとき、相関係数 $C_{xy}$ は次のように定義できる。

$$
C_{xy} = \frac{\sum_{i=1}^{n}(x_i - \overline{x})(y_i - \overline{y})}
              {\sqrt{\sum_{i=1}^{n}(x_i - \overline{x})^2}\sqrt{\sum_{i=1}^{n}(y_i - \overline{y})^2}}
$$

$$\overline{x} = \frac{1}{n}\sum_{i=1}^{n}x_i, \quad \overline{y} = \frac{1}{n}\sum_{i=1}^{n}y_i$$

### Preprocessing

> 前処理

### Analysis

> 分析

### Postprocessing

> 後処理

---

## Recommendations focusing on content

> コンテンツに注目したレコメンド

商品のレビュー・評価など、顧客の好みといった要素に着目したレコメンド。

### Preprocessing

> 前処理

### Analysis

> 分析

### Postprocessing

> 後処理

---
