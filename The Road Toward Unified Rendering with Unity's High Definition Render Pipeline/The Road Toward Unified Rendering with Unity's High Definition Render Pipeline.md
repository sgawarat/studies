---
title: The Road Toward Unified Rendering with Unity's High Definition Render Pipeline [@Lagarde2018]
---
# The Road Toward Unified Rendering with Unity's High Definition Render Pipeline

# HDレンダリングパイプラインのデザイン目標

- クロスプラットフォーム
    - PC (DX11, DX12, Vulkan)、XBox One、PS4、Mac (Metal)
- 至るところに物理ベースレンダリング
- 統一されたライティング
    - 不透明、透明、ボリューメトリックで同じライティング機能
- 一貫性のあるライティング
    - すべてのライトタイプがすべてのマテリアルで、および、大域照明で機能する
    - 可能な限りdouble lighting / double occlusionを回避する

# レンダリングパイプラインアーキテクチャ

# レンダリングパイプラインアーキテクチャ

- 重要な要素:
    - ライティングおよびマテリアルのアーキテクチャ
    - Gバッファのデザイン
    - フォワード/ディファードパスの機能的同等性[features parity] (*機能的同等性[features parity]*として知られる)
    - デカールのアーキテクチャ
- 以下によってフォローアップする
    - マテリアルの概要
    - ボリューメトリックライティング

# ライティングアーキテクチャ

- ライティングアーキテクチャは*可視*のシーンのライトでのループによって定義される

光源ごとに
↑
物体を評価する

# ライティングアーキテクチャ

- ライティングアーキテクチャは*可視*のシーンのライトでのループによって定義される

光源ごとに　　　⇦ライトのループはCPUの"マルチパス"かGPUの"シングルパス"で行うことができる
↑
物体を評価する
⇧
GPUで評価する

# ライティングアーキテクチャ

- ライティングアーキテクチャは*可視*のシーンのライトでのループによって定義される

光源ごとに
↑
物体を評価する

GPUやCPUパスのどちらかに対するライト区画構造[partitioning structure] (タイル、クラスタ…)によって最適化できる

そのようなループはマテリアルに影響を与えないライトを取り除くCPUやGPUの助けによって最適化できる

# ライティングアーキテクチャ

- ライティングアーキテクチャは*可視*のシーンのライトでのループによって定義される

光源ごとに
↑
物体を評価する

ディファードやフォワードレンダラでは同じ
マテリアル特性の源のみが異なる

このライトループはディファードでもフォワードでも概念的に同一であることに注意する。マテリアルの特性の源のみが異なる。ディファードでは、Gバッファに由来し、フォワードでは、オブジェクトのユニフォーム/テクスチャに由来する。

# ライティングアーキテクチャ

- ライティングアーキテクチャは*可視*のシーンのライトでのループによって定義される

光源、太陽、IBLごとに
↑
物体を評価する

# ライティングアーキテクチャ

- ライティングアーキテクチャは*可視*のシーンのライトでのループによって定義される

光源ごとに
↑
物体を評価する

太陽ごとに
↑
物体を評価する

IBLごとに
↑
物体を評価する

パフォーマンス上の理由により、ゲームではしばしばライトタイプとマテリアル評価応答[evaluation response]との結合が存在する。例えば、我々はマテリアルのライトモデルによってIBLを事前計算する。
つまり、我々はライトタイプごとに1つのループを行う必要がある。

# ライティングアーキテクチャ

- ライティングアーキテクチャは*可視*のシーンのライトでのループによって定義される

光源ごとに
↑
物体を評価する

太陽ごとに
↑　　　　　　　　　　　　⇦別個のパスで計算したりしなかったり
物体を評価する

IBLごとに
↑
物体を評価する

この種のライトタイプループは時折異なる呼び出しに分割され、パフォーマンスが変化するかもしれない。

# ライティングアーキテクチャ

- HDRPにおけるあるGPUライトループ(太陽、パンクチュアル、エリア、IBL、空)

光源ごとに
↑
物体を評価する

太陽ごとに
↑
物体を評価する

IBLごとに
↑
物体を評価する

HDRPにおいて、我々はすべてのライトタイプで単一のライトループを用いる。太陽、大きさのないライト(スポット、ポイント)、エリアライト、空がある。

# ライティングアーキテクチャ

- ディファードパス

HDRPはディファードとフォワードの両方のレンダラをサポートする。ディファードレンダラの例を見ていこう。Gバッファを満たすディファードマテリアルが1つと使う透明マテリアルが1つある。
同じライトループ。統一されたライティング。

# ライティングアーキテクチャ

- フォワードパス

フォワードレンダリングの例。フォワードの不透明および透明マテリアルがある。

# ライティングアーキテクチャ

- ディファードおよびフォワードの混合パス

HDRPは同時にフォワードとディファード両方をサポートしてもいる。この場合、ディファードパスで見てきたものに加えて、フォワードの不透明マテリアルもある。

# ライティングアーキテクチャ

- 完全なフォワードに切り替え可能

# マテリアルアーキテクチャ

- アーティストフレンドリーなデータ VS エンジンデータ

このスキーマはディファードおよびフォワードアーキテクチャで機能するためにHDRPにおいてマテリアルを記述する方法を示している。これらはガイドラインである。そして、我々はアーティストフレンドリーなデータとエンジンフレンドリーなデータの概念を導入する。
Uiかシェーダグラフでアーティストが埋めた入力をアーティストフレンドリーなデータとしよう。smoothnessのようなやつ。我々はライティングエンジンが使えるエンジンデータへの変換関数を追加する(例えば、ラフネス)。
Gバッファは単なる中間ストレージである。これは圧縮可能である。
HDRPのマテリアルはライティングアーキテクチャに合わせるためにこのマテリアルガイドラインに従う必要がある。

# ライティングアーキテクチャ

- 復習

光源ごとに
↑
物体を評価する

太陽ごとに
↑
物体を評価する

IBLごとに
↑
物体を評価する

# ライティングアーキテクチャ

- ボリューメトリックマテリアルでは

光源ごとに
↑
ボリュームを評価する

太陽ごとに
↑
ボリュームを評価する

IBLごとに
↑
ボリュームを評価する

ボリューメトリックでは、その概念は、代わりにボリューメトリックマテリアル(吸収、散乱)を用いたことを除いて、きっかり同じである。

# ライティングアーキテクチャ

- ボリューメトリックマテリアルでは

Gバッファと同様に、我々は計算のためのボリューメトリックマテリアルの入力としてVバッファを用いる。

# ライティングアーキテクチャ

- 実践では: より低い分解能[resolution]でライティングパスを分割する

そして、実践において、我々はライティングパスを分割し、froxelのセルごとに計算し、その後、別個のパスで不透明および透明マテリアルに結果を適用する。

# ライティングアーキテクチャ

- タイルおよびクラスタの両方のアプローチで最適化する
- 目標
    - 偽陽性を取り除くことに焦点を当てる
        - 例: 狭いシャドウをキャストするスポットライト
    - 偽陽性はライティングパスにおいてより高価である
        - ライトカリングはシャドウレンダリング中に非同期に実行する
        - ディファードライティングパスは非同期に実行していない
        - 隠せる所にコストを移す
        - ライティングパスにおける高いregister pressure

1. 球冠[sphere cap]付きスポットライトにも対応するアグレッシブ(だが高速)な偽陽性の除去を重要視する
    - スポットライトはしばしばシャドウをキャストし、狭いので、重要である
    - 基本的な境界球テストを用いたリスト構築は極めて不十分[highly insufficient]である
2. すべてのリスト構築の処理は非同期コンピュートの活用によって吸収される
3. 偽陽性は早めにやるよりライティング中に扱ったほうがもっと高価である
    - 最終ライティングシェーダはより高いループの複雑さとより大きなregister pressureを持つ
    - 最終ライティングシェーダはシャドウマップのレンダリングの間に非同期コンピュートを活用できない
4. そのリストはディファードでもフォワードでもその両方でも使える
5. リストはライティング中のスレッドdivergenceを減らすのに役立つタイプによる順序を保持するための増加するインデックスのためにもたらされる。

# ライティングアーキテクチャ

- 階層的なアプローチ
    1. 可視ライトごとにスクリーン空間のAABBを求める
    2. Big tile 64x64 prepass
        - 粗い交差テスト
    3. タイルまたはクラスタのライトリストを構築する
        - 狭い交差テスト

1. 可視ライトごとにスクリーン空間AABBを求める
2. Big tile 64x64 tile pre-pass. 初期の早期脱出[early out]のためにAABBを使う(2Dで、深度なし)
    - タイルと凸包との厳密な交差テストで追跡調査する
    - 追加のテスト基準[criteria]として境界球を使う(ポイントライトと球冠付きスポットライトで役立つ)

基本的に、最初にAABBパスが来て、その後、AABBパスが生成したものを使うbig tileパスが来て、その後、big tileパスだけでなくAABBパスが生成したものを両方使うFPTLやClusteredのリスト構築パスが来る。FPTLとClusteredの両方はbig tileプリパスで生成されたオーバーラップし得るタイルのリストを用いる。
これら両方はbig tileプリパスからのリストに残っていることをテストするためにAABBを用いる。
凸包は、2つのスケール値を持つ以外は、いわゆるOBBである。なので、我々は、角錐[pyramid]や楔[wedge]のどちらかを作るために、別個の軸XとYに沿って4つの頂点をしぼる[squeeze]ことができる。スケールを1.0に設定すると、それは単なるOBBであり、1.0より小さくすると、内側にすスケールする。両方を限界まで小さくすると角錐になり、片方だけだと楔型になる。境界球はスポットライトの球冠部分をもたらすのに役立つ。OOBと2つのスケール値は凸包として使うものだが、我々はより多くのタイルを除外するための追加の制約として境界球も用いる。これは、ポイントライトだけでなく球冠に対しても重要である。FPTLおよびClusteredでは、64x64タイルごとにbig tileパスで生成されたものをリストとしてループする。これら両方はまずリストに対してAABBを確認し、LDSに粗いリストを作り上げる。これら両方は境界球の輪郭が粗いリストにおけるライトごとにタイルと重なるかどうかの確認によって追跡調査する。最後にFPTLは詳細な剪定を行い、Clusteredは残りのリストに対してクラスタを確認する。
big tileプリパスがAABBを確認するとき、それは2Dであり、64x64のタイルに対してである。FPTLがそれを行うとき、一例として、3DのAABBテストであり、16x16のタイルに対してである。

FPTLでは、それは、不透明ピクセルをカバーするために必要であるのみなので、とりわけタイトである。つまり、我々は交差テストに最大/最小深度を含めることによってその早期パスだけで大量に除外できる。big tileはAABBテストにおいてXYを行う。FPTLはXYZを行う。タイルに対する球のオーバーラップはすべての場合で2Dのオーバーラップテストである。しかし、もちろん、それがより小さなタイルであることから、こまごましたもの[little extra]を取り除くことができる。なので、私は、Clusteredではかなり安価であるので、再びそのテストを行うことに決めた。2回目のAABBテストは依然として2Dであるが、64x64ではなく32x32で実行するので、こまごましたものを剪定する。そして、それは非常に高速なテストである。

#

# Tiledライティング

- 3. タイル16x16
    - Fine Prune Tile Lighting (FPTL) [@Mikkelsen2016]に基づく
