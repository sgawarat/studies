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

- キューブマップは統一されたimage_sampleを用いてサンプルされる
- サンプリングのために面UVや面IDを計算する必要がある
    - すべてのハードウェアはフルレートの独自の操作で高速化される

```gcn
v_cubetc_f32 v1, v2, v3, v0  // tc座標を計算する
v_cubesc_f32 v4, v2, v3, v0  // sc座標を計算する
v_cubema_f32 v5, v2, v3, v0  // 主軸[major axis]を計算する
v_cubeid_f32 v8, v2, v3, v0  // 面IDを計算する
v_rcp_f32 v2, abs(v5)
s_mov_b32 s0, 0x3fc00000
v_mad_f32 v7, v1, v2, s0  // 最終的な面UVを計算する
v_mad_f32 v6, v4, v2, s0  // 最終的な面UVを計算する
image_sample v[0:3], v[6:9], s[4:11], s[12:15]  // テクスチャ配列
```

# 特殊なALU操作: 主軸

```hlsl
// v_cubeid_f32の参考実装
float CubeMapFaceID(float inX, float inY, float inZ) {
    float3 v = float3(inX, inY, inZ);
    float faceID;

    if (abs(v.z) >= abs(v.x) && abs(v.z) >= abs(v.y)) {
        faceID = (v.z < 0.0) ? 5.0 : 4.0;
    } else if (abs(v.y) >= abs(v.x)) {
        faceID = (v.y < 0.0) ? 3.0 : 2.0;
    } else {
        faceID = (v.x < 0.0) ? 1.0 : 0.0;
    }
    return faceID;
}
```

# 特殊なALU操作: 主軸

- 主軸の問題でv_cubeid_f32、v_cubema_f32を使う
    - 法線圧縮
    - クオータニオン圧縮
    - キューブマップの境界での(ユニフォームな)独自のカーネルフィルタリング
    - アトラス化されたキューブマップ
    - キューブマップのレイマーチング最適化
    - レイキャスティングでのいくつかの問題

# 特殊なALU操作: 法線ストレージ精度

- 正規化されたベクトル
    - $1 = \sqrt[2]{x^2 + y^2 + z^2}$
- X、Yを格納して、Zを再構築する
    - $z = \sqrt[2]{1 - (x^2 + y^2)} = \sqrt[2]{1 - d}, d = x^2 + y^2$
- Zの精度は以下に依存する
    - $E(z) = dd * Er(x, y)$
        - ここで、$E(x)$はストレージと再構築の誤差関数
    - $\frac{d}{dd}(z) = \frac{d}{dd}(\sqrt{1 - d}) = -\frac{1}{2\sqrt{1 - d}}$
- 精度誤差は以下に起因する
    - $\lim_{d \to 1} -\frac{1}{2\sqrt{1 - d}} = \infty$
        - $d = x^2 + y^2 \to 1 \implies E(z) \to \infty$

# 特殊なALU操作: 法線ストレージ精度

- 誤差を最小化する一般的な方法
    - 境界で誤差関数を制限する
- $E(z)$を最小化するため、関数$d$を最小化する必要がある
- 単純な解決法
    - $d(x, y) = m^2 + n^2$
        - ここで、$m = \min(x,y,z), n = \text{med}(x,y,z), 1 = \sqrt[2]{x^2+y^2+z^2}$
- $d(x,y)$は以下よって上限付きになる: $\frac{2}{3}$
- $\lim_{d \to \frac{2}{3}} -\frac{1}{2\sqrt{1-d}} = -\frac{\sqrt{3}}{2}$

# 特殊なALU操作: 法線ストレージ精度

