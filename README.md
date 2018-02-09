---
bibliography: 'reading-list.bib'
numberSections: false
---
# memo

## いま読む

- [@Enderton2010]
    - OIT

## あとで読む

- [@Heitz2017]
    - 線光源と円光源をリアルタイムで扱う手法。
- [@Lottes2016]
    - AMDの次世代カラーグレーディング。
- [@True2017]
    - NVIDIAの次世代カラーグレーディング。
- [@Tatarchuk2017]
    - グラフィクスエンジンから見るプログラミングモデルの進化。
- [@Hodes2016]
    - DirectX12のtips。
- [@Hodes2017]
    - 非同期コンピュートのAMDとNVIDIAのプラクティス。
- [@Thomas2016]
    - DirectX12のプラクティス。
- [@Drobot2014]
    - GCNアーキテクチャを対象とした低レベルな最適化。
- [@Cigolle2014]
    - GPUで単位ベクトルを表現する方法について調査。
- [@Jimenez2016]
    - Filmic SMAA

### 保留

- [@Peters2016]
    - Moment Shadow Mapsの応用。
- [@Jansen2010]
    - Fourier Opacity Mapsの提案。
- [@Fernando2005]
    - Percentage-Closer Soft Shadowsの提案。


## 注目人物

- Bruce Walter
    - GGXの発案。[@Walter2007]
- Eric Heitz
    - シェーディング[@Heitz2014]やライティング[@Heitz2017]の研究。
    - 2017年現在はUnity所属。
