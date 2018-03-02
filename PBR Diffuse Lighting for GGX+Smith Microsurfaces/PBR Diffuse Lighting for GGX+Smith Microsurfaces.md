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

- $\color{indianred}{\int_\Omega} \rho_m(L, V, m) D(m)G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} \color{indianred}{dm}$
    - すべてのマイクロファセット法線上の積分

# 一般的なマイクロファセットBRDF

- $\int_\Omega \color{indianred}{\rho_m(L, V, m)} D(m)G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - 個々のファセットが光に反応する方法
        - すなわち、$m$に集まるマイクロファセットBRDF
        - 通常は理想的な鏡か理想的なディフューズ

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) \color{indianred}{D(m)} G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - 法線$m$の確率密度
    - 配置(形状)ではなく、どんなファセット法線が存在するか

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) D(m) \color{indianred}{G_2(L, V, m)} \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - マイクロファセットの実際の配置(形状)による遮蔽
        - シャドウイング/マスキング関数とも
    - マイクロファセット$m$が光$L$と視線$V$の両方から見える確率

# マイクロファセットBRDF: $G_2$対$G_1$

- <font color="indianred">$G_2(L, V, m)$</font>は2方向から見える割合
- <font color="skyblue">$G_1(V, m)$</font>は1方向から見える割合
    - 実際には、$G_2$は$G_1$から導出される

# マイクロファセットBRDF: $D(m)$の特性

- $\int_\Omega \rho_m(L, V, m) \color{indianred}{D(m)} G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
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

- $\int_\Omega \rho_m(L, V, m) \color{indianred}{D(m) G_2(L, V, m)} \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - 光と視点の両方から見えるマイクロファセット法線$m$がある確率密度
        - すなわち、BRDF$\rho_m(L, V, m)$を用いる確率密度

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) D(m) G_2(L, V, m) \color{indianred}{\frac{\langle m \cdot L \rangle}{|N \cdot L|}} \color{skyblue}{\frac{\langle m \cdot V \rangle}{|N \cdot V|}} dm$
    - $\color{indianred}{\frac{\langle m \cdot L \rangle}{|N \cdot L|}}$ --- ファセット$m$が光からどれだけ大きく見えるか
    - $\color{skyblue}{\frac{\langle m \cdot V \rangle}{|N \cdot V|}}$ --- ファセット$m$が視点からどれだけ大きく見えるか
        - すなわち、<font color='indianred'>光から</font>と<font color='skyblue'>視点へ</font>の寄与を正規化する

# 一般的なマイクロファセットBRDF

- $\int_\Omega \rho_m(L, V, m) \color{indianred}{D(m) G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|}} dm$
    - マイクロファセット法線$m$を1回だけバウンスして$V$に到達する$L$からの光の確率密度
    - 要件: $\int_\Omega \color{indianred}{D(m) G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|}} dm \le 1$
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

- マイクロファセットBRDFは完璧な鏡である
    - すなわち、光は$m = H$の場合にのみ反射する
        - 数学的には、BRDFはスケールされたDiracのデルタ$\delta_m(H, m)$である

# PBRスペキュラマイクロファセットBRDF

- 純粋な鏡のBRDF: $k\delta_m(H, m)$
    - $\delta_m(H, m)$は大きさ[measure]$m$を用いるDiracのデルタである
    - $k$は我々が見つけなければならない正規化ファクターである
- 正規化済みBRDF: $\int_\Omega \rho(L, V, N)\cos\theta_V dV = 1$
    - 任意の光と法線に対して、すべてのエネルギーは厳密に1つの視点に反射する

# スペキュラBRDFの正規化

- 一般的な場合: $\int_\Omega \color{skyblue}{\rho(L, V, N)}\cos\theta_V \color{indianred}{dV} = 1$
- 我々の場合: $\int_\Omega \color{skyblue}{k\delta_m(H, m)}\cos\theta_V \color{indianred}{\frac{dV}{dm} dm} = 1$
    - $\delta_m$を評価するために$dm$で積分しなければならないので、積分領域を変更するために$\frac{dV}{dm}$を見つける

