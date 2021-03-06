---
title: PBR Diffuse Lighting for GGX+Smith Microsurfaces [@Hammon2017]
bibliography: bibliography.bib
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
    - Lambertianディフューズ: $\color{skyblue}{\rho_m(L, V, m) = \frac{1}{\pi}}$
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
    - 相関のない vs 高さ相関のある$G$
    - Smithのシャドウイング/マスキング
    - 新しいSmith+GGXの$G_2$近似
    - Smithの偉大さと奇妙さ
- パストレーシング

# ディフューズシミュレーションのサブトピックマップ

- シャドウイング/マスキング関数 ($G_1$, $G_2$)
    - <font color='skyblue'>相関のない vs 高さ相関のある$G$</font>
    - Smithのシャドウイング/マスキング
    - 新しいSmith+GGXの$G_2$近似
    - Smithの偉大さと奇妙さ
- パストレーシング

# 高さ相関のある$G$

- $G_n$は$n$個の方向からの幾何的な可視性である
- 相関がない場合:
    - $G_2(L, V, m) = G_1(L, m) G_1(V, m)$
    - 現実的じゃない！点が高い位置にあるほど、$L$と$V$の両方から見える可能性が大きくなる(低ければ、可能性は小さくなる)
    - 依然として、実践では、驚くほど優秀である

# 高さ相関のある$G$

- 相関のない$G$はハイトフィールドで法線に当たる光を取り…

# 高さ相関のある$G$

- …そして、その法線を持つ各マイクロファセットのいたるところに均等にばらまく

# 高さ相関のある$G$

- これは光を小さくする傾向にあるので、その可視性を減少したり、スペキュラを暗くしたりする

# 高さ相関のある$G$

- 相関のない$G$の誤差は遮蔽に関係している
- サーフェスが粗くなるほど、誤差が大きくなる
- $L$と$V$がよりglancing[表面すれすれ]であるほど、誤差が大きくなる
    - $L = N$および/または$V = N$ならば誤差なし

# 相関のない vs 相関のある$G$

高さ相関あり(下)は粗いサーフェスでのglancing反射を増幅する

黒のアルベド
光の強度 = $\pi$

# 相関のない vs 相関のある$G$

# 厳密 vs 近似の相関のある$G$

近似(下)はかなり優秀だが、依然として中くらいの角度とラフネスでほんの少し暗くなりすぎている

# 厳密 vs 近似の相関のある$G$

#

高さ相関のある$G$ 相関のない$G$ 差異

# 相関のある$G$

- 角度相関も存在する
- $L = V$のときに必要: $G_2(V, V, m) = G_1(V, m)$
- 相関のない形式: $G_2(V, V, m) = G_1(V, m)^2$
- 高さ相関のある$G_2$では$L = V$の内のどこか[?Height correlated G_2 somewhere in between when L = V]

# ディフューズシミュレーションのサブトピックマップ

- シャドウイング/マスキング関数 ($G_1$, $G_2$)
    - 相関のない vs 高さ相関$G$
    - <font color='skyblue'>Smithのシャドウイング/マスキング</font>
    - 新しいSmith+GGXの$G_2$近似
    - Smithの偉大さと奇妙さ
- パストレーシング

# Smithのシャドウイング/マスキング

- すべての法線が平等に遮蔽されると仮定する
    - すなわち、$G_1$と$G_2$は$m$に依存しない
    - 可能な限り最もバランスの取れた仮定

# Smithのシャドウイング/マスキング

- 正規化制約から導出できる:
    - $G_1(V) \int_\Omega D(m) \langle m \cdot V \rangle dm = |N \cdot V|$
- 確率的ハイトフィールドのレイトレーシングからも導出できる

# Smithのシャドウイング/マスキング

- 超基本的なレイトレース導出:
    - PDF$P_{22}(p, q)$によって2D平面に$m$を射影する
    - $D(m)$は等方的なので、PDF$P_2(q)$による1Dスライスを用いる
    - 傾き$\mu$を持つレイがハイトフィールドをウォークしつつ、レイ-サーフェスのコリジョンのPDFを得るために$P_2(q)$を用いる
    - $G_1$を得るためにコリジョンのPDFを用いる

# Smithのシャドウイング/マスキング

