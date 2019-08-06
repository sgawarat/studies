# まえがき[Introduction]

TODO

## 基本的定義[Basic definitions]

我々の表記法は主にCohen-Or and Kaufman[@COK95]に従う。整数の座標を持つ3次元の点の集合を$\mathbb{Z}^3$する。我々は点$p(x, y, z) \in \mathbb{R}^3$の集合とボクセル$V = (x, y, z) \in \mathbb{Z}^3$を関連付ける。ここで、$V_x \le p_x \le V_x + 1$であり、$y$と$z$に対して同様である。$\mathbb{R}^3$に組み込まれた連続で単純で閉じた2次元多様体の表面$S$を考えるとすると、$\mathbb{R}^3 - S$は過不足なく[exactly]2つの連結成分[connected components]$I$および$O$を持つ。$I$と$O$の全体に含まれるボクセルの空でない集合をそれぞれ$I^d$と$O^d$とする
表面$S$のボクセル化とは$I^d$や$O^d$と共通の要素を持たないボクセルの離散集合$S^d$である。ボクセルの$k$連結経路[k-connected path]$\Pi_k = (V_0, \dots, V_n)$が存在しないならば、$S^d$は$k$分離[k-separating]であると言われる。ここで、$V_{i + 1} \in N_k(V_i)$、すなわち、$V_0 \in I^d$であり、$V_n \in O^d$であり、$\Pi_k \cap S^d \ne \emptyset$である。別の言い方をすれば、$I^d$と$O^d$の間のすべての$\Pi_k$が$S^d$にあるボクセルを含むならば、$S^d$は$S$の$k$分離ボクセル化である。$N_k(V)$はボクセル$V$の$k$近傍の集合であり、$\mathbb{Z}^3$において、我々にとって関心のある近傍集合は以下である。

$$
N_6(x, y, z) = (x \pm 1, y, z) \cup (x, y \pm 1, z) \cup (x, y, z \pm 1)
$$

$$
N_{26}(x, y, z) = {x - 1, x, x + 1} \times {y - 1, y, y + 1} \times {z - 1, z, z + 1} - (x, y, z)
$$

ここで、$\times$はデカルト積[cartesian product]を表す。ゆえに、任意の$V$に対して、$N_6(V)$は6個の要素を持ち、$N_{26}(V)$は26個の要素を持つ。したがって、6連結経路$\Pi_6$はボクセルの面を通って隣接するボクセルをたどる[walk into]ことができるのみであり、26連結経路$\Pi_{26}$ではそれに加えて[as well]対角線上にたどること[diagonal walks]が可能である。紙面の都合により[due to space constraints]、本論では18分離および18連結のボクセル化および経路を考えない[we shall not consider]。
2Dボクセル化においても状況は変わらず、この場合においては、我々はそれらに対応する意味で$S \subset \mathbb{R}^2$、$S^d \subset \mathbb{Z}^2$、$V$、$I$、$O$、$\Pi_k$を用いる。その次元は文脈により明らかとなるだろう。2Dにおける近傍関係[neighborship relations]は以下である。

$$
\begin{aligned}
N_4(x, y) &= (x \pm 1, y) \cup (x, y \pm 1) \\
N_8(x, y) &= {x - 1, x, x + 1} \times {y - 1, y, y + 1} - (x, y)
\end{aligned}
$$

ここで、$N_4$は3Dにおける$N_6$と類似し、$N_8$は$N_{26}$と類似している。近傍集合の図は、例えば、[@HYFK98]に見ることができる。
我々は、その表面が依然として$\mathbb{R}^3$において対応するボリュームを2つの連結成分に分けるようなボクセル空間$\mathbb{Z}^3$の部分集合を考慮することによって、閉じていない表面に分離可能性[separability]の概念を拡張できる。これについては[@fig:1]を参照のこと。閉じている表面と閉じていない表面のどちらでも、対応する$I^d$および$O^d$が空でない場合に制限していることに注意したい。そうでない場合、$\mathbb{Z}^3$で分離する手段が存在しないので、分離可能性を論じる意味がなくなってしまう[it makes no sense to speak of separability]。

![閉じていない表面への$k$分離可能性の概念の拡張。（a）$S$が依然として$\mathbb{R}^3$における対応するボリュームを2つの連結成分$I$および$O$に分離するような部分集合$\boldsymbol{z} \subset \mathbb{Z}^3$が存在する閉じていない表面$S$を考える。そして、対応する$I^d$および$O^d$は空ではない。（b）$\boldsymbol{z}$におけるボクセルに制限される$I^d$と$O^d$の間のすべての$\Pi_k$が$S^d$におけるボクセルを含むならば、$S^d$は$S$の$k$分離ボクセル化であると言える。そして、これは部分集合$\boldsymbol{z}$の任意の有効な選択に対して成り立つ[holds for]。この2Dの例では、$S^d$は8分離ではなく4分離である。](fig/1.png){#fig:1}

# 関連研究[Previous work]

TODO

# 交差対象を伴うボクセル化
