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

- モデルからクリップへは3つの座標変換が必要:
    - モデルからワールドへ
    - ワールドからビューへ
    - ビューからクリップへ
- ユニフォームとして頂点シェーダに渡される4x4行列を用いる

<!-- p.15 -->

- 実践では、モデル、ビュー、プロジェクション行列は一般にひとつの行列に焼き込まれるが、なぜ？

<!-- p.17 -->

- モデルからクリップへの座標変換は頂点シェーダの使い方のほんの一面に過ぎない
- 他の使い方: アニメーション
- どうやって振動[pulsing]を実装する？

<!-- p.18 -->

- 時間と共に表面の法線に沿って位置を変位させる
- どうやって変位を計算する？

<!-- p.19 -->

- 検討: `float displacement = 0.5 * (sin(u_time) + 1.0);`
- 欠点は？

<!-- p.20 -->

- 検討: `float displacement = u_scaleFactor * 0.5 * (sin(u_frequency * u_time) + 1.0);`
- 他の欠点は？

<!-- p.21 -->

- 可変の膨らみ[bulge]を得るには？

<!-- p.22 -->

- 検討: `float displacement = u_scaleFactor * 0.5 * (sin(position.y * u_frequency * u_time) + 1.0);`

<!-- p.23 -->

- 何が頂点ごとに変化して、何がしないのか？

<!-- p.24 -->

- すべてのモダンなGPUでは、頂点シェーダはユニフォーム変数と同様にテクスチャから読み出すことができる
- これは何に役に立つか？

<!-- p.25 -->

- 例: テクスチャはディスプレイスメントマッピング用のハイトマップを提供できる

<!-- p.26 -->

- RenderMonkeyのデモ
    - バウンス
    - モーフ
    - パーティクルシステム

# Primitive Assembly

- 頂点シェーダは頂点ひとつを処理する。*Primitive Assembly* はトライアングル、ラインなどのようにプリミティブひとつの形に頂点をグループ化する

# Perspective Division and Viewport Transform

- Primitive AssemblyとRasterizationの間の一連のステージが存在する
    - *透視除算[perspective division]*
        - `P_ndc = P_clip.xyz / P_clip.w`
    - *ビューポート変換[viewport transform]*
        - `P_window = M_viewport_transform * P_ndc`

<!-- p.36 -->

- Primitive AssemblyとRasterizationの間の一連のステージが存在する
    - *クリッピング[clipping]*

# Rasterization

- プリミティブがどのピクセルにオーバーラップするかを決定する
- どうやってこれを実装する？
- エイリアシングは？
- ラスタライゼーション中に位置ではない頂点属性に何が起こる？
- トライアングルからフラグメントへの比率はどれくらい？

# Fragment Shader

- ライトやマテリアルとの相互作用をシミュレートすることでフラグメントをシェードする
- 大まかに、フラグメントシェーダやそのユニフォーム入力の組み合わせが *マテリアル* である
- *ピクセルシェーダ* とも呼ばれる (D3D)
- フラグメントシェーダは計算的に強烈になり得る
- フラグメント入力とは？
- 有用なユニフォーム/テクスチャの例とは？

<!-- p.39 -->

- 例: Blinn-Phongライティング

```hlsl
    float diffuse = max(dot(N, L), 0.0);
```

<!-- p.40 -->

```hlsl
    float diffuse = max(pow(dot(N, L), u_shininess), 0.0);
```

- なぜ頂点ごとに評価してラスタライゼーション中に補間しない？

<!-- p.41 -->

- フラグメントごとのライティング　VS　頂点ごとのライティング
- どちらがどちら？

<!-- p.42 -->

- 例: テクスチャマッピング

<!-- p.43 -->

- 例: ライティングとテクスチャマッピング

<!-- p.44 -->

- 例: バンプマッピング

<!-- p.46 -->

- 例: スペキュラマッピング

<!-- p.47 -->

- 例: ノンフォトリアリスティックレンダリング(NPR)

<!-- p.48 -->

- 例: リフレクションマッピング

<!-- p.49 -->

- さらなる例

<!-- p.51 -->

- フラグメントシェーダは色を出力できるが、他に何が役立つだろう？
    - フラグメントの破棄。なぜ？
    - 深度。なぜ？
    - 複数の色。なぜ？

# Vertex and Fragment Shader Examples

# Per-Fragment Tests

