---
title: Improved Moment Shadow Maps for Translucent Occluders, Soft Shadows and Single Scattering [@Peters2017]
---
# はじめに[Introduction]

　フィルタ可能なシャドウマップには、半透明な遮蔽物をフィルタ可能なシャドウマップに直接レンダリングする手法[@Delalandre2011; @McGuire2016]、PCSS[@Fernando2005]、Summed-Area Table[@Crow1984]を用いてソフトシャドウを固定コストで行う方法[@Lauritzen2007; @Annen2008; @Yang2010; @Shen2013]、視線に沿った単一散乱の累積をフィルタ可能なシャドウマップに事前計算する手法[@Klehm2014]、といった多種多様な応用が存在する。
　本論ではこれらすべてのアプローチがMoment Shadow Mappingと互換性があることを実証する。その導出の過程で、モーメントベースのブロッカーサーチや6モーメントのMoment Shadow Mappingといった、新しい構成要素を開発する。出来上がるテクニックは魅力的なクオリティ対ランタイムのトレードオフを提供する。また、フラグメントあたりのオーバーヘッドが小さく固定であるため、高出力解像度に合わせて特に良くスケールする。

# 改善されたMoment Shadow Maps[Improved Moment Shadow Maps]

## 深度分布[Depth Distributions]

　フィルタカーネルが$n \in \mathbb{N}$個の重み$w_0, ... , w_{n - 1} > 0$を持ち、フィルタ領域内でサンプルされる対応する深度が$z_0, ... , z_{n-1} \in \mathbb{R}$であるとすると、これらの量は以下の深度分布を定義する。

$$
Z := \sum_{l = 0}^{n - 1} w_l \cdot \delta_{z_l}
$$

　この表記は、深度$z_l$を描く確率がフィルタ重み$w_l$によって求まるようにフィルタ領域を無作為にサンプルする、という確率的な解釈を提案する。そして、各深度$z$をあるベクトル量$\boldsymbol{b}(z) \in \mathbb{R}^{m + 1}$(ここで、$m \in \mathbb{N}$)にマップすることを考える。無作為な深度を入力として用いると、出力の期待値は以下で求まる。

$$
b := \varepsilon_Z(\boldsymbol{b}) := \sum_{l = 0}^{n - 1} w_l \cdot \boldsymbol{b}(z_l) \in \mathbb{R}^{m + 1}
$$

　フィルタ可能なシャドウマップはちょうどこの方法で構築される。各テクセルにはある関数$\boldsymbol{b} : \mathbb {R} \rightarrow \mathbb{R}^{m + 1}$による$\boldsymbol{b}(z)$を格納して、上記のフィルタカーネルでシャドウマップをフィルタリングすることで$b = \varepsilon_Z(\boldsymbol{b})$を生成する。関数$\boldsymbol{b}$には$b$の情報[knowledge]が$Z$のおおよその再構築を可能にするものが選ばれる。

## 関連研究[Related Work]

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

　Moment Shadow Mapping[@Peters2015]は以下を用いる。

$$
\boldsymbol{b}(z) = (z^j)_ {j = 0}^m = (1, z, z^2, z^3, z^4)^T
$$

モーメント数$m \in \mathbb{N}$は任意の偶数を取ることができる。
　深度$z_f \in \mathbb{R}$のフラグメントをシェーディングするとき、シャドウ強度$Z(\boldsymbol{z} < z_f)$を推定する必要があるが、シャドウマップからフィルタされたサンプル$b = \varepsilon_Z(\boldsymbol{b})$のみが与えられている。一般に、これらのモーメントに互換性のある$\mathbb{R}$上の深度分布$S$は無数に存在する。誤ったセルフシャドウ(サーフェスアクネ)を減らすため、我々は以下で求まる最も小さなシャドウ強度を産む再構築を用いる。

