---
title: Wavefront Tracing for Precise Bokeh Evaluation [@Kakimoto2015b]
numberSections: false
---
# はじめに[INTRODUCTION]

# 波面トレーシングの応用[Applications of Wavefront Tracing] [@Kneisly1968]

- コースディクス[caustics] [@Stavroudis1972; @Mitchell1992]
- ボケ付きレンダリング
    - ボケの大きさの正確な評価
    - 人間の視力[visual acuity]を計算に入れることができる。
    - エンジニアリング分野で有効活用される(メガネレンズの設計、眼科学[ophthalmology])。

# 基本の前提[Basic Premises]

- レイトレーシングに対する選択肢
    - レイは与えられる入力である。
- 波面上の点に注意を払う。
    - レイと波面の交差点[cross point]
- 波面トレーシングは波動光学の手法ではなく幾何光学の技術であることに注意。

# 各トレースに対する入出力[Input / Output for Each Trace]

- 入力
    - (スクリーンから物体表面の点への)一連のレイ[a series of rays]
    - 波源点[wave source point] (レイの両端点[either end point])
    - 道中の媒質の屈折率または屈折力(メガネレンズ、眼球レンズ)
    - 開口(瞳孔[pupil])の直径
- 出力
    - 波面曲率
    - レイの任意の点におけるライトビームの大きさ[extent]

# 波面トレーシングの簡単な概要[BRIEF OVERVIEW OF WAVEFRONT TRACING]

# 物体の点からの波面トレーシング[Wavefront Tracing from an Object Point]

- 任意のオブジェクトの点からのバックトレーシングを行いながらボケを評価する[@Loos1998; @Kakimoto2007]。

- 角膜[cornea]
- 瞳孔[pupil]
- 網膜[retina]
- 中心窩[central fovea]

# 目からの波面トレーシング[Wavefront Tracing from the Eye]

- オブジェクト空間の点でボケを評価する[@Kakimoto2010]。
- ボケの空間的な分布を事前計算するのに効率的。

# 波面の記述[Descriptions of a Wavefront]

- 法線ベクトル$N$
- 主要曲率$\kappa_1$と$\kappa_2$
- 主要方向$\boldsymbol{e}_1$と$\boldsymbol{e}_1$

# 波面処理(1) 転送[Wavefront Operation (1) Transfer]

$$
\kappa_1' = \frac{\kappa_1}{1 - d \kappa_1} \\
\kappa_2' = \frac{\kappa_2}{1 - d \kappa_2}
$$

- $d$は進んだ距離。

# (2) 屈折率による屈折[(2) Refraction by Refractive Index]

- 波面形式におけるSnellの法則
    - 境界が同じ方法で表現される。

# その他の波面処理[Other Wavefront Operations]

- 屈折力による屈折
    - 人間の視力は屈折力で表される。
- 反射
- 円錐体[conoid]トレーシングによって必要に応じて同時に起こさせる。
    - 円形開口を仮定する。
    - レイに沿って楕円形状をトレースする。
    - 詳細は[@Kakimoto2011]を参照

# デフォーカスシミュレーションのための円錐体トレーシング[Conoid Tracing for Defocus Simulation]

# 円錐体: 光の束の形状[Conoid: A Bundle Shape of Light]

Sturm[シュトルム]の円錐体(眼科学用語)
: 非球面レンズの円形開口を通る光の束によって形成される円錐のような形状

- 乱視レンズ[astigmatic lens]

# 例[EXAMPLES]

TODO

# 参考文献[References]
