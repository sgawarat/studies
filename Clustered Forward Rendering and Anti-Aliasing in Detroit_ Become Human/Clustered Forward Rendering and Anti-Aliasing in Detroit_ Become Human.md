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
    - Morphologicalアンチエイリアシング

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
    - ライトごとに
        - 光の減衰を計算する
        - シャドウを計算する
        - 投影テクスチャを計算する
        - マテリアルのBRDFを持つ最終ライティング色を計算する

# Clustered forward rendering

- ライトループ最適化
    - ライトごとに
        - 光の減衰を計算する
        - シャドウを計算する → 太陽の影に対するより高いレジスタ使用率
        - 投影テクスチャを計算する
        - マテリアルのBRDFを持つ最終ライティング色を計算する

# Clustered forward rendering

- ライトループ最適化
    - 太陽の影を計算する → より低いレジスタ使用率
    - ライトごとに
        - 光の減衰を計算する
        - シャドウを計算する
        - 投影テクスチャを計算する
        - マテリアルのBRDFを持つ最終ライティング色を計算する

# Clustered forward rendering

- 可視性のライト拒否
    - 我々の可視性はポータル/ゾーンに基づく
    - 可視性情報はできるだけ早くにライトを拒否するのに使うことができる
    - 可視性情報はビットフィールドに格納されている(ゾーンあたり1ビット)
    - uiObjectVisibility & uiLightVisibility != 0ならば、ライトを拒否できる

# Clustered forward rendering

- ライトループ最適化
    - 太陽の影を計算する → より低いレジスタ使用率
    - ライトごとに
        - ビットフィールドの可視性テスト → 早期脱出
        - 光の減衰を計算する
        - シャドウを計算する
        - 投影テクスチャを計算する
        - マテリアルのBRDFを持つ最終ライティング色を計算する

# Clustered forward rendering

- 可能性のある他の早期脱出
    - N dot L
    - 光の減衰の結果
    - シャドウの結果

# Clustered forward rendering

- ライトループ最適化
    - 太陽の影を計算する → より低いレジスタ使用率
    - ライトごとに
        - ビットフィールドの可視性テスト → 早期脱出
        - 光の減衰を計算する → 早期脱出
        - N dot Lをテストする → 早期脱出
        - シャドウを計算する → 早期脱出
        - 投影テクスチャを計算する
        - マテリアルのBRDFを持つ最終ライティング色を計算する

#

#

#

#

#

#

# Clustered forward rendering

- 透明の最適化
    - 透明はパフォーマンスキラーとなり得る

# Clustered forward rendering

- 透明の最適化
    - 透明はパフォーマンスキラーとなり得る
    - 草
        - 画像ベースライティングのみ

# Clustered forward rendering

- 透明の最適化
    - 透明はパフォーマンスキラーとなり得る
    - 草
        - 画像ベースライティングのみ
    - パーティクル
        - 重心ごと
        - 球面調和関数
        - 半解像度

# Clustered forward rendering

- 髪
    - 完全に透明な髪によるパフォーマンス問題

# Clustered forward rendering

- 髪
    - 完全に透明な髪によるパフォーマンス問題
        - 加算ブレンディング
        - 調整されたアルファ透明度を出力する
        - 1/16の解像度

#

# Clustered forward rendering

- 髪
    - 透明度累積パス
    - 深度パス
        - 透明度累積パスから計算されるアルファテスト

#

# Clustered forward rendering

- 髪
    - 透明度累積パス
    - 深度パス
    - 後面トライアングルパス
    - 前面トライアングルパス
    - モーションベクトルパス

# Clustered forward rendering

- 以下の事は我々のエンジンでは未だにディファードで行われる
    - スクリーンスペースリフレクション
    - イメージベースライティング
    - 両方とも法線とラフネスを必要とする

# Clustered forward rendering

- デバッグ
    - ディファードシェーディングでは、Gバッファはデバッグに非常に便利
        - 法線、ラフネス、アルベド、他
    - デバッグシェーダ
    - なんでも出力する: 法線、正接、準法線、UV、他
    - Gバッファのレンダターゲットより強力
    - デバッグメモリに格納される。ゲームのリテールバージョンには使われない

# Clustered forward rendering

- 鏡
    - 可視の鏡ごとに一度だけクラスタを埋める

# Clustered forward rendering

- 他の利点
    - ボリューメトリックライティング

# Clustered forward rendering

- 他の利点
    - ボリューメトリックライティング
    - デカール

# Clustered forward rendering

- ライティングのパフォーマンス

#

# Clustered forward rendering

- ライティングのパフォーマンス
    - 124つのライトと32つのIBL
    - クラスタ埋めに1.23ms
    - 深度パスに1.92ms
    - 不透明パスに8.79ms
    - 透明パスに3.48ms

