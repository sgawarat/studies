---
title: Optics - Real-Time Rendering of Physically Based Optical Effects in Theory and Practice [@Kakimoto2015]
bibliography: bibliography.bib
numberSections: false
---
# はじめに[Introduction]

# 光の物理[Physics on Lights]

- 光学[optics]
    - 幾何光学[geometrical optics] --- 単純で実践的なモデル
    - 波動光学[wave optics]^[物理光学[physical optics]とも] --- より物理的に正しく複雑
- 電磁気学[electromagnetism] --- 古典的な物理モデル
- 量子光学[quantum optics] --- 近代的な物理モデル

# 光学とコンピュータグラフィックス理論[Optics and Computer Graphics Theories]

- コンピュータグラフィックス理論は光学に基づいている。
    - 理論と技術の大多数は幾何光学に基づく。
    - 1%だけ波動光学を勘定に入れている。
<!--  -->
フォトンマッピングは量子光学から'フォトン'の概念を借用して、幾何光学の枠組みで用いている。

# トピック[Topics]

- このコース
    - ほとんどのトピックは幾何光学に関連している。
    - いくつかは波動光学関連のものである。
- このトークは以下をカバーする。
    - 残りのコースのための基本的な幾何光学の知識。
    - この後のトークのための波関連のトピックの簡単な紹介。

# 基本的な幾何光学[Basic Geometrical Optics]

# CGの幾何光学モデル[Geometrical Optics Models for CG]

- ピンホールカメラモデル
    - ｜＋絞り[aperture]
    - ｜＋近似された屈折
    - ｖ
- 薄レンズ[thin lens]の近似
    - ｜＋厚さ[thickness]
    - ｖ
- 厚レンズ[thick lens]の近似
    - ｜＋正確な屈折
    - ｜＋複数の波長
    - ｜など
    - ｖ
- 完全なレンズシステム

# 幾何光学モデルと効果[Geometrical Optics Models and Effects]

- ピンホール
    - 透視投影[perspective projection]
    - モーションブラー
    - 自然の口径食[natural vignetting][^natural_vignetting]
- 薄レンズ/厚レンズ
    - ボケ(デフォーカス[defocus])
    - フォーカス呼吸[focus breathing]
    - 自然の口径食[natural vignetting]
- 完全にシミュレートされたレンズ
    - 複雑なボケ
    - 色収差[chromatic aberration]
    - 光学の口径食[optical vignetting][^optical_vignetting]
    - レンズゴースト

[^natural_vignetting]: "natural vignetting"は口径食のうち、光がその入射角の「コサイン4乗則」に従うために周辺光量が低下することを指す。[@Wikipedia:Vignetting]

[^optical_vignetting]: "optical vignetting"は口径食のうち、レンズの構成要素によって光が物理的に遮られることで周辺光量が低下することを指す。[@Wikipedia:Vignetting]

<!-- p.12 -->

- 本日のトピック
    - ピンホール
        - 自然の口径食[natural vignetting]
    - 薄レンズ/厚レンズ
        - ボケ(デフォーカス)
        - フォーカス呼吸[focus breathing]
        - 自然の口径食[natural vignetting]
    - 完全にシミュレートされたレンズ
        - 複雑なボケ
        - 色収差[chromatic aberration]
        - 光学の口径食[optical vignetting]
        - レンズゴースト

# 幾何光学モデルと実装[Geometrical Optics Models and Implementations]

- ピンホール
    - レイトレーシング
    - グラフィックスハードウェア(固定パイプライン)
    - <--- プログラマブルシェーダ技術 ---
- 薄レンズ/厚レンズ
    - <--- 分散レイトレーシング[distribution ray tracing] ---
    - 累積バッファ[accumulation buffer]
    - ポストプロセッシング
    - --- プログラマブルシェーダ技術 --->
- 完全なレンズシステム
    - --- 分散レイトレーシング[distribution ray tracing] --->
    - 波面トレーシング[wavefront tracing]

<!-- p.14 -->

- 本日のトピック(幾何光学)
- ピンホール
    - <--- プログラマブルシェーダ技術 ---
- 薄レンズ/厚レンズ
    - ポストプロセッシング
    - --- プログラマブルシェーダ技術 --->
- 完全なレンズシステム
    - 波面トレーシング[wavefront tracing]

# 薄いレンズ --- [Thin Lens --- Fundamentals to Understand Real-Time Special Effects]

TODO

# 参考文献[References]
