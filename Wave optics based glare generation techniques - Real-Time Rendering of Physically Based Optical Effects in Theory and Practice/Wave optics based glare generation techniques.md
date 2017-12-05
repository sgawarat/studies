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

TODO

# 参考文献[References]
