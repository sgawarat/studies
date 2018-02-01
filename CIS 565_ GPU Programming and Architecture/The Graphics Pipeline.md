---
title: The Graphics Pipeline [@Cozzi2017f]
numberSections: false
---
# Graphics Review: Rendering

- レンダリング
    - 目標: ピクセルに色を割り当てる
- 2つの部分
    - 可視表面
        - 与えられた視点の前列に何があるか
    - シェーディング
        - ピクセル色を生成するためにマテリアルとライトの相互作用をシミュレートする

# Graphics Pipeline Walkthrough

# Vertex Assembly

- 1つ以上のバッファから頂点を一緒に取り出す
- *primitive processing*(GL ES)や*input assembler*(D3D)とも呼ばれる

```
+----- 頂点[vertex] ------------------------------------------------+
+--- 属性[attribute] -----------+                                   |
+- 成分[component] -+           |                                   |
|       p.x        | p.y | p.z | n.x | n.y | n.z | t.s | t. t      |
+--- 位置 ----------------------+---- 法線 --------+-- テクスチャ座標　--+
```

<!-- p.7 -->

- OpenGLは異なるバッファからの頂点属性の取り出しに対する大量の柔軟性をもたらす
- 例えば(インデックスなし):

位置: P0 P1 P2
テクスチャ座標: U0 U1 U2
法線、従法線[binormal]、従正接[bitangent]: N0 B0 T0 N1 B1 T1 N2 B2 T2

頂点0: P0 U0 N0 B0 T0
頂点0: P1 U1 N1 B1 T1

# Vertex Shader

- 入ってくる頂点位置を *モデル[model]* から *クリップ[clip]* 座標に変換する
- パイプラインを伝ってきた属性の修正、追加、除去などの、追加の頂点ごと[per-vertex]の計算を処理する
- 以前は *Transform and Lighting*(T&L)ステージと呼ばれたが、なぜ？

<!-- p.11 -->

-
