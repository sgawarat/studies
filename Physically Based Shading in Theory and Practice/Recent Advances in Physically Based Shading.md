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

### "Extracting Microfacet-based BRDF Parameters from Arbitrary Materials with Power Iterations", Dupuy et al., EGSR 2015

この論文では、microfacetモデルをベースに、F項とD項を計測したBRDFデータ表を用いる手法を提案している。G項はこのD項をもとにSmithの関数により求めるとしている。NDFは、構築段階でshape-invariantになり、slope空間で展開される。
著者らは、解析的(analytic)なNDFを計測したものにフィッティングすることを試みた結果、GGXが多くのケースでよい(reasonable)合致(fit)を見せることを発見した。

microfacet理論の予測値が計測値と合致するかを見てみると、誘電体(dielectrics)は展開されたdiffuse lobeとうまくマッチしないが、金属では一見の価値がある。その他、NDFについては矛盾らしい矛盾は見当たらないが、Fresnel項の計測値では予測値と異なり、glancing angles下で減少する様が見て取れる。
2012年のBrent Burleyが示した同様の調査では、ほとんどの材質で増加が見られる。しかし、中には70°前後にピークが来るものがいくつかあり、これらは著者らが示したものと同じ材質である可能性がある。


### "A Non-Parametric Factor Microfacet Model for Isotropic BRDFs", Bagher et al., SIGGRAPH 2016

この論文では、isotropicなBRDFに絞ることで、さらなるフィッティングを行っており、さらには、ランバート項を導入する手法も提案している。

\[
\rho_d + \rho_s \frac{F({\bf l}, {\bf h}) G({\bf l}, {\bf h}) G({\bf v}, {\bf h}) D({\bf h})}{({\bf n} \cdot {\bf l})({\bf n} \cdot {\bf v})}
\]

Dupuyらの手法と以上に、microfacet理論に縛られていない。著者らは、色チャネル別に分けたDとGに対してフィッティングを行うとともに、どの理論のモデルにも見られないspecular係数を導入している。G項に関しては、Dupuyらと同様に一般化したSmithの方法を用いてD項から導出する方法と、独自の曲線をフィッティングする方法、の2つを選択肢として提示している。

著者らは、独自のG項が、特にdiffuseな材質で、SmithのG項と比べてよくできていると主張している。これが正しいとすると、正確性の面でひとつ有意差が生まれることになる。

microfacet理論との潜在的な矛盾点として、GやDが色ごとに大幅に異なっているかどうかという点がある。これについては、異なることが分かっている一部の構造化材質(structured materials)以外では非常に似通っているように見える。

## 解析的マイクロファセットモデル(Analytic Microfacet Models)

### "genBRDF: Discovering New Analytic BRDFs with Genetic Programming", Brady et al., SIGGRAPH 2014

この論文では、遺伝的プログラミングを用いて計測した材質にフィットする解析的モデルを探し出すという手法を提案している。
いくつかの有望な結果が得られた中、少なくとも1つは多少の調整を経て議題に上がるようなモデルが出来上がった。

### NDF: Generalized Beckmann

\[
D_{\text{GB}}(\theta) = \frac{\gamma}{\pi\alpha^2\Gamma(1/\gamma)\cos^4\theta}e^{-\left( \frac{\tan^2\theta}{\alpha^2} \right)^\gamma}
\]

Generalized Beckmannは、genBRDFで生まれた結果の1つをベースにして、shape-invarianceを付け加えたものである。
$\gamma$は分布の尖度(kurtosis)を操作し、$\gamma = 1$のときは通常のBeckmannと同等になる。

### NDF: Hyper-Cauchy

Hyper-Cauchy分布は2006年にWellemsらにより発表されたもので、Generalized Beckmannのように、shape controlパラメータとroughnessパラメータを持ち、shape-invariantである。

Butler[^Butler14]によれば、Hyper-CauchyはMERLにおけるニッケルのBRDFに対してBeckmannよりもさらに良くフィットする。さらなる興味深い点として、Hyper-Cauchyにおいて$\gamma = 2$としたときの数式がGGXと非常に似通っており、GTRの代替としてうまく機能するかもしれない、たぶん。

