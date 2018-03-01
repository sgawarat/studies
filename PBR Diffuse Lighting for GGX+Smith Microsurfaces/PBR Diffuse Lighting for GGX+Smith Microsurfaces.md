---
title: PBR Diffuse Lighting for GGX+Smith Microsurfaces [@Hammon2017]
---
# GGX+SmithマイクロサーフェスのためのPBRディフューズライティング

# 私はEarlという者ですが…

# Titanfall 2の開発中に行ったGGXディフューズの研究

# 主なおみやげ

- GGX+Smithスペキュラと同じ仮定から導出される新しいディフューズ方程式
- GGX+Smithスペキュラのための新しい安価で良好なシャドウイング/マスキング関数$G$
- シェーダ最適化のための新しい三角関数の公式

# 道すがらの面白い発見

- PBRスペキュラにおける"$4$"の理由は？
- Oren-Nayarディフューズにおける"$s$"とは？
- Smithのシャドウイング/マスキングの仮定
- Lambertの物理的解釈
- DisneyのBRDFの部分を解釈するのに役立つ

# ちょっとした余談: 元々のモチベーション

- Titanfall 2はOren-Nayarディフューズを用いた
- 疑問: GGXのラフネス$\alpha$からOren-Nayarのラフネス$s$を得るには？
- 発見: Orn-Nayarはまったく異なる仮定に由来する

# ちょっとした余談: 元々のモチベーション

||Oren-Nayar|GGX+Smith|
|-|-|-|
|シャドウイング/マスキング|V型空洞|Smith|
|法線分布|球面ガウシアン|GGX|
|ラフネスパラメータ|$s \in [0, \infty]$|$\alpha \in [0, 1]$|
|完全に平坦|$s = 0$|$\alpha = 0$|
|法線の傾き[slopes]の標準偏差|$s$|$\alpha = 0$なら$0$、$\alpha \ne 0$なら$\alpha^2 \infty$|

# ちょっとした余談: 元々のモチベーション

- Oren-NayarとSmith+GGXはマッチしない！
    - 標準偏差ですらマッチできない
    - ふむ…GGXの標準偏差は$\alpha^2\infty$
        - $\alpha^2$でミップマップ/フィルタするのがたぶん"最適"？
        - 2つのGGX分布の合計はGGXではないので、"適切"にミップマップ/フィルタできない

# 本日のトークのロードマップ

- 一般的なマイクロファセットベースBRDF
- GGX+Smithマイクロファセットモデルによるディフューズのシミュレーション
- 他のディフューズBRDFとの比較

# 本日のトークのロードマップ

- <font color='skyblue'>一般的なマイクロファセットベースBRDF</font>
- GGX+Smithマイクロファセットモデルによるディフューズのシミュレーション
- 他のディフューズBRDFとの比較

# マイクロファセットBRDFのサブトピックマップ

- 一般的な形式
- そこからPBRスペキュラを得る方法
- ディフューズBRDFへの拡張

# マイクロファセットBRDFのサブトピックマップ

- <font color='skyblue'>一般的な形式</font>
- そこからPBRスペキュラを得る方法
- ディフューズBRDFへの拡張

# マイクロファセットモデル

- 複雑な<font color="skyblue">マクロサーフェス</font>BRDFを大量の単純な<font color="indianred">マイクロファセット</font>BRDFで平均する
- 基本的には単なるサブピクセルシェーダアンチエイジング

# 現実世界の例

# "一般的"なマイクロファセットベースBRDF

- 完全に一般的ではなく、ハイトフィールドを仮定する
    - 編まれたもの[weaves]、半円形のもの[arches]、空洞のあるもの[caves]はない

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) D(m)G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$

# 一般的なマイクロファセットBRDF

- $\color{red}{\int_\Omega} \rho_m(L, V, m) D(m)G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} \color{red}{dm}$
    - すべてのマイクロファセット法線上の積分

# 一般的なマイクロファセットBRDF

