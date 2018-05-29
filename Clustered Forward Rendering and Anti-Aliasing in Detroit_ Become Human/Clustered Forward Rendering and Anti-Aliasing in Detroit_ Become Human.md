---
title: >
Clustered Forward Rendering and Anti-Aliasing in Detroit: Become Human
---
# Clustered Forward Rendering and Anti-Aliasing in Detroit: Become Human

# まえがき

- Quantic Dream
- Quantic Dreamの3Dエンジンの歴史
- "Detroit: Become Human"のための新技術の構築
- **Clustered forward rendering**
- **Temporal anti-aliasing**

# Quantic Dream

- パリを拠点とする独立系のフランスのスタジオ
- David Cageにより1997年に設立
- *Heavy Rain*からSonyとだけ仕事をしている
- "インタラクティブドラマ"に特化
- オーダーメイドの技術を開発
- 200人の従業員

# Quantic Dream

- リリースしたタイトル
    - Nomad Soul (1999)
    - Fahrenheit (2005)
    - Heavy Rain (2010)
    - Beyond: Two Souls (2013)
    - Detroit: Become Human (2018)

# Quantic Dream

- 技術デモ
    - The Casting (2006)
    - Kara (2012)
    - The Dark Sorcerer (2013)

# Quantic Dreamの3Dエンジンの歴史

- プロプライエタリなエンジン
    - プレイステーションのハードウェアに最適化されている
    - ツール用のPCのOpenGLバージョン
    - エンジンはアセット編集のためMayaに統合される

# Quantic Dreamの3Dエンジンの歴史

- Heavy Rain (2010)
    - プレイステーション3
    - Forward Rendering
    - 法線マップ付きper-pixelライティング
    - ライトあたりシェーダ1つ
    - シェーダツリー(Mayaで作られる)
    - MSAA 2X

# Quantic Dreamの3Dエンジンの歴史

- Beyond: Two Souls (2013)
    - プレイステーション3
    - Deferred Shading
    - ガンマ補正
    - 物理ベースレンダリング
    - Morphologicalアンチエイジング

# Quantic Dreamの3Dエンジンの歴史

- The Dark Sorcerer (2013)
    - プレイステーション4の技術デモ
    - 初期SDKでのPS4への我々の技術の最初の移植
    - Deferred Shading(レンダターゲット5つ)
    - 改善されたマテリアル(スペキュラ色付きCook-Torrance)

# 新技術の構築

- Detroit: Become Human
    - インタラクティブドラマ
    - パフォーマンスキャプチャ
    - 画像品質

# 新技術の構築

- Detroit: Become Human
    - 街中での出来事
    - 大量の夜間シーン
    - 大量の屋内シーン
    - 雨と雪

#

#

#

#

# 新技術の構築

- Detroit: Become Human
    - 30FPS / 1080p
        - アクションゲームじゃない！
        - より良いFPSではなくより良いグラフィクス

# 新技術の構築

- Detroit: Become Human
    - 30FPS / 1080p
        - アクションゲームじゃない！
        - より良いFPSではなくより良いグラフィクス
    - ローディング
        - ローディング画面を避ける

# 新技術の構築

- First list of features
- そのほとんどがGバッファにいくつかのスペースを必要とする

# 新技術の構築

- First list of features
- そのほとんどがGバッファにいくつかのスペースを必要とする
    - シャドウのための法線ベースのバイアス

# 新技術の構築

- First list of features
- そのほとんどがGバッファにいくつかのスペースを必要とする
    - シャドウのための法線ベースのバイアス
    - マルチレイヤード・マテリアル(肌、雨、など)

# 新技術の構築

- First list of features
- そのほとんどがGバッファにいくつかのスペースを必要とする
    - シャドウのための法線ベースのバイアス
    - マルチレイヤード・マテリアル(肌、雨、など)
    - 頂点ごとに格納される自己遮蔽

# 新技術の構築

- First list of features
- そのほとんどがGバッファにいくつかのスペースを必要とする
    - シャドウのための法線ベースのバイアス
    - マルチレイヤード・マテリアル(肌、雨、など)
    - 頂点ごとに格納される自己遮蔽
    - 眼球シェーダ

# 新技術の構築