# スペキュラBRDFの正規化

- $\frac{dV}{dm}$は、$m$に対して$V$がどれだけ早く変化するか、である
- これはPBRスペキュラの4を導入するだろう！
    - 以降のスライドはその方法を示す

# スペキュラBRDFの正規化

- $\frac{dV}{dm}$を得るために$dV$から$dm$を見つけたい
- まず、$\delta_m$は$m = H$を選び取るので、$dm = dH$である
    - すべてのベクトルは単位半球上に描かれる

# スペキュラBRDFの正規化

- 立体角$\color{lightgreen}{dV}$を移動して…

# スペキュラBRDFの正規化

- 立体角$\color{lightgreen}{dV}$を$L+V$に移動して…

# スペキュラBRDFの正規化

- 立体角$\color{lightgreen}{dV}$を移動して、スケールして…
    - $L+V$の球: $\color{indianred}{H \cdot V}$

# スペキュラBRDFの正規化

- 立体角$\color{lightgreen}{dV}$を移動して、スケールして…
    - $L+V$の球: $\color{indianred}{H \cdot V}$
    - 単位球: $\frac{4\pi \color{indianred}{1^2}}{4\pi \color{skyblue}{|L+V|^2}} = \frac{\color{indianred}{1}}{\color{skyblue}{|L+V|^2}}$

# スペキュラBRDFの正規化

- 立体角$\color{lightgreen}{dV}$を移動して、スケールして…
    - $L+V$の球: $\color{indianred}{H \cdot V}$
    - 単位球: $\frac{4\pi \color{indianred}{1^2}}{4\pi \color{skyblue}{|L+V|^2}} = \frac{\color{indianred}{1}}{\color{skyblue}{|L+V|^2}}$
- $dm = \frac{\color{indianred}{H \cdot V}}{\color{skyblue}{|L+V|^2}} \color{lightgreen}{dV}$

# スペキュラBRDFの正規化

- 立体角$\color{lightgreen}{dV}$を移動して、スケールして…
    - $L+V$の球: $\color{indianred}{H \cdot V}$
    - 単位球: $\frac{4\pi \color{indianred}{1^2}}{4\pi \color{skyblue}{|L+V|^2}} = \frac{\color{indianred}{1}}{\color{skyblue}{|L+V|^2}}$
- $dm = \frac{\color{indianred}{H \cdot V}}{\color{skyblue}{|L+V|^2}} \color{lightgreen}{dV}$

# スペキュラBRDFの正規化

- $|L+V| = H \cdot (L+V) = H \cdot L + H \cdot V = 2 H \cdot V$
    - この<font color='crimson'>2</font>の自乗がスペキュラBRDFの<font color='crimson'>4</font>である！
- $dm = \frac{H \cdot V}{|L+V|^2} dV = \frac{H \cdot V}{\color{crimson}{4} (H \cdot V)^2} dV$
- $\frac{dV}{dm} = \color{crimson}{4} H \cdot V$

# スペキュラBRDFの正規化

- $\int_\Omega k\delta_m(H, m) \color{skyblue}{\cos\theta_V} \color{indianred}{\frac{dV}{dm}} dm = 1$
    - $\int_\Omega k\delta_m(H, m) \color{skyblue}{(m \cdot V)} \color{indianred}{(4 H \cdot V)} dm = 1$
    - $\color{skyblue}{m = H}$かつ$\color{skyblue}{H \cdot V = H \cdot L}$なので、$k = \frac{1}{\color{crimson}{4} \color{skyblue}{(H \cdot L)} \color{indianred}{(H \cdot V)}}$
- よって、完璧な鏡のBRDF: $\frac{\delta_m(H, m)}{\color{crimson}{4} \color{skyblue}{(H \cdot L)} \color{indianred}{(H \cdot V)}}$

# スペキュラマイクロファセットBRDF

- 入射光のFresnel反射$F(L, m)$のみがスペキュラ反射を行う
- よって、最終的なスペキュラマイクロファセットBRDF:

