---
title: >
    A Certain Slant of Light: Past, Present and Future Challenges of Global Illumination in Games [@Barre-Brisebois2017]
---
# A Certain Slant of Light: Past, Present and Future Challenges of Global Illumination in Games

# 自己紹介

# アジェンダ

- この分野の現状
- 現在の課題
- 今後のこと
- 質疑応答

# あなたのために何が入っているか？

以下のことに関して知りたいので…

- 研究者向け
    - 学術研究[academia]におけるGIをゲームにおける**最新技術**と比較する方法
    - ゲームにおける最新のアート、エンジニアリング、パイプライン、**プロダクション上の課題**
    - アーティストおよびプログラマが現在のアプローチを持っている**根本的な問題**
    - ゲーム業界における**未解決のGI問題**、および、**将来の研究を適応させる**方法
- ゲーム開発者向け
    - あなたの分野における**GIの現状**！
    - 学術研究を目標達成のために**役立てる**方法
    - **共同研究の機会[collaboration opportunities]**を改善&生成するためにできること
    - GIが今後のゲームで向かう先

# 始める前に

- 連絡をくれたり、コンテンツを収集するのを手伝ってくれた**全員**に感謝
- 発売済みのゲームでのGIの**最新技術**を**まとめる**ためのベストを尽くした
- あれやこれやを見逃しているみたい？**教えてください**！
- すべての参考文献はトークの終わりに

# 免責 --- ストレージ対データ

***ストレージはフォーマットやエンコーディングから非依存にできる***

しばしば取り合わせは様々となり得る

# ゲームの大域照明(GI)とは？

[@Ritschel2011]

# この分野の現状

# Surface Caching

Quake [@Abrash2000]

- ゲームにおける大域照明の始まり…
- **事前計算済み放射照度[precomputed irradiance]**を格納する最初のゲームのひとつ
- 空間、ボリューム、張り[tension]の感じがより良い
- 頂点かテクセルに影響を与えるすべてのライトに対してひとつの**8ビット**値
- 後に色付けされる(Quake2)

画像は[@Bush2015]から

# Radiosity Normal Mapping

Mirror's Edge [@Halen2009]

- Half-Life2で見られるヤツ [@McTaggart2004]
- **平坦なアンビエントを回避する**
- TS^[訳注:恐らくTangent Spaceのこと]における3つの方向に、3つの完全なRGBのライティング値
- *cos(NormalMap, BasisDirections)*に基づいたブレンディング
- **スペキュラを近似できる**
    - ビューベクトルが基底に並行であるとき
    - 粗いスペキュラだと"OK"、低いラフネスだと駄目
- 多くのゲームやエンジンで使われている！

# Spherical Harmonics Lightmaps

[@Chen2008]
[@Sloan2008]

- "大局的"な改良された忠実性へのもうひとつの一歩
- **オフラインのフォトンマッパー**に由来する球面調和関数
- テクセルは、球面上の連続関数として、表面点[surface point]に放射照度を持つ
- 動的オブジェクトのためのプロブもまたSHとして格納される
- SHは**ディフューズには素晴らしい**が、スペキュラには…

Halo 3

# Spherical Gaussians Lightmaps

The Order: 1886 [@Neubelt2014]
[@Pettineo2016]

- 球面ガウス関数[spherical gaussians]を用いて入射する放射輝度を近似する
- **ディフューズとスペキュラ**に対する直観的かつコンパクトな表現
- 動的オブジェクトのためのプロブもまたSGとして格納される
- 広範囲に渡る実装がMattのブログで詳しく述べられている[@Pettineo2016]

# Heightfield GI

UE4 Kite Demo --- 高さフィールドGIなし(左)とあり(右) [@Seymour2015]
MotoGPにおけるStatic Terrain Radiosity Maps [@Hargreaves2003]

- 高さフィールド上の**広いエリア**や**環境光**用。動的にできる[@Nowrouzezahrai2009]
- MotoGPの事前計算済みラジオシティマップ(2003)以来の多くの改善[@Hargreaves2003; @Oat2007]
- 利点: 高さベースのワールドに対する巧みな解法
- 欠点: 高さフィールドのみ。屋内では機能しない

# Precomputed Form Factors

[@Enlighten]

- 有限要素にシーンを**分割[subdivide]**する
- 要素間の可視性を**事前計算**する
- *大量の動的ライト*に対する、ディフューズGIに対する実行時に使われるtransfer関数
- ライトマップやプロブに格納される
- ジオメトリ(可視性)が変化すると再計算しなければならない

# Probe-only* Approaches

Tom Clancy's The Division [@Stefanov2015]
Quantum Break [@Silvennoinen2015]

