---
title: Pipeline State Object Caching [@Archard2016]
bibliograhy: bibliograhy.bib
numberSections: false
---
# Why do we care?

- なぜなら、あなたが作るであろう最も簡単な最適化のひとつである。
- 完璧なPSO生成は常に実行可能であるとは限らない。
    - DX9やDX11のレンダリングインターフェース、スクリプト駆動レンダリングステート、など。
    - オンザフライで生成されるPSOは現実に存在する。
- パイプライン生成はめっっっっっっっっっっっっっっっっっっちゃ遅くなる可能性がある。
    - 狂ったようにヒッチングする。
- PSO生成中に発生する冗長な処理が大量に存在する。
    - GLESはあなたに代わりこれの世話をしていた。
- Epic GamesのProtostarからのユースケース。

# PSO create time break-down --- Epic Games Protostar

- リンク: 56%
    - ユニークなリンク: 54%
    - 冗長なリンク: 46%
- コンパイル: 42%
    - 冗長なコンパイル: 62%
    - ユニークなコンパイル: 38%
- その他の処理: 2%

# Possible solutions to speed up PSO creation

- 動的パイプラインステート。
    - 変化できるステートが制限される。
- パイプラインを継承する。
    - ベンダ固有。
    - ほとんどのエンジンで接続するのが難しい。
- パイプラインステートキャッシュ。

# Pipeline Cache

- パイプラインキャッシュは`vkCreatePipelineCache`で生成する。
    - キャッシュデータがあれば生成時に渡す。
- `vkCreateGraphicsPipelines`などでパイプラインを生成する。
    - `pipelineCache`に生成したパイプラインキャッシュを渡す。
- `vkGetPipelineCacheData`でキャッシュデータを取り出せる。
    - ディスクを介して次回以降の起動を高速化できる。

# References
