---
bibliography: 'reading-list.bib'
numberSections: false
---
# memo

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

## 文献目録

### マイクロファセット理論

- [x] [@Walter2007] [[paper](https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf)]
    - GGXの原著。
- [x] [@Heitz2014] [[paper](http://jcgt.org/published/0003/02/03/paper.pdf)]
    - マイクロファセット理論におけるG項の考察。

### マイクロフレーク理論

- [x] [@Heitz2016b] [[paper](https://drive.google.com/open?id=0BzvWIdpUpRx_cFVlUkFhWXdleEU)]
    - Smithモデルを用いた複数回散乱を持つマイクロファセットBSDFのモデル化。
- [x] [@Heitz2015] [[paper](https://drive.google.com/file/d/0BzvWIdpUpRx_dXJIMk9rdEdrd00/view), [sup1](https://drive.google.com/file/d/0BzvWIdpUpRx_djVyMG9jMnltdTg/view), [sup2](https://www.cs.cornell.edu/projects/diffusion-sg10/anisotropic_web.pdf)]
    - GGXをマイクロフレーク理論に応用したSGGX分布の提案。

### 物理ベースレンダリング

- [x] [@Burley2012] [[slides](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_slides_v2.pdf), [notes](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_notes_v3.pdf)]
    - 測定データから見るマイクロファセット理論の分析と映像分野での採用事例。
- [x] [@Hoffman2015] [[slides](http://blog.selfshadow.com/publications/s2015-shading-course/hoffman/s2015_pbs_physics_math_slides.pdf), [video](https://www.youtube.com/watch?v=j-A0mwsJRmk)]
    - 2015年最新の物理ベースシェーディング手法のまとめ。
- [x] [@Hoffman2016] [[slides](http://blog.selfshadow.com/publications/s2016-shading-course/hoffman/s2016_pbs_recent_advances_v2.pdf), [video](https://www.youtube.com/watch?v=zs0oYjwjNEo)]
    - 2016年最新の物理ベースシェーディング手法のまとめ。

### シェーディング技術

- [x] [@Olsson2012] [[paper](http://efficientshading.com/wp-content/uploads/clustered_shading_preprint.pdf)]
    - Clastered Shadingの原著。
- [x] [@Burns2013] [[paper](http://jcgt.org/published/0002/02/04/)]
    - Visibility Bufferの原著。
- [x] [@Olsson2014] [[slides](http://efficientshading.com/wordpress/wp-content/uploads/sa_2014_intro.pdf)]
    - Forward ShadingからTiled Shadingまでのメジャーどころを総ざらい。
- [x] [@Oosten2015] [[blog](https://www.3dgep.com/forward-plus/)]
    - Forward Shading、Deferred Shading、Forward+それぞれに対する実装の解説とパフォーマンスの比較。
- [x] [@Drobot2017]
    - "Call of Duty: IW"で用いたライト割り当ての改善方法。
- [x] [@Garawany2016]
    - "Uncharted 4"で用いたDeferred LightingとSpecular Occlusion。
<!-- あとで読む -->

### ライティング技術

- [x] [@Heitz2016] [[paper](https://drive.google.com/open?id=0BzvWIdpUpRx_d09ndGVjNVJzZjA)]
    - 多角形の面光源をリアルタイムで扱う手法。
- [x] [@Lecocq2016]
    - @Arvo1995 のIrradiance Tensorsをもとに多角形の面光源をリアルタイムで扱う手法。
<!-- あとで読む -->
- [ ] [@Heitz2017] [[slides](http://blog.selfshadow.com/publications/s2017-shading-course/heitz/s2017_pbs_ltc_lines_disks.pdf)]
    - 線光源と円光源をリアルタイムで扱う手法。

### シャドウイング技術

- [x] [@Peters2015]
    - Moment Shadow Mapsの提案。
- [x] [@Lauritzen2010]
    - Sample Distribution Shadow Mapsの提案。
- [x] [@Donnelly2006]
    - Variance Shadow Mapsの提案。
- [ ] [@Yang2010]
    - Variance Soft Shadow Mapsの提案。
<!-- あとで読む -->
- [ ] [@Peters2016]
    - Moment Shadow Mapsの応用。
- [ ] [@Peters2017]
    - Moment Shadow Mapsの拡張。
- [ ] [@Jansen2010]
    - Fourier Opacity Mapsの提案。
- [ ] [@Fernando2005]
    - Percentage-Closer Soft Shadowsの提案。

### 光学エフェクト

- [x] [@Guertin2015]
    - ベジェ曲線を用いた非線形モーションブラー。
- [x] [@Gotanda2015]
    - 現実に起こるカメラに関する効果のまとめ。
- [x] [@Kakimoto2015]
    - 幾何光学の基礎のおさらい。
- [x] [@Kawase2015]
    - カメラレンズに関するエフェクト。
- [x] [@Gotanda2015b]
    - ボケの基本的な実装。
- [x] [@Kawase2015b]
    - ペンシルマップを使ったボケの実装詳細。
<!-- あとで読む -->
- [ ] [@Guertin2014]
    - モーションブラーフィルタの提案。
- [ ] [@Kakimoto2015b]
    - 精密なボケ評価のための波面トレーシング

### HDRレンダリング

- [x] [@Fry2017] [[slides](http://www.gdcvault.com/play/1024466/High-Dynamic-Range-Color-Grading)]
    - FrostbiteのHDRディスプレイを前提とした次世代のカラーグレーディングパイプライン。
<!-- あとで読む -->
- [ ] [@Lottes2016]
    - AMDの次世代カラーグレーディング。
- [ ] [@True2017]
    - NVIDIAの次世代カラーグレーディング。

### グローバル・イルミネーション

- [x] [@McLaren2015] [[slides](http://fumufumu.q-games.com/archives/TheTechnologyOfTomorrowsChildrenFinal.pdf), [video](http://www.gdcvault.com/play/1022428/The-Technology-of-The-Tomorrow)]
    - Q-Gamesが"Tomorrow Children"から得た、Voxel Cone Tracingの大規模な活用方法。
- [x] [@McGuire2017] [[paper](https://casual-effects.com/research/McGuire2017LightField/McGuire2017LightField.pdf), [others](https://casual-effects.com/research/McGuire2017LightField/index.html)]
    - light field probeを用いたリアルタイムGI。
- [x] [@Mara2016]
    - Deep G-Buffer。
- [x] [@Loos2010]
    - Volumetric Obscuranceの提案。
- [x] [@McGuire2012]
    - Scalable Ambient Obscuranceの提案。
<!-- あとで読む -->
- [ ] [@Neubelt2015] [[slides](http://blog.selfshadow.com/publications/s2015-shading-course/rad/s2015_pbs_rad_slides.pdf)]
    - Ready At Dawn Studiosが"The Order: 1886"から得た、焼き込み系GIと太陽光ライティングの知見。

### レイトレージング

- [x] [@Zsolnai-Feher2015] [[slides](https://www.cg.tuwien.ac.at/courses/Rendering/VU.SS2017.html), [video](https://www.youtube.com/watch?v=pjc1QAI6zS0)]
    - レイトレーシング技術の概要。
- [x] [@McGuire2014] [[paper](http://jcgt.org/published/0003/04/04/paper.pdf), [others](http://jcgt.org/published/0003/04/04/)]
    - スクリーンスペースで効率的にレイトレースするアルゴリズム。

### レンダリングエンジン

- [x] [@ODonnell2017] [[slides](http://www.gdcvault.com/play/1024612/FrameGraph-Extensible-Rendering-Architecture-in)]
    - EAの"Frostbite"が採用した、FrameGraph(依存関係を定義するしくみ)とTransientリソース(フレーム内でリソースを共有するしくみ)。
- [x] [@Wihlidal2016] [[slides](https://www.gdcvault.com/play/1023109/Optimizing-the-Graphics-Pipeline-With)]
    - "Frostbite"における、大規模なカリングパイプラインとGCNアーキテクチャへの最適化。
- [x] [@Rodrigues2017]
    - Ubisoftの"Anvil Next"をDirecX12に移行したときの知見。
<!--  あとで読む -->
- [ ] [@Lagarde2014] [[notes](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf)]
    - EAが"Frostbite"を物理ベースにするときに採用した方法とその知見。
- [ ] [@Sousa2016] [[slides](http://advances.realtimerendering.com/s2016/Siggraph2016_idTech6.pdf)]
    - idSoftwareの"idTech6"が採用した、Clusterd Shadingとライティングシステム。
- [ ] [@Carpentier2017]
    - Guerrilla Gamesの"Decima Engine"のライティングとアンチエイリアシング。
- [ ] [@Sousa2013]
    - "CryEngine 3"のアンチエイリアシング、カメラのポストプロセスなど。
- [ ] [@Drobot2017b]
    - Call of Duty: Infinite Warfareのレンダリングエンジン。
- [ ] [@Sousa2011]
    - "CryEngine 3"の全体。
- [ ] [@Haar2015]
    - GPUによるカリングを取り入れたレンダリングパイプライン。
- [ ] [@Tatarchuk2017]
    - グラフィクスエンジンから見るプログラミングモデルの進化。
- [ ] [@Vale2017]
    - 可視性。

### ジョブシステム

- [x] [@Reinalter2015] [[blog](https://blog.molecular-matters.com/2015/08/24/job-system-2-0-lock-free-work-stealing-part-1-basics/)]
    - Lock-FreeなWork-Stealing Queueを使ったジョブシステムの設計。
- [x] [@Gyrling2015] [[slides](http://www.gdcvault.com/play/1022187/Parallelizing-the-Naughty-Dog-Engine), [video](http://www.gdcvault.com/play/1022186/Parallelizing-the-Naughty-Dog-Engine)]
    - FiberベースのジョブシステムとFrame centricなメモリ管理。

### グラフィクスAPI

- [x] [@Olson2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/1-Vulkan_101.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [x] [@Worcester2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/2-Command_buffers_and_pipelines.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [x] [@Barker2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/5-Feeding-your-shaders.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [x] [@Garrard2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/6-Vulkan-subpasses.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [x] [@Hector2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-vulkan-devday-uk/7-Keeping-your-GPU-fed.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
- [x] [@Chajdas2016] [[slides](http://www.gdcvault.com/play/1022999/D3D12-Vulkan-Lessons)]
    - D3D12とVulkanの横断的なtips。バリアとキューについて。
- [x] [@Thomas2017] [[slides](http://32ipi028l5q82yhj72224m8j.wpengine.netdna-cdn.com/wp-content/uploads/2017/03/GDC2017-D3D12-And-Vulkan-Done-Right.pdf)]
    - D3D12とVulkanの横断的なtips。バリアとキューについて。
- [x] [@Chajdas2017] [[slides](https://www.gdcvault.com/play/1024634/D3D12-Vulkan-Lessons)]
    - D3D12とVulkanの2017年の最新動向。
- [x] [@Daniell2015] [[slides](http://on-demand.gputechconf.com/siggraph/2015/presentation/SIG1501-Piers-Daniell.pdf)]
    - VulkanのチュートリアルとNVIDIAでの動き。
- [x] [@Sellers2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-gdc/Khronos-Vulkan-Sessions-Part%20I_GDC_Mar16.pdf)]
    - Vulkanのメモリ管理についてのtips。
- [x] [@Sellers2016b] [[slides](http://32ipi028l5q82yhj72224m8j.wpengine.netdna-cdn.com/wp-content/uploads/2016/03/VulkanFastPaths.pdf)]
    - GCNアーキテクチャにおけるVulkanのデスクリプタ、レンダパス、バリアのtips。
- [x] [@Caloca2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - UnrealEngine4のVulkan対応で得られた知見。
- [x] [@Gneiting2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - idTech6のVulkan対応で得られた知見。
- [x] [@Hebert2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - メモリ管理のtips。
- [x] [@Hector2016b] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - レンダパスのtips。
- [x] [@Archard2016] [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-siggraph/3D-BOF-SIGGRAPH_Jul16.pdf)]
    - パイプラインキャッシュのtips。
- [x] [@Witczak2016] [[slides](http://32ipi028l5q82yhj72224m8j.wpengine.netdna-cdn.com/wp-content/uploads/2016/05/Most-common-mistakes-in-Vulkan-apps.pdf)]
    - Vulkanのプラクティス集。
- [x] [@Schott2016] [[slides](http://developer.download.nvidia.com/gameworks/events/GDC2016/mschott_lbishop_gl_vulkan.pdf)]
    - OpenGLとVulkanのtips。
- [x] [@Lorach2016]
    - AZDOなOpenGLとVulkanのtips。
- [x] [@Themaister2017]
    - フレームグラフベースレンダラの実装とそこからの知見。
<!-- あとで読む -->
- [ ] [@Hodes2016]
    - DirectX12のtips。
- [ ] [@Hodes2017]
    - 非同期コンピュートのAMDとNVIDIAのプラクティス。
- [ ] [@Thomas2016]
    - DirectX12のプラクティス。

### 最適化

- [x] [@Gyrling2017]
    - マルチコア環境におけるキャッシュシステムの動作から学ぶメモリバリアの概要。
<!-- あとで読む -->
- [ ] [@Drobot2014]
    - GCNアーキテクチャを対象とした低レベルな最適化。

### データ表現

<!-- あとで読む -->
- [ ] [@Cigolle2014]
    - GPUで単位ベクトルを表現する方法について調査。

### References