- タイル16x16に対してFTPLライトリストを作る
    - Fine pruning: いずれかの深度ピクセルがボリューム内にある場合にテストする
    - 偽陽性のアグレッシブな除去
    - タイルあたり1つのライトリスト。スカラレジスタからの属性の読み出しが可能

FPTLの実装
1. FPTLでは、クラスタリングしない16x16のタイルを使う
2. まず平凡な[trivial]AABBテスト(3D)を行う。その後、タイル対境界球テストを行う。big tileではそのAABBテストにおいてxyのみを行うが、FPTLはxyzを行う
3. fine pruningが少なくとも1つの不透明ピクセル/点を真のボリューム内に持たないライト取り除く
    - 偽陽性のアグレッシブな除去だが、不透明でのみ機能する
4. すべての偽陽性が取り除かれる、また、FPTLが不透明専用であることから、タイルあたり1つのリストを書き出す
    - ディファードでは、これはその代わりにライト属性をスカラレジスタから読み出すことができる
    - タイルを処理するすべてのスレッドが同じリストを読むので、ディファードではスレッドdivergenceが起こらない

参考にした論文と比較して何が新しいか:
1. Clustered
2. big tileプリパス
3. 球の輪郭対2Dタイルの高速な重なりテスト

# Clusteredライティング

- 3. 64つのクラスタで32x32のタイルを構築する
    - クラスタの位置とサイズに対して等比級数[geometric series]を用いる
    - 半分のクラスタ(32つ)はニアとタイルあたりの最大深度の間を消費する
        - 可視範囲において良い分解能である
        - タイルあたりの最大深度以降のクエリを許可する
            - パーティクル、ボリューム、エフェクト

Clusteredの実装
1. クラスタの解像度は64つのクラスタを持つ32x32タイルである
2. 正確だが高速なクラスタ対ライトの交差テストを行う --- 球冠付きスポットライトに対してさえも
3. クラスタの位置とサイズを確立するために等比級数を用いる
4. クラスタの半分(32つ)がニア面と不透明の最大深度の間に使われるような、共通比率をタイルあたりに確立する
    - これは、(パーティクル、ボリュームライト、透過エフェクトのようなものに対して)不透明の最大深度以降のクエリを許可しつつ、可視範囲における高度に最適化したクラスタ分解能をもたらす

(3+4) そのタイルにおいて不透明の最大深度に達する回数できっちり半分のクラスタを使い果たすように等比級数に対するパラメータを選択している。

# ライティングアーキテクチャ

- パフォーマンス
    - PS4 @ 1080p

|シーン(マイクロ秒で計測)|フォワードタイル|フォワードクラスタ|
|-|-|-|
|不透明(約30つのパンクチュアルライト、3つのエリアライト、5つの環境ライト)|5675.5|7135.9|

- 注意: MSAAは無視できない影響を持つ可能性がある(ここでは計測していない)

# ライティングアーキテクチャ

- HDRPにおいて
    - 透明マテリアルはクラスタを使う
    - ディファードマテリアルはFPTLを使う
    - フォワード不透明マテリアルはFPTLとクラスタで選べる

# ライティングアーキテクチャ

- タイル/クラスタのパフォーマンス(MSAAなし)
    - 1080p PS4: タイル＋クラスタのリスト生成

エントリコストが高価である: 1ms、1つから10つまでのライトではほぼ同コストだが、上手くスケールする。

# ライティングアーキテクチャ

- VGPR pressureを減らしたい
- ディファードレンダラ[@Coffin2011; @Garawany2016]
    - マテリアルの分類
    - ライトの分類
        - エリアライトを扱うときに大きな利益
    - すべてのバリアントをカバーできない --- ワーストケースが必要
- フォワードレンダラ
    - 暗黙的なマテリアルの分類
    - ライトの分類ができない

# ライティングアーキテクチャ

- 分類のパフォーマンス
    - 1080p PS4 --- ディファードライティングパス

# Gバッファのデザイン

- Gバッファのデザインの設計
    - ブレンディングをサポートしない
        - アグレッシブな圧縮スキームを可能にする
            - 例: 法線を圧縮する
        - ブレンド可能なパラメータの位置による制約を回避する
            - 例: Smoothnessはアルファチャネルに置くことができる
    - 静的なディフューズライティング(ライトマップ/ライトプロブ)
    - 静的なシャドウマスク

静的なディフューズライティングはGバッファパス中にサンプルされる。

# Gバッファのデザイン

|標準|R|G|B|A|
|-|-|-|-|-|
|RT0 RGBA8 sRGB|ベース色.rgb|←|←|スペキュラオクルージョン|
|RT1 RGBA8|法線.xy(八面体 12/12)|←|←|知覚的smoothness|
|RT2 RGBA8|マテリアルデータ|←|←|特徴マスク(3)/マテリアルデータ|
|RT3 RGB111110f|静的ディフューズライティング|←|←|←|
|(任意)RT4 RGBA8|追加のスペキュラオクルージョンデータ|←|アンビエントオクルージョン|ライトレイヤリングマスク|
|(任意)RT5 RGBA8|4つのシャドウマスク|←|←|←|

- RT5がない場合、アンビエントオクルージョンはGバッファパス中に静的ライティングに関して適用される
    - これはSSAOと組み合わせたときのdouble occlusionを暗に示す
- ディファードマテリアル分類はRT2のみを用いる

Xbox Oneでは既定で4RT
ライトレイヤリングはライトの結合であり、一連のオブジェクトとライトの結合を意味する。つまり、これらのオブジェクトにのみ影響を与える。
RT4は動的に割り当てることができる。例えば、インゲームシネマティクスを行うときのみ有効化し、通常のゲームプレイでは無効化することができる。

# 機能的同等性

- エンジンの機能は選択したレンダリングパスによってしばしば変化する
    - SSAOが欲しい？SSRは？ --- ディファードパスを使おう
- HDRPはディファードおよびフォワードマテリアルの混ぜ合わせをサポートする
    - 同じ機能がサポートされる必要がある
- HDRPは最初から機能的同等性を持つよう設計されている

# 機能的同等性

- ディファードおよびフォワードで必要とされる機能
    - ライトリンキングとして知られるライトレイヤリング
        - ライトデータから利用可能なライトマスク
        - ディファード: オブジェクトマスクはRT4に格納する(必要に応じて)
        - フォワード: オブジェクトマスクは定数バッファを使う

ライトレイヤリングはカメラごとに有効化できる。これは、我々が必要な時にのみ追加のRTを割り当てたことを意味する。一般的にはインゲームシネマティクスに対して。
青色のドラゴンは青色のライトによってのみ影響を受け、白色の奴らの中間にある灰色のヤツは反射プロブによって影響を受けない。

# 機能的同等性

- ディファードおよびフォワードで必要とされる機能
    - SSAO、SSR、ディファード法線バイアスシャドウ
        - すなわち、ライティングの機能
        - ライティングパスの前に処理されなければならない

# 機能的同等性

- ディファード: Gバッファからのデータを使う

# 機能的同等性

- ディファード: Gバッファからのデータを使う
- フォワード: 深度プリパスからのデータを出力する

フォワード・パスでは、我々は深度プリパス間のデータを出力する必要がある。注意: 不透明フォワードマテリアルでは、我々はHDRPにおいて常に深度プリパスを処理する。

# 機能的同等性

- ディファード: Gバッファからのデータを使う
- フォワード: 深度プリパスからのデータを出力する
- 同じデータエンコーディングを用いなければならない

|標準|R|G|B|A|
|-|-|-|-|-|
|RT1 RGBA8|法線.xy(八面体 12/12)|←|←|知覚的smoothness|

# 機能的同等性

- ディファードおよびフォワードで必要とされる機能
    - スクリーン空間の表面下散乱(SSSSS)
        - ライティングパスの後に処理されなければならない

# 機能的同等性

- ディファードおよびフォワードで必要とされる機能
    - スクリーン空間の表面下散乱(SSSSS)
        - ライティングパスの後に処理されなければならない
        - 別個のディフューズ(RGB111110f)およびスペキュラライティング(RGBA16f)

# 機能的同等性

- ディファード: Gバッファからのデータを用いる

# 機能的同等性

- ディファード: Gバッファからのデータを用いる
- フォワード: プリパスからのデータを出力する？ --- 高コスト？

プリパス中にSSSデータを出力するとプリパスを高価にするだろう。我々はこれを回避することを優先する。

# 機能的同等性

- ディファード: Gバッファからのデータを用いる
- フォワード: フォワード不透明からのデータを出力する(SSSマテリアル用)

注意: XBox Oneでは、4つの32ビットのRTを維持することを目的とする。これは我々が得るものである。ディフューズ用RTを1つ、スペキュラ用RTを2つ、SSSデータ用RTを1つ。

# 機能的同等性

- ディファード: Gバッファからのデータを用いる
- フォワード: フォワード不透明からのデータを出力する(SSSマテリアル用)
- 同じデータエンコーディングを用いなければならない

|標準|R|G|B|A|
|-|-|-|-|-|
|RT0 RGBA8 sRGB|ベース色.rgb|←|←|スペキュラオクルージョン|

# 機能的同等性

- ディファード: Gバッファからのデータを用いる
- フォワード: フォワード不透明からのデータを出力する(SSSマテリアル用)
- 同じデータエンコーディングを用いなければならない

|標準|R|G|B|A|
|-|-|-|-|-|
|RT0 RGBA8 sRGB|ベース色.rgb|←|←|拡散プロファイル/SSSマスク|

注意: これは後に述べられるが、SSSマテリアルの場合、我々はRT0にスペキュラオクルージョンではなく拡散プロファイルとSSSマスクを格納し、スペキュラオクルージョンはSSSマテリアルではRT2に格納される。

# 不透明マテリアルレンダパス

- ステンシルの使い方
    - ステンシルはフレームの始めで0にクリアされる
    - ディファードマテリアルのtag stencil

# 不透明マテリアルレンダパス

- ステンシルの使い方
    - ディファードライティングパス
    - フォワードマテリアルと空に関してライティングを行わない
    - SplitLightingがタイルごとに行われる

# 不透明マテリアルレンダパス

- ステンシルの使い方
    - SplitLightingに対するフォワード不透明のtag stencil
    - SplitLightingのタグに対してSSSSSを処理する

# 不透明マテリアルレンダパス

- 深度プリパス
    - _
        - ディファードマテリアル: 任意
        - フォワードマテリアル: 法線バッファを出力する
    - Gバッファ
        - 通常ライティングまたは分割[split]ライティングに対するtag stencil
    - シャドウレンダリング
        - ライトリスト生成＋ライト/マテリアル分類と非同期に行う
        - SSAOと非同期に行う(法線バッファを使う)
        - SSRと非同期に行う(法線バッファを使う)
    - ディファード方向性カスケードシャドウ
        - (法線シャドウバイアスのために法線バッファを使う)
- タイルディファードライティング
    - シェーダバリアントごとに間接ディスパッチ
        - ステンシルを読む
            - ライティングなし: フォワードマテリアルと空をスキップする
            - 通常ライティング: ライティングを出力する
            - 分割ライティング: ディフューズとスペキュラを分ける
- フォワード不透明
    - (任意) ベース色＋拡散プロファイルを出力する
    - (任意) 分割ライティングに対して出力＋tag stencilする