- $E(n, n') = 1 - n \cdot n'$
- 標準の再構築
    - 7ビットSNormのXとY + 1ビットの符号
    - X、Y領域上で$MSE(n, n') \approx \frac{3.04}{10000}$
        - ここで、$1 = \sqrt[2]{x^2+y^2+z^2}$
    - 役に立たない$n'$: $E(n,n') > \frac{1}{1024} \approx 5.4%$

# 特殊なALU操作: 法線ストレージ精度

- $E(n, n') = 1 - n \cdot n'$
- 主軸(x,y,zからm,nを最小化する)
    - 7ビットSNormのMとN + 2.5ビットの符号/order index
    - X、Y領域上で$MSE(n, n') \approx \frac{1.18}{10000}$
        - ここで、$1 = \sqrt[2]{x^2+y^2+z^2}$
    - 役に立たない$n'$: $E(n,n') > \frac{1}{1024} \approx 0.022%$

# 特殊なALU操作: 主軸

```hlsl
float3 PackNormalMajorAxis(float3 inNormal) {
    uint index = 2;
    if (abs(inNormal.x) >= abs(inNormal.y) && abs(inNormal.x) >= abs(inNormal.z)) {
        index = 0;
    } else if (abs(inNormal.y) > abs(inNormal.z)) {
        index = 1;
    }

    float3 normal = inNormal;
    normal = index == 0 ? normal.yzx : normal;
    normal = index == 1 ? normal.xzy : normal;

    float s = normal.z > 0.0 ? 1.0 : -1.0;
    float3 packedNormal;
    packedNormal.xy = normal.xy * s;
    packedNormal.z = index / 2.0f;
    return packedNormal;
}

// コンパイルすると:
// フルレートの28つのALU ＋ (16フルレート以上の)2つのBRANCH
```

# 特殊なALU操作: 主軸

```hlsl
float3 PackNormalMajorAxis(float3 inNormal) {
    uint index = CubeMapFaceID(inNormal.x, inNormal.y, inNormal.z) * 0.5f;

    float3 normal = inNormal;
    normal = index == 0 ? normal.yzx : normal;
    normal = index == 1 ? normal.xzy : normal;

    float s = normal.z > 0.0 ? 1.0 : -1.0;
    float3 packedNormal;
    packedNormal.xy = normal.xy * s;
    packedNormal.z = index / 2.0f;
    return packedNormal;
}

// コンパイルすると:
// フルレートの17つのALU
```

# Interpolator: 補間

- GCNでのVS->PSの補間は'手動'である
    - コンパイラによってアンロールされる
    - ハードウェアで最適化される
- LDSはラスタライズされるトライアングルあたりの頂点データが含まれる
- PSはデータをフェッチして、手動で補間する

# Interpolator: 補間

- P0、P1、P2
    - 頂点データを保持する
- Vi Vj
    - 重心座標
- 補間式セッティングに依存する
    - 補間する
        - 中心で、サンプルで、質量中心[centroid]
    - 補間なし
        - INT型でも強制される
        - V0(頂点0)からデータをフェッチする

```hlsl
float4 Interpolate(float4 A, float4 B, float4 C, float2 Vij) {
    return A *  (1.0 - Vij.x - Vij.y) + B * Vij.x + c * Vij.y;
}
```

# Interpolator: モード

```hlsl
struct Interpolants {
    float4 position : SV_POSITION;
    float4 color : COLOR0;
};

float4 main(Interpolants In) : COLOR {
    float4 Out;
    Out = In.color;
    return Out;
}
```

```gcn
v_interp_p1_f32 v2, v0, attr0.x  // LDSからAttr0のためにデータをロードして、Vi(補間の第1パート(V00, V01を使って))処理する
                                 //
v_interp_p2_f32 v2, v1, attr0.x  // LDSからAttr0のためにデータをロードして、Vj(補間の第2パート(V01、V10を用いて))を処理する
v_interp_p1_f32 v3, v0, attr0.y
v_interp_p2_f32 v3, v1, attr0.y
v_interp_p1_f32 v4, v0, attr0.z
v_interp_p2_f32 v4, v1, attr0.z
v_interp_p1_f32 v0, v0, attr0.w
v_interp_p2_f32 v0, v1, attr0.w
```

# Interpolator: モード

```hlsl
struct Interpolants {
    float4 position : SV_POSITION;
    float4 color : COLOR0;
};

float4 main(Interpolants In) : COLOR {
    float4 Out;
    Out = In.color;
    return Out;
}
```

```gcn
v_interp_mov_f32 v0, p0, attr0.x  // LDSからのAttr0に対して頂点p0からデータをロードする
v_interp_mov_f32 v1, p0, attr0.y  // LDSからのAttr0に対して頂点p0からデータをロードする
v_interp_mov_f32 v2, p0, attr0.z  // LDSからのAttr0に対して頂点p0からデータをロードする
v_interp_mov_f32 v3, p0, attr0.w  // LDSからのAttr0に対して頂点p0からデータをロードする
```

# 特殊なALU操作: Interpolator圧縮

- GCNはハードウェアラスタライザーのVi、Vj(重心座標)をpollすることができる
    - セットされたInterpolatorフラグに応じて計算される
- 独自の補間とパッキングの可能性が開ける
- ジオメトリシェーダとテッセレーションパイプライン --- データ増幅
    - 大きな帯域幅が必要
    - 帯域幅を最適化するためにinterpolator圧縮を使う
- PSはLDSによってボトルネックともなり得る
    - 'fat'な頂点データに対して多すぎるLDSを用いる
- トライアングル定数データを一切補間しない！

# Interpolator: パッキング

- 頂点データを読む
    - v_interp_mov_f32 v0, p0, attr0.x  // 頂点P00
    - v_interp_mov_f32 v0, p10, attr0.x  // 頂点P10
    - v_interp_mov_f32 v0, p20, attr0.x  // 頂点P20
- 重心座標Vi Vj
    - VGPRにプリロードされる(コンパイラがよしなにしてくれる)

# Interpolator: パッキング

```hlsl
float4 Interpolate(float4 A, float4 B, float4 C, floatt3 barycentric) {
    return A * barycentric.z + B * barycentric.x + C * barycentric.y;
}
```


```hlsl
float3 barycentric;
barycentric.xy = GetBarycentricCoordsPerspectiveCenter();  // ハードウェアからVi Vjを読む
barycentric.z = 1 - barycentric.x - barycentric.y;

uint rawA = (GetVertexParameterP0(In.color_packed));  // V00から生のUINTデータを読む
uint rawB = (GetVertexParameterP1(In.color_packed));  // V01から生のUINTデータを読む
uint rawC = (GetVertexParameterP2(In.color_packed));  // V10から生のUINTデータを読む

float4 decompressedA = UnpackColor(rawA);  // UINTからByteをアンパックして、floatに変換する
float4 decompressedB = UnpackColor(rawB);  // UINTからByteをアンパックして、floatに変換する
float4 decompressedC = UnpackColor(rawC);  // UINTからByteをアンパックして、floatに変換する

float4 Out;
Out = Interpolate(decompressedA, decompressedB, decompressedC, barycentric);
```

# PSのLDSアクセス: トライアングルデータ

- PSはラスタライズされたトライアングルから頂点データを直接読むことができる
- 以前はGSのものだった[reserved for]複数のアルゴリズムがPSでできる
    - 視差曲率の推定[Parallax Curvature estimation]
    - エッジへの(最も近い)距離
    - 頂点への(最も近い)距離
    - スプライン補間された法線/曲率

# PSのLDSアクセス: トライアングルデータ

# PSのLDSアクセス: トライアングルデータ

# PSのLDSアクセス: エッジへの距離

- 例: Distance to Edge AA
    - 最も近いエッジへの距離を出力する
    - GSを迂回してPSから直接的に
    - 複数の解析的AA手法で使われる
        - GBAA
        - DEAA

# PSのLDSアクセス: エッジへの距離

# PSのLDSアクセス: エッジへの距離

- すべてのメッシュでの高価なGSにより以前は実践的ではなかった
- 現在はすっかり実現可能な選択肢である
    - 素晴らしいパフォーマンス
- HUMUSのGBAAをチェックしてね
    - ジオメトリシェーダの部分をピクセルシェーダに移しただけ

# 超越関数

- rcp(x)、sqrt(x)、rsqrt(x)
    - レンダリングにおいて最も一般的な超越関数
    - クオーターレート --- 各16サイクル
    - ループでは一般的
        - ライトの反復
        - SSAO
        - マルチサンプリング
        - レイマーチング
    - マクロで使われる
        - Length(x)
        - Normalize(x)

# 超越関数: 例

```hlsl
// 低速なコード --- いくつかのコンパイラはマクロを最適化するほどアグレッシブではない
float3 vector;
float vectorLength = length(vector);  // コンパイラの最良ケース: sqrt(dot(vector, vector))を展開する
float3 normalVector = normalize(vector);  // コンパイラの最良ケース: vector * rsqrt(dot(vector, vector))を展開する
```

```gcn
// タイミング: (FR --- フルレートサイクル --- 4サイクル)
v_mov_b32     v0, s2           // 1FR
v_mul_f32     v1, s2, v0       // 1FR
v_mov_b32     v2, s1           // 1FR
v_mac_f32     v1, s1, v2       // 1FR
v_mov_b32     v3, s0           // 1FR
v_mac_f32     v1, s0, v3       // 1FR
v_sqrt_f32    v1, v1           // 4FR
v_mul_f32     v0, s2, v0       // 1FR
v_mac_f32     v0, s1, v2       // 1FR
v_mac_f32     v0, s0, v3       // 1FR
v_rsq_f32     v0, v0           // 4FR
v_mov_b32     v2, #0x7f7fffff  // 1FR
v_mov_b32     s3, #0xff7fffff  // 1FR
v_med3_f32    v0, v0, s3, v2   // 1FR
v_mul_f32     v2, s0, v0       // 1FR
v_mul_f32     v3, s1, v0       // 1FR
v_mul_f32     v0, s2, v0       // 1FR
// MOVを含めない合計               // 18FR
```

# 超越関数: 例

```hlsl
// 手動でマクロをアンロールすることによりコンパイラを補助する
// これは常に良いプラクティスである
float dotVector = dot(inVector, inVector);
float vectorLength = sqrt(dotVector);
float3 normalVector = inVector * rcp(vectorLength);
```

```gcn
// タイミング : (FR - フルレートサイクル - 4サイクル):
v_mov_b32     v0, s2       // 1 FR
v_mul_f32     v0, s2, v0   // 1 FR
v_mov_b32     v1, s1       // 1 FR
v_mac_f32     v0, s1, v1   // 1 FR
v_mov_b32     v1, s0       // 1 FR
v_mac_f32     v0, s0, v1   // 1 FR
v_rsq_f32     v1, v0       // 4 FR
v_sqrt_f32    v0, v0       // 4 FR
v_mul_f32     v2, s0, v1   // 1 FR
v_mul_f32     v3, s1, v1   // 1 FR
v_mul_f32     v1, s2, v1   // 1 FR
// MOVを含めない合計           //14 FR
```

# 超越関数: 例

```hlsl
// FR数と悪用[exploiting]によるパイプライン化でより良くできる
// sqrt(x) = rsqrt(x) * x
// rcp(x) = rsqrt(x) * rsqrt(x)  // Xが正のときのみ
float dotVector = dot(inVector, inVector);
float rcpVectorLength = rsqrt(dotVector);
float vectorLength = rcpVectorLength * dotVector;
float3 normalVector = inVector * rcpVectorLength;
```

```gcn
// この結果は:
v_mov_b32     v0, s2       // 1 FR
v_mul_f32     v0, s2, v0   // 1 FR
v_mov_b32     v1, s1       // 1 FR
v_mac_f32     v0, s1, v1   // 1 FR
v_mov_b32     v1, s0       // 1 FR
v_mac_f32     v0, s0, v1   // 1 FR
v_rsq_f32     v1, v0       // 4 FR
v_mul_f32     v0, v0, v1   // 1 FR
v_mul_f32     v2, s0, v1   // 1 FR
v_mul_f32     v3, s1, v1   // 1 FR
v_mul_f32     v1, s2, v1   // 1 FR
// MOVを含めない合計           //11 FR
```

# 近似的な超越関数

- HWでの超越関数は約1の精度の最小桁単位[ULP]をもたらす
- 我々は常にそこまで必要としていない
    - 特にF16、F11、UNorm8データでは
- クオーターレートのハードウェアより良い処理ができるか？

# 特殊なALU操作: 整数の計算

- 汎用用途レジスタ
    - 整数の計算
    - 再翻訳コストがない
- 整数のサポート
    - 整数ベースの浮動小数点数の計算を許可する

```c++
// asint() / asfloat()はreinterpret_castとして動作する
// タダ --- 異なる命令セットを用いるデータを扱うためにコンパイラにヒントを与えるだけ

#define asint(_x) *reinterpret_cast<int*>(&_x);
#define asfloat(_x) *reinterpret_cast<float*>(&_x);
```

# 0x5f3759df WTF?

- 高速な平方根の逆数
    - 整数の計算を用いてSGIで実装された
    - Quake 3のソースコードのために有名

```c
float Q_rsqrt(float number) {
    long i;
    float x2, y;
    const float threehalfs = 1.5f;

    x2 = number * 0.5f;
    y = number;
    i = *(long*)&y;                       // floatからintへの再翻訳
    i = 0x5f3759df - (i >> 1);            // WTF?
    y = *(float*)&i;                      // intからfloatへの再翻訳
    y = y * (threehalfs - (x2 * y * y));  // ニュートン・ラフソン法の1回目

    return y;
}
```

# 0x5f3759df WTF?

- 高速な平方根の逆数
    - 浮動小数点数の二進表現により動作する
- 速度を大切にする
    - ニュートン・ラフソン法を取り除く
- GCNで速くなるだろうか？
    - rsqrt()より2倍速い

```hlsl
int x = asint(inX);
x = 0x5f3759df - (x >> 1);
return asfloat(x);
```

```gcn
v_ashr_i32 v0, v0, 1
v_sub_i32 v0, # 0x5f3759df, v0
```

# More Magic!

- Using original idea derive
    - $x^n \approx qpow(x,n) = K+n(asInt(x) - K), n:[-1, 1]$
    - $K$は定数
- $E(x,n)=|x^n - qpow(x,n)|$
- $(x,n)$領域上で$E(x,n)$を最小化するために$K$を探す
- $E(K) = \sum_x E(x,n)$、$n$は定数 --- 停留点[stationary point]を持つ
- asInt(x)は対数関数に近い
    - $E(K)$は与えられた$x$領域と$n$に対して最小値[global minima]を持つ

# More Magic!

- gradient binary searchの使用
    - 以下に対する最適なKを見つける
        - n
        - x --- 領域
    - すべてに対して理にかなうKを見つけることができる
- 誤差関数を最小化するために推奨される特殊化
    - sqrt()、rsqrt()、rcp()に対する最適なKを見つける
    - 領域を制限する
        - つまり、カメラ空間での距離計算用 --- ファー面で上限を設ける

# 0x5f3759dfのrsqrt()に打ち勝とう

- 0x5f3759df --- rsqrt()に対する普遍的なKとして見つかった
- 我々の領域はx(0,1000)に制限される
- %でのRMSE(K): x(0, 1000), n = -1/2

# 0x5f3759dfのrsqrt()に打ち勝とう

- E(0x5f33aa52), x(0, 1000), n = -1/2

# 高速なシェーダライブラリ

```hlsl
// 2フルレート
float rcpSqrtIEEEIntApproximation(float inX, const int inRcpSqrtConst) {
    int x = asint(inX);
    x = inRcpSqrtConst - (x >> 1);
    return asfloat(x);
}

// 2フルレート
float sqrtIEEEIntApproximation(float inX, const int inSqrtConst) {
    int x = asint(inX);
    x = inSqrtConst + (x >> 1);
    return asfloat(x);
}

// 1フルレート
float rcpIEEEIntApproximation(float inX, const int inRcpConst) {
    int x = asint(inX);
    x = inRcpConst - x;
    return asfloat(x);
}
```

# ユースケース: Kのやつの例

- rsqrt()
    - 0x5f341a43 RME:1.72% (0.0, 1.0)
    - 0x5f33e79f RME:1.62% (0.0, 1000.0)
- sqrt()
    - 0x1FBD1DF5 RME:1.42% (0.0, 1.0)
    - 0x1FBD22DF RME:1.44% (0.0, 1000.0)
- rcp()
    - 0x7EEF370B RME:2.92% (0.0, 1.0)
    - 0x7EF3210C RME:3.20% (0.0, 1000.0)

# ユースケース: SSAO/バイラテラルフィルタ

- SSAO
    - Distance() sqrt()
    - Normalize() rsqrt()
- バイラテラルフィルタ
    - Divide() rcp()
    - Normalize() rsqrt()
- すべて高速シェーダライブラリに切り替えると
    - コンソールで13%の合計時間の改善
    - 見た目に差はない

#

#

#

# クリエイティブなコードNinjaになろう！

- GPUはCPUに相当近い
- SPU/CPUのNinjaスキルを使おう！
- 古の事物に驚かされることもある
- 帯域幅のためにALUを差し出す
- 氷山の一角
    - スケジューリング
    - 非同期コンピュート
    - レイテンシー隠蔽
    - キャッシング
    - 我々のまだ知らないたくさんのこと
    - fun ahead!

# 質疑応答

- 更なるTips&Tricks:
- MICHALDROBOT.COM
- TWITTERで宜しく
- \@MichalDrobot
- emailはこちら
- HELLO@DROBOT.ORG

# 参考文献

- GCN
    - "Low-level Shader Optimization for Next-Gen and DX11" --- Emil Persson
    - "The AMD GCN Architecture: A Crash Course" --- Layla Mah
    - "GCN - Two ways of latency hiding and wave occupancy" --- Bart  Wronski
    - "Compute Shader Optimizations for AMD GPUs: Parallel Reduction" --- Wolfgang Engel
    - GCN Performance Tweets
- Inverse Sqrt
    - "Fast inverse Square Root" --- Chris Lomont
    - "The Mathematics Behind the Fast Inverse Square Root Function Code" --- Charles McEniry
    - Quake 3 Source Code --- github.com/id-Software/Quake-III-Arena

# 謝辞

- Ubisoftの3Dチーム
- 特に:
    - Bart Wronski
    - Jeremy Moore
    - Steve McAuley
    - Stephen Hill
- AMDのDeveloper Relationチーム
- 特に:
    - Layla Mah
    - Chris Brennan

# ボーナススライド

# IEEEパフォーマンスモード

- IEEE準拠[compliance] (-fastmath)を無効化する
    - IEEE strictとも
    - コンパイラは以下を扱わないだろう
        - 非正規数
        - QNaN
        - 0除算
        - 他の安全でないケース
    - 近似的な超越関数を使うだろう
        - cleanupまたはaccuracy操作なしで
        - 精度は変化するが約1ULPであることが保証される(IEEEは0.5ULPを必要とする)

# IEEE Strict 対 非Strict: X/Y

```hlsl
float r = inV.x / inV.y;
```

```gcn
// IEEE strictなし
// xはv1、yはv2
v_rcp_f32 v0, v1  // 安全でないrcp()はNaNが生成され得る
v_mul_f32 v0, v2, v0
```

```gcn
// IEEE strict安全な-fastmath
// xはv1、yはv2
v_rcp_f32 v0, v1  // 安全でないrcp()はNaNが生成され得る
v_mov_b32 v1, #0x7f7fffff  // MAX_FLT
v_mov_b32 s1, #0xff7fffff  // MIN_FLT
v_med3_f32 v0, v0, s1, v1  // NaNを消去するための安全なクランピング
v_mul_legacy_f32 v0, v2, v0
```

# IEEE Strict 対 非Strict: X/Y

```hlsl
float r = inV.x / inV.y;
```

```gcn
// IEEE strict safe accurateは非正規数を洗い流す
// xはv1、yはv2
v_rcp_f32 v0, v1
v_mul_f32 v0, v0, v2
v_div_fixup_f32 v3, v0, v1, v2  // -/+INF NaN QNaNを直す
```

# IEEE Strict 対 非Strict: X/Y

```hlsl
float r = inV.x / inV.y;
```

```gcn
// IEEE strict safe accurateは非正規数に対応する
// 丸めモードや非正規数の出力に応じて
// コンパイラは以下を追加できる
v_rcp_f32
v_mul_f32
v_div_scale_f32
nop
nop
nop
nop
v_div_fmas_f32
```