- プロブにおける**PRT**(FarcryやDivision)、**放射照度ボリューム**([@Tatarchuk2005]やQuantum Break)
- 様々に統一化された[various unified]CPUまたはGPUのコンソールで使いやすい[console friendly]大域照明の近似
    - *FarCry4*: GPUページテーブルへのプロブセルのストリーミング(radiance tranfer、clipmap injection and sampling) [@McAuley2015]
    - *The Division*: surfels-to-probe、surfel共有付き (Gバッファのキューブマップ) [@Stefanov2015]
    - *Quantum Break*: 八分木を介する適応的放射照度ボリューム[@Silvennoinen2015]
- \*ライトマップとの組み合わせで用いることもできる。後に、ライティングと可視性を分ける[@Iwanicki2017]
- 利点: 同じデータで動的および静的オブジェクトが照らされる。*ボリューメトリック*！
- 欠点: 間接スペキュラの品質が高くない。ので、スペキュラはIBLやSSRから

# Virtual Point Lights

Gears of War 3 [@Malmros2017]
The Last of Us

- **Reflective Shadow Maps**(RSMs)から**Virtual Point Lights**(VPLs)を生成する[@Dachsbacher2004]
- プレイヤーのフラッシュライト用だが、他のスポットライト用にも
- 利点: バウンスを近似する安価なポイントライト。かなり説得力を強くできる！
- 欠点: 不安定になり得る(VPLの選択)。局所化された解法。精度が気になるかも。

# Light Propagation Volumes

FableにおけるCascaded Light Propagation Volumes [@Woodhouse2014]
Clipmap
CrytekのLight Propagation Volumes [@Kaplanyan2009]

- RSMsにシーンをレンダリングして、ボリュームにVPLsを**注入**して、**伝播**して、**適用**する[@Kaplanyan2009]
- **clipmap**でカスケードする: 近くが細かくなり、距離に一致する[consistent]
- 利点: **事前処理なし**。コンソールハードウェアで使いやすく、**ボリューメトリック**！
- 欠点: グリッドの不整合[misalignment]がbleedingを引き起こし得る。VPLエイリアシングに起因する安定性の問題

# Voxel Cone Tracing

Sparse Voxel Octree Cone Tracing [@Crassin2011]
The Tomorrow Children's Cascaded Voxel Cone Tracing [@McLaren2015]

- 元々のアイデアは[@Crassin2011]に由来、**Sparse Voxel Tree**を用いる
- **clipmap**を使う[@McLaren2014; @Pantaleev2015]、八分木と比べてよりGPUフレンドリー[@Mittring2012]
- 利点: **ディフューズ**および**スペキュラ**のGI、ボクセルによる**中から大縮尺のAO**[@Crytek2016]
- 欠点: いくらかのGIのリーク、低ラフネスのスペキュラコーンのコストが重大、clipmapの遷移

# Ambient Occlusion?

- 複数の頻度
    - **近距離場[near-field]**: 頂点やライトマップ、空洞[cavity]、スクリーンスペース(SSAO) [@Mittring2007]
    - **中距離場[mid-field]**: 頂点やライトマップ、SSAO、ボリューム、カプセル [@Hill2010]
    - **遠距離場[far-field]**: [@Sloan2016]、高さベース[@McAuley2015]、距離フィールド(UE4) [@Wright2015]

GPU上でベイクするAmbient Obscurance [@Sloan2016]
SH Sky Occlusion [@McAuley2015]
AO Fields and AO Capsules [@Hill2010]

# Screen Space GI?

SSDO [@Ritschel2009]
Quantum Break [@Silvennoinen2015]

- スクリーンスペースを介する追加の近距離場の局所的なディフューズGI
    - Screen-Space Diffuse Lighting [@Silvennoinen2015]
    - SSDOに由来する第1バウンス近似に触発される[@Ritschel2009]

# Specular GI?

Stochastic SSR [@Stachowiak2014]
Cubemap Relighting [@McAuley2015]
Box Projected Cube Maps [@Bech2010]

- 以下を介して大抵は近似される
    - Parallax-Corrected Cubemaps [@Bech2010; @Lagarde2012]
    - Screen-Space Reflections (SSR) [@Sousa2011; @Stachowiak2014]
    - Cubemap relighting [@Stachowiak2014; @McAuley2015]
    - 2次のSHから支配的な色[dominant color]を抽出する
    - SGに対するスペキュラ[@Neubelt2014]
- ほとんどのゲームはスペキュラ光輸送のためのスペキュラを行わない

# アート、エンジニアリング、パイプライン、プロダクション上の課題

TODO
