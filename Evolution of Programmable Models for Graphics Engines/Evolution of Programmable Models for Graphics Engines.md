---
title: Evolution of Programmable Models for Graphics Engines [@Tatarchuk2017]
numberSections: false
---

# グラフィクスのプログラミングモデルは複雑である[Programming Models for Graphics are Complex]

今日のトークでは、リアルタイムグラフィクスで用いるプログラミングモデルと、これらを改善できそうな方法に触れたい。会場の皆が容易に同意するであろう通りに、グラフィクスエンジニアリングのプログラミングモデルとして我々が現時点で持っているものはかなり複雑な猛獣[beast]である。我々はその領域で数々の特徴を持つ。

# 問題空間[Problem space]

## プラットフォーム及びAPIの多様性[Platform and API divergence]

近年のグラフィクスプログラミングはとても断片化した複雑なプラットフォームやAPIのエコシステムの上に生きている。

##リソース管理は必須である[Resource Management is Required]

グラフィクスに関連することを行うには何事にも、リソース(テクスチャ、シェーダ、メッシュ、アニメーション、など)をどうにかして得なければならない。これはしばしば大量の定型句[boiler plate]を意味し得る。

## ヘテロジニアスな計算モデル[Heterogenous Computational Models]

その上、ヘテロジニアスなマシンへのプログラミングについて考える必要がある --- GPUとCPUは大幅に異なるプログラミングモデルを持ち、ゲーム及びグラフィクスエンジンのプログラミングはすべての構成要素を提供する必要がある。グラフィクスアルゴリズム研究のアプリケーションも同様[ditto]。

##高級なアルゴリズム空間は深遠な構成可能性を必要とする[Rich Algorithm Space Needs Deep Configurability]

リアルタイムグラフィクスで用いる高レベルパイプラインアルゴリズムは急増している[have an explosion of] --- フォワードレンダリング、Forward+、ディファードレンダリング、Clusteredアルゴリズム、Z-Binning、GPUオクルージョン管理、例えばfine-prune tiled lightingのような様々なTiledアプローチ、などなど。これらすべては特定のパイプライン目標に対する有用性[utility]を持つ。例えば、VRはシングルパスのフォワードレンダラから利益を得るが、多くのコンソールゲームはディファードレンダリングを採用している。これらすべてのアルゴリズムは、それをサポートする予定である場合、基礎にあるエンジンの深い構成可能性を必要とする。

## コンテンツクリエイターとエンジニアリングの橋渡し[Bridge Content Creators and Engineering]

もちろん、ゲームエンジンやグラフィクスアプリケーションはしばしばコンテンツとエンジニアリングの橋渡しとして役立つ。

### グラフィクスプログラマーの務めはコンテンツ(アート)とプラットフォーム(ハードウェア)の間のインターフェイスを定義、熟考することである[The task of a graphics programmer is to define and mediate the interface between content (art) and platform (HW)]

[Foley2016: Open Problems in Real-Time Rendering course. SIGGRAPH 2016]

### コンテンツとプラットフォーム[Content and Platform]

このスライドの上部分には *コンテンツ* がある。これはアーティストが理解して作成しようとする概念のすべてである。我々には形状、動作、外観、照明なようなものがある。
このスライドの下部分には、我々が対象とする *ハードウェアプラットフォーム* がある。これは、スレッドやSIMDのサポートを持つCPUコアや伝統的なラスタライゼーションパイプラインとある種の汎用用途の計算の両方をサポートするGPUハードウェアがあり、近年似たようなものが見られる。
アーティストが生成するものを得るために、我々はコンテンツをハードウェアプラットフォームにマップする方法に対する計画を持つ必要がある。

### アセット毎、プラットフォーム毎マッピングを特定する？[Specify mapping per-asset, per-platform?]

これを行い得る(悪い)方法のひとつはアセット毎、プラットフォーム毎の基準でコードを書くことによるものである。これは明らかにスケールしない。つまりは、我々が実際に行っていることではない。And that gets us to the heart of the definition...

### アプリケーション固有のインターフェイスを用いてマップする[Map using application-specified interface]

TODO