極[polar]の法線$m$、PDF$D(m)$

# Smithのシャドウイング/マスキング

<font color='skyblue'>2Dの傾き$p, q$、PDF$P_{22}(p, q)$</font>

<font color='skyblue'>$(p, q, 1)$</font>

極[polar]の法線$m$、PDF$D(m)$

# Smithのシャドウイング/マスキング

<font color='skyblue'>2Dの傾き$p, q$、PDF$P_{22}(p, q)$</font>

<font color='indianred'>$q$</font>

<font color='lightgreen'>$q$に対するすべての$p$</font>

<font color='skyblue'>$(p, q, 1)$</font>

極[polar]の法線$m$、PDF$D(m)$

# Smithのシャドウイング/マスキング

<font color='skyblue'>2Dの傾き$p, q$、PDF$P_{22}(p, q)$</font>

<font color='indianred'>$q$　　　1Dの傾き$q$、PDF$P_2(q)$</font>

<font color='lightgreen'>$q$に対するすべての$p$</font>

<font color='skyblue'>$(p, q, 1)$</font>

極[polar]の法線$m$、PDF$D(m)$

# Smith: 任意の$D(m)$

- $P_{22}(p, q) = \cos^4 \theta_m D(m)$
- $P_2(q) = \int_{-\infty}^{\infty} P_{22}(p, q) dp$
- $\Lambda(\mu) = \frac{1}{\mu} \int_{\mu}^{\infty} (q - \mu) P_2(q) dq$
- $G_1(V) = \frac{1}{1 + \Lambda(V)}, G_2(L, V) = \frac{1}{1 + \Lambda(L) + \Lambda(V)}$

# Smith: 相関あり vs 相関なし

- $G_1(V) = \frac{1}{1+\Lambda(V)}$
- 相関あり: $G_2(L, V) = \frac{1}{1+\Lambda(L)+\Lambda(V)}$
- 相関なし: $G_2(L, V) = \frac{1}{1+\Lambda(L)+\Lambda(V)+\color{indianred}{\Lambda(L)\Lambda(V)}}$
    - $L$または$V$に対して$\Lambda = 0$(すなわち、$G_1 = 1$)でない限り、小さくなりすぎる

# GGXのSmith: $\Lambda(V)$

- GGXでは: $D(m) = \frac{\alpha^2}{\pi (\cos^4 \theta_m (\alpha^2 + \tan^2 \theta_m)^2)}$
- $P_{22}(p, q) = \frac{\alpha^2}{\pi (\alpha^2 + \tan^2 \theta_m)^2} = \frac{\alpha^2}{\pi (\alpha^2 + p^2 + q^2)^2}$
- $P_2(q) = \int_{-\infty}^{\infty} \frac{\alpha^2}{\pi (\alpha^2 + p^2 + q^2)^2} dp = \frac{\alpha^2}{2 (\alpha^2 + q^2)^{3/2}}$

# GGXのSmith: $\Lambda(V)$

- $\Lambda(\mu) = \frac{1}{\mu} \int_{\mu}^{\infty} (q - \mu) P_2(q) dq = \frac{1}{2} \left( \frac{\sqrt{\alpha^2 + \mu^2}}{\mu} - 1 \right)$
- $\mu = \cot \theta_V$
- $\cos \theta_V = N \cdot V$
- $\Lambda(V) = \frac{1}{2} \left( \frac{\sqrt{\alpha^2 + (1 - \alpha^2) (N \cdot V)^2}}{N \cdot V} - 1 \right)$

# GGXのSmith: $G_1(V), G_2(L, V)$

- $G_1(V) = \frac{2 N \cdot V}{\sqrt{\alpha^2 + (1 - \alpha^2) (N \cdot V)^2} + N \cdot V}$
- $G_2(L, V) = \frac{2(N \cdot L)(N \cdot V)}{N \cdot V \sqrt{\alpha^2 + (1 - \alpha^2) (N \cdot L)^2} + N \cdot L \sqrt{\alpha^2 + (1 - \alpha^2) (N \cdot V)^2}}$
    - もっと安価な近似にしたい！

# ディフューズシミュレーションのサブトピックマップ