- $\int_\Omega \color{red}{\rho_m(L, V, m)} D(m)G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - 個々のファセットが光に反応する方法
        - すなわち、$m$に集まるマイクロファセットBRDF
        - 通常は理想的な鏡か理想的なディフューズ

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) \color{red}{D(m)} G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - 法線$m$の確率密度
    - 配置(形状)ではなく、どんなファセット法線が存在するか

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) D(m) \color{red}{G_2(L, V, m)} \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - マイクロファセットの実際の配置(形状)による遮蔽
        - シャドウイング/マスキング関数とも
    - マイクロファセット$m$が光$L$と視線$V$の両方から見える確率

# マイクロファセットBRDF: $G_2$対$G_1$

- <font color="indianred">$G_2(L, V, m)$</font>は2方向から見える割合
- <font color="skyblue">$G_1(V, m)$</font>は1方向から見える割合
    - 実際には、$G_2$は$G_1$から導出される

# マイクロファセットBRDF: $D(m)$の特性

- $\int_\Omega \rho_m(L, V, m) \color{red}{D(m)} G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - 法線$m$の確率<u>密度</u>
        - 累積確率が$m$近くでどれだけ素早く<u>変化</u>するか
            - より可能性の高い領域ではより素早く変化するだろう
        - 範囲$[0, \infty]$であり、$[0, 1]$ではない！
            - その確率が0でないいずれかの$m$に対して$D(m) = \infty$！

# マイクロファセットBRDF: $D(m)$の特性

- $\int_\Omega D(m) dm = ?$
    - すべてのマイクロファセットの総表面積
    - 少しでもラフネスがあれば、常に1以上になる！
- $\int_\Omega D(m) \cos \theta_m dm = 1$
    - 総面積を正規化するため、$\cos \theta_m = m \cdot N$を用いてマイクロファセットをマクロサーフェスに射影する

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) \color{red}{D(m) G_2(L, V, m)} \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - 光と視点の両方から見えるマイクロファセット法線$m$がある確率密度
        - すなわち、BRDF$\rho_m(L, V, m)$を用いる確率密度

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) D(m) G_2(L, V, m) \color{indianred}{\frac{\langle m \cdot L \rangle}{|N \cdot L|}} \color{skyblue}{\frac{\langle m \cdot V \rangle}{|N \cdot V|}} dm$
    - $\color{indianred}{\frac{\langle m \cdot L \rangle}{|N \cdot L|}}$ --- ファセット$m$が光からどれだけ大きく見えるか
    - $\color{skyblue}{\frac{\langle m \cdot V \rangle}{|N \cdot V|}}$ --- ファセット$m$が視点からどれだけ大きく見えるか
        - すなわち、<font color='indianred'>光から</font>と<font color='skyblue'>視点へ</font>の寄与を正規化する

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) \color{red}{D(m) G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|}} dm$
    - マイクロファセット法線$m$を1回だけバウンスして$V$に到達する$L$からの光の確率密度
    - 要件: $\int_\Omega \color{red}{D(m) G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|}} dm \le 1$
        - 平坦な$D(m)$でのみ1になる --- **いずれのラフネスでも暗くなりすぎる！**

# 一般的なマイクロファセットBRDF

- 関連する要件:
    - $\int_\Omega D(m) G_1(V, m) \langle m \cdot V \rangle dm = |N \cdot V|$
    - 任意の方向$V$において、見えているマイクロファセットの総面積はマクロサーフェスの面積に等しい

# マイクロファセットBRDFのサブトピックマップ

- 一般的な形式
- <font color='skyblue'>そこからPBRスペキュラを得る方法</font>
    - <font color='skyblue'>$\frac{F(L, H) D(H) G_2(L, V, H)}{4|N \cdot L||N \cdot V|}$</font>
    - <font color='skyblue'>それと: $4$はどこから来たのか？$\pi$ではいけない理由は？</font>
- ディフューズBRDFへの拡張

# PBRスペキュラマイクロファセットBRDF

TODO
