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

TODO
