---
title: Improved Moment Shadow Maps for Translucent Occluders, Soft Shadows and Single Scattering [@Peters2017]
---
# Introduction

フィルタ可能なシャドウマップには、半透明な遮蔽物をフィルタ可能なシャドウマップに直接レンダリングする手法[@Delalandre2011; @McGuire2016]、PCSS[@Fernando2005]、Summed-Area Table[@Crow1984]を用いてソフトシャドウを固定コストで行う方法[@Lauritzen2007; @Annen2008; @Yang2010; @Shen2013]、視線に沿った単一散乱の累積をフィルタ可能なシャドウマップに事前計算する手法[@Klehm2014]、といった多種多様な応用が存在する。
本論ではこれらすべてのアプローチがMoment Shadow Mappingと互換性があることを実証する。その導出の過程で、モーメントベースのブロッカーサーチや6モーメントのMoment Shadow Mappingといった、新しい構成要素を開発する。出来上がるテクニックは魅力的なクオリティ対ランタイムのトレードオフを提供する。また、フラグメントあたりのオーバーヘッドが小さく固定であるため、高出力解像度に合わせて特に良くスケールする。

# Improved Moment Shadow Maps

## Depth Distributions

フィルタカーネルが$n \in \mathbb{N}$個の重み$w_0, ... , w_{n - 1} > 0$を持ち、フィルタ領域内でサンプルされる対応する深度が$z_0, ... , z_{n-1} \in \mathbb{R}$であるとすると、これらの量は以下の深度分布を定義する。

$$
Z := \sum_{l = 0}^{n - 1} w_l \cdot \delta_{z_l}
$$

この表記は、深度$z_l$を描く確率がフィルタ重み$w_l$によって求まるようにフィルタ領域を無作為にサンプルする、という確率的な解釈を提案する。そして、各深度$z$をあるベクトル量$\boldsymbol{b}(z) \in \mathbb{R}^{m + 1}$(ここで、$m \in \mathbb{N}$)にマップすることを考える。無作為な深度を入力として用いると、出力の期待値は以下で求まる。

$$
b := \varepsilon_Z(\boldsymbol{b}) := \sum_{l = 0}^{n - 1} w_l \cdot \boldsymbol{b}(z_l) \in \mathbb{R}^{m + 1}
$$

フィルタ可能なシャドウマップはちょうどこの方法で構築される。各テクセルにはある関数$\boldsymbol{b} : \mathbb {R} \rightarrow \mathbb{R}^{m + 1}$による$\boldsymbol{b}(z)$を格納して、上記のフィルタカーネルでシャドウマップをフィルタリングすることで$b = \varepsilon_Z(\boldsymbol{b})$を生成する。関数$\boldsymbol{b}$には$b$の情報[knowledge]が$Z$のおおよその再構築を可能にするものが選ばれる。

## Related Work

Percentage-Closer Filtering[@Reeves1987]はシャドウマップから$n \in \mathbb{N}$個のサンプルを取ることで無理やり[through brute force]深度分布を再構築する。

$$
Z(\boldsymbol{z} < z_f) := \sum_{l = 0}^{n - 1} w_l \cdot
\begin{cases}
1 & \text{if } z_l < z_f \\
0 & \text{otherwise}
\end{cases}
\text{ where } \boldsymbol{z}(z) := z
$$

Variance Shadow Maps[@Lauritzen2006]は$\boldsymbol{b}(z) := (1, z, z^2)^T$を用いる。1つ目の要素は$\varepsilon_Z(\boldsymbol{b_0}) = 1$なので格納する必要がなく、残りの2つで$Z$の平均と分散を計算して、Cantelliの不等式^[単一テールの場合でのチェビシェフの不等式の一般化 --- [Wikipedia](https://en.wikipedia.org/wiki/Cantelli%27s_inequality)]で再構築できる。

$$
\mu := b_1, \sigma^2 := b_2 - b_1^2, Z(\boldsymbol{z} < z_f) \ge 1 - \frac{\sigma^2}{\sigma^2 + (z_f - \mu)^2} \text{ if } z_f \ge \mu
$$

同様に、Exponential Shadow Maps[@Salvi2008; @Annen2008b]は$\boldsymbol{b}(z) := (1, \exp(c_{esm} \cdot z))^T$(ここで、$c_{esm} \gg 1$)を用いて、Markovの不等式で再構築する。

$$
Z(\boldsymbol{z} < z_f) \ge 1 - \frac{b_1}{\exp(c_{esm} \cdot z_f)}
$$

Convolution Shadow Maps[@Annen2007]は$\boldsymbol{b}$の関数にフーリエ基底関数を設定し、打ち切られたフーリエ級数を用いる。Exponential Variance Shadow Maps[@Lauritzen2008]はパラメータ$c_{evsm}^+, c_{evsm}^- > 0$を固定し、2つの指数関数的に曲げられた[warped]深度にVariance Shadow Mappingを適用する。すなわち、以下を用いる。

$$
\boldsymbol{b}(z) = (1, \exp(c_{evsm}^+ \cdot z), \exp(c_{evsm}^+ \cdot z)^2, -\exp(c_{evsm}^- \cdot z), \exp(c_{evsm}^- \cdot z)^2)^T
$$

### Moment Shadow Mapping

TODO