## MicroflakeとMultiple Surface Scattering

### Microflakes

マイクロフレークは、衣類や繊維質の織物のような異方的構造を持つボリュームをモデル化するために2010年に初めて導入された。これは、粒子による散乱についての考え方を拡張したものである。

2015年にはHeitzら[^Heitz15]により、マイクロファセットにおけるGGXを全球に拡張したをマイクロフレークにおけるSGGX分布がもたらされた。これには、線形に補間できる、解析的に評価できる、可視法線の重点サンプリングが可能になる、という既存のマイクロフレーク分布を超える多くの優位性を持っている。

### Multiple Surface Scattering

マイクロファセット理論は、複数回に及ぶ表面上での散乱を考慮していない。
その失ったエネルギーを是正しようとする試みは、KelemenとSzirmay-KalosによるEurographics 2001の論文やJakobらによるSIGGRAPH 2014の論文など、これまでにいくつか存在している。

### "Multiple-Scattering Microfacet BSDFs with the Smith Model" Heitz et al., SIGGRAPH 2016

この論文では、ある特性を持ったマイクロフレークのボリュームとして表面をモデル化する。関与媒質をレンダリングする典型的な方法ならば、マイクロファセット上の複数回バウンスを効果的にレンダリングすることができる。とはいえ、このモデルは確率的であり、リアルタイムレンダリングには適さない。

### "Additional Progress Towards the Unification of Microfacet and Microflake Theories", Dupuy et al., EGSR 2016

この論文では、先の可変密度ボリュームの代わりに、半無限の均質(homogeneous)ボリュームを用いたマイクロフレークに基づく多重表面上散乱の導出を行う。

## Coarse Microgeometry

### "Discrete Stochastic Microfacet Models", Jakob et al., SIGGRAPH 2014

この論文では、通常の連続的な分布の代わりとして、散乱粒子の離散的な分布によるマイクロファセットモデルを用いる。粒子は、表面上のピクセル面積とマイクロファセットの方向の集合を考慮した四次元領域上に、時間的に一貫性のある方法により確率的に生成される。

## Multiscale BRDF

*multiscale BRDF* は、従来の無限小(infinitesimal)の点と光線の代わりに、patches of surfaceとcones of areaにより定義される重要な概念である。この概念は、実際にBRDFが実装される方法に近く、スケーリングに関するBRDFモデルの密接な関係を備えており、原義よりfundamentalであると考えられるのではないかと思われる。

### "Real-time Rendering of Procedural Multiscale Materials", Zirr & Kaplanyan, I3D 2016

この論文では、multiscale NDFを評価するためにリアルタイム処理に適したデータ構造を選定することで高速化を図った。

### "Multi-Scale Rendering of Scratched Materials using a Structured SV-BRDF Model", Raynond et al., SIGGRAPH 2016

この論文は、引っかき傷(scratches)に特化して、傷内部で複数回バウンスするものも含めた、さまざまなBRDFを事前計算する手法を提案している。データは2Dパラメータ化した上で保存され、local scratch densityと組み合わせて、空間的に変化する(spatially varying)BRDFを組み立てる。

## Physical Optics (Wave) Models

TODO

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
- https://www.slideshare.net/imagire/the-sggx-microflake-distribution

[^GGX]: https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf
[^GTR]: https://disney-animation.s3.amazonaws.com/library/s2012_pbs_disney_brdf_notes_v2.pdf
[^Heitz14]: http://jcgt.org/published/0003/02/03/paper.pdf
[^Gulbrandsen14]: http://jcgt.org/published/0003/04/03/
[^Butler14]: "Robust Categorization of Microfacet BRDF Models to Enable Flexible Application-specific BRDF Adaptation", Butler & Marciniak, Proc. SPIE 9205, Reflection, Scattering, and Diffraction from Surfaces IV, 2014
[Heitz15]: https://drive.google.com/file/d/0BzvWIdpUpRx_dXJIMk9rdEdrd00/view