- SS表面下散乱
    - 分離ライティングでのステンシルをテストする
    - ライティングを組み合わせる

ここに話してきたパスのすべてがある。これらの詳細には触れないだろうが、興味があれば、スライドはカンファレンスの後に入手できるようになるだろう。

# デカールアーキテクチャ

- 望ましいデカールの機能
    - ディファードとフォワードの両方
    - Gバッファレイアウトに制約されない
    - マテリアル(PBR)と適切にブレンドする
    - 静的ライティングに作用する
    - 透過のサポート
    - Normal orientation fadingのサポート

Normal orientation fadingは引き伸ばされてしまうエッジに沿ったデカールを投影するときにアーティファクトを回避するためにアーティストによって望まれる。Goald^[訳注:Goalsのtypo?]はこの場合にデカールをなめらかにoutすることであるが、これは下地の法線を必要とする。

# デカールアーキテクチャ

- 3つの取り得るアプローチ
    - '古典的'なディファードデカール
        - 直接的にGバッファ内で属性をブレンドする
    - Dバッファ(デカールバッファ)
        - 別のDバッファに属性をブレンドする
        - 通常パスでライティングの前にDバッファを適用する
    - Cluster decals [@Sousa2016]
        - デカールがclusteredライトのようになる
        - 通常パスでライティングの前に適用する

DバッファはUnreal Engine 4で用いられるデカールバッファのアプローチである(私が知る限りこれに関するプレゼンテーションはない)。これはGバッファに似ているがデカール用である。

#

|機能:|ディファードデカール|Dバッファ|クラスタデカール|
|-|-|-|-|
|任意のGバッファレイアウト|<font color="red">No</font>|<font color="green">YES</font>|<font color="green">YES</font>|
|ブレンディングモード|多数(ただし、バリアント地獄)|Lerp|Lerp|
|静的ライティングへの影響|<font color="red">No</font>|<font color="green">YES</font>|<font color="green">YES</font>|
|ディファードおよびフォワードのサポート|<font color="red">No</font>|<font color="green">YES</font>|<font color="green">YES</font>|
|透明物のサポート|<font color="red">No</font>|<font color="red">No</font>|<font color="green">YES</font>|
|デカールメッシュのサポート|<font color="green">YES</font>|<font color="green">YES</font>|<font color="red">No</font>|
|法線フェーディングのサポート|<font color="green">YES</font>|<font color="red">No</font>|<font color="green">YES</font>|

銀の弾丸はない！
注意: 我々はプリパス中にフォワードマテリアルに対して法線バッファを出力する。つまり、我々はフォワードパスに対してDバッファと共に法線フェーディングを行うことができない。

# デカールアーキテクチャ

- HDRPは不透明マテリアルに対してDバッファを使う
    - 完全な深度プリパスを必要とする
    - Gバッファの前にプロジェクタとメッシュデカールをレンダリングする

# Dバッファのデザイン

- Dバッファアプローチはデカールのアルファ合成を用いる
    - 半解像度のパーティクル合成と同じ[@Cantlay2007]
- 別個の属性ブレンディングをサポートする
    - 属性あたりの不透明度
    - AOとMetalに対してパッキングされることに注意する --- 選択式のサポート

||R|G|B|A|
|-|-|-|-|
|RT0 RGBA8 sRGB|ディフューズ色.rgb|←|←|ディフューズの不透明度|
|RT1 RGBA8|法線.rgb|←|←|法線の不透明度|
|RT2 RGBA8|Metallic|AO|Smoothness|Smoothnessの不透明度|
|RT3 RG8(任意)|Metallicの不透明度|←|←|AOの不透明度|

別個の属性ブレンディングをサポートするため、各属性は不透明度を持つ必要がある。ここにDバッファのレイアウトがどのように属性と不透明度をパックしたかを示す。
注意: 乗算ブレンドモードはサポートしていない。我々はlerpのみを用いる。

# デカールアーキテクチャ

- ディファード: Dバッファを使う

# デカールアーキテクチャ

- ディファード: Dバッファを使う
- フォワード: Dバッファを使う

# デカールアーキテクチャ

- ディファード: Dバッファを使う
- フォワード: Dバッファを使う --- DepthPrepassで法線バッファを出力する？
- ライティング機能？

ライティング機能に対して、我々はプリパス中に法線バッファを用いることを思い出して欲しい。しかし、この場合、Dバッファはライティング機能の効果に対して法線に影響を与えない。

# デカールアーキテクチャ

- ディファード: Dバッファを使う
- フォワード: Dバッファを使う --- DepthPrepassで法線バッファを出力する
- Dバッファの後に法線バッファをパッチする

# デカールアーキテクチャ

- ディファード: Dバッファを使う
- フォワード: Dバッファを使う --- DepthPrepassで法線バッファを出力する
- Dバッファの後に法線バッファをパッチする
- 法線バッファのパッチ当てを最適化するのにステンシルを使う

# 不透明マテリアル＋デカールレンダパス

- 深度プリパス
    - _
        - フォワードマテリアル: 法線バッファを出力する
            - DecalNormalに対してtag stencilする
    - Dバッファ
        - Decalに対してtag stencilする
    - 法線バッファをパッチする
        - DecalとDecalNormalに対してステンシルをテストする
    - Gバッファ
        - 通常ライティングまたは分割ライティングのためにtag stencilする
    - シャドウのレンダリング
        - 非同期処理
    - ディファード方向性カスケードシャドウ
        - 法線シャドウバイアスのために法線バッファを使う
- タイルディファードライティング
    - シェーダバリアントごとに間接ディスパッチする
        - ステンシルを読む
            - ライティングなし: フォワードマテリアルと空をスキップする
            - 通常ライティング: ライティングを出力する
            - 分割ライティング: ディフューズとスペキュラを分割する
- フォワード不透明
    - (任意)ベース色＋拡散プロファイルを出力する
    - (任意)分割ライティングを出力する
- SS表面下散乱
    - 分割ライティングに対してステンシルをテストする
    - ライティングを組み合わせる

注意: 今や深度プリパスは必須である。
すべての非同期処理(SSR、SSAO)は削って、非同期処理の題でまとめてある。

これはディファードおよびフォワードの間で機能的同等性が可能となる我々のレンダリングフレームである。

# デカールアーキテクチャ

- HDRPは透明物に対してクラスタデカールを用いる
    - プロジェクタのみ
    - 選択式
    - 別個の属性ブレンディングをサポートする
    - クラスタライトリストのように準備されるクラスタデカールリスト
    - ひとつのアトラスにテクスチャを集める

# デカールアーキテクチャ

- HDRPは透明物に対してクラスタデカールを用いる

GPUライトカリング

クラスタライトリスト
クラスタデカールリスト

フォワード透明パス

デカールごとに
↑
累積する

マテリアル属性に結果を適用する

ライトごとに
↑
物体を計算する

# デカールアーキテクチャ

- HDRPは透明物に対してクラスタデカールを用いる
    - ループ中にデカール修正[modification]を累積する
        - ミップに対して通常のUVを使えない
        - 代わりにワールド空間の位置の微分を用いる

```hlsl
// mipmapのlod計算に後で使うために隣接ピクセルに対してワールド空間のddx/ddyを得る
float positionRWSDdx = ddx(positionRWS);
float positionRWSDdy = ddy(positionRWS);

for (uint i = 0; i < decalCount; i++) {
    DecalData decalData = FetchDecal(decalStart, i);

    // ループ内でサンプリングしているので、mipmapのLODを手動で計算する必要がある
    float3 positionDSDdx = mul(worldToDecal, float4(positionRWSDdx, 0.0)).xyz;
    float3 positionDSDdy = mul(worldToDecal, float4(positionRWSDdy, 0.0)).xyz;

    float2 sampleDiffuseDdx = positionDSDdx.xz * decalData.diffuseScaleBias.xy;  // アトラスのスケールにおけるファクタ
    float2 sampleDiffuseDdy = positionDSDdy.xz * decalData.diffuseScaleBias.xy;
    float lodDiffuse = ComputeTextureLOD(sampleDiffuseDdx, sampleDiffuseDdy, _DecalAtlasResolution);
}
```

それが自明でないクラスタデカールを持つ正しいmipをサンプルするため、我々はデカール空間に変換した位置の微分を用いることができることに注意する。アトラスの座標を担当する[take care of]。

# デカールアーキテクチャ

- GPUデカールのパフォーマンスの数値(エントリコストを強調するための単純なシーン)


Decal offはデカールのコードを取り除いた状態を意味する。この目標は0つのデカールがあるときとデカールのコードがないときとでDバッファおよびクラスタアプローチによって誘発するオーバーヘッドを計測することである。
現在我々のアプローチで見られるように、誘発する追加コストは無視できるようなものではない。しかし、これは上手くスケールする。透明マテリアルはデカールを受けるかどうかを選択できる。
Dバッファアプローチでは、我々は多少のパフォーマンスを節約する'デカール分類'の追加ステップを処理する。

これらの計測結果は複数の単純なオブジェクトからなるシーンで行われた。これは複数のテクスチャをフェッチする必要があるデカールは多くを傷つけることを意味する。マテリアルが大量のALUを持ち、すでにいくつものテクスチャをフェッチする複雑なレイヤリングを伴う現実世界のシーンでは、デカールオフとデカールゼロの差異は些細なものである。
また、ここで示される追加コストはシーン全体に対してのものである。

# 不透明マテリアルレンダパス

- 追加の最適化
    - ディファード方向性カスケードシャドウ
        - スクリーン空間でカスケードシャドウマップを投影する
        - ライティングパスの外側でより良いwavefront占有率
    - 不透明なアルファテストされるマテリアルを最適化する
        - プリパス中に不透明アルファテストをレンダリングする
        - アルファテストを無効化して、Gバッファまたはフォワード中にZ-equalを使う

パフォーマンス数値でのテストシーンは様々な草木＋いくつかのテッセレーションを伴う複雑にレイヤー化された地面を持つFountainebleauデモの一般的なエリアである。

# マテリアル

# HDRPのBRDF

- Litシェーダ
    - HDRPのデフォルトシェーダ
    - ディファードマテリアル(フォワードマテリアルに切り替え可能)
    - マテリアル機能のまとめ

# HDRPのBRDF

- Litシェーダ
    - ディフューズ項: Disneyディフューズとして知られるBurleyのディフューズ[@Burley2012]
    - ベース色/メタリックによるパラメータ化

# HDRPのBRDF

- Litシェーダ
    - ディフューズ項: Disneyのディフューズとして知られるBurleyのディフューズ[@Burley2012]
    - ディフューズ色/スペキュラ色によるパラメータ化

# HDRPのBRDF

- Litシェーダ
    - SSS項: DisneyのSSS

# HDRPのBRDF

- Litシェーダ
    - 半透明項: Disneyのディフューズに基づく

# HDRPのBRDF

- Litシェーダ
    - スペキュラ項: 多重散乱[multiscattering]の等方的GGX[@Heitz2016]

# HDRPのBRDF

- Litシェーダ
    - スペキュラ項: 多重散乱の異方的GGX[@Heitz2014]
        - ハック

# HDRPのBRDF

- Litシェーダ
    - 玉虫色[iridescence]項: フレネル項を置き換える[@Belcour2017]