- シャドウイング/マスキング関数 ($G_1$, $G_2$)
    - 相関のない vs 高さ相関のある$G$
    - Smithのシャドウイング/マスキング
    - <font color='skyblue'>新しいSmith+GGXの$G_2$近似</font>
    - Smithの偉大さと奇妙さ
- パストレーシング

# Smith: 近似GGX $G_1(V)$

- $G_1$の分母:
    - $\sqrt{\alpha^2 + (1 - \alpha^2) (N \cdot V)^2} + N \cdot V$
    - $\sqrt{\text{lerp}((N \cdot V)^2, 1, \alpha^2)} + N \cdot V$
    - 近似: $\text{lerp}(N \cdot V, 1, \alpha) + N \cdot V$

# Smith: 近似GGX $G_1(V)$

- $G_1(V) \approx \frac{2 N \cdot V}{\text{lerp}(N \cdot V, 1, \alpha) + N \cdot V} = \frac{2 N \cdot V}{N \cdot V (2 - \alpha) + \alpha}$
- UnrealのSmithと同一と判明:
    - $G_1(V) \approx \frac{N \cdot V}{N \cdot V (1 - k) + k}, k = \frac{\alpha}{2}$

# Smith: 近似GGX $G_2(L, V)$

- $\Lambda(V)$に対するこの$G_1$を解いて、$G_2(L, V)$に接続する:
    - $G_2(L, V) = \frac{2|N \cdot V| |N \cdot V|}{\text{lerp}(2|N \cdot L| |N \cdot V|, |N \cdot L| + |N \cdot V|, \alpha)}$
    - $G_2$の分子は完全なスペキュラBRDFでは打ち消される:
        - $\frac{F(L, H) D(H) G_2(L, V)}{4|N \cdot L| |N \cdot V|} = \frac{F(L, H) D(H)}{2 \text{lerp}(2|N \cdot L| |N \cdot V|, |N \cdot L| + |N \cdot V|, \alpha)}$

# Smithの近似のコスト

- 分母のコストを比較する:
    - $G_1(L)G_1(V)$: $\frac{F(L, H) D(H)}{(|N \cdot L| (2 - \alpha) + \alpha)(|N \cdot V| (2 - \alpha) + \alpha)}$　約4サイクル
    - $G_2(L, V)$: $\frac{F(L, H) D(H)}{2 \text{lerp}(2|N \cdot L| |N \cdot V|, |N \cdot L| + |N \cdot V|, \alpha)}$　約6サイクル
        - コストは$N \cdot L$と$N \cdot V$の計算を除く
        - 高さ相関のある形式は無視できる追加コストを持つ

# Smithの近似のクオリティ

- glancing角での粗い誘電体に役立つ

# Smithの近似のクオリティ

- glancing角での粗い誘電体に役立つ

#

相関のない$G$
近似

差の画像:

赤 = 相関
緑 = 近似

#

相関のある$G$
近似

差の画像:

相関のない近似との比較

#

相関のある$G$
厳密

差の画像:

相関のある近似との比較

# ディフューズシミュレーションのサブトピックマップ

- シャドウイング/マスキング関数 ($G_1$, $G_2$)
    - 相関のない vs 高さ相関のある$G$
    - Smithのシャドウイング/マスキング
    - 新しいSmith+GGXの$G_2$近似
    - <font color='skyblue'>Smithの偉大さと奇妙さ</font>
- パストレーシング

# Smithのマイクロサーフェスのハイトフィールド

[@Walter2007]におけるSmithのレイトレーシング導出の図から1Dハイトフィールドを例示する

# Smithのマイクロサーフェスのハイトフィールド

- 導出は幅が0に近いほとんど独立したスラブの1Dハイトフィールド**も**使う
    - 突如としてハイトフィールドの下にあることのみを禁止する

# Smithのマスキングが奇妙な理由

- レイトレーシング導出は異なる段階で矛盾する仮定を持つ:
    - 次の$d\tau$における高さはこの高さから独立している
        - 連続でないと仮定している
    - ハイトフィールドは任意の微分可能な関数である
        - 連続であると仮定している

# Smithのマスキングが奇妙な理由

