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

TODO

## 英語表現

- state of the art
  - [形] 最新式の、最高水準の

## 参考文献

- http://d.hatena.ne.jp/hanecci/20130511/p1
- http://graphicrants.blogspot.jp/2013/08/specular-brdf-reference.html
- http://graphics.hatenablog.com/entry/2014/02/12/060548
- http://qiita.com/\_Pheema_/items/f1ffb2e38cc766e6e668

[^GGX]: https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf
[^GTR]: https://disney-animation.s3.amazonaws.com/library/s2012_pbs_disney_brdf_notes_v2.pdf
[^Heitz14]: http://jcgt.org/published/0003/02/03/paper.pdf
[^Gulbrandsen14]: http://jcgt.org/published/0003/04/03/
