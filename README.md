# memo

## 参考資料

<!-- 書式: 著者1, 著者2, 著者3. タイトル. 発表媒体名(Youtubeやブログでは省略). [paper, slides, notes, video] -->

### 読むべきリスト

#### シェーディング

- Bruce Walter, Stephen R. Marschner, Hongsong Li, Kenneth E. Torrance. "Microfacet Models for Refraction through Rough Surfaces". *EGSR 2007*. [[paper](http://fumufumu.q-games.com/archives/TheTechnologyOfTomorrowsChildrenFinal.pdf)]
    - GGXの原著。

#### シェーディング技法

- Wolfgang Engel. "The filtered and culled Visibility Buffer". *GDCE 2016*. [[slides](http://www.conffx.com/Visibility_Buffer_GDCE.pdf)]
    - Visibility Bufferの実践編。

#### ライティング

- Stephen Hill, Eric Heitz. "Real-Time Area Lighting: a Journey from Research to Production". *SIGGRAPH 2016*. [[slides](http://advances.realtimerendering.com/s2016/s2016_ltc_rnd.pdf), [notes](http://advances.realtimerendering.com/s2016/s2016_ltc_fresnel.pdf)]
    - リアルタイムでエリアライトを扱う手法の実践編。

- Eric Heitz, Stephen Hill. "Real-Time Line- and Disk-Light Shading with Linearly Transformed Cosines". *SIGGRAPH 2017*. [[slides](http://blog.selfshadow.com/publications/s2017-shading-course/heitz/s2017_pbs_ltc_lines_disks.pdf)]
    - リアルタイムで線光源と円光源を扱う手法。

#### HDRレンダリング

- Alex Fry. "High Dynamic Range color grading and display in Frostbite". *GDC 2017*. [[slides](http://www.gdcvault.com/play/1024466/High-Dynamic-Range-Color-Grading)]

#### レンダリングエンジン

- James McLaren. "The Technology of The Tomorrow Children". *GDC 2015*. [[slides](http://fumufumu.q-games.com/archives/TheTechnologyOfTomorrowsChildrenFinal.pdf), [video](http://www.gdcvault.com/play/1022428/The-Technology-of-The-Tomorrow)]
    - Q-GamesのTomorrow Children。

- Tiago Sousa, Jean Geffroy. "The devil is in the details: idTech 666". *SIGGRAPH 2016*. [[slides](http://advances.realtimerendering.com/s2016/Siggraph2016_idTech6.pdf)]
    - idSoftwareのidTech6。

- Yuriy O'Donnell. "FrameGraph: Extensible Rendering Architecture in Frostbite". *GDC 2017*. [[slides](http://www.gdcvault.com/play/1024612/FrameGraph-Extensible-Rendering-Architecture-in)]
    - EAのFrostbiteの新旧アーキテクチャ比較。

### 文献目録

#### まとめ

- Károly Zsolnai-Fehér. "Two Minute Papers". 2015-. [[video](https://www.youtube.com/channel/UCbfYPyITQ-7l4upoX8nvctg)]
    - 最新技術を数分でまとめる動画群。

#### シェーディング

##### 理論

- Brent Burley. "Physically Based Shading at Disney". *SIGGRAPH 2012*. [[slides](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_slides_v2.pdf), [notes](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_notes_v3.pdf)]
    - マイクロファセット理論の総合的な調査報告書。

- Eric Heitz. "Understanding the Masking-Shadowing Function
in Microfacet-Based BRDFs". *JCGT 2014*.
 [[paper](http://jcgt.org/published/0003/02/03/paper.pdf)]
    - マイクロファセット理論のG項の考察。

- Naty Hoffman. "Physics and Math of Shading". *SIGGRAPH 2015*. [[slides](http://blog.selfshadow.com/publications/s2015-shading-course/hoffman/s2015_pbs_physics_math_slides.pdf), [video](https://www.youtube.com/watch?v=j-A0mwsJRmk)]
    - 最新技術の概要まとめ。

- Naty Hoffman. "Recent Advances in Physically Based Shading". *SIGGRAPH 2016*. [[slides](http://blog.selfshadow.com/publications/s2016-shading-course/hoffman/s2016_pbs_recent_advances_v2.pdf), [video](https://www.youtube.com/watch?v=zs0oYjwjNEo)]
    - 最新技術の概要まとめ。

##### 実践



#### シェーディング技法

##### 理論

- Ola Olsson, Markus Billeter, Ulf Assarsson. "Clustered Deferred and Forward Shading". *HPG 2012*. [[paper](http://efficientshading.com/wp-content/uploads/clustered_shading_preprint.pdf)]
    - Clastered Shadingの原著。

- Christopher A. Burns, Warren A. Hunt. "The Visibility Buffer: A Cache-Friendly Approach to Deferred Shading". *JCGT 2013*. [[paper](http://jcgt.org/published/0002/02/04/)]
    - Visibility Bufferの原著。

- Ola Olsson. "Introduction to Real-Time Shading with Many Lights". *SIGGRAPH 2014*. [[slides](http://efficientshading.com/wordpress/wp-content/uploads/sa_2014_intro.pdf)]
    - Forward ShadingからTiled Shadingまでを総ざらい。

##### 実践

- Jeremiah van Oosten. "Forward vs Deferred vs Forward+ Rendering with DirectX 11". 2015. [[blog](https://www.3dgep.com/forward-plus/)]
    - Forward Shading、Deferred Shading、Forward+の実装とパフォーマンス比較。

#### グローバル・イルミネーション

- Károly Zsolnai-Fehér, Thomas Auzinger. "TU Wien Rendering". 2015. [[slides](https://www.cg.tuwien.ac.at/courses/Rendering/VU.SS2017.html), [video](https://www.youtube.com/watch?v=pjc1QAI6zS0)]
    - レイトレーシング技術の概要まとめ。

#### ジョブシステム

- Stefan Reinalter. "Job System 2.0: Lock-Free Work Stealing". 2015. [[blog](https://blog.molecular-matters.com/2015/08/24/job-system-2-0-lock-free-work-stealing-part-1-basics/)]

- Christian Gyrling. "Parallelizing the Naughty Dog engine using fibers". *GDC 2015*. [[slides](http://www.gdcvault.com/play/1022187/Parallelizing-the-Naughty-Dog-Engine), [video](http://www.gdcvault.com/play/1022186/Parallelizing-the-Naughty-Dog-Engine)]
    - FiberベースのジョブシステムとFrame centricなメモリ管理。

#### Vulkan API

- Or-bach et al. "Moving to Vulkan: How to make your 3D graphics more explicit". *SIGGRAPH 2016*. [[slides](https://www.khronos.org/assets/uploads/developers/library/2016-uk-chapter-moving-to-vulkan/Moving-to-Vulkan_Khronos-UK_May16.pdf)]
    - モバイル系メーカーによるVulkan APIの概要。