- 計算曰く、可視性は非対称である: 下向きのレイは上向きのレイより同じハイトフィールド経路を生き延びる可能性が低い！
    - $\Lambda(\mu) = \frac{1}{\mu} \int_{\mu}^{\infty} (q - \mu) P_2(q) dq$
    - $\Lambda(\mu)$はすべての$q > \mu$を積分するので、$\mu < 0$は$\mu > 0$のときより多くの$q$の値にヒットする可能性がある

# Smithのマスキングが素晴らしい理由

- すべてのファセットの法線が同じ可視割合を持つ所では$G$はエネルギー保存則を満たす
- その他の$m$を使わない$G$ではいくつかの方向で間違った可視面積を得る
    - 可視面積が大きければ、反射が強くなるので、エネルギーを作る
    - 可視面積が小さければ、反射が弱くなるので、エネルギーを吸収する

# ディフューズシミュレーションのサブトピックマップ

- シャドウイング/マスキング関数 ($G_1$, $G_2$)
    - 相関のない vs 高さ相関のある$G$
    - Smithのシャドウイング/マスキング
    - 新しいSmith+GGXの$G_2$近似
    - Smithの偉大さと奇妙さ
- <font color='skyblue'>パストレーシング</font>

# パストレースによるディフューズの解法

- Smithの奇妙な点は現実のハイトフィールドがその仮定と一致するのを妨げる
    - 連続的であり、かつ、非連続的であることは不可能である
- 数学モデルをレイトレースしなければならない
    - 数多の詳細はボーナススライドを参照

# 最初のレイトレース結果

- GGXスペキュラかLambertianディフューズを選ぶためのFresnel付きの単純なレイトレーサー
- 結果のBRDFは対称ではなかった！
    - $\rho(L, V, N) = \rho(V, L, N)$
- 何か間違ったかな？両部分は対称なBRDFなのに！

# 非対称なBRDFの原因

- 本質的には、マージされたBRDFを持っていた:
    - $\rho = F(L, N) \rho_{spec} + (1 - F(L, N)) \rho_{diff}$
    - Fresnel補間が非対称！
- 物理的にもっともらしい方法で直すには？

# なぜLambertianディフューズなのか？

- Lambertianディフューズは何をシミュレートする？
    - BRDF$\rho = \frac{1}{\pi}$: すべての視点で同じ
    - 放射輝度$= \rho \cos\theta_V$: 法線でより多くのフォトン
        - $V$から見える総表面積に対して$\frac{1}{\cos\theta_V}$でバランスを取る
        - なぜコサインでエネルギーが減少するのか？答えは驚くほどわかりにくいけど、かなり単純！

# Lambertianディフューズの説明

- BRDFはサーフェスに与えられるが、ディフューズ光は単に表面を通り過ぎる
    - Lambertは内部の光がすべての方向に同じ密度を持つことを仮定する
    - コサインでの減少は光方向に相対的なサーフェス角度に起因する

# Lambertianディフューズの説明

- 方向ごとに<font color="indianred">面積</font>あたりの同じエネルギー
- 表面の方を向いた方向がより大きな<font color='skyblue'>面積</font>に投影される
    - $1 / \cos\theta$でスケールされる単位光あたりの面積
    - $\cos\theta$でスケールされる面積あたりの光

# Lambertianディフューズの説明

- 光は侵入[enter]し、周囲でバウンスし、脱出[exit]する
- 脱出方向は多数のバウンス事象の後に無作為である
- アルベドは周波数依存の吸収事象に起因する

# 対称なBRDFのためのディフューズ修正

- 侵入には反射/透過のためにFresnelを使う
- 脱出は常に透過すると仮定する
- 脱出にもFresnelが必要！

# 対称なBRDFのためのディフューズ修正

- 反射: $F = F_0 + (1 - F_0)(1 - N \cdot V)^5$
- 透過: $1 - F = (1 - F_0)(1 - (1 - N \cdot V)^5)$
- Fresnelの法則は対称なので、視点からサーフェスに侵入する割合は視点に向かってサーフェスを脱出する割合に等しい

# 対称なBRDFのためのディフューズ修正

