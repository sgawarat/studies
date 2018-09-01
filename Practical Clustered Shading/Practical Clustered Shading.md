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

- 偽陽性を最小化したい
- 保守的でなければならない
    - ただし、依然としてタイトである
    - できれば[preferably]厳密である
        - だだし、高価すぎない
        - 驚くほど難しい！
- 99%の錐台カリングのコードは役に立たない
    - 視錐台カリングのために作られる
        - 多くな錐台 対 小さな球
        - 我々は小さな錐台 対 大きな球を必要とする
    - 球 対 6つの平面はやりたくない

# カリング

- あなたの錐台の心象[mental picture]は間違っている！

# カリング

- "小"ネタ["fun" facts]:
    - スクリーンに投影された球は円ではない
    - 投影下にある球は球ではない
    - スクリーン上の球の最も幅の広い部分は中心に並行でない
    - 円錐(スポットライト)はさらに難しい
- 錐台はスイスイいかない[frustums are frustrating] (洒落のつもりです[pun intended])
- 有効な[workable]ソリューション
    - 各クラスタのAABBに対してカリングする

# ポイントライトカリング

- 我々のアプローチ
    - 反復的な球の精錬
        - Z上でループして、球を減らす
        - Y上でループして、球を減らす
        - X上でループして、球に対してテストする
    - AABBより良好にカリングする
        - 似たようなコスト
        - 一般的に20〜30%をカリングする

# ポイントライトカリング

# カリングの擬似コード

```cpp
for (int z = z0; z <= z1; z++) {
    float4 z_light = light;
    if (z != center_z) {  // 中央にあれば元の球を使い、そうでなければ縮めた球を使う
        const ZPlane& plane = (z < center_z) ? z_planes[z + 1] : -z_planes[z];
        z_light = project_to_plane(z_light, plane);
    }
    for (int y = y0; y < y1; y++) {
        float3 y_light = z_light;
        if (y != center_y) {  // 中央にあれば元の球を使い、そうでなければ縮めた球を使う
            const YPlane& plane = (y < center_y) ? y_planes[y + 1] : - y_planes[y];
            y_light = project_to_plane(y_light, plane);
        }
        int x = x0;  // 球に当たるまで左から走査する
        do {
            ++x;
        } while (x < x1 && GetDistance(x_planes[x], y_light_pos) >= y_light_radius);

        int xs = x1;  // 球に当たるまで右から走査する
        do {
            --xs;
        } while (xs >= x && -GetDistance(x_planes[xs], y_light_pos) >= y_light_radius);

        for (--x; x <= xs; x++) {  // その範囲におけるクラスタを満たす
            light_lists.AddPointLight(base_cluster + x, light_index);
        }
    }
}
```

# スポットライトカリング

- 我々のアプローチ
    - 反復的な平面の狭小化[narrowing]
        - 球とクラスタの境界を求める
        - 6つの方向ごとに、平面-円錐テストを行い、縮める
    - 円錐対境界球は残りの"立方体[cube]"をカリングする

我々の円錐は平面で覆われる[flat-capped]^[訳注:底面が平坦である、一般的な円錐]のでなく球で覆われる[sphere-capped]^[訳注:底面が頂点を中心とする球冠[spherical cap]である円錐]ことに注意する。これは(そうすべきであるように)光の減衰が深度ではなく距離に基づくためである。球で覆われる円錐は一般に広い角度でより良く振る舞い、平面で覆われる円錐が取り得るような極端に大きな値にはならない。

# スポットライトカリング

- 我々のアプローチ
    - 反復的な平面の狭小化[narrowing]
        - 球とクラスタの境界を求める
        - 6つの方向ごとに、平面-円錐テストを行い、縮める
    - 円錐対境界球は残りの"立方体[cube]"をカリングする

# スポットライトカリング

# ポイントライトとスポットライト

# シャドウ

- 前もってすべてのシャドウバッファを必要とする
    - 古典的ディファードとは異なり…
        - 次世代ではメモリはそれほど問題にならない
    - 1つの大きなアトラス
        - 可変サイズのバッファ
        - 動的に調節可能な解像度
- ライトは安価だが、シャドウマップはそうではない
    - シャドウキャスタについては依然として保守的である必要がある

ROP
: 深度/ステンシルテストやブレンディングなどを行うユニットのこと

# シャドウ