# Clustered forward rendering

- 今後の課題
    - クラスタ埋めのパス数をへらす

# Clustered forward rendering

- 今後の課題
    - クラスタ埋めのパス数をへらす
    - より良い深度分布

# Clustered forward rendering

- 今後の課題
    - クラスタ埋めのパス数をへらす
    - より良い深度分布
    - クラスタ埋め中にいくつかのライトを取り除く
        - シャドウマップとともに
        - 可視性情報とともに

# Clustered forward rendering

- 今後の課題
    - クラスタ埋めのパス数をへらす
    - より良い深度分布
    - クラスタ埋め中にいくつかのライトを取り除く
    - VR: 両眼に対して一度にクラスタを埋める

# Temporal anti-aliasing

# Temporal anti-aliasing

- エイリアシング
    - より良い解像度だけに頼ることはできない
    - HDRとPBRはエイリアシングを増加させる

# Temporal anti-aliasing

- アンチエイリアシング
    - マルチサンプリング
    - ポストプロセッシング
    - シェーディング

# Temporal anti-aliasing

- マルチサンプリング
    - ハードウェア
    - レンダターゲットの大きさが増える(2x、4x、など)
    - シェーディングがポリゴンのエッジでより多く処理される
    - パフォーマンスを減少させる
    - スペキュラエイリアシングを改善しない

# Temporal anti-aliasing

- ポストプロセッシング
    - Morphological Anti-Aliasing (MLAA)
    - Fast Approximate Anti-Aliasing (FXAA)

# Temporal anti-aliasing

- ポストプロセッシング
    - Morphological Anti-Aliasing (MLAA)
    - Fast Approximate Anti-Aliasing (FXAA)
    - Temporal Anti-Aliasing (TAA)

# Temporal anti-aliasing

- ポストプロセッシング
    - Morphological Anti-Aliasing (MLAA)
    - Fast Approximate Anti-Aliasing (FXAA)
    - Temporal Anti-Aliasing (TAA)
        - *Bryan Karis*の"High Quality Temporal Supersampling"に基づく
        - UnrealのInfiltratorリアルタイムデモ

# Temporal anti-aliasing

- TAA
    - 異なるオフセットで各フレームをジッターする

# Temporal anti-aliasing

- TAA
    - 異なるオフセットで各フレームをジッターする
    - フレームを累積する

# Temporal anti-aliasing

- TAA
    - 異なるオフセットで各フレームをジッターする
    - フレームを累積する
    - 前のピクセル位置を回収するのにモーションベクトルを使う

# Temporal anti-aliasing

- TAA
    - 異なるオフセットで各フレームをジッターする
    - フレームを累積する
    - 前のピクセル位置を回収するのにモーションベクトルを使う
    - 前のフレームのピクセルを拒否するためにヒューリスティックを用いる

# Temporal anti-aliasing

- TAAを適用する所
    - 最終画像
        - ブルーム、DOF、モーションブラーのエイリアシングを防止しない
    - ポストプロセッシング前
        - 安定性において最適
    - 特定のフィーチャのために
        - SSR、ボリューメトリックライティング

# Temporal anti-aliasing

- ジッタリング
    - 固定の8タップ
        - 8xMSAAのような

# Temporal anti-aliasing

- モーションベクトル
    - オプションとして透明オブジェクトの上に書き込みできる

# Temporal anti-aliasing

- モーションベクトル
    - オプションとして透明オブジェクトの上に書き込みできる
    - 布、Skinnedキャラクター

# Temporal anti-aliasing

- モーションベクトル
    - オプションとして透明オブジェクトの上に書き込みできる
    - 布、Skinnedキャラクター
    - 草木(Speedtree)

# Temporal anti-aliasing

- モーションベクトル
    - オプションとして透明オブジェクトの上に書き込みできる
    - 布、Skinnedキャラクター
    - 草木(Speedtree)
    - 頂点アニメーション

# Temporal anti-aliasing

- ピクセル拒否
    - 近傍クランプ
        - *Bryan Karis*のプレゼンテーションに触発される

# Temporal anti-aliasing

- ピクセル拒否
    - 近傍クランプ
        - *Bryan Karis*のプレゼンテーションに触発される
    - 深度のdisocclusion^[遮蔽されていたものが見えるようになること]
    - 速度の類似性

# Temporal anti-aliasing

- スキンシェーダ
    - カメラがズームイン/ズームアウトするときにTAAの強さを減らす

# Temporal anti-aliasing

- パフォーマンス
    - 1.14ms

# Temporal anti-aliasing

