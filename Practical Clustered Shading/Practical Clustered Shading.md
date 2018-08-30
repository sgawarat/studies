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

TODO
