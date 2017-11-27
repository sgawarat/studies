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

TODO

# 参考文献[References]