# HDRPのBRDF

- Litシェーダ
    - クリアコートのスペキュラ項: 多重散乱の等方的GGX
        - ハック

# HDRPのBRDF

- Litシェーダ
    - HDRPにおけるマテリアルID: マテリアル機能のビットマスク
    - 例: 標準＋半透明
    - 例: 標準＋クリアコート＋異方性
    - 例: 標準＋玉虫色＋表面下散乱
- Gバッファ制約
    - ストレージ容量のための排他的なマテリアル機能
    - 玉虫色、異方性、表面下散乱/半透明

# Gバッファのデザイン

- 標準

|標準|R|G|B|A|
|-|-|-|-|-|
|RT0 RGBA8 sRGB|ベース色.rgb|←|←|スペキュラオクルージョン|
|RT1 RGBA8|法線.xy(八面体 12/12)|←|←|知覚的smoothness|
|RT2 RGBA8|フレネル0.rgb|←|←|機能マスク(3)/塗装マスク(3)|

- クリアコートはすべてのバリアントで利用可能
- メタリックなし --- フレネル0に展開する[decompress]
    - 最適化＋両方のパラメータ化(メタリック/スペキュラ色)を扱う

# 異方性

- 異方的GGX[@Heitz2014]
    - 高さ相関の可視性項付き
    - 単純化[@McAuley2015]＋最適化

```hlsl
// roughnessT -> tangent方向におけるラフネス
// roughnessB -> bitangent方向におけるラフネス
float D_GGXAnisoNoPI(float TdotH, float BdotH, float NdotH, float roughnessT, float roughnessB) {
    float a2 = roughnessT * roughnessB;
    float3 v = float3(roughnessB * TdotH, roughnessT * BdotH, a2 * NdotH);
    float s = dot(v, v);

    return INV_PI * a2 * (a2 / s) * (a2 / s);
}

float V_smithJointGGXAniso(float TdotV, float BdotV, float NdotV, float TdotL, float BdotL, float NdotL, float roughnessT, float roughnessB) {
    real lambdaV = NdotL * length(float3(roughnessT * TdotV, roughnessB * BdotV, NdotV));
    real lambdaL = NdotV * length(float3(roughnessT * TdotL, roughnessB * BdotL, NdotL));

    return 0.5 / (lambdaV + lambdaL);
}
```

# 異方性

- [@Kulla2017]の異方性パラメータ化を用いる

```hlsl
roughnessT = roughness * (1 + anisotropy);
roughnessB = roughness * (1 - anisotropy);
```

- [@Revie2011; @McAuley2015]由来の法線ベクトルのハックを再検討する
    - tangentおよびbitangentの伸縮[stretching]をサポートする
    - IBLテクスチャMIPをフェッチするためにラフネスの使い方を修正する
    - マジックナンバーを目測する[eye-ball]

```hlsl
// 異方比[anisotropic ratio](0 -> 等方性なし、1 -> tangent方向における完全な異方性) --- 正だとbitangentWSを使う --- 負だとtangentWSを使う
void GetGGXAnisotropicModifiedNormalAndRoughness(float3 bitangentWS, float3 tangentWS, float3 N, float3 V, float anisotropy, float perceptualRoughness) {
    // 正の異方性の値では: tangent = highlight stretch(異方性)の方向、bitangent = grain (brush)の方向
    float3 grainDirWS = (anisotropy >= 0.0) ? bitangentWS : tangentWS;
    // (perceptualRoughness < 0.2)で伸縮を減らす
    float stretch = abs(anisotropy) * saturate(1.5 * sqrt(perceptualRoughness));
    // grain方向(例、髪やbrushの方向)は法線と直交すると仮定する
    float3 B = cross(grainDirWS, V);
    float3 grainNormal = cross(B, grainDirWS);
    iblN = normalize(lerp(N, grainNormal, stretch));
    iblPerceptualRoughness = perceptualRoughness * saturate(1.2 - abs(anisotropy));
}
```

ハックは純粋に経験則的である☺

# 法線ベクトルのハック

伸縮を確認するためにこの較正キューブマップを用いる。
異方性は左から右へ: -1から1
知覚的smoothnessは下から上へ: 1から0

# リファレンス(エンジン内)

かなり不正確に見えるが、高周波なキューブマップでは…

# 法線ベクトルのハック

大丈夫そうであり、(遠くから)リファレンスのように見ることができるであろうものがほとんどである。

# リファレンス(エンジン内)

将来的に、我々はこのハックの代わりに異方的フィルタリングで行いたいと考えている。

# Gバッファのデザイン

- 異方性

|異方性|R|G|B|A|
|-|-|-|-|-|
|RT0 RGBA8 sRGB|ベース色.rgb|←|←|スペキュラオクルージョン|
|RT1 RGBA8|法線.xy(八面体 12/12)|←|←|知覚的smoothness|
|RT2 RGBA8|異方性|tangent frame angle(11)/メタリック(5)|←|機能マスク(3)/塗装マスク(3)|

- 実際のtangent frameとデフォルトのそれとの間の角度を用いる

```hlsl
// tangent frameをエンコードする
// デフォルトのtangent frameを再構築する
float3x3 frame = GetLocalFrame(normalWS);
// デフォルトのそれに関して実際のtangent frameの回転角を計算する
float sinFrame = dot(tangentWS, frame[1]);
float cosFrame = dot(tangetnWS, frame[0]);
uint storeSin = abs(sinFrame) < abs(cosFrame) ? 4 : 0;
uint quadrant ((sinFrame < 0) ? 1 : 0) | ((cosFrame < 0) ? 2 : 0);
// sin[とcos]は最大で[後の]45度で近似的に線形である
float sinOrCos = min(abs(sinFrame), abs(cosFrame)) * sqrt(2);
outGBuffer2.rgb = float3(surfaceData.anisotropy * 0.5 + 0.5, sinOrCos, PackFloatInt8bit(metallic, storeSin | quadrant, 8));
```

```hlsl
// tangent frameをデコードする
float unused;
uint tangentFlags;
UnpackFloatInt8bit(inGBuffer2.b, 8, unused, tangentFlags);
// デフォルトのそれに関して実際のtangent frameの回転角を得る
uint quadrant = tangentFlags;
uint storeSin = tangentFlags & 4;
float sinOrCos = inGBuffer2.g * rsqrt(2);
float cosOrSin = sqrt(1 - sinOrCos * sinOrCos);
float sinFrame = storeSin ? sinOrCos : cosOrSin;
float cosFrame = storeSin ? cosOrSin : sinOrCos;
sinFrame = (quadrant & 1) ? -sinFrame : sinFrame;
cosFrame = (quadrant & 2) ? -cosFrame : cosFrame;
// 法線のまわりで再構築されたtangentを回転する
tangentWS = sinFrame * frame[1] + cosFrame * frame[0];
bitangentWS = cross(frame[2], frame[0]);
```

# クリアコート

- 物理的でない --- 単純化されたアプローチ
    - ベースの上にある等方的GGX
    - 1.5の固定された屈折率(0.04のF0)と0.03のラフネス
    - 塗装の界面を計算に入れたF0でベースのスペキュラを計算する
        - ゲームでは入力のF0(つまり、スペキュラ色)によるSchlick Fresnelを使った
            - ゲームにおけるF0は空気の界面(屈折率1)で計算される

# クリアコート

- 誘電体では、以下で塗装の界面(屈折率1.5)に対してベースのF0を適合させられる

```hlsl
float IorToFresnel0(float transmittedIor, float incidentIor) {
    return Sq((transmittedIor - incidentIor) / (transmittedIor + incidentIor));
}

float Fresnel0ToIor(float f0) {
    return ((1.0 + sqrt(f0)) / (1.0 - sqrt(f0)));
}

float ConvertF0ForAirInterfaceToF0ForClearCoat15(float f0) {
    return IorToFresnel0(Fresnel0ToIor(f0), 1.5);
}

// 最適化: 範囲[0.04(0を返すべき), 1(1を返すべき)]に対する関数(3 mad)のフィッティング
float ConvertF0ForAirInterfaceToF0ForClearCoat15Fast(float f0) {
    return saturate(-0.0256868 + f0 * (0.326846 + (0.978946 - 0.283835 * f0) * f0));
}
```

- 導体では、スペクトル的複素屈折率[spectral complex IOR]の畳み込みを必要とする
- アイデア
    - 屈折率1.5の界面に対するリファレンスを計算する[@Lagarde2011]
    - 導体に対して上記の式と比較してみる

# クリアコート

- そのコストが実行時パフォーマンスに対してそれほど悪くないと仮定すれば
    - より低い値で誤差が増加する
    - クリアコートが有効化されるときにベースのF0を更新するためにこのアプローチを使う

# クリアコート

- すべてのライトタイプに対して
    - 塗装の界面を計算に入れたF0を持つベースのスペキュラを計算する
    - クリアコートに対してフレネル項を計算する
    - ベースに関する近似的なエネルギー保存則を適用する
    - ベースのスペキュラに塗装のスペキュラ寄与を追加する

```hlsl
baseF0 = ConvertF0ForAirInterfaceToF0ForClearCoat15Fast(fresnel0);
(...)  // BaseSpecularの計算

float coatF = F_Schlick(CLEAR_COAT_F0, LdotH);
baseSpecular *= Sq(1.0 - coartF);  // 交差する界面の出入りに対してベースのスペキュラをスケールする
float DV = DV_SmithJointGGX(NdotH, NdotL, NdotV, CLEAR_COAT_ROUGHNESS);
baseSpecular += coatF * DV;
```

- IBL: NdotVによるSchlick Fresnelを使う＋追加のIBLフェッチ

エリアライトは追加のLTC計算を使う。
将来的に、我々は、我々のstacklitシェーダアプローチに基づいた、より物理的な方法を用いるだろう。

#

# 多重散乱GGX

- エネルギー保存を改善する
    - GGX式における多重散乱の欠落
        - furnace test(一様なHDRI)における最大60%の光の喪失(rough case)
- 現在のトレンドは追加の埋め合わせ[compensation]項で近似することである
    - [@Kulla2017; @Hill2018]

# 多重散乱GGX

- [@Heitz2016]はground truthな挙動をもたらす
    - より荒くなると、ディフューズとスペキュラの両方がより飽和する
    - シミュレーションはバウンスローブが似ていることを示している
        - 単一散乱GGXでのスケールファクタで十分であることを暗示している

# 多重散乱GGX

- Emmanuel Turquinの功績
- スケールファクタはFresnelに依存しなければならない
- $\rho(\omega_o, \omega_i) = \rho_{ss}(\omega_o, \omega_i) + F_{ms} k_{ms}(\omega_o) \rho_{ss}(\omega_o, \omega_i)$
- ここで、$k_{ms}(\omega_o) = \frac{1 - E_{ss}(\omega_o)}{E_{ss(\omega_o)}}$であり、$E_{ss}(\omega_o) = \int_{\Omega_i} \rho(\omega_o, \omega_i) |\omega_i \cdot n| d\omega_i$である
- フレネル項はHDRPにおいてコサイン加重平均Schlick Fresnelである
    - $F_{ms} \approx F_{ss} = 2 \int_0^1 F(\mu) \mu d\mu = \frac{(1+20F_0)}{21} \approx F_0$

