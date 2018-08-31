---
title: Practical Clustered Shading [@Persson2015]
---
# SIGGRAPH 2015

# Practical Clustered Shading

# 近況

- 2年半の間、Clusteredの教義[gospel]を説き勧めている[preaching]
    - Nordic Game 2013
    - SIGGRAPH 2013
    - CEDEC 2013
    - SIGGRAPH Asia 2014
    - SIGGRAPH 2015
- Avalanche Studiosは依然としてプロダクションでコレを使っている
    - **Just Cause 3**に搭載される[ship in]だろう
    - これに非常に満足している
    - 昔のライティングパスをすべて取り除いた

# 近況

# 近況

- 興味深いClustered Shadingの広がり
    - Forza Horizon 2に搭載された
    - IntelのPCとAndroid向けサンプル[@Intel14]
    - GDCでのAMDのトーク[@Thomas15]
    - ゲーム開発者によってよく取り上げられている[gets name-dropped]
        - とはいえ、実際に実装したのはまだほとんどいない

# アジェンダ

- 昔のヤツ
    - Avalanche Engineにおけるライティングの歴史
    - なぜClustered Shadingか？
    - Avalanche Engineへの適合
    - パフォーマンス
    - 今後の作業
- 今のヤツ
    - 新しい研究と洞察[insights]
    - 代替実装

# 昔のヤツ

# Avalanche Engineにおけるライティング

- Just Cause 1
    - フォワードレンダリング
    - 3つのグローバルなポイントライト
- Just Cause 2
    - フォワードレンダリング
    - World-space XZ-tiled light-indexing [@Persson10]
        - 4m×4mタイルあたり4つのライト
        - 128x128のRGBA8のライトインデックステクスチャ
        - ライトは定数レジスタ(PC/Xenon)または1Dテクスチャ(PS3)に置く
    - オブジェクト毎のライティング
    - ソリューションをカスタムする

# Avalanche Engineにおけるライティング

- Mad Max
    - 古典的なディファードシェーディング
        - 3つのGバッファ
        - 柔軟なライティングセットアップ
            - ポイントライト
            - スポットライト
                - 任意のシャドウキャスタ
                - 任意の投影テクスチャ
        - HDR
    - 透明物[transparency]が問題となる
        - 透明物を一切使わないことで解決した
    - アンチエイリアシングにはFXAA

# Avalanche Engineにおけるライティング

- Just Cause 3
    - Clustered deferred shading
        - 4つのGバッファ
        - 柔軟なライティングセットアップ
        - 物理ベースライティング
    - ライティング付き透明物はそのまま動作する
        - この種のゲームでは本当に必要とされる
        - Wire AAが使えるようになる[@Persson12]

# Avalanche Engineにおけるライティング

# 注視してきたソリューション

- Tiled deferred [@Olson et al. 11]
    - プロダクション実績あり[production proven] (Battlefield 3)
    - 古典的ディファードより高速
    - 古典的ディファードのすべての短所を踏襲する
        - 透明物、MSAA、メモリ、独自のマテリアル/ライトモデル、など
    - 古典的ディファードよりモジュールが少ない
- Tiled Forward [@Harada e al. 12]
    - プロダクション実績あり(Dirt Showdown)
    - プリZパスを強制する
    - MSAAが上手く機能する
    - 透明物は追加のパスを必要とする
    - 古典的ディファードよりモジュールが少ない
- Clustered shading [@Olsson et al. 12]
    - プロダクション実績あり(Forza Horizon 2)
    - プリZの必要なし
    - MSAAが上手く機能する
    - 透明物が上手く機能する
    - 古典的ディファードよりモジュールが少ない

# なぜClustered Shadingか？

- 柔軟性
    - フォワードレンダリング互換
        - 独自のマテリアルやライトモデル
        - 透明物
    - ディファードレンダリング互換
        - スクリーン空間デカール
        - パフォーマンス
- 単純性
    - 統一されたライティングソリューション
    - 本格的な[full blown]Tiled Deferred/Tiled Forwardより実装しやすい
- パフォーマンス
    - 一般的にTiled Deferredと同等かそれより良い
    - ワーストケースでのより良いパフォーマンス
    - 深度の不連続性？"It just works"

# 深度の不連続性

# 深度の不連続性

# 深度の不連続性

# 実践的Clustered Shading

- 我々が必要としなかったこと
    - 何百万のライト
    - 手の込んだ[fancy]クラスタリング
    - 法線円錐カリング[normal-cone culling]
    - 明示的な境界
- 我々が必要としたこと
    - 大きなオープンワールドのソリューション
    - プリZパスを強制しない
    - スポットライト
    - シャドウ
- 我々が好んだ[preferred]こと
    - DX10レベルのハードウェアで動作する
    - タイトなライトカリング
    - シーン非依存性

# Avalancheのソリューション

- 未だにディファードシェーディングエンジンである
    - ただし、フォワードパス付きの統一されたライティングソリューション
- 空間的クラスタリングのみ
    - 64x64ピクセル、16深度スライス