- Gバッファにすべてをパックしたい場合、レンダターゲットが8つ以上になる可能性がある
- 様々な種類のマテリアルがディファードシェーディングでかち合う
- ディファードシェーディングは高速だが、良いパフォーマンスを得るために事を単純にし続けなければならない
- 我々はフォワードシェーディングに回帰することに決めた

# 新技術の構築

- Detroitの3Dエンジンの柱

# 新技術の構築

- Detroitの3Dエンジンの柱
    - Clustered forward rendering

# 新技術の構築

- Detroitの3Dエンジンの柱
    - Clustered forward rendering
    - Temporal anti-Aliasing

# 新技術の構築

- Detroitの3Dエンジンの柱
    - Clustered forward rendering
    - Temporal anti-Aliasing
    - 物理ベースレンダリング

# 新技術の構築

- Detroitの3Dエンジンの柱
    - Clustered forward rendering
    - Temporal anti-Aliasing
    - 物理ベースレンダリング
    - キャラクターレンダリング

# 新技術の構築

- Detroitの3Dエンジンの柱
    - Clustered forward rendering
    - Temporal anti-Aliasing
    - 物理ベースレンダリング
    - キャラクターレンダリング
    - FX

# 新技術の構築

- Detroitの3Dエンジンの柱
    - Clustered forward rendering
    - Temporal anti-Aliasing
    - 物理ベースレンダリング
    - キャラクターレンダリング
    - FX
    - ローディング

# 新技術の構築

- Detroitの3Dエンジンの柱
    - **Clustered forward rendering**
    - **Temporal anti-Aliasing**
    - 物理ベースレンダリング
    - キャラクターレンダリング
    - FX
    - ローディング

# Clustered forward rendering

# Clustered forward rendering

- GPUはより柔軟で効率的である

# Clustered forward rendering

- GPUはより柔軟で効率的である
- 新しいライティングアルゴリズム
    - Tiled rendering
    - Forward+ rendering
    - Clustered forward rendering

# Clustered forward rendering

- Tiled rendering
    - スクリーンはタイルにカットされる
    - タイルごとにライトのリストを埋める
    - タイルごとにライティングを処理する
    - 多くのライトに対して一度だけGバッファを読むので帯域幅を節約する
    - 透明をサポートしない

#

#

# Clustered forward rendering

- Forward+ rendering
    - タイルが奥に拡張される
    - ライトのリストはタイルのZ farからカメラのZ nearの間のすべてのライトを含む
    - 透明をサポートする

#

# Clustered forward rendering

- Clustered forward rendering
    - タイルが3Dのクラスタで置き換えられる
    - 深度分布は線形でない
    - Forward+ renderingの場合よりクラスタあたりのライト数が少ない
    - ただし、クラスタ数はタイル数より多い

#

# Clustered forward rendering

- 最初の実装
    - "Clustered Deferred and Forward Shading" by *Ola Olson et al.*, HPG 2012
    - Just Cause 3 (Avalanche)
        - "Practical Clustered Shading" by *Emil Persson*
    - Doom (idSoftware)
        - "The evil is in the details" by *Tiago Sousa* and *Jean Geffroy*

#

# Clustered forward rendering

- データ構造
    - ひとつのバッファにクラスタデータを含める
        - 3D配列
        - Width:36、Height:20、Depth:64
        - 最初のライトインデックス + ライト数

# Clustered forward rendering

- データ構造
    - ひとつのバッファにクラスタデータを含める
    - ひとつのバッファにライトデータを含める
        - 1D配列
        - ライトタイプ、位置、色、減衰、など
        - サイズ = 最大ライト数

# Clustered forward rendering

- データ構造
    - ひとつのバッファにクラスタデータを含める
    - ひとつのバッファにライトデータを含める
    - ひとつのバッファにライトインデックスを含める
    - 16ビットのインデックス
    - サイズは最大ライト密度に依存する

# Clustered forward rendering

- クラスタを満たす
    - 非同期コンピュートシェーダで満たされる
        - 深度およびシャドウパスの間に

# Clustered forward rendering

- クラスタを満たす
    - 非同期コンピュートシェーダで満たされる
    - 各クラスタはすべてのライトとテストされる
        - スポット/錐台、ポイント/錐台、ボックス/錐台
        - "Practical Clustered Shading" by Emil Persson
        - "Cull that cone!" by Bart Wronski

