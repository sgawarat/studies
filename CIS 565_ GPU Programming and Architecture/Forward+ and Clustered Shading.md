---
title: Forward+ and Clustered Shading [@Cozzi2017h]
numberSections: false
---
# Deferred Shading Limitations

- 高いメモリ**帯域幅**
- 限定された**マテリアル**の柔軟性
- **半透明** なし
- **MSAA** が高いメモリ利用率を必要とする
- **冗長** なライティング計算

# Memory Bandwidth

# 2D Tiling

# Forward+

- スクリーン空間のタイルのライトカリング付きフォワードシェーディング
- こんにちのGPUに対して設計される
    - 高いコンピュート対メモリ比
    - コンピュートシェーダ
- 3パス
    1. 深度/Zプリパス
    2. ライトカリング
    3. 最終シェーディング
- **Tiled Forward** とも呼ばれる

# 1. Depth/Z Pre-Pass

- フラグメントシェーダをパススルーする
- ライト累積パスでオーバードローを軽減する
- ライトカリングパスへの入力の一部

# 2. Light Culling

- どのライトが各タイルと**重なる**かを計算する

<!-- p.11 -->

- コンピュートシェーダ
    - タイルごとの錐台を計算する: **最大** と **最小** のZ
    - ライト/錐台の重なりをテストする
- 入力
    - Zバッファ
    - ライトバッファ(SSBO)
- 出力
    - タイルごとのライトリスト(SSBO)

<!-- p.12 -->

- 2.a. タイルごとの **最大** と **最小** のZを計算する
    - ピクセルあたり1スレッド。タイルあたり1ワークグループ/ブロック
    - ナイーブだと、総計n個のアトミック比較
- 2.b. ライト/錐台の重なり
    - タイルあたり1ワークグループ/ブロック。ライトあたり1スレッド(多くのライトには複数パスを必要とする)
    - ローカル/共有メモリにタイルごとに累積して、アトミックを使ってSSBO/バッファに書き込む
- 2.aと2.bに対してコンピュートシェーダひとつ

# 3. Final Shading

- マテリアルグループごとのドローコール
    - タイルごとのライトでライトを累積する

# Forward+ vs. Forward

<!-- p.18 -->

- 多少のライトを持つ複雑度の高いシーンではフォワードが勝る

<!-- p.19 -->

- Foward+はライト半径が増加するごとに遅くなる

# Tile Size

- 大きすぎる: 多くのライトが重なる
- 小さすぎる: 高いライトカリングのオーバーヘッド

# % Time in Each Pass

- 右: 大きなタイル: ライトカリング時間が小さく、ライト累積パス時間が大きい

# Clustered Shading

- 最大で約100万光源
- クラスタごとのライト背面カリング
- 大きな深度不連続性を持つ良くないケースのパフォーマンスの改善
- 以下で使われる
    - Just Cause 2 (Avalanche Engine)
    - Oculus
    - Doom (idTech)
    - Forza Horizon 2

<!-- p.23 -->

1. 深度プリパス
2. クラスタ割り当て: ピクセルをクラスタに割り当てる
3. ユニーク/空でないクラスタを見つける
4. ライトカリング: ライトを空でないクラスタに割り当てる
5. 最終シェーディング

# Clusters

# Depth Slicing

- 例: 64x64ピクセルのタイル、16深度スライス
- Doom: Zを対数関数的に割った16x8x24クラスタ

<!-- p.28 -->

- 指数階数的/カスタムなニア[near]クラスタ
- ファー制限; 遠方のライトを無視する
- 前面のクラスタでのみのスペキュラ

# Culling

- 小さな錐台/大きな球
    - 視錐台カリングとの比較: 大きな視錐台/小さな球
- AABBでカリング、または、反復的な球の改善[refinement]

# Tiles vs. Clusters