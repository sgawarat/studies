---
title: Deferred Lighting in Uncharted 4 [@Garawany2016]
bibliography: bibliography.bib
numberSections: false
---
# DEFERRED LIGHTING

# 目標と動機付け(Goals and Motivations)

- マテリアルデータの読み込み/修正を行う必要がある多数のスクリーンスペースエフェクト。
    - パーティクル。
    - デカール。
    - より重要: SSR、キューブマップ、間接シャドウ、など。
- マテリアルデータの節約から逃れられない。

# 選択肢(Options)

- フォワードジオメトリパスに伴ういくつかの出力(色、法線、支配的間接方向、など)を持った"安価"なGバッファパスで実験した。
    - 現在のハードウェアにおいてジオメトリパスは安価ではない。
    - 我々は…密なオブジェクト…がある。

<!-- p.13 -->

- フォワードシェーダでのライティングの複雑さは信じられないレジスタプレッシャーを追加する。これはジオメトリパスをさらに遅くする。
- 両方の世界の最善となる方法はあるのか？

# 解決法(Solution)

- 完全にディファード化する！

<!-- p.16 -->

- Gバッファはゲーム中のすべてのマテリアルを残念ながらサポートしなければならない。
- 幸運なことに、我々には大量のメモリと大量の帯域幅がある。

# Gバッファ(GBuffer)

- 16ビット毎ピクセルの符号なしバッファ。
- プロダクション中は機能の間を巡ってビットが絶えず移動していて、どれだけのビットが様々な機能に対して必要とされたかを厳密に決定するための大量のビジュアルテストを行った。
- GCNのパラメータパッキング命令を多用。
- さらなる詳細は水曜日の"The Technical Art of Uncharted 4"を確認して。

||||
|-|-|-|-|
|R|r|g||
|G|b|spec||
|B|normalx|normaly||
|A|iblUseParent|normalExtra|roughness|
: Gバッファ0

||||||
|-|-|-|-|-|
|R|ambientTranslucency|sunShadowHigh|specOcclusion||
|G|hightmapShadowing|sunShadowLow|metallic||
|B|dominantDirectionX|dominantDirectionY|||
|A|ao|extraMaterialMask|sheen|thinWallTranslucency|
: Gバッファ1

# 任意のGバッファ(Optional GBuffer)

- 任意の第3Gバッファはより複雑なマテリアルで使われる。マテリアルのタイプに基づいてさまざまに解釈される。
- 任意のGバッファを用いるマテリアルの例として織物(fabric)、髪、肌、絹(silk)がある。
- Gバッファの解釈は相互に排他的である(つまり、織物と肌を同じピクセルに持つことはできない)。この制約はマテリアルオーサリングパイプラインで強制される。
- 任意のGバッファはマテリアルが必要としない限り読み書きされない。

# 問題(Problems)

- ディファードシェーダはすぐさま肥大化する。
- すべてのライトタイプは言うまでもなく、肌、織物、植物、金属、髪などをサポートシなければならない。

# ディファードパイプライン(Deferred Pipeline)

- マテリアル"ID"テクスチャを保存する。
    - 実際ホントのマテリアルIDではなく、単なる使用するシェーダ機能のビットマスクである。
    - 12ビットが8ビットに圧縮される。(機能の相互排他性を計算に入れることで)

# 分類(Classification)

- 16x16タイルごとに、ルックアップテーブルを指すためにタイル全体に対するマテリアルマスクを用いる。
- ルックアップテーブルは事前計算され、タイル内ですべての機能をサポートするできるだけ最も単純なシェーダを保持する。

```hlsl
uint materialMask = DecompressMaterialMask(materialMaskBuffer.Load(int3(screenCoord, 0)));

uint orReducedMaskBits;
ReduceMaterialMask(materialMask, groupIndex, orReducedMaskBits);

short shaderIndex = shderTable[orReducedMaskBits];

if (groupIndex == 0) {
    uint tileIndex = AtomicIncrement(shaderGdsOffsets(permutationIndex));

    tileBuffers[shaderIndex][tileIndex] = groupId.x | (groupId.y << 16);
}
```

<!-- p.22 -->

- シェーダがライティングするタイルのリストへタイル座標をアトミックにプッシュする。
- アトミック整数はDispatchIndirectの引数バッファのディスパッチ数でもある。

<!-- p.23 -->

- すでに大きな改善がなされている。
- [@Coffin2011]以前には似たテクニックが使われていた。

# 最適化(Optimization)

TODO

# 参考文献(References)

[1] [@Coffin2011]
[2] [@Lagarde2014]
[3] [@Drobot2014]
[4] [@Mazonka2012]
[5] [@Klehm2011]
