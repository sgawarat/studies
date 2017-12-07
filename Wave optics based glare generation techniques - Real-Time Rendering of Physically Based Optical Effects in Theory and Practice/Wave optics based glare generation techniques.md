---
title: Wave optics based glare generation techniques [@Kakimoto2015c]
numberSections: false
---
# はじめに[Introduction]

- コンピュータグラフィクス理論の大多数は幾何光学に基づいている。
- 波動光学は1%程度が計算に入っている。
- グレア効果にリアリティが必要なら、
    - 波動光学が役立つだろう。
- こんにちの計算パワーは有利[advantageous]である。

# 波関連の現象と効果[Wave-Related Phenomena and Effects]

<!-- 左側 -->

- 回折[diffraction]
    - グレア
    - エアリーディスク[airy disc]
- 干渉[Interference]
    - 表面塗装[surface coating]
    - 薄膜の色効果[thin film color effects]
- 偏光[polarization]
    - 複雑な反射
    - 画像の霞除去[dehazing]
- このコースの波動光学トピックは回折に焦点を当てる。

<!-- 右側 -->

- 波動光学を必須とする。
- 拡張レイでシミュレートできない。
- 拡張レイ理論でシミュレートできる。[@Cook1981; @Gondek1994; @Wolff1999; @Schechner2001]

# グレアの例[An Example of Glare]

# グレアの単純な実験 (1)[A Simple Experiment of Glare (1)]

- 実験に使ったペンライト
- 光の直接スナップショット

# グレアの単純な実験 (2)[A Simple Experiment of Glare (2)]

- 付けた偽のまつげ[eyelash]
- 光の直接スナップショット

# グレアの単純な実験 (3)[A Simple Experiment of Glare (3)]

- 90度回転させたまつげ
- 光の直接スナップショット

# 関連研究[RELATED WORK]

# グレア効果の初期研究[Early Work for Glare Effect]

- クロスフィルタのレンズフレア効果[@Shinya1989]
- 夜間運転シーンに対するまつげによるグレア[@Nakamae1990]

# グレア効果の初期研究 (続き)[Early Work for Glare Effect (cont'd)]

- グレアビルボード[@Rokita1993]
- 目構造の解析とグレアフィルタコンポジター[@Spencer1995]
- HDR画像上のグレアフィルタ[@Debevec1997]

# グレアのリアルタイムテクニック[Real-Time Techniques for Glare]

- リアルタイム環境ライティング[@Mitchell2002]
- レーシングゲームの実装[@Kawase2002; @Kawase2003]

# 物理ベースアプローチ[Physically-Based Aproaches]

- Fraunhofer回折によって引き起こされるグレア[@Kakimoto2004; @Kakimoto2005]
- 眼内Fresnel回折[@Ritschel2009]
- リアルタイムレンズフレア[@Hullin2011]

# 基礎理論[FUNDAMENTAL THEORY]

# 回折 --- グレアの主要因[Diffraction - A Major Cause of Glare]

# Huygens-Fresnelの原理は回折を説明する[Huygens-Fresnel Principle Accounts for Diffraction]

- 波は波面上の **どこへでも** 同心円状に伝播する。
- 次の波面からの第2波の包絡曲線[envelope curve]

# 回折の解析[An Analysis of Diffraction]

# 回折のモデル[A Model for Diffraction]

- 開口$S$
- $\lambda$:波長
- $U_f(x_f, y_f)$:点$(x_f, y_f)$における複合波[complex wave]の振幅

# 解析の付録を参照[See Appendix for the Analysis]

# Fraunhofer回折[Fraunhofer Diffraction]

$$
I_f(\frac{x_f}{\lambda R}, \frac{y_f}{\lambda R}) = \left| \frac{A}{\lambda R} \right|^2 |F[t_o(x_o, y_o)]|^2
$$

- $I_f \equiv |U_f|^2$: 波の強度
- $A$: 入射光の振幅
- $F[\cdot]$: Fourier変換オペレータ
- $R$: 十分に大きな距離
    - 開口サイズ$5\text{mm}^2$と$\lambda = 500\text{nm}$で$R \gg 50m$

# レンズシステムでのFraunhoferの近似[Fraunhofer Approximation in a Lens System]

$$
I_f(\frac{x_f}{\lambda f}, \frac{y_f}{\lambda f}) = \left| \frac{A}{\lambda f} \right|^2 |F[t_o(x_o, y_o)]|^2
$$

