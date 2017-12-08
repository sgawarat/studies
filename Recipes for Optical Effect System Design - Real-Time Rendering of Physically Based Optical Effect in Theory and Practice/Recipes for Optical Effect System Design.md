---
title: Recipes for Optical Effect System Design
numberSections: false
---
# 光学シミュレーションのパラメータ[Optical Simulation Parameters]

# 光学に基づかないパラメータの使用[Use of Parameters Not Based on Optics]

- 長所
    - アーティストは光学的整合性[optical consistency]に制限されない。
- 短所
    - 出力はアーティストに強く依存する。
    - 多くの経験か知識が必要になる。
    - **写真撮影[photography]に親しみのない人は不自然な結果にほとんど気付かないかも。**
        - 物理的に怪しい[implausible]結果が定期的に見つかる。
            - 例: 背景のボケが近、中距離と比較して大きすぎる。
        - 写真撮影に親しみのある人がみれば、何かがおかしいと感じるかも。

# 不自然なDOFハンドリング[Unnatural DOF Handling]

- F値はフォーカシングかズーミングで過度に[immoderately]変化する。
- FOVが広いとき、背景のボケが大きすぎてしまう。
    - ミニチュアに見えるシーン

# 自然なDOFハンドリング[Natural DOF Handling]

- 背景のCoCサイズはフォーカス距離やFOVにより異なる。
    - 広いFOV、または、遠いフォーカスは小さなボケを生み出す。
    - 狭いFOV、または、近いフォーカスは大きなボケを生み出す。
- F値は変えなくても良い。
    - フォーカシング時はF値を一定にしたままにする。
    - 最小F値はそのメカニズムに依存してズーミングにより変化するかも。

# 光学に基づくパラメータの使用[Use of Parameters Based on Optics]

- アーティスト設定から必要なパラメータを計算する。
    - アーティスト設定
        - FOV、絞り[aperture]、フォーカス距離、など
- 長所
    - **光学的整合性を維持できる。**
    - 設定するパラメータが少ない。
- 短所
    - アーティストは光学的整合性により制限される。
        - アーティストが思い通りにDOFを制御できないかも。

# 写真撮影に基づかないパラメータの使用[Use of Parameters Not Based on Photography]

- 光学パラメータは依然として不自然になるかも。
    - 10,000mmのフォーカス距離
    - 0.1のF値
    - 1cmの望遠レンズ[telephoto lens]のフォーカス距離
    - などなど…

# よくある間違い[A Frequently Made Mistake]

- 人物にピントを合わせて背景をボヤけさせたいとき
    1. 広角FOVを用いて人物をフレームに収める。
    1. 人物にピントを合わせる。
        - 背景をボヤかしづらい、ので
    1. 背景がボヤけるまで絞り[aperture]を開く。
        - F値が小さくなりすぎてしまうかも。

⇒ **シーンが不自然にミニチュアっぽくなる。**

# 背景をボヤかす間違った方法 f/0.3(写真撮影に基づかない)[The Wrong Way to Make a Background Blurry f/0.3 (Not Based on Photography)]

# パラメータは写真撮影に基づくべき[Parameters Should be Based on Photography]

- 不十分なブラーは絞り[aperture]設定の問題ではない。
    - カメラ位置とFOVの関係の問題である。
        - カメラを前方に動かしたりズームインするとボケが大きくなる。
- **写真撮影に基づかないパラメータは不自然に見える結果を引き起こす。**
    - 光学パラメータを適切な範囲に制限する。
        - F値はf/1.0からf/32.0までにするべき。
            - ズームレンズではf/2.0以上
        - フォーカス距離の下限は焦点距離の2倍にすべき。
            - つまり、マクロ1:1
        - など

# 写真撮影に基づくパラメータ[Parameters Based on Photography]

- **アーティストはカメラを移動したりズームインしたりしなければならないだろう。**
    - その結果は自然に見えるだろう。

# 背景をボヤかす間違った方法 f/0.3(写真撮影に基づかない)[The Wrong Way to Make a Background Blurry f/0.3 (Not Based on Photography)]