- ライトとシャドウバッファを分離する
    - 似たライトはシャドウバッファを共有できる
    - 車のライトなどで有用

# CPUパフォーマンス

- コア1つをミリ秒で計測する。Intel Core i7-2600K

# GPUパフォーマンス

- ミリ秒で計測する

# 今後の仕事

- クラスタリング戦略
    - スクリーン空間タイル、深度対距離
    - ビュー空間カスケード
    - ワールド空間
        - 視錐台の外側のライトの計算を可能にする(反射など)
    - 動的な調整[adjustments]？
- シャドウ
    - シャドウバッファで最大Z値に基づいてクラスタをカリングする？
- ライト割り当て
    - 非同期コンピュート

# おわりに

- Clustered shadingはゲームに対して実用的である
    - 高速であり
    - 柔軟性があり
    - 単純である
    - 新たな可能性をもたらす
        - どこでもライト計算
        - ボリューメトリックフォグのレイトレース

# 新しいヤツ

- *Clustered shadingn: Assigning convex light shapes using conservative rasterization in DirectX12*
    - 修士学生[master thesis student]との研究プロジェクト
    - Clustered Shadingのためのライト割り当てを改善する
        - "完全なクラスタリング"
    - GPU Pro 7の中に掲載予定

# アルゴリズム

- メッシュとしてのライト
    - 一般に極めて低解像度
    - LODできる
- Shellパス
    - 保守的ラスタライゼーション
- Fillパス
    - コンピュートシェーダ
- シェーディング

# Shellパス

- 頂点シェーダ
    - 単位メッシュ
    - ライトタイプあたり1ドローコール
- ジオメトリシェーダ
    - 配列IDを割り当てる
        - 今ではVSで可能
- ピクセルシェーダ
    - 厳密な深度範囲を計算する
- テクスチャ配列、例えば、24x16xN、R8G8フォーマット

# Shellパス

- 保守的ラスタライゼーション
    - 関連するすべてのタイルに触る
- ピクセル内で厳密な深度範囲を計算する
    - トライアングルはピクセルを完全にカバーする
        - 深度の勾配から最大値と最小値を計算する
    - ピクセルはトライアングルを完全にカバーする
        - 頂点から最大値と最小値を使う
    - Partial coverage
        - 交差点で最大値と最小値を計算する
- MINブレンディングはオーバーラップを解決する
    - MINMAXを達成するために1-GをGへ出力する

# Fillパス

- コンピュートシェーダ
    - ライトあたりのタイルあたりに1スレッド
    - ライトの連接リスト

# 結果

- 任意の凸型ライトタイプで動作する
- 今ある中で[out there]一番タイトなライト割り当て
    - シェーディング時間を節約する

# パフォーマンス

# 指数関数的な深度スライシング

広い深度範囲を持つ、大きなオープンワールドゲームでは指数関数的のほうが明らかに勝っているが、狭い深度範囲を持つ、屋内シューターや見下ろし型のゲームでは話が変わってくるかもしれない。

# 線形な深度スライシング

# 代替実装

# 代替実装

- 代替のクラスタリングスキーム
    - ワールド空間、固定グリッド
- 代替のライトリスト
    - ライトのビットフィールド
        - 単一フェッチ
        - 一定で低いメモリ要件
    - 小から中程度のライト数に適している

# シェーダ

```hlsl
// クラスタを計算して、そのライトマスクをフェッチする
int4 coord = int4(In.WorldPos * Scale + Bias, 0);
uint light_mask = Clusters.Load(coord);

while(light_mask) {
    // マスクからライトを抽出して、そのビットを無効化する
    uint i = firstbitlow(light_mask);
    light_mask &= ~(1 << i);

    // ライティング
    const Light light = Lights[i];
    // ...
    // ...
}
```

# 代替実装

- Clustered lightmapping?
    - 伝統的[old school]なライトマップとClustered Shadingが交わる
    - LightMapはテクセルあたりのライトのビットフィールドを格納する
    - シャドウは有効化されたライトに対してフェッチされる
        - Dead spaceの最適化？
- 限定的な動的ライティングのサポート
    - ライトをオン/オフする
    - ライトの色、強度、フォールオフを変化させる
    - 半径を減らす

# おわりに

- 多くの未踏の地
    - 今後の研究のための機会
- Clustered Shadingは多数のフレーバーが現れている
    - 最適な選択は要件に依存する

# 参考文献

# ご質問は？