- Natty Hoffman
- Sébastien Lagarde [[blog](https://seblagarde.wordpress.com/)]
    - Frostbiteでエンジン大改造をした。[@Lagarde2014]
    - 2017年現在はUnityのレンダリングリサーチ・ディレクター。
- Stephen Hill [[blog](http://blog.selfshadow.com/)]
    - SIGGRAPHのPBSコースの資料を毎年まとめているSelf Shadowブログの管理人。
    - 2017年現在はLucasfilm所属。
- Michal Drobot [[blog](https://michaldrobot.com/)]
    - 2017年現在はinfinity wardの主席レンダリングエンジニア。
- Tiago Sousa
    - id softwareの人。
    - CryENGINE3の開発にも参加している。
- Morgan McGuire [[blog](https://casual-effects.com/)]
    - Williams大学の准教授、NVIDIAの名誉研究員(?)[Distinguished Research Scientist]、他
    - AO、AA、OIT、GIなどの研究

## 文献目録

### アンチエイリアシング

- [@Yang2011]
    - DLAA
- [@Reshetov2011]
    - MLAA
- [@Jimenez2011]
    - Jimenez式MLAA＆SMAA
- [@McGuire2011]
    - SRAA
- [@Lottes2011]
    - FXAA
- [@Malan2011]
    - DEAA
- [@Persson2011]
    - GBAA
- [@Sousa2011b]
    - CryENGINE 3 のAA
- [@Andreev2011]
    - DLAA
- [@Jimenez2017]
    - ポストプロセスAAの変遷
- [@Jimenez2012]
    - SMAA

### グローバル・イルミネーション

- [@McLaren2015] [[slides](http://fumufumu.q-games.com/archives/TheTechnologyOfTomorrowsChildrenFinal.pdf), [video](http://www.gdcvault.com/play/1022428/The-Technology-of-The-Tomorrow)]
    - Q-Gamesが"Tomorrow Children"から得た、Voxel Cone Tracingの大規模な活用方法。
- [@McGuire2017] [[paper](https://casual-effects.com/research/McGuire2017LightField/McGuire2017LightField.pdf), [others](https://casual-effects.com/research/McGuire2017LightField/index.html)]
    - light field probeを用いたリアルタイムGI。
- [@Mara2016]
    - Deep G-Buffer。
- [@Loos2010]
    - Volumetric Obscuranceの提案。
- [@McGuire2012]
    - Scalable Ambient Obscuranceの提案。
- [@Neubelt2015] [[slides](http://blog.selfshadow.com/publications/s2015-shading-course/rad/s2015_pbs_rad_slides.pdf)]
    - Ready At Dawn Studiosが"The Order: 1886"から得た、焼き込み系GIと太陽光ライティングの知見。


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

- [@Olson2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/1-Vulkan_101.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [@Worcester2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/2-Command_buffers_and_pipelines.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [@Barker2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/5-Feeding-your-shaders.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [@Garrard2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/6-Vulkan-subpasses.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [@Hector2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/7-Keeping-your-GPU-fed.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [@Chajdas2016] [[slides](http://www.gdcvault.com/play/1022999/D3D12-Vulkan-Lessons)]
    - D3D12とVulkanの横断的なtips。バリアとキューについて。
- [@Thomas2017] [[slides](http://32ipi028l5q82yhj72224m8j.wpengine.netdna-cdn.com/wp-content/uploads/2017/03/GDC2017-D3D12-And-Vulkan-Done-Right.pdf)]
    - D3D12とVulkanの横断的なtips。バリアとキューについて。
- [@Chajdas2017] [[slides](https://www.gdcvault.com/play/1024634/D3D12-Vulkan-Lessons)]
    - D3D12とVulkanの2017年の最新動向。
- [@Daniell2015] [[slides](http://on-demand.gputechconf.com/siggraph/2015/presentation/SIG1501-Piers-Daniell.pdf)]
    - VulkanのチュートリアルとNVIDIAでの動き。
- [@Sellers2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-gdc/Khronos-Vulkan-Sessions-Part%20I_GDC_Mar16.pdf)]
    - Vulkanのメモリ管理についてのtips。
- [@Sellers2016b] [[slides](http://32ipi028l5q82yhj72224m8j.wpengine.netdna-cdn.com/wp-content/uploads/2016/03/VulkanFastPaths.pdf)]
    - GCNアーキテクチャにおけるVulkanのデスクリプタ、レンダパス、バリアのtips。
- [@Caloca2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - UnrealEngine4のVulkan対応で得られた知見。
- [@Gneiting2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - idTech6のVulkan対応で得られた知見。
- [@Hebert2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - メモリ管理のtips。
- [@Hector2016b] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - レンダパスのtips。
- [@Archard2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - パイプラインキャッシュのtips。
- [@Witczak2016] [[slides](http://32ipi028l5q82yhj72224m8j.wpengine.netdna-cdn.com/wp-content/uploads/2016/05/Most-common-mistakes-in-Vulkan-apps.pdf)]
    - Vulkanのプラクティス集。
- [@Schott2016] [[slides](http://developer.download.nvidia.com/gameworks/events/GDC2016/mschott_lbishop_gl_vulkan.pdf)]
    - OpenGLとVulkanのtips。
- [@Lorach2016]
    - AZDOなOpenGLとVulkanのtips。
- [@Themaister2017]
    - フレームグラフベースレンダラの実装とそこからの知見。

### HDRレンダリング

- [@Fry2017] [[slides](http://www.gdcvault.com/play/1024466/High-Dynamic-Range-Color-Grading)]
    - FrostbiteのHDRディスプレイを前提とした次世代のカラーグレーディングパイプライン。

### ジョブシステム

- [@Reinalter2015] [[blog](https://blog.molecular-matters.com/2015/08/24/job-system-2-0-lock-free-work-stealing-part-1-basics/)]
    - Lock-FreeなWork-Stealing Queueを使ったジョブシステムの設計。
- [@Gyrling2015] [[slides](http://www.gdcvault.com/play/1022187/Parallelizing-the-Naughty-Dog-Engine), [video](http://www.gdcvault.com/play/1022186/Parallelizing-the-Naughty-Dog-Engine)]
    - FiberベースのジョブシステムとFrame centricなメモリ管理。

### ライティング技術

- [@Heitz2016] [[paper](https://drive.google.com/open?id=0BzvWIdpUpRx_d09ndGVjNVJzZjA)]
    - 多角形の面光源をリアルタイムで扱う手法。
- [@Lecocq2016]
    - @Arvo1995 のIrradiance Tensorsをもとに多角形の面光源をリアルタイムで扱う手法。

### メモリ割り当て

- [@Schneider2006]
    - Streamflow

### マイクロファセット理論

- [@Walter2007] [[paper](https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf)]
    - GGXの原著。
- [@Heitz2014] [[paper](http://jcgt.org/published/0003/02/03/paper.pdf)]
    - マイクロファセット理論におけるG項の考察。

### マイクロフレーク理論

- [@Heitz2016b] [[paper](https://drive.google.com/open?id=0BzvWIdpUpRx_cFVlUkFhWXdleEU)]
    - Smithモデルを用いた複数回散乱を持つマイクロファセットBSDFのモデル化。
- [@Heitz2015] [[paper](https://drive.google.com/file/d/0BzvWIdpUpRx_dXJIMk9rdEdrd00/view), [sup1](https://drive.google.com/file/d/0BzvWIdpUpRx_djVyMG9jMnltdTg/view), [sup2](https://www.cs.cornell.edu/projects/diffusion-sg10/anisotropic_web.pdf)]
    - GGXをマイクロフレーク理論に応用したSGGX分布の提案。

### 光学エフェクト

- [@Guertin2015]
    - ベジェ曲線を用いた非線形モーションブラー。
- [@Gotanda2015]
    - 現実に起こるカメラに関する効果のまとめ。
- [@Kakimoto2015]
    - 幾何光学の基礎のおさらい。
- [@Kawase2015]
    - カメラレンズに関するエフェクト。
- [@Gotanda2015b]
    - ボケの基本的な実装。
- [@Kawase2015b]
    - ペンシルマップを使ったボケの実装詳細。
- [@Guertin2014]
    - モーションブラーフィルタの提案。
- [@Kakimoto2015b]
    - 精密なボケ評価のための波面トレーシング
- [@Kakimoto2015c]
    - 波動光学ベースのグレア生成テクニック
- [@Kawase2015c]
    - 光学エフェクトシステムの設計
- [@Gotanda2015c]
    - 光学エフェクトシステムの設計
- [@McGuire2012b]
    - モーションブラーフィルタの提案。

### 最適化

- [@Gyrling2017]
    - マルチコア環境におけるキャッシュシステムの動作から学ぶメモリバリアの概要。

### OIT

- [@McGuire2017b]
    - 統合的なOITフレームワーク

### 物理ベースレンダリング

- [@Burley2012] [[slides](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_slides_v2.pdf), [notes](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_notes_v3.pdf)]
    - 測定データから見るマイクロファセット理論の分析と映像分野での採用事例。
- [@Hoffman2015] [[slides](http://blog.selfshadow.com/publications/s2015-shading-course/hoffman/s2015_pbs_physics_math_slides.pdf), [video](https://www.youtube.com/watch?v=j-A0mwsJRmk)]
    - 2015年最新の物理ベースシェーディング手法のまとめ。
- [@Hoffman2016] [[slides](http://blog.selfshadow.com/publications/s2016-shading-course/hoffman/s2016_pbs_recent_advances_v2.pdf), [video](https://www.youtube.com/watch?v=zs0oYjwjNEo)]
    - 2016年最新の物理ベースシェーディング手法のまとめ。

### レイトレージング

- [@Zsolnai-Feher2015] [[slides](https://www.cg.tuwien.ac.at/courses/Rendering/VU.SS2017.html), [video](https://www.youtube.com/watch?v=pjc1QAI6zS0)]
    - レイトレーシング技術の概要。
- [@McGuire2014] [[paper](http://jcgt.org/published/0003/04/04/paper.pdf), [others](http://jcgt.org/published/0003/04/04/)]
    - スクリーンスペースで効率的にレイトレースするアルゴリズム。

### レンダリングエンジン

- [@ODonnell2017] [[slides](http://www.gdcvault.com/play/1024612/FrameGraph-Extensible-Rendering-Architecture-in)]
    - EAの"Frostbite"が採用した、FrameGraph(依存関係を定義するしくみ)とTransientリソース(フレーム内でリソースを共有するしくみ)。
- [@Wihlidal2016] [[slides](https://www.gdcvault.com/play/1023109/Optimizing-the-Graphics-Pipeline-With)]
    - "Frostbite"における、大規模なカリングパイプラインとGCNアーキテクチャへの最適化。
- [@Drobot2017b]
    - Call of Duty: Infinite Warfareのレンダリングエンジン。
- [@Rodrigues2017]
    - Ubisoftの"Anvil Next"をDirecX12に移行したときの知見。
- [@Lagarde2014] [[notes](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf)]
    - EAが"Frostbite"を物理ベースにするときに採用した方法とその知見。
- [@Vale2017]
    - Decima Engineの可視性。
- [@Sousa2013]
    - "CryEngine 3"のアンチエイリアシング、カメラのポストプロセスなど。
- [@Haar2015]
    - GPU駆動カリングパイプラインと仮想テクスチャ。
- [@Carpentier2017]
    - Guerrilla Gamesの"Decima Engine"のライティングとアンチエイリアシング。
- [@Sousa2011]
    - "CryEngine 3"の全体。
- [@Sousa2016] [[slides](http://advances.realtimerendering.com/s2016/Siggraph2016_idTech6.pdf)]
    - idSoftwareの"idTech6"が採用した、Clusterd Shadingとライティングシステム。

### シェーディング技術

- [@Olsson2012] [[paper](http://efficientshading.com/wp-content/uploads/clustered_shading_preprint.pdf)]
    - Clastered Shadingの原著。
- [@Burns2013] [[paper](http://jcgt.org/published/0002/02/04/)]
    - Visibility Bufferの原著。
- [@Olsson2014] [[slides](http://efficientshading.com/wordpress/wp-content/uploads/sa_2014_intro.pdf)]
    - Forward ShadingからTiled Shadingまでのメジャーどころを総ざらい。
- [@Oosten2015] [[blog](https://www.3dgep.com/forward-plus/)]
    - Forward Shading、Deferred Shading、Forward+それぞれに対する実装の解説とパフォーマンスの比較。
- [@Drobot2017]
    - "Call of Duty: IW"で用いたライト割り当ての改善方法。
- [@Garawany2016]
    - "Uncharted 4"で用いたDeferred LightingとSpecular Occlusion。

### シャドウイング技術

- [@Peters2015]
    - Moment Shadow Mapsの提案。
- [@Lauritzen2010]
    - Sample Distribution Shadow Mapsの提案。
- [@Donnelly2006]
    - Variance Shadow Mapsの提案。
- [@Yang2010]
    - Variance Soft Shadow Mapsの提案。
- [@Peters2017]
    - Moment Shadow Mapsの拡張。

# References