# Clustered forward rendering

- クラスタを満たす
    - 非同期コンピュートシェーダで満たされる
    - 各クラスタはすべてのライトとテストされる
    - 3パス

# Clustered forward rendering

|クラスタ|
|-|
|クラスタ0|
|クラスタ1|
|クラスタ2|
|...|


# Clustered forward rendering

|クラスタ||ライト数|
|-|-|-|
|クラスタ0| → |2|
|クラスタ1| → |3|
|クラスタ2| → |2|
|...||...|

第1パス: ライト数を計算する

# Clustered forward rendering

|クラスタ||ライト数||第1ライトインデックス|
|-|-|-|-|-|
|クラスタ0| → |2| → |0|
|クラスタ1| → |3| → |2|
|クラスタ2| → |2| → |5|
|...||...||...|

第2パス: 第1ライトインデックスを計算する

# Clustered forward rendering

|クラスタ||ライト数||第1ライトインデックス|
|-|-|-|-|-|
|クラスタ0| → |2| → |0|
|クラスタ1| → |3| → |2|
|クラスタ2| → |2| → |5|
|...||...||...|

↓

|ライトインデックス|
|-|
|クラスタ0のインデックス0|
|クラスタ0のインデックス1|
|クラスタ1のインデックス0|
|クラスタ1のインデックス1|
|クラスタ1のインデックス2|
|クラスタ2のインデックス0|
|クラスタ2のインデックス1|

第3パス: ライトインデックスを満たす

# Clustered forward rendering

- クラスタを満たす
    - 非同期コンピュートシェーダで満たされる
    - 各クラスタはすべてのライトとテストされる
    - 3パス
    - 2つの階層レベル
        - 18x10x32
        - 36x20x64


# Clustered forward rendering

- クラスタ埋めのパフォーマンス
    - 124つのライトと32つのIBL

#

#

#

#

# Clustered forward rendering

- クラスタ埋めの結果
    - "Night of the long knives"のステージ
    - 124つのライトと32つのIBL
    - クラスタ埋めに1.23ms

# Clustered forward rendering

- ライティング
    - 現在のピクセルに対するクラスタを見つけるために深度とピクセル位置を使う
    - ライトのリストをパースする

# Clustered forward rendering

- 最初の結果
    - とても印象的とはいえない
    - 大量のレジスタを用いる重いシェーダ

# Clustered forward rendering

- 最適化
    - ベクタレジスタではなくスカラレジスタを使うようにライトループを強制して、ライトをソートする
        - "The devil is in the details" by Tiao Sousa and Jean Geffroy

# Clustered forward rendering

- 最適化
    - ベクタレジスタではなくスカラレジスタを使うようにライトループを強制して、ライトをソートする
        - "The devil is in the details" by Tiao Sousa and Jean Geffroy
    - すべてができるだけ同じ空間を用いることを確認する(ビュー空間)

# Clustered forward rendering

- 最適化
    - TAAでより少ないシャドウテクスチャサンプルを用いる(8だけ)
    - 2x4のテクスチャシャドウサンプルでループを用いるようにコンパイラに強制する
    - ある距離では、1つのテクスチャシャドウサンプルだけを持つ焼き込みシャドウテクスチャを用いる

# Clustered forward rendering

- 最適化
    - 深度パスは必要

# Clustered forward rendering

- 最適化
    - 深度パスは必要
    - クラスタはper-pixelライティングで使うことができる…そして、per-vertexライティングでも！

# Clustered forward rendering

- 最適化
    - 深度パスは必要
    - クラスタはper-pixelライティングで使うことができる…そして、per-vertexライティングでも！
    - 画像ベースライティングはできる時にディファードパスに移す

# Clustered forward rendering

- ライトループ最適化
    - 4つのライトタイプがある(point、spot、directional、projector)
    - シャドウと投影テクスチャ
    - 最初のバージョンは4つのループを用いた(ライトタイプごとに1つ)
    - すべてのライトタイプを扱う1つのループに切り替えた


# Clustered forward rendering

- ライトループ最適化
    - TODO