$$
F_{ms} \approx F_{ss} = 2 \int_{0}^{1} F(\mu)\mu \mathrm{d}{\mu} = \frac{(1 + 20 F_0)}{21} \approx F_0 \\
E_{ss}({\omega_o}) = \int_{\Omega_i} \rho({\omega_o}, {\omega_i}) |{\omega_i \cdot n}| \mathrm{d}{\omega_i} \\
k_{ms}({\omega_o}) = {\frac{1 - E_{ss}({\omega_o})}{E_{ss}({\omega_o})}} \\
\rho(\omega_o, \omega_i) = \rho_{ss}(\omega_o, \omega_i) + F_{ms} k_{ms}(\omega_o) \rho_{ss}(\omega_o, \omega_i)
$$

# 多重散乱GGX

- 直接および間接の両方のスペキュラにライトループの最後でファクタを適用する
    - オリジナルのGGXローブのスケールとして機能する
    - $E_{ss}(\omega_o)$をテクスチャに格納する。キューブマップの事前統合で共有する

```hlsl
specularLighting = lighting.direct.specular + lighting.indirect.specularReflected;
// y = Integral{(BSDF / F) * <N, L> dw}
float3 preFGD = SAMPLE_TEXTURE2D_LOD(_PreIntegratedFGD_GGXDisneyDiffuse, s_linear_clamp_sampler, float2(NdotV, perceptualRoughness), 0).xyz;
// 多重散乱を計算に入れるためにGGXを再スケールする
specularLighting *= 1.0 + fresnel0 * ((1.0 / reflectivity) - 1.0);
```

- ディフューズ項に対しては多重散乱はない
    - Disneyのディフューズは経験則的であり、エネルギー保存的ではない
        - ラフネスを増やしても暗くならない

#

多重散乱なし
１行目は純白の誘電体 --- ここでディフューズ項はDisneyのディフューズである
２行目はF0 = 1 --- 導体
３行目はF0 = 金 --- 導体
４行目はF0 = 銅 --- 導体

#

多重散乱あり
１行目は純白の誘電体 --- ここでディフューズ項はDisneyのディフューズである
２行目はF0 = 1 --- 導体
３行目はF0 = 金 --- 導体
４行目はF0 = 銅 --- 導体

#

多重散乱なし
１行目は純白の誘電体 --- ここでディフューズ項はDisneyのディフューズである
２行目はF0 = 1 --- 導体
３行目はF0 = 金 --- 導体
４行目はF0 = 銅 --- 導体

#

多重散乱あり
１行目は純白の誘電体 --- ここでディフューズ項はDisneyのディフューズである
２行目はF0 = 1 --- 導体
３行目はF0 = 金 --- 導体
４行目はF0 = 銅 --- 導体

#

多重散乱なしでの異方性

#

多重散乱ありでの異方性

#

多重散乱なしでの異方性 => 伸縮ハック付きの偽の異方性を用いているので、上記の見た目は異方性なしの場合ときっかり同じである。

#

多重散乱ありでの異方性は完璧にエネルギー保存している！全体としては偽物であるが、見た目は悪くない。

# 多重散乱GGX

- 検証のためにMitsubaと比較させた
- 注意: Mitsubaは金のスペクトル的複素屈折率を使う＋相互反射がある

上がHDRP、下がMitsuba
Mitsubaとの比較はそれほど悪くない。しかし、Mitsubaはより正確で光輸送(球の中での球の反射)を含む方法であるので、公平な比較を行うことはそう単純ではない。

# マテリアル最適化

- $E_{ss}(\omega_o)$は他のアルゴリズムと共有できる
- キューブマップ事前統合FGD項を再設計する[@Karis2013]
    - $FGD = \int_{\Omega_i} (F_0 + (1 - F_0) * (1 - |V \cdot H|)^5) \rho(V, L)|L \cdot N| dL$
    - $FGD = (1 - F_0) * \int_{\Omega_i} (1 - |V \cdot H|)^5 \rho(V, L)|L \cdot N| dL + F_0 * \int_{\Omega_i} \rho(V, L)|L \cdot N| dL$
    - $FGD = (1 - F_0) * x + F_0 * y$
- FGD.yは以下のために使う
    - 事前統合されたFGD
    - 多重散乱
    - エリアライト: LTCフレネル近似[@Hill2016]

# 玉虫色

- Unity Labsの研究に基づく
    - Laurent Belcour: A Practice Extension to Microfacet Theory for the Modeling of Varying Iridescence
    - コードはBRDF Explorerに提供される
    - リアルタイムで計算するには高コストであり、近似が必要である

# 玉虫色

- リアルタイム用の近似(功績: Laurent Belcour)
    - Schlick Fresnelを使う
        - Schlick Fresnelは屈折率が1.4〜2.2の範囲の外だと誤ったものになる[@Lagarde2013]
        - 屈折率1.0は効果を取り消すことを想定する --- 代わりにマスクパラメータを使う
    - RGB色空間のみを使う
        - オリジナルのコードはXYZ色空間を使う
    - 位相変位[phase shift]を単純化する
    - より小さい反射率項を使う

# 玉虫色

```hlsl
// 誘電体の媒質の上部にある薄膜層[thin-film layer]に対する反射率を計算する
float EvalIridescence(float eta_1, float cosTheta1, float iridesenceThickness, float3 baseLayerFresnel0) {
    // iridescenceThicknessの単位はここのこの式ではマイクロメートルである。0.5は500nmを意味する。
    float Dinc = 3.0 * iridesenceThickness;
    float eta_2 = lerp(2.0, 1.0, iridesenceThickness);
    float sinTheta2 = Sq(eta_1 / eta_2) * (1.0 - Sq(cosTheta1));
    float cosTheta2 = sqrt(1.00 - sinTHeta2);

    // 第１界面
    float R0 = IorToFresnel0(eta_2, eta_1);
    float R12 = F_Schlick(R0, cosTheta1);
    float R21 = R12;
    float T121 = 1.0 - R12;
    float phi12 = 0.0;
    float phi21 = PI - phi12;

    // 第２界面
    float R23 = F_Schlick(baseLayerFresnel0, cosTheta2);
    float phi23 = 0.0;

    // 位相変位
    float OPD = Dinc * cosTheta2;
    float phi = phi21 + phi23;

    // 項を混ぜ合わせる
    float3 R123 = R12 * R23;
    float3 r123 = sqrt(R123);
    float3 Rs = Sq(T121) * R23 / (float3(1.0, 1.0, 1.0) - R123);

    // m = 0に対する反射率項(DC項の増幅)
    float3 C0 = R12 + Rs;
    float3 I = C0;

    // m > 0に対する反射率項(ディラックのペア)
    float3 Cm = Rs - T121;
    for (int m = 1; m <= 2; ++m) {
        Cm *= r123;
        float3 Sm = 2.0 * EvalSensitivity(m * OPD, m * phi);
        I += Cm * Sm;
    }

    return I;
}
```

コードはリファレンスとして提供する。EvalSensitivityは論文のオリジナルコードと同じ関数である。

# 玉虫色

- EvalIridescenceはベースのFresnel0を置き換えるために呼び出される
    - すべてのライトタイプで行われる
    - 理論上パンクチュアルライトごとこれを呼び出すようになっている --- 高価すぎる
- クリアコートと組み合わせる(ハックな方法[hacky way])

```hlsl
float topIor = 1.0;  // デフォルトでは空気
if (HasFlag(bsdfData.materialFeatures, MATERIALFEATUREFLAGS_LIT_CLEAR_COAT)) {
    topIor = CLEAR_COAT_IOR;
    // HACK: 事前畳み込みされた環境マップに対するフレネル係数を特定するために反射方向を使う
    NdotV = sqrt(1.0 + Sq(1.0 / topIor) * (Sq(dot(bsdfData.normalWS, V)) - 1.0));
}
fresnel0 = EvalIridescence(topIor, NdotV, iridescenceThickness, fresnel0);
```

- パラメータ化は依然としてフレンドリーではない

# Gバッファのデザイン

- 玉虫色

|玉虫色|R|G|B|A|
|-|-|-|-|-|
|RT0 RGBA8 sRGB|ベース色.rgb|←|←|スペキュラオクルージョン|
|RT1 RGBA8|法線.xy(八面体 12/12)|←|←|知覚的smoothness|
|RT2 RGBA8|屈折率|厚さ|未使用(3)/メタリック(5)|機能マスク(3)/塗装マスク(3)|

- 3ビットは最適化目的のために未使用
    - メタリックを異方性のエンコーディングと合わせる

# 表面下散乱

- このセッションの次のトークを参照ください！
    - Efficient Screen-Space Subsurface Scattering Using Burley's Normalized Diffusion

# 半透明

- このセッションの次のトークを参照ください！
    - Efficient Screen-Space Subsurface Scattering Using Burley's Normalized Diffusion
- 表面下散乱からの別個のマテリアル機能
- HDRPのサポート:
    - 表面下散乱＋半透明
    - 表面下散乱のみ
    - 半透明のみ
        - foliage

# Gバッファのデザイン

- 表面下散乱および/または半透明

|SSS+透過|R|G|B|A|
|-|-|-|-|-|
|RT0 RGBA8 sRGB|ベース色.rgb|←|←|拡散プロファイル(4)/サブサーフェスマスク(4)|
|RT1 RGBA8|法線.xy(八面体 12/12)|←|←|知覚的smoothness|
|RT2 RGBA8|スペキュラオクルージョン|厚さ|拡散プロファイル(4)/サブサーフェスマスク(4)|機能マスク(3)/塗装マスク(3)|

- SSSSSパスはRT0のみを必要とする
    - 帯域幅を節約するためにスペキュラオクルージョンの位置を入れ替える
- 最適化目的のために拡散プロファイル/サーフェスマスクを複製する

この奇妙な配置は帯域幅を節約し、SSSに必要なすべての情報をRT0ひとつに格納することを可能とするためにある。ライティングコードはかなり最後になるまでRT0を読むことを必要としないので、拡散プロファイルとサブサーフェスマスクは先にRT0を読まなくて良いように複製される。

# Litシェーダのパフォーマンス

- 基本のPS4 --- 1080p --- 単純な入力を持つマテリアルでのフルスクリーンクアッド
- 単一のライト＋空によって影響を受ける --- シャドウなし
    - マテリアルとライトの分類を用いる
    - 表面下散乱機能は2つのレンダターゲットに書き込む
    - フォワードパスは大域照明をサンプルする(追加コスト)

認められることは、我々のエリアライトのコストはパンクチュアルライトの価格の倍あることである。
反射プロブは複合的なマテリアルでよいパフォーマンスを持つ(我々が行う様々な近似のため)。

# StackLitシェーダ

- VFX/映像/映画を対象とする
    - Litの正確なバージョン
        - いくつかの近似を取り除く
    - 同時にすべてのマテリアル機能をサポートする
        - 例: 玉虫色＋SSS＋半透明＋塗装
    - 常にフォワードマテリアル
- 垂直な2レイヤシェーダ
    - ベース＋塗装レイヤ

# StackLitシェーダ

- Unity Labsの研究に基づく --- Laurent Belcour
    - Efficient Rendering of Layered Materials using an Atomic Decomposition with Statistical Operators
        - 10月14日(火) 午前10:45

# キャラクター

- キャラクターシェーダ
    - 常にフォワードマテリアル
        - 髪
        - 布
        - 目