> レンズシステムを通した回折像は回折を引き起こす物体の二次元Fourier変換を用いて表すことができる。[@Goodman1968]

# グレアパターン画像生成[GLARE PATTERN IMAGE GENERATION]

# 波長に関する回折[Diffraction w.r.t. Wave Length]

# グレアパターン画像と波長[Glare Pattern Image and Wave Lengths]

- 2Dパターンのスケーリングは$\lambda$に比例する。[$\propto \lambda$]
- 回折の強度は$\lambda^{-2}$に比例する。[$\propto \lambda^{-2}$]

# 六角形絞りによるグレア[Glare by a Hexagonal Diaphragm]

# クロスフィルタパターン[A Cross Filter Pattern]

# まつげと虹彩絞り[Eyelashes and Iris Diaphragm]

- まぶた[eyelid]

# 動的グレア[Dynamic Glare]

- どうやってグレアは移動中に形状を変化するのか。
    - 光源位置
    - 入力障害物[obstacle]画像$t(x_o, y_o)P(x_o, y_o)$を選ぶ。
    - 出力グレア画像

# 特例: 円形開口

- FFTではなく'エアリーディスク'用の解析的式を使う。
    - $\theta$: 画角
    - $r$: 開口の半径
    - $J_1(\cdot)$: 第1種Bessel関数
    - $k \equiv 2\pi / \lambda$

$$
I(\theta) = \frac{\pi r^2 A^2}{\lambda^2 f^2} \left( \frac{2J_1(kr \sin\theta)}{kr \sin\theta} \right)^2
$$

- 矩形開口には、他の式を使うことができる。

# 実装と例[AN IMPLEMENTATION AND EXAMPLES]

# マルチスペクトル積分[Multi-Spectra Integration]

$$
G_{RGB} = M \int_{\lambda_{min}}^{\lambda_{max}} I_f(x_f, y_f, \lambda) C_{XYZ}(\lambda)S(\lambda)d\lambda
$$

- $G_{RGB}$: 光源に対するRGBグレア画像
- $M$: XYZからRGBへの変換(3x3行列)
- $I_f$: グレア強度(2DのFFTの結果)
- $C_{XYZ}$: カラーマッチっグ関数
- $S$: 光源のスペクトルパワー分布

# 処理の流れ[Processing Flow]

# 波長に沿ったサンプリングとアキュムレーション[Sampling and Accumulation along Wave Lengths]

- 可視光波長(380-700nm)に沿った100サンプルで十分かも。

# シードとなるグレア画像をスケール、累積する[Scale and Accumulate a Seed Glare Image]

- 各$\lambda$でFFTを計算する必要はない。
    - $\lambda = \lambda_0$と仮定したシードグレア画像
    - $\lambda / \lambda_0$で画像をスケールする。$(\lambda / \lambda_0)^2$でピクセル値をスケールする。

# 結果とリファレンス[A Result and a Referrence]

# 様々な光源での結果[Results for Different Light Sources]

# 様々な明るさでの結果[Results for Different Brightness]

- 単一のHDRグレア画像から色んな結果
- 入力シーンで現在のピクセルの明るさを乗算する。

<!--  -->

- 計測したヘッドランプの強度分布。単位:cd
- $L$は入射光の振幅の二乗$A^2$と等価である。

# 直視した光源からグレアをレンダリングする[Rendering Glare from Light Sources Directly Viewed]

1. スクリーンスペースで光源を探す。
1. ディレクショナルライト分布に従って明るさを乗算する。
1. グレア画像を散乱またはオーバーレイする。

# 高反射性の表面でのグレアをレンダリングする[Rendering Glare on Highly Reflective Surfaces]

1. 明るい光源のライトマップを準備する。
1. スクリーンスペースで反射する点を特定する。
1. 対応するテクセルの明るさを乗算する。
1. グレア画像を散乱またはオーバーレイする。

# ヘッドランプ評価への応用 [@Kakimoto2010][An Application to Headlamp Evaluation [Kakimoto et al. 2010]]

# おわりに[Conclusions]

- グレア画像は遮蔽物画像の2次元Fourier変換である。
    - FFTによりシードグレア画像を作る。
    - リサイズ、増幅、$\lambda$に沿ったシードグレアの累積により中間HDRグレア画像を計算する。
- '明るい'と特定した各ピクセルに対するスペクトル分布を使う。
    - ピクセルの明るさで中間グレアを乗算する。

# 参考文献[References]
