---
title: Low Level Optimization for GCN [@Drobot2014]
---
# MICHAL.DROBOT 3D @ UBISOFT MTL

# Low Level Optimization for GCN

# お品書き

- GCNアーキテクチャ
- 命令セット
- ALUの最適化
    - 新しい方法
    - アイデア
    - Tips & Tricks
- ユースケース

# GCNアーキテクチャ

- AMD Radeon HD7xxx R7x R9x
- Microsoft Xbox One
- Sony Playstation 4

# GCNアーキテクチャ

- AMDによる開かれた文書
    - ISA
- 良く網羅された基礎
    - "Low-level Shader Optimization for Next-Gen and DX11" --- Emil Persson
    - "The AMD GCN Architecture: A Crash Course" --- Layla Mah
- 基礎
    - 横に広げる[Keep it Wide]、占有率、低いリソース(VGPR^[Vector General Purpose Register])使用率
    - <font color="red">'賢い'ALUをいっぱい --- 帯域幅とトレードオフ</font>
    - GCNはレイテンシーを隠蔽するのに超良い --- ただし、お膳立てが必要

# ALUの最適化

- GCNのCU^[Compute Unit]は4クロックで256ヶの単精度[SP]ベクトルALUを実行できる
- 各レーンはクロックあたり1ヶの単精度ALU操作[SP ALU op]をディスパッチする
- 各単精度ALUは4クロックかかる
- SQ^[Shader seQuencer?]はクロックごと異なるwavefrontからディスパッチできる

# 基本の操作のパフォーマンス

- 32ビット算術および論理操作
    - フルレート
    - 例外として、int32のmul/madはクオーターレート
        - できるなら、フルレートのint24のmul/madを使う
- 64ビット算術および論理操作
    - ハーフレート
    - 例外として、mul/madはクオーターレート
- 変換およびパッキング操作
    - オペランドが32ビット以下ならフルレート
    - オペランドが32ビットより大きいならハーフレート

# 特別な操作のパフォーマンス

- 超越関数[transcendental functions]
    - 線形近似を用いる
        - 単一の関数がすべて(4つ)のSIMDで同時に動作する
    - クオーターレート
    - rcp、sqrt、rsqrt、log、pow、exp、sin、cos
    - 補助的操作
        - cleanup、accuracy、denormal flushingはフルレート
- 32ビットのグラフィクスの特殊な操作
    - フルレート
    - キューブマップ操作、パック済みバイト操作

# マクロな操作のパフォーマンス

- マクロなアンロールされる操作
    - tan、div、atan、acos、asin
    - smoothstep
    - length
    - normalize
- IEEE準拠の近似にアンロールする
    - 非常に高価
- 整数の除算
    - 浮動小数点数の計算でエミュレートされる
    - フルレートとクオーターレートの操作のかけ算

# コードフローのパフォーマンス: 分岐

- 分岐
    - 4つのWAITステート以上≒16サイクル以上≒16フルレート以上
        - 分岐サポートと論理演算
    - 潜在的なIキャッシュミスによる追加のレイテンシー
    - IC/ラベルに対する追加のVGPRの使用
    - すべての操作をスキップすることができる
        - BRANCHやSELECTよりさらに高速
    - 冗長なバッファ/テクスチャメモリ操作の場合に常に使う
        - 分岐のレイテンシーはL1キャッシュのレイテンシー以下

# コードフローのパフォーマンス: VSKIP

- VSKIP
    - 特殊な制御フローモード
        - クロックあたり10ヶのwavefrontsのレートでベクトル操作をスキップする
        - VMEM操作をVSKIPできない
    - ベクトルのみの小さなコード片用
        - BRANCHやSELECTよりさらに高速
    - コンパイラは依然として追い付こうとしている
        - 可能なら直接アセンブリを書く

# コードフローのパフォーマンス: SELECT

- SELECT
    - 標準のセレクター
    - 2つのコードパスを実行する
        - 比較に基づいて結果のひとつをSELECTする
    - [flatten]
    - 3項論理演算子
    - CndMask()

# VOP3 --- 3つのオペランドを持つベクトル命令

- モディファイヤのためのIEEEフラグに関係のない命令バンク
    - 入力モディファイヤ
        - abs、neg
    - 出力モディファイヤ
        - mul2、mul4、mul8、div2、div4
        - saturate

# VOP3 --- 3つのオペランドを持つベクトル命令

- ほとんどのコンパイラは以下のときに自動的にVOP3を使うだろう
    - 許可された (-fastmath -IEEEStrict disabled)
    - (-x)
    - saturate()
    - \*2.0、\*4.0、\*8.0、\*0.5、\*0.25