# LayeredLit

- いくつかのLitマテリアルを一緒にミックスするための設備[facilities]
    - マスキングに対する様々な重みをサポートする
    - フォトグラメトリを対象とする*influence mode*
- "Photogrammetry workflow Layered Shader"という電子書籍で詳しく述べる
- 複合的でリッチな環境を作りたい
    - 法線マップの複合的なレイヤリングを意味する
    - しかし…

# 法線マッピングの問題

- しかし、法線マッピングは多くの問題がある
    - UVごとにtangent frameを1つ必要とする、プロシージャルなUVでさえ
    - ボリュームバンプマッピングを扱いづらい(triplanar/ノイズ)
    - 複数ブレンディングの定式化[@Brisebois2012]
        - ある程度の順序依存
    - プロシージャルなジオメトリで実践的でない
        - 多くのブレンドシェイプを扱いづらい

# 法線マッピング

- 位置、法線、UVから構築されるオンザフライなタンジェント基底[@Mikkelsen2010]

通常のタンジェント基底^[訳注:MikkTSpace = Morten S. Mikkelsenの修士論文中に登場するタンジェント空間を扱うコードを指す。]
```hlsl
// これはmikktspace変換である(正規化済みでない属性を使う)
float3x3 worldToTangent = CreateWorldToTangent(unnormalizedVertexNormalWS, vertexTangentWS.xyz, flipSign);

// 定式化に基づく表面の勾配は単位長の初期法線を必要とする。我々は、摂動法線[perturbed normal]の正規化が打ち消されるので、すべての3つのベクトルを一様にスケーリングすることによってmikktsによる整合性[compliance]を維持できる。
float renormFactor = 1.0 / length(unnormalizedNormalWS);
worldToTangent[0] = worldToTangent[0] * remormFactor;
worldToTangent[1] = worldToTangent[1] * remormFactor;
worldToTangent[2] = worldToTangent[2] * remormFactor;  // 保管された頂点法線を正規化する

float3 vertexNormalWS = worldToTangent[2];
// UVに対するタンジェント基底を0に設定する
vertexTangetWS0 = input.worldToTangent[0];
vertexBitangetWS0 = input.worldToTangent[1];
```

オンザフライなタンジェント基底
```hlsl
// PositionWSはカメラに相対的[camera-relative]である
float3 dPdx = ddx_fine(positionWS);
float3 dPdy = ddy_fine(positionWS);

float3 sigmaX = dPdx - dot(dPdx, vertexNormalWS) * vertexNormalWS;
float3 sigmaY = dPdy - dot(dPdy, vertexNormalWS) * vertexNormalWS;
float flipSign = dot(dPdy, cross(vertexNormalWS, dPdx)) < 0.0 ? -1.0 : 1.0;

// 複数のUVセット(1〜3)に対するタンジェント基底を計算する
SurfaceGradientGenBasisTB(vertexNormalWS, sigmaX, sigmaY, flipSign, texCoord1, out vertexTangentWS1, out vertexBitangentWS1);
SurfaceGradientGenBasisTB(vertexNormalWS, sigmaX, sigmaY, flipSign, texCoord2, out vertexTangentWS2, out vertexBitangentWS2);
SurfaceGradientGenBasisTB(vertexNormalWS, sigmaX, sigmaY, flipSign, texCoord3, out vertexTangentWS3, out vertexBitangentWS3);
```

# 法線マッピング

- 各UVセットはGPUコスト[ALUs]を追加する

```hlsl
// これはプロシージャルに生成されるものを含めたいずれかのUVに対して頂点レベルの正接/従正接なしで正接および従正接の正規直交基底をもたらす
void SurfaceGradientGenBasisTB(real3 nrmVertexNormal, real3 sigmaX, real3 sigmaY, real flipSign, real2 texST, out real3 vT, out real3 vB) {
    real2 dSTdx = ddx_fine(texST), dSTdy = ddy_fine(texST);

    real det = dot(dSTdx, real2(dSTdy.y, -dSTdy.x));
    real sign_det = det < 0 ? -1 : 1;

    // invC0は(dXds, dYds)を表すが、我々は行列式で除算しない(代わりに符号でスケールする)
    real2 invC0 = sign_det * real2(dSTdy.y, -dSTdx.y);
    vT = sigmaX * invC0.x + sigmaY * invC0.y;
    if (abs(det) > 0.0) vT = normalize(vT);
    vB = (sign_det * flipSign) * cross(nrmVertexNormal, vT);
}
```

# 法線マッピング

- 実践では
    - UV0に基づくタンジェント基底はMikktspaceを使う[@Mikkelsen2008]
        - オンザフライなTBNはhardな表面を持つローポリメッシュを扱う点で良くない
            - つまり、法線マップがメッシュの正確な形状へ使われるとき
    - 他のUV1〜3ではオンザフライな基底を生成する
    - 法線マッピングの問題のごく一部のみを解決するにすぎない…

# 表面勾配フレームワーク

- 表面勾配ベースのアプローチ[@Mikkelsen2010]
    - 表面勾配は表面の傾きの方向におけるベクトルである
    - 高さの微分から作る

スカラの高さフィールドがあるとすると、そのフィールドの勾配は各ベクトルが最も大きな変化の方向を指す2Dベクトル場である。ベクトルの長さは変化率[rate of change]に対応する。

# 表面勾配フレームワーク

- 摂動法線はn' = n - SurfGrad(Height)として表現できる

```hlsl
float3 SurfaceGradientResolveNormal(float3 normalizedVertexNormal, float3 surfGrad) {
    return normalizedVertexNormal - surfGrad;
}
```

- SurfGrad()は線形な操作である
    - いずれのバンプの影響度の重み付き組み合わせに対しても機能する
- すべては表面の勾配に変換できる
    - 通常のタンジェント基底
    - UV、位置、法線から作られるオンザフライなタンジェント基底
    - オブジェクト空間の法線
    - ボリュームバンプマップ

1. 表面勾配ベースのアプローチ[@MM2010 - sfgrad]はひとつのフレームワークにこのすべてを統一することを可能にする。
2. Blinnの摂動法線はn' = n - SurfGrad(H)で表現できる、と[@MM2010 - sfgrad]に示されている。
3. SurfGrad(H)は線形操作であり、任意のバンプ影響度の重み付き組み合わせに対して機能するだろう。
    - オブジェクト空間の法線はオンザフライに(ピクセルシェーダで)表面の勾配へ変換できる。
    - 因習的なmikktspace対応の頂点レベルのタンジェント空間はオンザフライに表面法線へ変換できる。
        - 我々は頂点のタンジェント空間なしでUV、位置、法線からオンザフライに表面法線を生成することもできる。
        - ボリュームバンプマップに対して、我々はオンザフライに表面法線を生成できる。これは[@MM2010 - sfgrad]に示されるように正しい結果をもたらす。

# 表面法線フレームワーク

```hlsl
// オンザフライなTBN(tspaceNormalToDerivative()を用いて得られるderiv)から、または
// 因習的な頂点レベルのTBN(mikktspace対応およびtspaceNormalToDerivative()を用いて得られるderiv)からの表面勾配
float3 SurfaceGradientFromTBN(float2 deriv, float3 vT, float3 vB) {
    return deriv.x * vT + deriv.y * vB;
}

// オブジェクトまたはワールド空間の法線マップからのような、すでに生成された"法線"からの表面勾配
// vは、それが方向を立証[establish]している限り、単位長である必要はない
float3 SurfaceGradientFromPerturbedNormal(float3 nrmVertexNormal, float3 v) {
    float3 n = nrmVertexNormal;
    float s = 1.0 / max(FLT_EPS, abs(dot(n, v)));
    return s * (dot(n, v) * n - v);  // 訳注:画像が不明瞭で判別できない
    // return s * (dot(n, v) * n + v);
}

// パーリンノイズのボリュームのようなボリュームバンプ関数の勾配から表面勾配を生成するのに使われる
// "bump mapping unparametrized surfaces on the GPU"の式2
// 図2ではその勾配を使う場合とバンプマッピングを行うための表面勾配を使う場合との間の差異を確認できる(オリジナルの手法は論文中では誤って証明されている！)
float3 SurfaceGradientFromVolumeGradient(float3 nrmVertexNormal, float3 grad) {
    return grad - dot(nrmVertexNormal, grad) * nrmVertexNormal;
}

// ボリュームバンプマップの特例を考慮したtriplanar投影
// deriv等はtspaceNormalToDerivative()を用いて得られ、computeTriplanarWeights()を用いて重み付けする
float3 SurfaceGradientFromTriplanarProjection(float3 nrmVertexNormal, float3 triplanarWeights, float2 deriv_xlane, float2_deriv_yplane, float2 deriv_zplane) {
    const float w0 = triplanarWeights.x, w1 = triplanarWeights.y, w2 = triplanarWeights.z;

    // driv_xplane、deriv_yplane、derive_zplaneが個々に(z,y)、(z,x)、(x,y)を用いてサンプルされると仮定する
    // ルックアップ座標の正のスケールは同様に機能するだろうが、微分要素の負のスケールは適宜に否定を取る必要があるだろう
    float volumeGrad = float3(w2 * deriv_zplane.x + w1 * deriv_yplane.y, w2 * deriv_zplane.y + w0 * deriv_xplane.y, w0 * deriv_xplane.x + w1 * deriv_yplane.x);

    return SurfaceGradientFromVolumeGradient(nrmVertexNormal, volumeGrad);
}
```

# 表面勾配フレームワーク

```hlsl
// この128は微分が数値的に128より大きくならないであろうことを意味する(1は45度なので、128は非常に急勾配)
// 基本的にはtan(angle)を128に制限する
// なので、最大角度は89.55度となり、90度の垂直限界に十分近いと考える
// vTは[-1; 1]にあるタンジェント空間の法線のchannels.xyである
// 出力: vTを微分に変換する
float2 tspaceNormalToDerivativeRGB(float4 packedNormal, float scale = 1.0) {
    const float fS = 1.0 / (128.0 * 128.0);
    float3 vT = packedNormal.xyz * 2.0 - 1.0;
    float3 vTsq = vT * vT;
    float maxcompxy_sq = fS * max(vTsq.x, vTsq.y);
    float z_inv = rsqrt(max(vTsq.z, maxcompxy_sq));
    float2 deriv = -z_inv * float2(vT.x, vT.y);
    return deriv * scale;
}
```

タンジェント空間の法線から微分への変換は、一様なスケーリングを表現するので、表面勾配としてTBN変換を書き換えることが可能になる。

n = (nx, ny, nz)の微分はd = (-nx/nz, -ny/nz)

なので、最後の正規化の後、vT*n.x+vB*n.y+vN*n.zはvN-(d.x*vT+d.y*vB)と同じであるTBN変換を得る。ここで、括弧内の部分は基本的にTBNが一様にスケールされるあなたの他のスライドと一緒に使われるときのTBNスタイルの表面勾配である。
worldToTangetを含むコードを伴う法線マッピングのスライド。つまり、前者では、vT、vB、vNは補間以降すべて正規化されていない。後者の表面勾配のバリアントでは、一様にスケールされる(表面勾配の式が必要とするのでvNを正規化するためのトリックとして)。このファクタは一様なスケーリングである微分を作るためにnzでの除算と共に最終的な正規化では打ち消される。