- CPUのライト割り当て
    - DX10ハードウェアで動作する
    - コンパクトなメモリ構造にしやすい
- 暗黙的クラスタ境界のみ
    - シーン非依存
    - ディファードパスでは明示的クラスタ境界が使えそう[could potentially use]

# Avalancheのソリューション

- 指数関数的な深度スライシング
    - 広大な深度範囲！[0.1m〜50,000m]
    - 既定のリスト
        - [0.1, 0.23, 0.52, 1.2, 2.7, 6.0, 14, 31, 71, 161, 365, 828, 1880, 4270, 9696, 22018, 50000]
        - 活用不十分
    - ファーを500に制限する
        - それ以降のライトを可視化するための"遠方ライト[distant light]"システムがある
        - [0.1, 0.17, 0.29, 0.49, 0.84, 1.43, 2.44, 4.15, 7.07, 12.0, 20.5, 35, 59, 101, 172, 293, 500]
    - 特別なニアの0.1〜5.0のクラスタ
        - 平坦な地面に立っているプレイヤーからビジュアル的に調整される
        - [0.1, 5.0, 6.8, 9.2, 12.6, 17.1, 23.2, 31.5, 42.9, 58.3, 79.2, 108, 146, 199, 271, 368, 500]

# Avalancheのソリューション

- 別個の遠方ライトシステム

# Avalancheのソリューション

# Avalancheのソリューション

# Avalancheのソリューション

- 既定の指数関数的なもの
- 特別なニアクラスタ

# データ構造

- 3Dテクスチャにクラスタ"ポインタ"
    - R32G32_UINT
        - R=オフセット
        - G=[ポイントライト数, スポットライト数]
- テクスチャバッファにライトインデックスリスト
    - R16_UINT
    - タイトにパックされる
- 定数バッファにライトとシャドウのデータ
    - ポイントライト: 3つのfloat4
    - スポットライト: 4つのfloat4

# シェーダ

```hlsl
int3 tex_coord = int3(In.Position.xy, 0);  // スクリーン空間の位置…
float depth = Depth.Load(tex_coord);  // …と深度

int slice = int(max(log2(depth * ZParam.x + ZParam.y) * scale + bias, 0));  // クラスタを探す
int4 cluster_coord = int4(tex_coord >> 6, slice, 0);  // TILE_SIZE = 64

uint2 light_data = LightLookup.Load(cluster_coord);  // ライトリストをフェッチする
uint light_index = light_data.x;  // パラメータを抽出する
const uint point_light_count = light_data.y & 0xffff;
const uint spot_light_count = light_data.y >> 16;

for (uint pl = 0; pl < point_light_count; pl++) {  // ポイントライト
    uint index = LightIndices[light_index++].x;

    float3 LightPos = PointLights[index].xyz;
    float3 Color = PointLights[index + 1].rgb;
    // ここでポイントライトを計算する…
}

for (uint sl = 0; sl < spot_light_count; sl++) {  // スポットライト
    uint index = LightIndices[light_index++].x;

    float3 LightPos = SpotLights[index].xyz;
    float3 Color = SpotLights[index + 1].rgb;
    // ここでスポットライトを計算する…
}
```

# データ構造

- メモリ最適化
    - ナイーブなアプローチ: 理論上の最大値を割り当てる
        - すべてのクラスタがすべてのライトにアドレスする
            - それほど起こらない
        - 数メガバイトになるかも
        - ほとんどが一切使われない
    - 準保守的なアプローチ
        - 大規模なワーストケースシナリオを構築する
            - 2の倍数、または、安心できるくらいの数
            - 理論上の最大値のほんの一部に過ぎない[still only a small fraction of]可能性が高い
    - 過剰な割り当てをしていないことを実行時にアサートする
        - 近づいただけでも警告する

# クラスタリングと深度

- 深度で錐台をサンプルする

# クラスタリングと深度

- タイル化された錐台

# クラスタリングと深度

- Tiled Deferred/Forward+用の深度範囲

# クラスタリングと深度

- 2.5Dカリング付きTiled Deferred/Forward+用の深度範囲[@Harada12]

# クラスタリングと深度

- クラスタ化された錐台

# クラスタリングと深度

- Clustered shading用の暗黙的な深度範囲

# クラスタリングと深度

- Clustered shading用の明示的な深度範囲

# クラスタリングと深度

- 明示的な深度範囲 対 暗黙的な深度範囲

# クラスタリングと深度

Tiled 対 暗黙的な深度範囲 対 明示的な深度範囲

# 広範囲の深度

- AからFまでの深度の不連続性の範囲
    - 既定のTiled: A+B+C+D+E+F
    - 2.5D付きTiled: A+F
    - Clustered: 約max(A, F)
- AからFまでの深度の傾斜[slope]の範囲
    - 既定のTiled: A+B+C+D+E+F
    - 2.5D付きTiled: A+B+C+D+E+F
    - Clustered: 約max(A, B, C, D, E, F)

# データ一貫性

# データ一貫性

# カリング

TODO