# VOP3 --- 制約

- VOP3(VDST, VSRC0, VSRC1, VSRC2)
    - VSRC0 --- リテラル、VGPR、SGPR
    - VSRC1/VSRC2 --- ある組み合わせでいくつかの制約がある
- すなわち、SGPR^[原文ではSGRPRs]からVSRC1とVSRC2の両方を発行できない
- suboptimalを強制される
    - SGPRからVGPRへのプリロード
    - VOP3の無効化

# VOP3 --- 制約 --- 例

```hlsl
float2 TexcoordToScreenPos(float2 inUV, float4 inFov) {
    float2 p = inUV;
    p.x = p.x * inFov.x + inFov.z;
    p.y = p.y * inFov.y + inFov.w;
    return p;
}
```

```gcn
s_buffer_load_dwordx4 s[0:3], s[12:15], 0x08
s_waitcnt lgkmcnt(0)
v_mov_b32 v2, s2
v_mov_b32 v3, s3
s_waitcnt vmcnt(0) & lgkmcnt(15)
v_mac_f32 v2, s0, v0
v_mac_f32 v3, s1, v1
```

# VOP3 --- 制約 --- 例 --- パッチ

```hlsl
float2 TexcoordToScreenPos(float2 inUV) {
    float2 p = inUV;
    p.x = p.x * 2.0 + (-1.0);
    p.y = p.y * -2.0 + 1.0;
    return p;
}
```

```gcn
v_mad_f32 v0, v0, 2.0, -1.0
v_mad_f32 v1, v1, -2.0, 1.0
```

# VOP3 --- 定数パッチング

- ビルトインの高速なリテラル定数[コード中に直接書き込まれる定数。定数リテラルとも]
    - ビルトインの+/-1、2、4、8の定数 --- すべてのVOP3で使える
- 単一のリテラル定数のサポート
    - v_madak_f32
    - v_madmk_f32

# VOP3 --- 定数パッチング

- Uniform Patchingを検討する
    - ユニフォームが定数の場合に
        - アーティストが生成したシェーダ
        - パーティクルシステム
        - トランスフォーム
- 有益である
    - より良いスケジューリングの機会が高まる
    - 効率的でタイトなループ(すなわち、スクリーンスペースのレイマーチング)
- ユニークなシェーダとパフォーマンスとのバランスをとる
    - 極めて重要なシェーダは"実行中に"常にパッチできる --- PS3スタイル

# 特殊なALU操作: リダクション

- min3
- max3
- med3
    - 中央値
    - clamp()
- 最適化
    - フィルタリング
    - ソート

# 特殊なALU操作: パッキング

- GCNは複数のパッキングおよび変換操作を公開している(圧縮されたMRTのために使われる)
    - F32 -> F16
    - F16 -> F32
    - F32 -> SNorm/UNorm
    - ...
    - 対でも: 2xF32 -> 2xF16
    - v_cvt_* --- ISA操作
- アンパッキング関数は手で書く必要がある

# 特殊なALU操作: BFE

- GCNは完全な32ビットUINT/INTサポートがある
    - マスキング、シフト、整数算術演算のための特殊な操作
- v_bfe_i32
    - 整数ベースのパッキングを扱うための符号拡張付きBitFieldExtract
        - 2の補数の整数形式のための符号拡張を気にすることを避ける
- v_bfe_u32
    - 符号なし整数ベースのパッキングを扱うためのBitFieldExtract
        - ビットマスク、フラグ、シフト＋マスク

```hlsl
// v_bfe_u32の参考実装
uint BitFieldExtract(uint inSrc, uint inOffset, uint inSize) {
    return (inSrc >> inOffset) & ((1 << inSize) - 1);
}
```

# 特殊なALU操作: BFE

```hlsl
// v_bfe_i32の参考実装
uint BitFieldExtractSignExtend(uint inSrc, uint inOffset, uint inSize) {
    uint size = inSize & 0x1f;
    uint offset = inOffset & 0x1f;
    uint data = inSrc >> offset;
    uint signBit = data & (1 << (size - 1));
    uint mask = (1 << size) - 1;
    return (-int(signBit)) | (mask & data);
}
```

```gcn
// 127, -1, -126をRGB_11_11_10にパックする
// 整数は2の補数である
// packed_data = 00001111111 11111111111 1110000010
// R(127)をアンパックする
BFE(packed_data, 21, 11) = 0000 0000 0000 0000 0000 0000 0111 1111
// B(-126)をアンパックする
BFE(packed_data, 0, 11) = 1111 1111 1111 1111 1111 1111 1000 0010
```