# 表面勾配フレームワーク

- triplanar法線マッピングは実装するにはトリッキーとなり得る
    - しばしば誤った向きになる

# 表面勾配フレームワーク

- 表面勾配はより単純でこの問題を示さない

# 表面勾配フレームワーク

- 法線マッピング問題に対する良い解法
    - 使用例
        - レイヤー化したマテリアル
            - ベース＋３レイヤ
            - ベースのUV0
            - レイヤのUV0〜2またはplanar
            - ディテールマップのためのUV3
        - 様々なブレンドマスクモード

# 表面勾配フレームワーク

- パフォーマンス数値
    - オンザフライなタンジェント基底によって暗示されるコスト
    - 複合的なマテリアル、木、foliage、地面を持つシーン

# ライティング

# 物理的な光の単位

- [@Lagarde2014]に基づく --- 同じ定式化

|||
|-|-|
|パンクチュアルライト|光量(lm)、光度(cd)|
|エリアライト|光量(lm)、輝度(cd/m^2)またはEV値(K=12.5)|
|発光|輝度(cd/m^2)|
|環境|上半分の半球の照度(lux)|
|太陽|天頂[zenith]の太陽によるグラウンドレベルの照度(lux)|

- スポットライトの絞りに対する2つのモード
- Occlusion
- Reflector
    - 光度が絞りで変化する

```cs
// Occlusion
public static float ConvertPointLightLumenToCandela(float intensity) {
    return intensity / (4.0f * Mathf.PI);
}

// Reflector
public static float ConvertPointLightLumenToCandela(float intensity, float angle) {
    return intensity / (2.0f * (1.0f - Mathf.Cos(angle / 2.0f)) * Mathf.PI);
}
```

皆[people]は気付いていないかもしれないが、逆二乗の減衰を用いることは物理単位を用いることを意味する。すなわち、それはカンデラであり、ディレクショナルライトではルクス(PIで割れば)であり、その他ではPI*ルクスである

# 物理的な光の単位

- パンクチュアルライトおよび太陽光の色
    - フィルタ＋色温度
    - 正確なフィッティング近似(最大誤差: 0.008)

```cs
// 相関のある色温度(ケルビン)として、等価なRGBを推定する。曲線フィッティング誤差は最大0.008
// 線形RGB空間における色を返す
public static Color CorrelatedColorTemperatureToRGB(float temperature) {
    float r, g, b;

    // 温度は1000から40000度の間になければならない
    // フィッティングはケルビンを1000で割る必要がある(より高い精度を可能にする)
    float kelvin = Mathf.Clamp(temperature, 1000.0f, 40000.0f) / 1000.f;
    float kelvin2 = kelvin * kelvin;

    // ピボットは近似であるので、6570を用いる。pivot pointは赤で約6580であり、青と緑で6560である
    // 順次各色を計算する(注、すべての値は[0,1]に属するので、クランプは本当は必要ないが、局地で役に立つ可能性がある)
    // 赤
    r = kelvin < 6.57f ? 1.0f : Mathf.Clamp((1.35651f + 0.216422f * kelvin + 0.000633715f * kelvin2) / (-3.24223f + 0.918711f * kelvin), 0.0f, 1.0f);
    // 緑
    g = kelvin < 6.570f ? Mathf.Clamp((-399.809f + 414.271f * kelvin + 111.543f * kelvin2) / (2779.24f + 164.143f * kelvin + 84.7356f * kelvin2), 0.0f, 1.0f) : Mathf.Clamp((1370.38f + 734.616f * kelvin + 0.689955f * kelvin2) / (-4625.69f + 1699.87f * kelvin), 0.0f, 1.0f);
    // 青
    b = kelvin > 6.570f ? 1.0f : Mathf.Clamp((348.963f - 523.53f * kelvin + 193.62f * kelvin2) / (2848.82f - 214.52f * kelvin + 78.8614f * kelvin2), 0.0f, 1.0f);

    return new Color(r, g, b, 1.0f);
}
```

# 物理的な光の単位

- HDRI
    - 大量の相対的HDRIが入手できるが、絶対的HDRIが欲しい
        - 計測データを必要とする[@Lagarde2016]
    - 上半分の半球に対してではなく望ましい照度の値(ルクス)を入力する
    - DHRIの実効値との比を計算し、乗数を適用する
        - エディタでGPUによるブルートフォース的な球の一様サンプリングを用いる

```hlsl
float GetUpperHemisphereLuxValue() {
    float sum = 0.0;
    float dphi = 0.005;
    float dtheta = 0.005;
    for (float phi = 0; phi < 2.0 * PI; phi += dphi) {
        for (float theta = 0; theta < PI / 2.0; theta += dtheta) {
            float3 L = SphericalToCartesian(phi, cos(theta));
            real val = Luminance(SAMPLE_TEXTURECUBE_LOD(skybox, sampler_skybox, L, 0).rgb);
            sum += cos(theta) * sin(theta) * val;
        }
    }
    return sum * dphi * dtheta;
}

float luxMultiplier = desiredLuxValue / upperHemisphereLuxValue;
```

# 物理的な光の単位

- エリアライト[@Heitz2017]
    - Linearly Transformed Cosineに基づく
    - Unity LabsとLucasfilm researchの共作
        - 矩形と線をサポートする
        - 球と円板はもうすぐ
    - 線ライトは面積を持たないので、線に沿って放射輝度を積分する

```cs
public static float CalculateLineLightLumenToLuminance(float intensity, float lineWidth) {
    // 円筒[tube]ライトと異なり、我々の線ライトは表面積を持たず、すなわち、その積分は以下のようになる:
    // power = Integral{length, Integral{sphere, radiance}}
    // 等方的な線ライトでは、放射輝度は一定である。故に:
    // power = length * (4 * Pi) * radiance
    return intensity / (4.0f * Mathf.PI * lineWidth);
}
```

シャドウはありません☹

#

続いては: ボリューメトリクス！SIGGRAPH 2018 HDRP volumetrics.mp4をご覧ください。

# ボリューメトリックライティング

# 概要

- 人気の錐台に並行な3Dテクスチャ(ボクセルバッファ)技術を使う[@Vos2014; @Wronski2014; @Hillaire2015; @Wright2017]
    - フォワードおよびディファード不透明を扱う、透明オブジェクトも同様に
    - 準ネイティブ解像度のレンダリングとテンポラル再投影をサポートする

# 関与媒質のオーサリング

我々はシーンにフォグを追加する方法を2つサポートする。

- アーティストはグローバルで境界のない[unbounded]フォグ、または、
- グレイスケールの3Dテクスチャ付きOBBで表現されるローカルな密度ボリュームを追加できる

# 関与媒質のオーサリング

- 単一散乱のアルベドと平均自由行程のパラメータ化[@Fong2017]
    - 単色の消散と透過

両方の場合で、我々は単一散乱のアルベドと平均自由行程のアーティストフレンドリーなボリューメトリックマテリアルのパラメータ化を開示する。これは後に散乱および消散係数に内部的に変換する。
パフォーマンス上の自由のため、我々は単色の平均自由行程のみをサポートする。これは、消散係数も単色であることを意味する。
結果として、ライトバウンスは散乱光に色を付ける[tint]ことが可能である一方、フォグの減衰は(カメラレイのような)直線路[straight paths]に沿って伝わる光の色に影響を一切与えないだろう。

# 関与媒質のオーサリング

- Cornette-Shanksの異方的位相関数

我々はグローバルな異方性パラメータを持つCornette-Shanksの異方的位相関数をサポートすることを選択した。Henyey-Greensteinと比べて、これは"真"のMie位相関数により良い一致をもたらす。

メモ: Cornette-Shanks anisotropic phase function [@Cornette1992; @Toublanc1996]

# 関与媒質のオーサリング

例えば、これはスポットライトが高い前方散乱性のフォグの中でどのように振る舞うかである(gifを参照)。
ローカルフォグでは、ボリューメトリックライティングは多くの入り組んだ信号がすぐさまアンダーサンプリングされる、すなわち、エイリアスとなるかなり低い変化量[rate]で計算されるので、我々は関与媒質を表現するのに3Dテクスチャを用いる。

# 関与媒質のオーサリング

どこもかしこもエイリアシング

これはシャドウマップ、ライトクッキー^[訳注:光源のマスク処理]、密度テクスチャを含む。幸いにも、ジオメトリのLODの扱いがより複雑になる一方で、テクスチャでは我々はMIPMAPを単に(注)使うことができる。

注: "未解決問題と今後の課題"を参照

# 深度分布

我々の実装は分布をスライスすることに関しては[when it comes to]かなり柔軟である。

我々はOuterra社のBrano Kemenの研究からスタートした。彼は自身のブログで対数深度分布を述べた[@Klemen2012]。
ある意味[in some sense]、彼の分布は最適である。しかし、異なる内容は異なる必要性を持つかもしれない、故に、我々は*一般化した*対数深度分布を制御する調整パラメータを開示する。これは線形と対数の間をなめらかに遷移する。

# 深度分布

このグラフの読み方(ttps://www.desmos.com/calculator/qrtatrlrba):

- X軸には、0から64までの、バッファの深度スライスがある。
- Y軸には、0.5から64メートルまでの、このスライスに対応する線形深度がある。
- 中間、32スライスの所に縦線が描いてある。

# 深度分布

赤には典型的な逆Z分布がある。これは予想通り酷く、0.5から1メートルの範囲をカバーする。
緑には標準の対数分布があり、最大5.6メートルの距離をカバーする。
青には0.5にセットした調整パラメータを持つ一般化した分布がある。これは最大10メートルの距離をカバーする。
調整パラメータの値に依存して、緑の対数分布から紫の線形分布まで範囲を広げることができる。

# 実装

- 3パス:
    - ボリュームのボクセル化
    - ボリューメトリックライティング
    - テンポラル再投影
- 計算されたライティングと不透明度はバイラテラルアップサンプリングされ、後続のメッシュレンダリングパス中に適用される

# ボリュームのボクセル化

初めに、密度ボリュームの保守的で中身の詰まったボクセル化[conservative solid voxelization]を行う。平たく言えば[to put it plainly]、我々は箱に重なるボクセルを決定する。

結果のバッファをアンチエイリアシングし、時間的な安定性を達成するためにパーシャルカバレッジを計算したい。これは保守的ラスタライザに対する良い応用のように思えるかもしれないが、我々の目標は非同期コンピュートを使うことである。
我々はまだこの問題への効果的な解法を述べる論文を見つけられていない。なので、自分で考え出した。
これは"A Topological Approach to Voxelization"[@Laine2013]というタイトルのSamuli Laineの論文で示される技術に触発されている。

同様にClusteredライティングでは、一連のボリュームを剪定し、局所化するためにclusteredプリパスから始める。
そして、ボクセルごとに、クラスタに重なる一連のボリュームを探し出し、それらをボクセル化する。

# ボリュームのボクセル化

パーシャルカバレッジを決定するため、箱の最も近い面を取り、その法線を計算する。
その後、この法線と最も並行なボクセルの対角線を取り、箱とこの対角線の重なりを計算する。
これはボクセルのパーシャルカバレッジの近似をもたらす。

# ボリュームのボクセル化

- リサンプリングを伴う大雑把な近似
- 正確な解法はコスト$O(線分の数 \times ライトの数) \times 積分コスト$を持つ
- 我々の解法はコスト$O(ボリュームの数) \times ボクセル化コスト + O(ライトの数) \times 積分コスト$を持つ
- 現在のGPUにより良くフィットする
    - 1つの巨大なuberシェーダの代わりに2つのより単純なシェーダ

ボクセル化を伴う解法はその性質上近似である。最も大きな問題は情報の不可逆的喪失を引き起こす。

理想的には、ライティングパス中、密度ボリュームに重なる個々のレイの線分上で積分して、完全にボクセル化を省略したい。

しかし、ボリューメトリックライティングはすでにかなり高価であるので、我々は入れ子となったループを含む巨大なuberシェーダより2つの単純なシェーダを伴う近似を持つ方が良い。

# ボリューメトリックライティングなぞ朝飯前よ！

- モンテカルロ推定器を評価する:

我々はモンテカルロ法を用いてボリュームレンダリング方程式を解く。これは"単なる"昔そのままの再帰的多次元積分である。

このスライドに10分費やす代わりに…

*注: ジョークスライドです。こいつの解読に時間を無駄にしないでね:-)*

