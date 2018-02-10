---
bibliography: 'reading-list.bib'
numberSections: false
---
# memo

## いま読む

- [@McGuire2011b]
    - 色付き確率的シャドウマップ
- [@Tatarchuk2017]
    - グラフィクスエンジンから見るプログラミングモデルの進化

## あとで読む

- [@Heitz2017]
    - 線光源と円光源をリアルタイムで扱う手法
- [@Lottes2016]
    - AMDの次世代カラーグレーディング
- [@True2017]
    - NVIDIAの次世代カラーグレーディング
- [@Hodes2016]
    - DirectX12のtips
- [@Hodes2017]
    - 非同期コンピュートのAMDとNVIDIAのプラクティス
- [@Thomas2016]
    - DirectX12のプラクティス
- [@Drobot2014]
    - GCNアーキテクチャを対象とした低レベルな最適化
- [@Cigolle2014]
    - GPUで単位ベクトルを表現する方法について調査
- [@Jimenez2016]
    - Filmic SMAA
- [@Jakob2010]
    - マイクロフレーク理論
- [@McGuire2013]
    - OIT
- [@Sumaili2017]
    - "Horizon Zero Dawn"のツールパイプラインの作成
- [@Rouwe2014]
    - エンティティ更新のスレッド化
- [@Hammon2017]
    - 物理ベースのディフューズライティング

## 注目人物

