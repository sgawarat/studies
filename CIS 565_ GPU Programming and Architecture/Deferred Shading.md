---
title: Deferred Shading [Cozzi2017g]
numberSections: false
---
# Renderer Design

- 大量の以下をレンダリングするエンジンを設計する
    - 様々なマテリアルを持つオブジェクト
    - 動的ライト
    - 様々なライトタイプ
- 綺麗に、効率的に

# Forward Rendering - Multi-Pass

```pseudo
foreach ライト {
    foreach 可視オブジェクト {
        このマテリアル/ライトタイプに対するシェーダを用いてレンダリングする;

        フレームバッファに累積する;
    }
}
```

- 利点と欠点は？

<!-- p.4 -->

- マテリアル/ライトタイプあたりシェーダひとつ
- パフォーマンス
    - 頂点変換、ラスタライゼーション、フラグメントシェーダのマテリアル部分などをオブジェクトごとに複数回行う必要がある
    - 遮蔽されるフラグメントがシェードされる
    - すべてのライトがオブジェクト全体に影響を与えるわけではない

# Forward Rendering - Single Pass

```pseudo
foreach 可視オブジェクト {
    オブジェクトに影響するライトを見つける;

    単一のシェーダを用いてすべてのライトとマテリアルをレンダリングする;
}
```
- 利点と欠点は？

<!-- p.6 -->

- 大量のシェーダ
    - マテリアル/ライトの組み合わせごとにひとつのシェーダ
    - シェーダを作成するのが難しい
    - 実行時コンパイル/リンクを必要とするかも
    - 長いubershaderがコンパイル時間を増加させる
    - More potential shaders to sort by
- マルチパスと同じ
    - 遮蔽されるフラグメントがシェードされる
    - すべてのライトがオブジェクト全体に影響を与えるわけではない

# Deferred Rendering

```pseudo
foreach 可視オブジェクト {
    特性をGバッファに書き込む;
}

foreach ライト {
    Gバッファを用いてライトを計算する;
    フレームバッファに累積する;
}
```

- 利点と欠点は？

<!-- p.8 -->

- シーンの複雑さからライティングを分離する
- 少ないシェーダ
    - マテリアルごとにひとつ
    - ライトタイプごとにひとつ
- 各オブジェクトを一度だけ変換してラスタライズする
- 遮蔽されていないオブジェクトのみライティングする

<!-- p.9 -->

- メモリ帯域幅の使用 --- ライトごとにGバッファを読み込む
- ライトごとに完全なライティング方程式を再計算する
- Gバッファでの限定されたマテリアル特性
- MSAAや半透明は難しい

# G-Buffer Layout in Leadwerks 2.1

アルベド/深度/法線/スペキュラファクタ
↓
ディフューズライティング/スペキュラ反射
↓
最終イメージ

<!-- p.11 -->

|バッファ|フォーマット|ビット|値|
|-|-|-|-|
|色|GL_RGBA8|32|赤/緑/青/アルファ|
|深度|GL_DEPTH_COMPONENT24|24|深度|
|法線|GL_RGBA16FかGL_RGBA8|64か32|X/Y/Z/スペキュラファクタ|

# G-Buffer Layout in Killzone 2

ターゲット画像
深度
ビュー空間の法線
スペキュラ強度
スペキュララフネス/パワー
スクリーン空間の2Dモーションベクトル
アルベド(テクスチャ色)
ディファード合成
ポストプロセッシング付き画像(被写界深度、ブルーム、モーションブラー、色付け[colorize]、ILR)

<!-- p.21 -->

|R8|G8|B8|A8||
|-|-|-|-|-|
|深度|←|←|ステンシル|DS|
|ライティング累積RGB|←|←|強度|RT0|
|法線X(FP16)|←|法線Y(FP16)|←|RT1|
|モーションベクトルXY|←|スペキュラパワー|スペキュラ強度|RT2|
|ディフューズアルベドRGB|←|←|太陽遮蔽|RT3|

# Light Accumulation Pass

- ライトごとのジオメトリ
    - フルスクリーンクアッド/トライアングル
        - シザー/ステンシルテスト付き
    - 3D境界ジオメトリ。例えば:
        - ポイントライト --- 球
        - スポットライト --- 円錐

# Optimizing Light Accumulation

重なる球: 加算ブレンディングを使う

# WebGL Demo

# WEBGL_draw_buffers performance impact

Gバッファパスが高価な(シーンの複雑さが高い)ときに一番役立つ

ライト累積パスが高価なときはあまり役立たない

# Tile-Based Deferred Shading

- 2Dタイル、例えば16x16ピクセル、にスクリーンを分割する
- どのライトがどのタイルに影響を与えるかを決定する -> ライトタイル情報
    - CPUかコンピュートシェーダ！
- ライト累積パス
    - Gバッファを一度だけ読み込む
        - ライトごとに一度だけと比較して帯域幅を節約する
    - ライトタイル情報を用いて、どのライトがピクセルに影響を与えるかを見つける

# Optimizing Light Accumulation

- 背面のみをレンダリングする(前面カリングを用いる)
- 深度テストを`GREATER`にセットする
- ピクセルがオブジェクトに属し、球の内部にある必要がある