# 特殊なALU操作: BFEパック --- アンパック

- 高速な整数パッキングおよびアンパッキング

```hlsl
// Int16のパッキング
int PackInt2ToInt(in int inX, in int inY) {
    return (clamp(inX, -int(0x8000), 0x7fff) & 0xffff) | (clamp(inY, -int(0x8000), 0x7f00) <<16);
}

int2 UnpackInt2FromInt(in int inPackedInt) {
    return int2(
        BitFieldExtractSignExtend((int)inPackedInt, uint(0), uint(16)),
        BitFieldExtractSignExtend((int)inPackedInt, uint(16), uint(16)),
    );
}
```

# 特殊なALU操作: BFEパック --- アンパック

- 高速なSNorm16パッキングおよびアンパッキング

```hlsl
// SNorm16のパッキング
int PackSNorm2ToUInt(in float inX, in float inY) {
    return (clamp(int(inX * 0x7fff), -int(0x7fff), 0x7fff) & 0xffff) | (clamp(int(inY * 0x7fff), -int(0x7fff), 0x7fff) << 16);
}

int2 UnpackSNorm2FromUInt(in uint inPackedUInt) {
    return float2(
        BitFieldExtractSignExtend((int)inPackedUInt, uint(0), uint(16)) / float(0x7fff),
        BitFieldExtractSignExtend((int)inPackedUInt, uint(16), uint(16)) / float(0x7fff),
    );
}
```

# 特殊なALU操作: パックされたデータのサンプリング

- 'fat'なフォーマットにデータをパックする
- GATHERでサンプルする
- 例: バイラテラルフィルタリング
- UINT32にすべてをパックする
    - 8ビットのデータ
    - 16ビットの深度
    - 8ビットの法線
        - 4ビットSNORMとして
- 1回のGatherで
- 4つのデータ
- 4つの深度
- 4つの法線

# 特殊なALU操作: パッキング

- 寿命のboolフラグでレジスタプレッシャーを削減するのにビットフィールドを使う
    - countbits()
    - firstbithigh()
    - firstbitlow()

```hlsl
uint FastLog2(uint inX) {
    return firstbithigh(inX) - 1;
}
```

# 特殊なALU操作: BFE --- 二値操作

```hlsl
// ソフトウェアのトライアングル錐台(ニア面)クリッピング
// 一次方程式[line equations]の前の頂点ソート
float v0 = b[0]; float v1 = b[1]; float v2 = b[2];
if (b[0] > near_z || b[1] > near_z || b[2] > near_z) {
    if (b[1] > near_z) { v0 = t[1]; v1 = t[2]; v2 = t[0]; }
    if (b[2] > near_z) { v0 = t[2]; v1 = t[0]; v2 = t[1]; }
}
if ((b[0] > near_z && b[1] > near_z) ||
    (b[0] > near_z && b[2] > near_z) ||
    (b[1] > near_z && b[2] > near_z)) {
    if (!(b[0] > near_z)) { v0 = t[1]; v1 = t[2]; v2 = t[0]; }
    if (!(b[1] > near_z)) { v0 = t[2]; v1 = t[0]; v2 = t[1]; }
}
// コンパイルすると: フルレートのALUが42つ ＋ (16フルレート以上の)BRANCHが4つ
```

# 特殊なALU操作: BFE --- 最適化済み二値操作

```hlsl
// ソフトウェアのトライアングル錐台(ニア面)クリッピング
// 一次方程式[line equations]の前の頂点ソート
uint bitfield = 0;
bitfield |= b[0] > near_z ? 0x1 << 0 : 0x0;
bitfield |= b[1] > near_z ? 0x1 << 1 : 0x0;
bitfield |= b[2] > near_z ? 0x1 << 2 : 0x0;

float v0 = b[0]; float v1 = b[1]; float v2 = b[2];
uint csb = CountSetBits(bitfield);
uint csb_eq2 = (csb >> 1) & 0x1;

if (bitfield & 0x2 & csb) {v0 = t[1]; v1 = t[2]; v2 = t[0]; }
if (bitfield & 0x4 & csb) {v0 = t[2]; v1 = t[0]; v2 = t[1]; }
if (!(bitfield & 0x1) && csb_eq2) {v0 = t[1]; v1 = t[2]; v2 = t[0]; }
if (!(bitfield & 0x2) && csb_eq2) {v0 = t[2]; v1 = t[0]; v2 = t[1]; }
```

# 特殊なALU操作: キューブマップ

TODO