- 内部的に反射する光はずっと透過する機会を得続けるので、正規化する必要がある！
    - $\color{skyblue}{2\pi} \int_0^{\pi/2} k(1-(1-\cos\theta)^5) \color{indianred}{\cos\theta} \color{skyblue}{\sin\theta d\theta} = 1$
        - 因子$(1 - F_0)$は正規化因子$k$に吸収される
        - $\color{indianred}{\cos\theta}$はBRDFを正規化するのに必要になる
        - $\color{skyblue}{2\pi}$と$\color{skyblue}{\sin\theta d\theta}$は半球上の積分に由来する

# 対称なBRDFのためのディフューズ修正

- <u>厳密に</u>簡単に解ける: $k = \frac{21}{20\pi} = \frac{1.05}{\pi}$
- マージされたディフューズ＋スペキュラマイクロファセットBRDF:
    - $F = F_0 + (1-F_0)(1-N \cdot V)^5$
    - $\rho = F \rho_{spec}+(1-F)\frac{1.05}{\pi}(1-(1-N \cdot V)^5)$

# おわりに！

- パストレーシングシミュレーションに必要なすべてが揃ったので、最終的にどうなったかというと…

#

シミュレーション
スペキュラ付き

アルベド:
{0.75, 0.5, 0.25}

#

シミュレーション
ディフューズのみ

アルベド:
{0.75, 0.5, 0.25}

#

近似
ディフューズのみ

アルベド:
{0.75, 0.5, 0.25}

# GGXディフューズ近似

- $facing = 0.5 + 0.5 L \cdot V$
- $rough = facing (0.9 - 0.4 facing) (\frac{0.5+N \cdot H}{N \cdot H})$
- $smooth = 1.05 (1-(1-N \cdot L)^5)(1-(1-N \cdot V)^5)$
- $single = \frac{1}{\pi} \text{lerp}(smooth, rough, \alpha)$
- $multi = 0.1159 \alpha$
- $diffuse = albedo * (single + albedo * multi)$

# 余談: 有用なシェーダ恒等式

- $|L + V|^2 = 2 + 2L \cdot V$
    - $0.5 + 0.5 L \cdot V = \frac{1}{4}|L+V|^2$
- $N \cdot H = \frac{N\cdot L+L\cdot V}{|L+V|}$
- $L \cdot H = V \cdot H = \frac{1}{2}|L+V|$

# 余談: 有用なシェーダ恒等式

- $H$を求めなくても$N \cdot H$と$L \cdot H$は求められる！

|計算|サイクル|レジスタ|
|-|-|-|
|$H = \text{normalize}(L+V)$を得る|13|4|
|$H$から$N \cdot H$を得る|16|4|
|$H$から$N \cdot H$と$L \cdot H$を得る|19|4|
|恒等式から$N \cdot H$を得る|7*|2|
|恒等式から$L \cdot H$と$V \cdot H$を得る|8*|2|

\* $L \cdot V$が計算済みでない場合、その分の+3サイクルが加わる

# 余談: 有用なシェーダ恒等式

- $lenSq_{LV} = 2+2 L \cdot V$
- $rcpLen_{LV} = rsqrt(lenSq_{LV})$
- $N \cdot H = (H \cdot L + N \cdot V) * rcpLen_{LV}$
- $L \cdot H = V \cdot H = rcpLen_{LV} + rcpLen_{LV} * L \cdot V$
    - ($L \cdot H = \frac{1}{2}|L + V| = \frac{1}{2}\sqrt{2+2L \cdot V} = \frac{1}{2} \left( \frac{2+2L \cdot V}{\sqrt{2+2L \cdot V}} \right) = (1+L \cdot V)\frac{1}{\sqrt{2+2L \cdot V}}$による)

# 本日のトークのロードマップ

- 一般的なマイクロファセットベースBRDF
- GGX+Smithマイクロファセットモデルによるディフューズのシミュレーション
    - シャドウイング/マスキング関数
    - パストレーシング
- <font color='skyblue'>他のディフューズBRDFとの比較</font>

# でも、まずは…

- DisneyのBRDFスライスをサクッと理解するのが良い

# DisneyのBRDFスライス

- BRDFは2つの極ベクトルの4D関数である
- 以前は、光線と視線のベクトル: $\theta_l, \phi_l, \theta_v, \phi_v$
- 以後は、半角と差分: $\theta_h, \phi_h, \theta_d, \phi_d$
    - 等方的なBRDFでは$\phi_h$に<u>一切</u>依存しない
    - $\phi_d$への依存はしばしば無視できる