# 背景をボヤかす適切な方法 f/2.8(写真撮影に基づく)[The Proper Way to Make a Background Blurry f/2.8 (Based on Photography)]

# 光学パラメータのアニメーション[Optical Parameter Animation]

- フォーカスアニメーション
- ズーミングアニメーション
- フォーカス呼吸
- ズーミング時の最大絞りの変化
- 口径食の変化
- などなど…

<!-- p.30 -->

- アーティストはこれらすべてを制御すべき？
    - 多くの経験か知識が必要になる。
        - フォーカシングと呼吸はまったく別[exclusive]の知識である。
    - **出力の品質はアーティストの経験か知識に強く依存する。**
- **アーティストにより指定された設定をオーバーライドする。**
    - フォーカシングアニメーションを自動化する。
        - アーティストはフォーカシングの対象を設定するだけ。
    - フォーカス呼吸によりFOVをオーバーライドする。
    - FOVに依存して最小F値をオーバーライドする。
    - などなど…

# フォーカス呼吸によるFOVのオーバーライド[Override FOV by Focus Breathing]

- フォーカス距離とフォーカシングメカニズムに依存してFOVを変更する。

# フォーカス呼吸の近似[Approximation of Focus Breathing]

- 無限遠[infinite focus]のFOVとフォーカス距離から現在の有限遠[finite focus]のFOVを計算する。
    - $f = h / (\tan(fov / 2) * 2)$
        - $f$: 無限遠の焦点距離
    - $d_i = d_o f / (d_o - f)$
        - $d_i$: 無限遠の像距離
    - $s = d_i / f = d_o / (d_o - f)$
    - $sr = \text{pow}(s, r)$
    - $d_i' = f * sr$
        - $d_i'$: 現在の有限遠の像距離(呼吸の結果)
    - $fov' = \text{atan}(h / (d_i' * 2)) * 2$
        - $fov'$: 現在の有限遠のFOV(呼吸の結果)
    - 呼吸の結果に対する光学の有効焦点距離
        - $f' = d_i / s = f * sr / s = (d_o h / 2) / (\tan(fov' / 2) * d_o + h / 2)$
        - フォーカス距離と$r$に依存してズレるだろう。
    - **$r$はフォーカス呼吸の度合い[degree]**
        - $r$を$-1$から$1$の間に設定することで呼吸を制御できる。

# 呼吸の度合い: r[Degree of the Breathing: r]

- **$1.0$は、FOVが単レンズのルールに応じて狭角になるであろうことを意味する。**
    - 全群繰り出しフォーカシング(典型的なマクロレンズ)
    - 光学の焦点距離は常に一定である。
- **$0.0$は、FOVが一定である(呼吸しない)ことを意味する。**
    - 焦点距離は像距離をズラす代わりに短くなる。
    - いくつかの高価なレンズ
- **$-1.0$は、有限遠に焦点を合わせるとき、FOVが広角になるであろうことを意味する。**
    - 焦点距離は更に短くなる。
    - 典型的なインターナルフォーカスレンズ(安価なレンズ)
- アーティストはレンズごとに$r$を選択する。
    - 直接$r$を設定する。
    - レンズのフォーカスメカニズムを選択する。

# 遠い焦点[Far Focus]

# 近い焦点(全群繰り出しフォーカシング: r = 1)[Close Focus (All-Group Focusing: r = 1)]

# 近い焦点(呼吸なし: r = 1)[Close Focus (with No Breathing: r = 0)]

# 近い焦点(インナーフォーカシング: r = -1)[Close Focus (Inner Focusing: r = -1)]

# 可変最大絞りズームレンズ[Variable Maximum Aperture Zoom Lens]

- 焦点距離に依存して最小F値を変更する。
    - 現在のF値を最小F値に制限する。
- 現在のF値と最小F値の比に依存して絞り形状を変更する。
    - 円形絞りから多角形絞りへ

# 可変絞りズームレンズの近似[Approximation of Variable Aperture Zoom Lens]

TODO

# 参考文献[References]
