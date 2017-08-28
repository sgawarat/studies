---
bibliography: 'reading-list.bib'
link-citations: true
---
# memo

## 注目人物

- Bruce Walter

## 文献目録

### 物理ベースシェーディング

- [x] [@Walter2007] [[paper](https://www.cs.cornell.edu/~srm/publications/EGSR07-btdf.pdf)]
    - GGXの原著。

- [x] [@Burley2012] [[slides](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_slides_v2.pdf), [notes](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_notes_v3.pdf)]
    - 測定データから見るマイクロファセット理論の分析と映像分野での採用事例。

- [x] [@Heitz2014]
 [[paper](http://jcgt.org/published/0003/02/03/paper.pdf)]
    - マイクロファセット理論におけるG項の考察。

- [x] [@Hoffman2015] [[slides](http://blog.selfshadow.com/publications/s2015-shading-course/hoffman/s2015_pbs_physics_math_slides.pdf), [video](https://www.youtube.com/watch?v=j-A0mwsJRmk)]
    - 2015年最新の物理ベースシェーディング手法のまとめ。

- [x] [@Hoffman2016] [[slides](http://blog.selfshadow.com/publications/s2016-shading-course/hoffman/s2016_pbs_recent_advances_v2.pdf), [video](https://www.youtube.com/watch?v=zs0oYjwjNEo)]
    - 2016年最新の物理ベースシェーディング手法のまとめ。

- [ ] [@Heitz2016a] [[paper](https://drive.google.com/open?id=0BzvWIdpUpRx_cFVlUkFhWXdleEU)]
    - Smithモデルを用いた複数回の散乱のマイクロファセットBSDFをモデル化。

### シェーディング技術

- [x] [@Olsson2012] [[paper](http://efficientshading.com/wp-content/uploads/clustered_shading_preprint.pdf)]
    - Clastered Shadingの原著。

- [x] [@Burns2013] [[paper](http://jcgt.org/published/0002/02/04/)]
    - Visibility Bufferの原著。

- [x] [@Olsson2014] [[slides](http://efficientshading.com/wordpress/wp-content/uploads/sa_2014_intro.pdf)]
    - Forward ShadingからTiled Shadingまでのメジャーどころを総ざらい。

- [x] [@Oosten2015] [[blog](https://www.3dgep.com/forward-plus/)]
    - Forward Shading、Deferred Shading、Forward+それぞれに対する実装の解説とパフォーマンスの比較。

### ライティング技術

- [x] [@Heitz2016] [[paper](https://drive.google.com/open?id=0BzvWIdpUpRx_d09ndGVjNVJzZjA)]
    - 多角形の面光源をリアルタイムで扱う手法。

- [ ] [@Heitz2017] [[slides](http://blog.selfshadow.com/publications/s2017-shading-course/heitz/s2017_pbs_ltc_lines_disks.pdf)]
    - 線光源と円光源をリアルタイムで扱う手法。

### HDRレンダリング

- [ ] [@Fry2017] [[slides](http://www.gdcvault.com/play/1024466/High-Dynamic-Range-Color-Grading)]
    - HDRディスプレイを前提とした次世代のカラーグレーディングパイプライン。

### グローバル・イルミネーション

- [ ] [@Heubelt2015] [[slides](http://blog.selfshadow.com/publications/s2015-shading-course/rad/s2015_pbs_rad_slides.pdf)]
    - Ready At Dawn Studiosが"The Order: 1886"から得た、焼き込み系GIと太陽光ライティングの知見。

- [ ] [@McLaren2015] [[slides](http://fumufumu.q-games.com/archives/TheTechnologyOfTomorrowsChildrenFinal.pdf), [video](http://www.gdcvault.com/play/1022428/The-Technology-of-The-Tomorrow)]
    - Q-Gamesが"Tomorrow Children"から得た、Voxel Cone Tracingの大規模な活用方法。

### レイトレージング

- [x] [@Zsolnai2015] [[slides](https://www.cg.tuwien.ac.at/courses/Rendering/VU.SS2017.html), [video](https://www.youtube.com/watch?v=pjc1QAI6zS0)]
    - レイトレーシング技術の概要。

### レンダリングエンジン

- [ ] [@Lagarde2014] [[notes](https://seblagarde.files.wordpress.com/2015/07/course_notes_moving_frostbite_to_pbr_v32.pdf)]
    - EAの"Frostbite"が採用した、物理ベースのレンダリングシステム。

- [ ] [@Sousa2016] [[slides](http://advances.realtimerendering.com/s2016/Siggraph2016_idTech6.pdf)]
    - idSoftwareの"idTech6"が採用した、Clusterd Shadingとライティングシステム。

- [ ] [@ODonnell2017] [[slides](http://www.gdcvault.com/play/1024612/FrameGraph-Extensible-Rendering-Architecture-in)]
    - EAの"Frostbite"が採用した、FrameGraph(依存関係を定義するしくみ)とTransientリソース(フレーム内でリソースを共有するしくみ)。

#### ジョブシステム

- [x] [@Reinalter2015] [[blog](https://blog.molecular-matters.com/2015/08/24/job-system-2-0-lock-free-work-stealing-part-1-basics/)]
    - Lock-FreeなWork-Stealing Queueを使ったジョブシステムの設計。

- [x] [@Gyrling2015] [[slides](http://www.gdcvault.com/play/1022187/Parallelizing-the-Naughty-Dog-Engine), [video](http://www.gdcvault.com/play/1022186/Parallelizing-the-Naughty-Dog-Engine)]
    - FiberベースのジョブシステムとFrame centricなメモリ管理。

#### Vulkan API

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

- [ ] [@Chajdas2016] [[slides](http://www.gdcvault.com/play/1022999/D3D12-Vulkan-Lessons)]
    - D3D12とVulkanの横断的なtips。バリアとキューについて。

- [x] [@Thomas2017] [[slides](http://32ipi028l5q82yhj72224m8j.wpengine.netdna-cdn.com/wp-content/uploads/2017/03/GDC2017-D3D12-And-Vulkan-Done-Right.pdf)]
    - D3D12とVulkanの横断的なtips。バリアとキューについて。

### References