- Michal Drobot [[blog](https://michaldrobot.com/)]
    - 2017年現在は、Infinity Wardの主席レンダリングエンジニア
- Eric Heitz
    - シェーディング[@Heitz2014]やライティング[@Heitz2017]の研究
    - 2017年現在は、Unity所属
- Stephen Hill [[blog](http://blog.selfshadow.com/)]
    - SIGGRAPHのPBSコースの資料を毎年まとめている、Self Shadowブログの管理人
    - 2017年現在は、Lucasfilm所属
- Natty Hoffman
- Sébastien Lagarde [[blog](https://seblagarde.wordpress.com/)]
    - Frostbiteのエンジン大改造をした人[@Lagarde2014]
    - 2017年現在は、Unityのレンダリングリサーチ・ディレクター
- Morgan McGuire [[blog](https://casual-effects.com/)]
    - AA[@McGuire2011]、AO[@McGuire2012]、OIT[@McGuire2013]、GI[@McGuire2017]など、リアルタイム系の研究が主
    - 2018年現在、Williams大学の准教授、NVIDIAのDistinguished Research Scientist、など
- Tiago Sousa
    - id Softwareの人[@Sousa2016]
    - CryENGINE3の開発に参加している[@Sousa2013]
- Bruce Walter
    - GGX[@Walter2007]

## 文献目録

### アンチエイリアシング

- [@Andreev2011]
    - DLAA
- [@Jimenez2011]
    - GPUベースのMLAAとSMAA
- [@Jimenez2012]
    - SMAA
- [@Jimenez2017]
    - ポストプロセスAAの変遷
- [@Lottes2011]
    - FXAA
- [@Malan2011]
    - DEAA
- [@McGuire2011]
    - SRAA
- [@Persson2011]
    - GBAA
- [@Reshetov2011]
    - MLAA
- [@Sousa2011b]
    - CryENGINE 3のAA
- [@Yang2011]
    - DLAA

### グローバル・イルミネーション

- [@Loos2010]
    - Volumetric Obscurance
- [@Mara2016]
    - Deep G-Buffer
- [@McGuire2012]
    - Scalable Ambient Obscurance
- [@McGuire2017]
    - light field probeを用いたリアルタイムGI
- [@McLaren2015]
    - Q-Gamesが"Tomorrow Children"から得た、Voxel Cone Tracingの大規模な活用方法
- [@Neubelt2015]
    - Ready At Dawn Studiosが"The Order: 1886"から得た、焼き込み系GIと太陽光ライティングの知見


### GPUアーキテクチャ

- [@Cozzi2017]
    - GPUアーキテクチャの概要
- [@Cozzi2017b]
    - 並列アルゴリズムの基礎
- [@Cozzi2017c]
    - 並列アルゴリズムの応用
- [@Cozzi2017d]
    - CUDAのパフォーマンス
- [@Cozzi2017e]
    - CUDAのアトミック
- [@Cozzi2017f]
    - グラフィクスパイプライン
- [@Cozzi2017g]
    - Deferred Shading
- [@Cozzi2017h]
    - Forward+とClustered Shading

### グラフィクスAPI

- [@Archard2016]
    - Vulkanのパイプラインキャッシュのtips
- [@Barker2016]
    - モバイル系メーカーによるVulkan APIの概要
- [@Caloca2016]
    - UnrealEngine4のVulkan対応で得られた知見
- [@Chajdas2016]
    - バリアとキューについてのD3D12とVulkanの横断的なtips
- [@Chajdas2017]
    - D3D12とVulkanの2017年の最新動向
- [@Daniell2015]
    - VulkanのチュートリアルとNVIDIAでの動き
- [@Garrard2016]
    - モバイル系メーカーによるVulkan APIの概要
- [@Gneiting2016]
    - idTech6のVulkan対応で得られた知見
- [@Hebert2016]
    - Vulkanのメモリ管理のtips
- [@Hector2016]
    - モバイル系メーカーによるVulkan APIの概要
- [@Hector2016b]
    - Vulkanのレンダパスのtips
- [@Lorach2016]
    - AZDOなOpenGLとVulkanのtips
- [@Olson2016]
    - モバイル系メーカーによるVulkan APIの概要
- [@Themaister2017]
    - Vulkanベースのフレームグラフの実装とそこからの知見
- [@Thomas2017]
    - バリアとキューについてのD3D12とVulkanの横断的なtips
- [@Schott2016]
    - OpenGLとVulkanのtips
- [@Sellers2016]
    - Vulkanのメモリ管理についてのtips
- [@Sellers2016b]
    - GCNアーキテクチャにおけるVulkanのデスクリプタ、レンダパス、バリアのtips
- [@Witczak2016]
    - Vulkanのプラクティス集
- [@Worcester2016]
    - モバイル系メーカーによるVulkan APIの概要

### HDRレンダリング

- [@Fry2017]
    - FrostbiteのHDRディスプレイを前提とした次世代のカラーグレーディングパイプライン

### ジョブシステム

- [@Gyrling2015]
    - FiberベースのジョブシステムとFrame centricなメモリ管理
- [@Reinalter2015]
    - Lock-FreeなWork-Stealing Queueを使ったジョブシステムの設計

### ライティング技術

- [@Heitz2016]
    - 多角形の面光源をリアルタイムで扱う手法
- [@Lecocq2016]
    - @Arvo1995 のIrradiance Tensorsをもとに多角形の面光源をリアルタイムで扱う手法

### メモリ割り当て

- [@Schneider2006]
    - Streamflow: マルチスレッド対応のメモリアロケータ

### マイクロファセット理論

- [@Heitz2014]
    - マイクロファセット理論におけるG項の考察
- [@Walter2007]
    - GGX

### マイクロフレーク理論

- [@Heitz2015]
    - GGXをマイクロフレーク理論に応用した、SGGX分布の提案
- [@Heitz2016b]
    - Smithモデルを用いた複数散乱を持つマイクロファセットBSDFのモデル化

### 光学エフェクト

- [@Gotanda2015]
    - 現実に起こるカメラに関する効果のまとめ
- [@Gotanda2015b]
    - ボケの基本的な実装
- [@Gotanda2015c]
    - 光学エフェクトシステムの設計
- [@Guertin2014]
    - モーションブラーフィルタの提案
- [@Guertin2015]
    - ベジェ曲線を用いた非線形モーションブラー
- [@Kakimoto2015]
    - 幾何光学の基礎のおさらい
- [@Kakimoto2015b]
    - 精密なボケ評価のための波面トレーシング
- [@Kakimoto2015c]
    - 波動光学ベースのグレア生成テクニック
- [@Kawase2015]
    - カメラレンズに関するエフェクト
- [@Kawase2015b]
    - ペンシルマップを使ったボケの実装詳細
- [@Kawase2015c]
    - 光学エフェクトシステムの設計
- [@McGuire2012b]
    - モーションブラーフィルタの提案

### 最適化

- [@Gyrling2017]
    - マルチコア環境におけるキャッシュシステムの動作から学ぶメモリバリアの概要

### OIT

- [@Enderton2010]
    - 深度バッファを活用した確率的OIT
- [@McGuire2017b]
    - 統合的なOITフレームワーク

### 物理ベースレンダリング

- [@Burley2012]
    - 測定データから見るマイクロファセット理論の分析と映像分野での採用事例
- [@Hoffman2015]
    - 2015年最新の物理ベースシェーディング手法のまとめ
- [@Hoffman2016]
    - 2016年最新の物理ベースシェーディング手法のまとめ

### レイトレージング

- [@Zsolnai-Feher2015]
    - レイトレーシング技術の概要
- [@McGuire2014]
    - スクリーンスペースで効率的にレイトレースするアルゴリズム

### レンダリングエンジン

- [@Carpentier2017]
    - Guerrilla Gamesの"Decima Engine"のライティングとアンチエイリアシング
- [@Drobot2017b]
    - Call of Duty: Infinite Warfareのレンダリングエンジン
- [@Haar2015]
    - GPU駆動カリングパイプラインとバーチャルテクスチャ
- [@Lagarde2014]
    - EAが"Frostbite"を物理ベースにするときに採用した方法とその知見
- [@ODonnell2017]
    - EAの"Frostbite"が採用した、依存関係を定義するしくみ(FrameGraph)とフレーム内でリソースを共有するしくみ(Transient Resource)
- [@Rodrigues2017]
    - Ubisoftの"Anvil Next"をDirecX12に移行したときの知見
- [@Sousa2011]
    - "CryEngine 3"の全体
- [@Sousa2013]
    - "CryEngine 3"のアンチエイリアシング、カメラのポストプロセスなど
- [@Sousa2016]
    - idSoftwareの"idTech6"が採用した、Clusterd Shadingとライティングシステム
- [@Wihlidal2016]
    - "Frostbite"における、大規模なカリングパイプラインとGCNアーキテクチャへの最適化
- [@Vale2017]
    - Decima Engineの可視性

### シェーディング技術

- [@Burns2013]
    - Visibility Buffer
- [@Drobot2017]
    - "Call of Duty: IW"で用いたライト割り当ての改善方法
- [@Garawany2016]
    - "Uncharted 4"で用いたDeferred LightingとSpecular Occlusion
- [@Olsson2012]
    - Clustered Shading
- [@Olsson2014]
    - Forward ShadingからTiled Shadingまでのメジャーどころを総ざらい
- [@Oosten2015]
    - Forward Shading、Deferred Shading、Forward+それぞれに対する実装の解説とパフォーマンスの比較

### シャドウイング技術

- [@Donnelly2006]
    - Variance Shadow Maps
- [@Lauritzen2010]
    - Sample Distribution Shadow Maps
- [@Peters2015]
    - Moment Shadow Maps
- [@Peters2017]
    - Moment Shadow Mapsの応用([@Peters2016]の拡張版)
- [@Yang2010]
    - Variance Soft Shadow Maps

## 保留文献

- [@Fernando2005]
    - Percentage-Closer Soft Shadows
    - 多くのソフトシャドウ手法が再掲しているので
- [@Jansen2010]
    - Fourier Opacity Maps
    - あまり主流ではなさそうなので
- [@Peters2016]
    - Moment Shadow Mapsの応用
    - 大体の内容が[@Peters2017]と重なっているので

# References