- 雨と雪
    - 雨と雪はTAAで完全に消失する可能性がある
    - The flags responsive fix missing particles
    - 雨の表面のエフェクトでは、rain normalの強さに応じてTAAの強さを減らす

#

#

#

#

# Temporal anti-aliasing

- TAAによる問題
    - いくつかのポストプロセスはTAAとうまく働かない
    - 深度がジッターされる
    - 輪郭のピクセルがアンチエイリアスされる
    - DOFとモーションブラーでの漏れ[leaking]や震え[vibration]

# Temporal anti-aliasing

- 被写界深度
    - 震えの除去
        - 深度でTAAを処理する
    - 漏れの除去
        - 漏れを避けるために錯乱円をerodeする

#

#

#

#

# Temporal anti-aliasing

- モージョンブラー
    - 半解像度のモーションブラー(540p)
    - それぞれ8回のテクスチャフェッチを伴う2つのパス
    - TAAは深度不連続な部分の近くのピクセルでにじむ
    - モーションブラーのサンプリング中にこれらのピクセルを拒否する
        - => 望ましくないTAAのにじみ縞を避ける

#

#

#

#

#

# Temporal anti-aliasing

- PS4 Proの検討事項
    - Temporal anti-aliasingはcheckerboardと互換性がある

# Temporal anti-aliasing

- PS4 Proの検討事項
    - Temporal anti-aliasingはcheckerboardと互換性がある
    - チェッカーボードはtemporal AAとともに解決されるべき
        - ジッタリングはチェッカーボードのピクセル間で分ける

# Temporal anti-aliasing

- PS4 Proの検討事項
    - Temporal anti-aliasingはcheckerboardと互換性がある
    - チェッカーボードはtemporal AAとともに解決されるべき
    - でも、4Kがポストプロセッシングのパフォーマンスをメチャクチャにする

# Temporal anti-aliasing

- PS4 Proの検討事項
    - Temporal anti-aliasingはcheckerboardと互換性がある
    - チェッカーボードはtemporal AAとともに解決されるべき
    - でも、4Kがポストプロセッシングのパフォーマンスをメチャクチャにする
    - 我々はポストプロセッシングのあとにチェッカーボードを解決する

#

# Temporal anti-aliasing

- TAAはいくつかのシチュエーションでは十分ではない
    - 我々は8xのみ
    - 孤立したピクセルに影響を与える非常に高いスペキュラは依然として問題となり得る

# Temporal anti-aliasing

- シェーディングのアンチエイリアシング
    - GPU derivativesを用いることで、複数回のシェーディングを行うことができる
    - 良い結果ではあるが、コストがかかる

# Temporal anti-aliasing

- シェーディングのアンチエイリアシング
    - 法線分布関数(NDF)のフィルタリング
    - "Filtering Distributions of Normals for Shading Antialiasing" by A. S. Kaplanyan, S. Hill, A. Patney and A. Lefohn

# Temporal anti-aliasing

- シェーディングのアンチエイリアシング
    - 法線分布関数(NDF)のフィルタリング
    - より高速なバージョン: "Error Reduction and Simplification for Shading Anti-Aliasing" by Yusuke Tokuyoshi

# Temporal anti-aliasing

- シェーディングのアンチエイリアシング
    - 法線分布関数(NDF)のフィルタリング
        - TAAととてもうまく動作する
        - 雨のディテールがよりよく見える

#

#

#

# Temporal anti-aliasing

- 他のtemporalエフェクト
    - シャドウ
    - HBAO
    - SSR
    - 肌のスクリーンスペース表面下散乱
    - ボリューメトリックライティング

# Temporal anti-aliasing

- ブルーノイズ
    - 最小の低周波要素を持ち、エネルギー的に集中したスパイクを持たないノイズ
    - "The rendering of inside" by Mikkel Gjel & Mikkel Svendsen
    - ブルーノイズ生成器 by Bart Wronski

# Blue noise

# Blue noise (with tiling)

# Temporal anti-aliasing

- Temporalシャドウ
    - 毎フレーム回転した8タップのポアソンカーネルを使う
    - 我々は16x16のブルーノイズの2Dテクスチャでピクセルあたりで異なる回転を用いる
    - その結果はTAAによって平滑化される

# Temporal anti-aliasing

```glsl
const float fAARotation = pPassSRT->_fAARotation;  // from 1 to 8. Change at each frame.
const float fScale = 1.f / 4.f + 1.f / 8.f;
float fRand = tex2Dfetch(BlueNoiseTexture, int2(sSurface.fFragCoord.xy) % 16, 0).x;
float fAngle = 2.0f * PI * (fRand + fAARotation * fScale);
```

#

#

#

# Temporal anti-aliasing

TODO