# ボリューメトリックライティング

- 我々はモンテカルロ(MC)積分法を用いてボリューメトリックレンダリング方程式(VRE)を解く
    - このプレゼンテーションはすでに数学過多なので:-)
        - CGにおけるMC理論の入門には[@Dutre2006; @Veach1997]を参考に
        - VREの入門には[@Fong2017; @Novak2018]を参考に

数学をボクセルバッファライティングの我々のユースケースに適用する方法をカバーするのみとしたい。

どうしたら良いか分からない君は、参考文献を確かみてみろ

# ボリューメトリックライティング

$$
L_i = T(d) L_r(d) + \int_0^d T(t) L_s(t) dt
$$

$d$はレイに沿った最も近い不透明表面までの距離を示す。

この場合、入射する放射輝度の量$L_i$は透過率$T$で減衰する反射した放射輝度$L_r$とレイに沿って透過率$T$で連続的に減衰するin-scatterした放射輝度$L_s$の合計である。

# ボリューメトリックライティング

$$
L_i = T(d) L_r(d) + \int_0^x T(t) L_s(t) + \int_0^d T^x(t) L_s(t) dt
$$

レイに沿って2つのボクセルがある場合、積分を2つに分けるのは明らかである。

# ボリューメトリックライティング

$$
L_i = T(d) L_r(d) + \int_0^x T(t) L_s(t) + \int_0^d T^x(t) L_s(t) dt \\
\Downarrow \\
L_i = T(d) L_r(d) + \int_0^x T(t) L_s(t) + \int_0^{d-x} T(t) L_s(x+t) dt
$$

最後に、我々は独立したボクセル積分を得るために透過率のかけ算的な特性を活用することができる。

# ボリューメトリックライティング

1. モンテカルロを用いてボクセル積分を計算する
2. カメラレイに沿ってボクセル透過率をかけ算的に累積する
3. 透過率によって減衰するボクセル積分のprefix sumを計算する

$$
L_i = T(d) L_r(d) + \int_0^x T(t) L_s(t) + \int_0^d T^x(t) L_s(t) dt \\
\Downarrow \\
L_i = T(d) L_r(d) + \int_0^x T(t) L_s(t) + \int_0^{d-x} T(t) L_s(x+t) dt
$$

# ボクセル積分の計算

- ボクセル化の後、もうボリューム境界の情報を持てない
    - ボクセルの関与媒質を均質であると見なす
- MCを用いてボクセル積分を計算する
    - $I = \int_0^x T(t) L_s(t) dt \approx \frac{1}{n} \sum_{i=1}^n \frac{f(X_i)}{p(X_i)}$

密度ボリュームを事前にボクセル化するので、我々はボクセルの関与媒質を均質であると見なす。これは積分計算を相当に単純化する。

我々はこの仕事に対してモンテカルロのツールを使う。特に、分散低減[variance reduction]のために重点サンプリングを用いる。

# ボクセル積分の計算

- 重点サンプリングを介した分散低減
    - ディレクショナルライト→解析的自由行程サンプリング[@Novak2018]
    - パンクチュアルライト→等角[equiangular]サンプリング[@Kulla2011]
    - エリアライト→nullサンプリング\*

ディレクショナルおよびボックスプロジェクタライトでは、解析的な距離サンプリングを用いる。
パンクチュアルおよび粗ポットライトでは、等角サンプリングを用いる。これは逆二乗の減衰を扱うために設計される。
エリアライトでは、nullサンプリングを用いる。これは、エリアライトがまだサポートされていないので、0つのサンプルを取ることを意味する(ごめんね、Eric)。

# ボクセル積分の計算

ここにいくつかの等角サンプリングで実行中のプログラマアートがある。これはグローバルフォグで…

# ボクセル積分の計算

こっちが、、フォグに空間的に変化するテクスチャを与える密度ボリュームを使っている。

# 時間的積分

- MCを用いてボクセル積分を計算する
    - $I = \int_0^x T(t) L_s(t) dt \approx \frac{1}{n} \sum_{i=1}^n \frac{f(X_i)}{p(X_i)}$
- フレームあたりのボクセルあたりに十数サンプルを取る
- 前のフレーム全体で指数関数的な重み付き平均と組み合わせる[@Yang2009]

モンテカルロ積分は通常いくつかのサンプルを取ることを伴う。しかし毎フレームにボクセルあたり1より多いサンプルを取ることは一般に高価すぎる。特に現世代のコンソールハードウェアでは。

故に、我々は代わりにフレームあたりにボクセルあたり1つのサンプルを取り、前のフレームの上に指数関数的に重み付けされた平均と組み合わせる。

# 時間的積分

- 実践では:
    i. ボクセルあたりに放射輝度と透過率を計算する
    ii. 履歴バッファの内容と組み合わせる
    iii. フィードバックバッファに結果を書き込む
    iv. 履歴バッファとフィードバックバッファをスワップする

ワールド空間で再投影を行い、トライリニア補間して放射輝度と透過率を8つの最も近いボクセルから推定する。

# Will it Blend?

どうやって時間的ブレンディングを処理するか？

時間0でボクセルの放射輝度と透過率の推定値を計算し、時間1でこれらを再投影して時間1での推定値と組み合わせたいとしよう。
高速な前方へのカメラの動きがある場合、後ろにあるボクセルから前にあるボクセルへの再投影となるかもしれない。
すぐに分かることはこれらの大きさ[dimensions]が非常に異なるということである。故に放射輝度と透過率の推定値は似たようにはならないだろう。

例えば、あなたは高速なカメラ動作の結果としてシーン全体の明るさの変化を経験するかもしれない。

# Will it Blend?

さて、どうすればよいだろうか？

そのアイデアはブレンド可能な密度を得るためにボクセルの大きさに関してどうにかして放射輝度と透過率の両方の推定値を"正規化"することにある。
しかし、ボクセルの長さと透過率の上での放射輝度の積分は長さの線形関数ではない。
透過率は光学的な深度の指数関数である。これは長さの線形関数であり、簡単に用いることができる。

放射輝度の積分に関して言うと、話はより複雑になる。
グラフでは、長さの関数として、解析的な距離サンプリングの重みによって与えられる積分の推定値をプロットした。
実線[solid line]は推定値を表し、破線[dotted line]は長さに関するその一階微分を表す。

#

微分は定数でないので、この関数は線形ではない。しかし、変位が小さいので、線形近似はそれほど悪くない。

# Will it Blend?

我々は、ここでは緑で示されるように、長さで推定値を割ることでこれを少しだけ改良できる。これは線形にさらに近づいている。

もうひとつのアイデアはボクセルの実際の長さではなく単位間隔に沿って入射する放射輝度を積分し、それを再投影に対して使うことにある。
これら両方の手法はディレクショナルライトに対して比較的単純であるのに対して、等角サンプリングでパンクチュアルライトを正確に扱うことは未解決問題のままである。

また、私はよりエレガントで一般的な解法が存在すると考えている[suspect]。そのようなものがあれば、ぜひ教えてください！

"正規化された"放射輝度と透過率で再投影されたボクセルがあるとすると、我々は現在のボクセルの長さを用いて再スケールして戻し、現在のフレームからの推定値とブレンドすることができる。ボリュームブレンディングの正確なやり方はTom Duffによって彼の"Deel Compositiong Using Lie Algebras"[@Duff2017]という題の論文で与えられる。

# Will it Blend?

しかし、落とし穴[catch]がある: これのすべては位相関数が等方的であることを仮定することで機能する。一般的には、そうはならない。

強烈に前方散乱する位相関数があるとしよう。ディレクショナルライトを正視する[look straight at]場合、我々はボクセルに対して高い放射輝度の推定値を得る。

# Will it Blend?

カメラ、すなわち、ボクセルを90度回転する場合、媒質は前方散乱しており、我々はライトに対して直角[right angle]に面しているので、我々は非常に低い放射輝度の推定値を得る。

それ故に、前のフレームから再投影される高い値は現在の状況ではもはや正しくない[no longer valid]。

あなたは賢くしてみても良く、ライト方向を知っているので、我々は現在のフレームの向きに合わせて前のフレームの位相関数を再スケーリングできる、と言える…

# Will it Blend?

しかし、ボクセルを照らしているいくつかのライトを持ってしまえば、このアプローチでは"ゲームオーバー"である。

では、解法とは？
私はワークアラウンドを提案できるのみである。それに関わらず、これは実践で理に適って上手く機能する。
ボクセル積分を計算するとき、我々は2つの推定値を計算する。ひとつは位相関数をかけたもので、もうひとつはかけていないものである。我々は履歴バッファに異方的なバージョンを格納する。そして、これが我々が再投影するものである。
現在のフレーム中に位相関数の影響度を再構築するため、位相関数を伴う推定値を伴わないもので割り、再投影される放射輝度を再スケールするための比率を用いる。
これは時間的積分処理から異方性を除外する。これはもちろん悪手であるが、大量の目障りな[jarring]再投影アーティファクトを取り除きもする。

# Will it Blend?

時間的積分の最も明らかな結果のひとつはシャドウのエイリアシングおよびバンディングにおける低減である。ここで確認できるように。

*行ったり来たり*

# Will it Blend?

# サンプリングと再構築

高品質なサンプリングはモンテカルロアルゴリズムの素早い収束に必須である。
我々は高周波にそのエネルギーのほとんどを含めるためにサンプリングパターンのスペクトルも欲しい。これは人間の観測者には知覚されづらく、**そして**、再構築フィルタのローパス要素によって減衰するだろう[@Mitchell1991]。
これらは青色ノイズ、または、Poisson-hypersphereの特性である。

# サンプリングと再構築

加えて、Timothy Lottesが知覚的に気付いたこととして、サンプリングパターンの形状は良い再構築フィルタを近似するはずである。時間的積分の文脈で空間的に変化する重みを持つことは難しいが、我々は少なくともパターンのフットプリントが四角ではなく円であることを確かめることができる[@Smith1995]。

最後に、無作為なパターンを用いることはノイズによる構造化されたアーティファクトをマスクすることができる一方、結果の分布の品質を制御するのがさらに難しくなる。

# サンプリングと再構築

TODO