$$
\inf \{ S(\boldsymbol{z} < z_f) | Sは\varepsilon_S(\boldsymbol{b}) = bを持つ\mathbb{R}上の深度分布 \}
$$ {#eq:1}

　この最小化問題は非自明であるが、すでに解かれている[@Krein1977]。最適な深度分布$S$は深度値$z_f$と他の$\frac{m}{2} = 2$つの深度値のみを用いる。そのような深度分布は唯一であり、アルゴリズム1で効率良く計算する。
　アルゴリズム1では、行列$B(b)$は正定値[positive definite]でないと正しく動作しない可能性があるが、有意な入力$b = \varepsilon_Z(\boldsymbol{b})$に対して半正定値[positive semi-definite]となることが知られている。丸め誤差が入力を不正にする可能性があるが、これらは小さな補間重み$0 < \alpha_b \ll 1$を用いた固定の定数ベクトルに向かう線形補間によって相殺される。このバイアスは、近似の正確さを低下させたり、ライトリークを増やしたりするので、最小限に留めるべきである。
。

- アルゴリズム1 --- [@eq:1]を解く
- 入力: モーメント$b \in \mathbb{R}^{m+1}$、フラグメントの深度$z_f \in \mathbb{R}$
- 出力: [@eq:1]での表現を最小化する深度分布$S$
    1. $B(b) := (b_{j+k})_ {j,k=0}^{\frac{m}{2}} = \left( \begin{array} \\ b_0 & b_1 & \cdots & b_{\frac{m}{2}} \\ b_1 & b_2 & \cdots & b_{\frac{m}{2}+1} \\ \vdots & \vdots & \ddots & \vdots \\ b_{\frac{m}{2}} & b_{\frac{m}{2}+1} & \cdots & b_m  \end{array} \right) \in \mathbb{R}^{(\frac{m}{2}+1)\times(\frac{m}{2}+1)}$をセットする。
    2. $q \in \mathbb{R}^{\frac{m}{2}+1}$に対する$B(b) \cdot q = (1, z_f^1, \dots, z_f^{\frac{m}{2}})^T$を解く。
    3. $z$に対する多項式$\sum_{j=0}^{\frac{m}{2}} q_j \cdot z^j = 0$を解き、$z_1, \dots , z_{\frac{m}{2}} \in \mathbb{R}$による個別の解を示す。
    4. $z_0 := z_f$と$A := (z_l^j)_ {j,l=0}^{\frac{m}{2}} = \left( \begin{array} \\ z_0^0 & \cdots & z_{\frac{m}{2}}^0 \\ \vdots & \ddots & \vdots \\ z_0^{\frac{m}{2}} & \cdots & z_{\frac{m}{2}}^{\frac{m}{2}} \end{array} \right) \in \mathbb{R}^{(\frac{m}{2}+1)\times(\frac{m}{2}+1)}$をセットする。
    5. $w \in \mathbb{R}^{\frac{m}{2}+1}$に対する$A \cdot w = (b_0, b_1, \dots, b_{\frac{m}{2}})^T$を解く。
    6. $\sum_{l=0}^{\frac{m}{2}} w_l \cdot \delta_{z_l}$を返す。

　4つのモーメントにそれぞれ16ビットを単に充てるとすると丸め誤差が強く出てしまうので、出力データ型の表現できる範囲を侵さないで行列式を最大化する数値的な最適化となるようなアフィン変換を施してから16ビット浮動小数点数に格納する。

## 符号付き深度[Signed Depth]

　元々[@Peters2015]では深度を区間$[0, 1]$で定義するよう提案したが、この選択は最適な選択である$[-1, 1]$より悪いことが判明した。このときニア面が$-1$、ファー面が$1$に位置する。
　この定義を支持する理由はモーメントでの線形変換の効果に基づいている。$x \ne 0$である$x, y \in \mathbb{R}$が深度値の線形変換$x \cdot z + y$を記述すると、以下のモーメントに対する線形変換を導く。

$$
\varepsilon_Z((x \cdot \boldsymbol{z} + y)^j) =
\varepsilon_Z \left( \sum_{k=0}^j \left(
    \begin{array} \\
    j \\
    k
    \end{array}
\right) \cdot (x \cdot \boldsymbol{z})^k \cdot y^{j - k}
\right) = \sum_{k=0}^j \left(
    \begin{array} \\
    j \\
    k
    \end{array}
    \right) \cdot x^k \cdot y^{j-k} \cdot b_k
$$ {#eq:2}

この線形変換は対角成分$x^0, \dots, x^m$を持つ下三角行列に対応する。すなわち、この行列式は$\prod_{j=1}^m x^j$である。
　区間$[-1, 1]$で深度を定義すると仮定するとして、上記の変換を適用すると、$j$番目のモーメントは最大で$(|x|+|y|)^j$の大きさを取り得る。我々は浮動小数点数を用いているので、大きさの最大値を1にするためにこの大きさで割る場合には相対的な精度は減少しない。これを先程の変換と組み合わせると、その行列式は以下のようになる。

$$
\prod_{j=1}^m \left( \frac{x}{|x|+|y|} \right)^j
$$

明らかに、この行列式の大きさは$y = 0$のときに最大であり、$x$の変化に対して不変である。
　[@eq:2]の変換は特殊な量子化変換のように働く。その行列式を最大化することはMoment Shadow Mapに格納されるデータのエントロピーに良い影響を持つ。故に、深度区間$[-1, 1]$の選択が確かに最適である。$m = 4$の場合、$[0, 1]$から$[-1, 1]$に変えると行列式は1024になる。これはおよそ$\log_2 1024 = 10$ビットのエントロピーの増加に対応する。

## 疎な量子化変換[Sparse Quantization Transform]

　元の量子化変換の適用に対するコストは重要であり、これはMoment Shadow Mapのテクセルごとやシェードされたフラグメントごとに密な4x4行列によるベクトルの乗算を必要とする。$[-1, 1]$での深度値に起因するモーメントの直接ストレージと比べて、この大局的に最適な変換は4.28ビットのエントロピー増加になる。
　我々は変換の最適化における探索空間をエントロピー的に大きく減らすことなく実質的に制限することができることを発見した。特に、以下の変換はエントロピーを4.21ビット増加させる。

$$
\Theta_4^* \left( \begin{array} \\
b_1 \\
b_2 \\
b_3 \\
b_4
\end{array} \right) := \left( \begin{array} \\
\frac{3}{2} & 0 & -2 & 0 \\
0 & 4 & 0 & -4 \\
\frac{\sqrt{3}}{2} & 0 & -\frac{2\sqrt{3}}{9} & 0 \\
0 & \frac{1}{2} & 0 & \frac{1}{2}
\end{array} \right) \cdot \left( \begin{array} \\
b_1 \\
b_2 \\
b_3 \\
b_4
\end{array} \right) + \left( \begin{array} \\
\frac{1}{2} \\
0 \\
\frac{1}{2} \\
0
\end{array} \right)
$$

故に、結果の品質は実践的には変化しないが、成分の半数が0であるので、この変換の適用は大まかに2倍速くなる。

## ワーストケースに対するバイアス[Biasing for the Worst Case]

TODO
