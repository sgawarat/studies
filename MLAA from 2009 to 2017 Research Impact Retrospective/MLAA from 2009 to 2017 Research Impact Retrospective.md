---
title: MLAA from 2009 to 2017 Research Impact Retrospective [Jimenez2017]
numberSections: false
---
# 金字塔: MSAA --- CSAA --- EQAA[Gold Standards: MSAA - CSAA - EQAA]

10年前、金字塔的存在はMSAAだった。

このテクニックは各ピクセルに一度だけのシェーディングに基づくが、個別に各サブピクセルをシェードする真のSSAAとは対称的に、複数の深度テストの結果を格納している。

これは時折高価なソリューションであったが、大方、代替案を探す研究はほとんど行われていなかった。

# エッジ検出とブラー[Edge Detection and Blur]

[@Leadbetter2009a]からの情報
[@Sousa2007]
[@Shishkovtsov2004]

最も使われるMSAAへの代替案は、S.T.A.L.K.E.R、Crysis1、Brutal Legendで使われるモノのような、エッジ検出とブラーである。

これは単純で安価であり、この時点においてディファードエンジンで利用できる最良の選択肢であった。

しかし、その出来[the results]はMSAAと比べてはるかに[significantly]低品質であった。

# モーフォロジカルアンチエイリアシング[Morphological Antialiasing]

[@Reshetov2009]

その後、2009年には、モーフォロジカルアンチエイリアシングが登場[come into play]し、常識が一変[changed the rules]した。

私の意見では、MLAAの最大の功績はコミュニティの考え方[mindset]を一変させたことである。

それはMSAAを用いるより高品質なこの問題に対する様々なアプローチがあるかもしれないことを証明した。

それはこの問題に対する標準的で一般に認められているソリューションに疑問を投げかけた。

# ポストAAテクニックの爆発[Explosion of Post-AA Techniques]

[@Jimenez2011a]

そして、これは同様に他の多くの人々にそれについて考えさせた。

2011年には、とてもたくさんのポストプロセッシングAAテクニックが存在した。それらはひとつのコースに一斉に実際に集合している。

次のいくつかのスライドではこれらのいくつかをカバーしたいと思う。

# モーフォロジカル対エッジ検出とブラー[Morphological versus Edge Detect and Blur]

- モーフォロジカル
    - エッジ検出＋**サーチ**＋ブラー
    - 非局所的情報が利用可能(パターン)
- エッジ検出とブラー
    - エッジ検出＋ブラー
    - 局所的情報のみ利用可能(近傍)

でも、続ける前に、モーフォロジカルとエッジ検出とブラーの区別を付け[make the distinction]よう。

私の見解では[from my point of view]、モーフォロジカルを区別するモノは、より正確なピクセルのブレンドを行う機会を与えるサーチ処理である。

# Saboteur (2009)

[@Leadbetter2009b]からの情報

SaboteurはPS3においてコンソールで今まで見たことのないアンチエイリアシング品質を伴って出荷され、高品質なポストプロセッシングアンチエイリアシングを伴って出荷した恐らく初めてのゲームである。

Digital Foundryはこれを詳細にカバーし、このテクニックがMLAAかそれと同種のものであると結論づけた。

しかし、残念ながら、実際の詳細は、そのプレゼンテーションが取りやめになってしまったので、公に知られることはなかあった。

# SonyのエッジMLAA(2009)[Sony's Edge MLAA (2009)]

[@Perthuis2011]

Sonyもまた、高速なPS3 SPU実装に焦点を当てた彼ら自身のMLAA実装を作成した。

<!-- p.9 -->

それは品質、特に、ディテールの保存に多大な注意を払っている。

とりわけGod of War3、Killzone 3、Little Big Planet 2がSonyのMLAAで出荷した。

それはPS3で開発しているサードパーティにさえも提供された。

# Double Fine's Costume Quest (2010)

Double Fine's Costume QuestはXbox360でMLAAを伴って出荷され、恐らくこのコンソールでは初めてのことであった。

<!-- p.11 -->

最も興味深い機能のひとつはCPU/GPUのハイブリッドなソリューションであることだった。

# Star Wars: The Force Unleashed II - DLAA (2010)

TODO