- フラグメントはフレームバッファを作るために一連のテストを通過しなければならない
    - どんなテストが役立つ？

# Scissor Test

- フラグメントがウィンドウ座標で定義される矩形の内部にある場合にそれを破棄する
    - なぜこれが役立つ？
    - これはフラグメントシェーディングの後に発生する必要があるか？

<!-- p.55 -->

- シザーテストはシェードする必要のないウィンドウの部分を"切り取る[scissoring out]"のに役立つ
    - パフォーマンスのため。なぜ？
    - ポストプロセッシング効果
    - マルチパスレンダリング

# Stencil Test

- ステンシルテストはウィンドウの任意の範囲を破棄したり、フラグメントごとに数え上げたりできる
- ステンシルはステンシルバッファに書き込まれ、後に、フラグメントはこのバッファに対してテストさせることができる
- これは何のために役立つ？

# Depth Test

- 可視の表面を見つける
- かつては"馬鹿馬鹿しいほど高価"と言われた
- *z-test* とも呼ばれる
    - フラグメントシェーディングの後に必要？
    - z-fighting

# Blending

- フラグメントの色をフレームバッファの色と組み合わせる
    - それぞれの色を重み付けできる
    - +、-、など、様々な演算子が使える
- なぜこれが役立つ？

<!-- p.61 -->

- 例: 半透明
- 加算[additive]ブレンディング
    - `c_dest = c_source.rgb * c_source.a + c_dest.rgb;`
- アルファブレンディング
    - `c_dest = c_source.rgb * c_source.a + c_dest.rgb * (1 - c_source.a);`

<!-- p.62 -->

- アルファブレンディングのソート

# Graphics Pipeline Walkthrough

- そんなこんなで、フレームバッファに書き込む！

# Example Performance Analysis

# Evolution of the Programmable Graphics Pipeline

- GPU以前
- 固定機能GPU
- プログラマブルGPU
- ユニファイドシェーダプロセッサ

# Early 90s - Pre GPU

# Why GPUs?

- 並列性の活用
    - パイプライン並列
    - データ並列
    - CPUとGPUの並列実行
- ハードウェア:
    - テクスチャフィルタリング、ラスタライゼーション、アルファブレンディング、…
    - MAD、sqrt、…

# Texture Hardware

- ソフトウェアは12-40倍遅い(Larrabee)

# 3dfx Voodoo(1996)

- ハードウェアでは:
    - 固定機能のラスタライゼーション、テクスチャマッピング、深度テスト、など
    - 4-6MBのメモリ
    - PCIバス
    - $299

# Aside: Mario Kart 64

- 高いフラグメント負荷/低い頂点負荷

# Aside: Mario Kart Wii

- 高いフラグメント負荷/低い頂点負荷？

# NVIDIA GeForce 256 (1999)

- ハードウェアでは:
    - 固定機能の頂点シェーディング (T&L)
    - マルチテクスチャリング: バンプマップ、ライトマップ、など
    - 1000万ポリゴン/秒
    - Direct3D 7
    - AGPバス

# NVIDIA GeForce 3 (2001)

- 固定機能T&Lをプログラマブル頂点シェーダで選択的に迂回する
- 固定機能フラグメントシェーディングをプログラマブルフラグメントシェーダで選択的に迂回する(結合器[combiners]を登録する)
- 多くのプログラミング制限
- Direct3D 8
- Pentium 4 --- 20ステージ
- GeForce 3 --- 600-8000ステージ

# NVIDIA GeForce 6 (2004)

- より良いプログラマブルフラグメントシェーダ
- 頂点シェーダがテクスチャを読み込むことができる
- 動的分岐
- 複数レンダターゲット
- PCIeバス
- OpenGL 2/Direct3D 9

# Dynamic Branches

- 最適なパフォーマンスのために、フラグメントシェーダの動的分岐はスクリーン空間で一貫性のあるものにすべき
- これはCUDAにおけるWarp分割とどのように関係する？

# NVIDIA GeForce 8 (2006)

- 一から[ground-up]のGPU再設計
- ジオメトリシェーダ
- トランスフォームフィードバック
- OpenGL 3 / Direct3D 10
- ユニファイドシェーダプロセッサ
- GPUコンピュートに対するサポート

# Why Unify Shader Processors?

# Evolution of the Programmable Graphics Pipeline