# DisneyのBRDFスライスの直観

- 各行は光線と視線の組である($\theta_d$)
    - 上端で反対、中央で垂直、下端で一致
- 左から右へはスペキュラハイライトの中心から離れることでの減衰を示す($\theta_h$)

# 照らされた球での間違った色の例

明るい線は球で使われる行を強調する

明るい線は$\phi_d = 90^\circ$を強調する
$\phi_d$は反時計回りに増加する

# DisneyのBRDFスライス

# 球上での$\theta_h, \theta_d, \phi_d$の振る舞い

# DisneyのBRDFスライス

- 様々な恒等式:
    - $\cos\theta_h = N \cdot H$
    - $\cos\theta_d = L \cdot H = V \cdot H$　$\cos 2\theta_d = L \cdot V$
    - $\cos\phi_d = \frac{N \cdot V - N \cdot L}{\sqrt{(2 - 2 L \cdot V) (1 - (N \cdot H)^2)}}$
- BRDFは主に$N \cdot H$と$L \cdot V$の関数である！

# BRDFを比較する準備はほぼ完了！

- まずは、比較フォーマットを導入する

#

新しいモデル
↑
タイトルはどのディフューズモデルが示されるかを表す。
このintroは新しいモデルを用いる。

#

新しいモデル

←このパネルは前の例と同様の照らされた球を示す。

#

新しいモデル


←一致するBRDFスライスがここに示される
←(相関のない$G$)
←

#

　　　　　　　　　　　　　　　　　　　　　　　　　↗ uffizi
0から1への$\alpha$を持つ同様の完全なBRDFだが、→ grace
Paul DebevecのHDR環境プロブで照らされる　　　↘ campus

#

Lambert

#

Disney

#

Oren-Nayar、$\sigma = 0.5\alpha$

#

新しいモデル

#

新しいモデル(ハイブリッド)

SmoothはDisneyの$f_{d90} = 0.5$を用いるので、$\alpha = 0$のときはDisneyと同様になる。

#

新しいモデル(安価)

SmoothはLambertを使用

#

Lambert

#

Disney

#

新しいモデル

#

Lambert

#

Disney

#

新しいモデル

#

Lambert

#

Disney

#

新しいモデル

# スペシャルサンクス

- Mark Cerny
    - GDCメンター
- Chad Barb、Xin Liu、Steve Marton
    - この研究に素晴らしいアイデア/フィードバックをくれたRespawnの同僚のエンジニア(いつも通りに)

# 選抜した参考文献

- [@Heitz2014]
- [@Walter2007]
- [@Burley2012]
- [@Karis2013]
- [@Shirley1997]
- [@Ashikmin2000]
- [@Oren1994]

# ご質問は？

# 付録

- 以下はレイトレーシング定式化からのSmithのシャドウイング/マスキングに対する導出のまとめであり、パストレーシングを実際に行うのにそれを使う方法である。これは私が先のプレゼンテーションに含まれる結果を得た方法である。
- これはかなり仰々しい数式である。なので、トークより付録にあるほうがふさわしい。聞き手が導出を読むのは難しいし、聴くのはもっと難しい！こういうのは自身のペースで進めることができるほうが良いし、必要に応じて振り返って見直しできるほうが良い。
- 最終警告: これらの付録スライドはそんなに推敲しなかった(例えば、図が完全に欠落していたり)。それでも、この情報と導出は由来を理解するのが好きな人やSmithのシャドウイング/マスキングの導出を理解したい人には有用だろう。

# 解法 --- パストレーシング

- 光の方向でマイクロサーフェスにフォトンを放つ
- これらのフォトンがどの視線方向に現れるかを確認する
- これはディフューズとスペキュラの相互作用をモデル化させる
- だが、はじめに、マイクロサーフェスをレイトレースできるようにしなければならない
- マイクロサーフェスは法線分布関数$D(m)$とシャドウイング/マスキング関数$G(L, V, N)$によって暗黙的に定義される
- $D(m)$から導出される$G(L, V, N)$は基本的にレイトレーシングである
- なので、Smithの$G(L, V, N)$が動作する方法を理解する必要がある

# Smithの導出を開始

TODO