\[
\rho_m(L, V, m) = F(L, m) \frac{\delta_H(m)}{\color{crimson}{4} |H \cdot L| |H \cdot V|}
\]

# スペキュラマイクロファセットBRDF

- $\int_\Omega \color{skyblue}{\rho_m(L, V, m)} D(m) G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
- $\int_\Omega \color{skyblue}{\frac{F(L, m) \delta_m(H, m)}{\color{crimson}{4} |H \cdot L| |H \cdot V|}} \color{lightgreen}{D(m) G_2(L, V, m)} \frac{\langle m \cdot L \rangle}{\color{lightgreen}{|N \cdot L|}} \frac{\langle m \cdot V \rangle}{\color{lightgreen}{|N \cdot V|}} dm$
    - $\delta_m(H, m)$の積分は、その積分記号を取り除いて$m$を$H$に置き換える、という操作を行う
- スペキュラBRDF: $\frac{\color{skyblue}{F(L, H)} \color{lightgreen}{D(H) G_2(L, V, H)}}{\color{crimson}{4} \color{lightgreen}{|N \cdot L| |N \cdot V}}$

# マイクロファセットBRDFのサブトピックマップ

- 一般的な形式
- そこからPBRスペキュラを得る方法
- <font color='skyblue'>ディフューズBRDFへの拡張</font>

# ディフューズマイクロファセットBRDF

- $\int_\Omega \color{skyblue}{\rho_m(L, V, m)} D(m) G_2(L, V, m) \frac{\langle m \cdot L \rangle}{|N \cdot L|} \frac{\langle m \cdot V \rangle}{|N \cdot V|} dm$
    - ランバートディフューズ: $\color{skyblue}{\rho_m(L, V, m) = \frac{1}{\pi}}$
    - 積分を取り除くためのDiracのデルタがない ☹️
    - GGX+Smithの閉形式の解がない ☹️☹️

# ディフューズマイクロファセットBRDF

- 数値的に積分を解くので、良い近似を見つけたい
    - Oren-Nayarの論文と同じアプローチ
- 最大半分の光が失われる！
    - 複数バウンスを無視できない…
        - (完全なOren-Nayarは二次バウンスも含む)

# 直接のみ

アルベド:
{0.75, 0.5, 0.25}

# 直接と間接

アルベド:
{0.75, 0.5, 0.25}

# 間接のみ

アルベド:
{0.75, 0.5, 0.25}

# 並べて表示

アルベド:
{0.75, 0.5, 0.25}

# 適切なアルベドの重要性

上:
正しい

下:
アルベド x 0.5、
光 x 2

# 適切なアルベドの重要性

上:
正しい

下:
アルベド x 2、
光 x 0.5

# 本日のトークのロードマップ

- <font color='indianred'>一般的なマイクロファセットベースBRDF</font>
- GGX+Smithマイクロファセットモデルによるディフューズのシミュレーション
- 他のディフューズBRDFとの比較

# 本日のトークのロードマップ

- 一般的なマイクロファセットベースBRDF
- <font color='skyblue'>GGX+Smithマイクロファセットモデルによるディフューズのシミュレーション</font>
    - シャドウイング/マスキング関数
    - パストレーシング
- 他のディフューズBRDFとの比較

# ディフューズシミュレーションのサブトピックマップ

- シャドウイング/マスキング関数 ($G_1$, $G_2$)
    - 無相関 対 高さ相関$G$
    - Smithのシャドウイング/マスキング
    - 新しいSmith+GGXの$G_2$近似
    - Smithの偉大さとおかしさ
- パストレーシング

# ディフューズシミュレーションのサブトピックマップ

- シャドウイング/マスキング関数 ($G_1$, $G_2$)
    - <font color='skyblue'>無相関 対 高さ相関$G$</font>
    - Smithのシャドウイング/マスキング
    - 新しいSmith+GGXの$G_2$近似
    - Smithの偉大さとおかしさ
- パストレーシング

# 高さ相関$G$

TODO
