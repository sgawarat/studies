# Recent Advances in Physically Based Shading

## 最新技術(State of the art)

最新のリアルタイムレンダリングでは、基本として拡散反射と鏡面反射の2つを計算する。

### 鏡面反射

鏡面反射はマイクロファセットモデルに従う。

\[
f_{\text{specular}}({\bf l}, {\bf v}) = \sum_{i = 0}^n k_i \frac{F_i({\bf l}, {\bf h}) G_i({\bf l}, {\bf v}, {\bf h}) D_i({\bf h})}{4({\bf n} \cdot {\bf l})({\bf n} \cdot {\bf v})}
\]

法線分布項Dには、GGX(Trowbridge-Reitz)モデル[^GGX]やそれを一般化したGTRモデル[^GTR]がよく用いられる。
ここでは、ラフネスに異方性を持たせることもできる。

\[
D_{\text{TR}}({\bf h}, \alpha_x, \alpha_y)
\]

幾何減衰項Gには、height-correlatedなSmithの式[^Heitz14]が用いられる。

\[
G_i({\bf l}, {\bf v}, {\bf h}) = \frac{\chi^+({\bf l} \cdot {\bf h}) \chi^+({\bf v} \cdot {\bf h})}{1 + \Lambda({\bf l}) + \Lambda({\bf v})}
\]

フレネル項Fには、Schlickの近似式が用いられる。
アーティストが調整しやすいように、色味から物理パラメータを計算する手法も提案されている。[^Gulbrandsen14]

### 拡散反射

多くでLambertをベースとした手法が用いられる。

## 問題と制限(Issues and Limitations)

### Coarse Microgeometry

こんにちのNDFはとてもきめ細かい(fine-grained)microgeometryをうまく表現できる。しかし、多くの表面はきめの荒い(coarse-grained)微小構造(microstructure)を持つため、複雑な"輝き(glint)"を見せるが、現在のモデルではこれを表現しきれない。

### Shape Control

GGXは比較的長いテールを持つが、多くの材質ではさらに長いテールが必要になる。そこでテールの長さを調節できるならばとても便利だし、実際にGTRがテールの長さを調節できる方法を提示したが、ここ数年を見てもあまり使われていない。
これは、形状の不変性(shape invariance)に起因していると思われる。

### Shape Invariance

2014年のEric Heitzの論文[^Heitz14]によれば、以下の形式に沿う場合、それはshape-invariantである、としている。これに従えば、GGXやBeckmannはshape-invariantであり、GTRはそうではない。

\[D(\theta_m, \alpha) = \frac{f(\frac{\tan\theta_m}{\alpha})}{\alpha^2\cos^4\theta_m}\]

二次元の傾斜(slope)分布で表すと以下のようになる。

\[
\begin{split}
P^{22}(x_{\tilde{m}}, y_{\tilde{m}}) &= D(\theta_m, \alpha) \cos^4\theta_m \\
&= \frac{1}{\alpha^2} f\left(\frac{\tan \theta_m}{\alpha}\right)
\end{split}
\]

つまり、slope空間ではラフネス$\alpha$のスケーリングに対して分布が線形に伸長することを示す。分布の伸長はmicrogeometryの伸長であるから、shape-invariantなNDFでは、ラフネスのスケーリングはその逆数倍でmicrogeometryが伸長することと等価であると言える。

NDFがshape-invariantであることには、多くの利点がある。

- anisotropicな形式への拡張が比較的容易になる。
- Smithのshadowing-masking関数の導出が比較的容易になる。
- 重点サンプリングの導出が比較的容易になる。

以上を踏まえて、GTRのようなshapeを変化させるパラメータを持ち、なおかつ、ラフネスに関してshape-invariantなNDFが望まれる。現状、このようなNDFはproduction use(仕事で使うような品質のもの)では存在しない。

### Microfacet Theory

microfacet理論は、表面の物理特性からそのままBRDFの数式を導くelegantな(スッキリと簡潔にまとめられた)ツールであるが、それ自体が思い切った仮定のもとに成り立っているため、シェーディングモデルの現実感や正確性に制約をかけてしまっている。

例えば、microfacet理論ではG項は、入射光を遮蔽するshadowingと反射光を遮蔽するmaskingを考慮するが、microfacet内で複数回反射する光を考慮しておらず、その分のエネルギー損失により若干暗くなってしまう。これを補正するため、Disneyのモデルにおける"sheen"項のような物理的でないパラメータをを導入することもある。

また、microfacet理論の根本的な制約として、光を光線として扱う幾何光学(geometric optics)をもとにしていることが挙げられる。つまり、波としての性質を取り扱う物理工学(physical optics)的要素は取り入れられていない。

## データ駆動microfacetモデル(Data-Driven Microfacet Models)

Extracting Microfacet-based BRDF Parameters from Arbitrary Materials with Power Iterations, Dupuy et al., EGSR 2015

この論文では、microfacetモデルをベースに、F項とD項を計測したBRDFデータ表を用いる手法を提案している。G項はこのD項をもとにSmithの関数により求めるとしている。NDFは、構築段階でshape-invariantになり、slope空間で展開される。
著者らは、解析的(analytic)なNDFを計測したものにフィッティングすることを試みた結果、GGXが多くのケースでよい(reasonable)合致(fit)を見せることを発見した。

microfacet理論の予測値が計測値と合致するかを見てみると、誘電体(dielectrics)は展開されたdiffuse lobeとうまくマッチしないが、金属では一見の価値がある。その他、NDFについては矛盾らしい矛盾は見当たらないが、Fresnel項の計測値では予測値と異なり、glancing angles下で減少する様が見て取れる。
2012年のBrent Burleyが示した同様の調査では、ほとんどの材質で増加が見られる。しかし、中には70°前後にピークが来るものがいくつかあり、これらは著者らが示したものと同じ材質である可能性がある。

## 用語

テール(tail)
: 縦軸の値が0に近づいてゆく関数をグラフで見たときの形状を動物の尻尾に見立てた呼び方。
  横軸の値が大きくなっても、縦軸の値がなかなか0にならないことを、"長い(long)"と形容する。

NDF(Normal Distribution Function)
: マイクロファセットモデルのD項に用いられる、微視的な法線分布を表す関数。

誘電体(dielectrics)
: プラスチック、セラミックなど。
  3DCGでは、そもそもの意味よりかは単に、金属でないものという意味合いが強い。[?]

## 英語表現

- state of the art
  - [形] 最新式の、最高水準の

- tabulate /tˈæbjʊlèɪt(米国英語)/
  - [動] 表に作る、作表する

- worth doing
  - する価値があって

## 参考文献

- http://d.hatena.ne.jp/hanecci/20130511/p1
- http://graphicrants.blogspot.jp/2013/08/specular-brdf-reference.html
- http://graphics.hatenablog.com/entry/2014/02/12/060548
- http://qiita.com/\_Pheema_/items/f1ffb2e38cc766e6e668

[^GGX]: https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf
[^GTR]: https://disney-animation.s3.amazonaws.com/library/s2012_pbs_disney_brdf_notes_v2.pdf
[^Heitz14]: http://jcgt.org/published/0003/02/03/paper.pdf
[^Gulbrandsen14]: http://jcgt.org/published/0003/04/03/
