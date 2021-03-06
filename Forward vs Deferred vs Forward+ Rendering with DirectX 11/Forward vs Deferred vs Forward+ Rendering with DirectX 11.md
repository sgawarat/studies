# Forward vs Deferred vs Forward+ Rendering with DirectX 11

https://www.3dgep.com/forward-plus/

## Introduction

### Forward Rendering

Forward Renderingは、ジオメトリオブジェクトごとにラスタライズすることでシーンを描画する手法である。
この手法では、ジオメトリオブジェクトを1つ描画するごとに、そのオブジェクトのライティング処理に必要なすべての光源を列挙する必要があるため、取り扱う数が大きくなるとその変化に対する計算量の変化量が大きくなる傾向がある。そのため、計算負荷を抑える目的で計算対象となる光源数に上限を設けることが多い。

### Deferred Rendering

Deferred Renderingは、ジオメトリ情報を格納した2D画像バッファ(G-buffer)を用いてスクリーン空間でライティング計算を行う手法である。ジオメトリ処理とライティング処理が分離しているため、オブジェクト数の増加に対する計算負荷の増加が比較的小さくなる。
G-bufferは、1ピクセルが32ビットのフルHD(1920x1080)サイズのテクスチャを用いるとすると、1枚につき約8MBになる。通常のG-bufferは4枚程度で構成されることが多いので、30から40MBのメモリ消費を見越しておかなければならない。
Deferred Renderingは不透明オブジェクトしか扱えないという欠点がある。これはG-bufferが1ピクセルに1つの情報しか保存できないために、透明オブジェクトを介して見るオブジェクトが正しくライティングされなくなるためである。この問題は透明オブジェクトを通常のforward renderingで描画することで回避する。
また、ライティングパスが基本的には単一のピクセルシェーダで動作するという都合上、ライティングモデルを動的に切り替えるシステムとうまく噛み合わない場合がある。

### Forward+

Forward+ (Tiled Forward Shading)は、考慮しなければならない光源数を削減するTiled Light CullingとForward Renderingを組み合わせた手法である。はじめに、スクリーン空間を等間隔のタイルに分割して、そのタイルごとに計算を行うであろう光源のリストを作成する。そして、シェーディングを行うピクセルを含むタイルの光源リストを参照してForward Renderingを行う。
これにより、計算する必要のない光源を大幅に取り除くことができ、パフォーマンスが飛躍的に向上する。また、本質的にはForward Renderingなので、不透明と透明の両方を同じ処理系で扱うことができるし、複数のマテリアルやライティングモデルを取り扱うこともできる。

## Definitions

以下の定義は基本的に一般的な解釈に基づくが、この記事中でのみ有効なニュアンスを含むことがあることに注意する。

シーン(scene)
: レンダリングされるオブジェクト群。
  シーングラフ(scene graph)と呼ばれるノードによる階層構造を構成することが多い。

パス(pass)
: レンダリングテクニックを構成する操作(operation)の基本単位。
  例えば、Deferred Renderingではジオメトリ情報を書き込むジオメトリパスとライティングを計算するライティングパスがある。

テクニック(technique)
: (主として、レンダリングアルゴリズムを実装するための)パスの組み合わせ。

パイプラインステート(pipeline state)
: レンダリングパイプラインの構成要素をまとめたもの。シェーダ、ラスタライザステート、ブレンドステート、デプスステンシルステート、レンダターゲットを含む。
  DirectX12のものとは若干異なる。

フォワードレンダリング(Forward Rendering)
: 伝統的なレンダリングテクニック。不透明(opaque)パス、透明(transparent)パスの2パスから成る。

遅延レンダリング(Deferred Rendering)
: 不透明オブジェクトをG-bufferに書き込むジオメトリパス、スクリーン空間でライティングを計算するライティングパスの2パスとForward Renderingと同じ方法で行う透明パスから成る。

Forward+ または Tiled Forward Rendering
: スクリーン空間のタイルごとに光源のリストを作成するライトカリング(light culling)パス、不透明パス、透明パスの3パスから成る。
  光源リストは不透明オブジェクト用と透明オブジェクト用にそれぞれ作られる。光源はその形を指定でき、Point、Spot、Directionalなどがある。

減衰(attenuation)
: ポイントライトやスポットライトで光源との距離に応じてその光量が減少すること。

## Forward Rendering

Forward Renderingは今回取り上げる3つの中では一番シンプルであり、ゲームなどのリアルタイム系でよく使われているテクニックである。このテクニックではライティング計算は比較的高価であるため、大量の動的な光源を扱えなくしてある場合が多い。一方で、光源が静的ならば、lightmappingやlight probesによりその寄与を事前計算することができるため、大量に取り扱うことができる。

本実験では、Forward Renderingは他のテクニックの比較を行うためのground truth[^ground_truth]として用いる。また、他のテクニックのパフォーマンスを比較するための基準値としても用いる。

[^ground_truth]: 実際のシステムを直接観察することで得た情報。計算機科学界隈では、アルゴリズムの特性を調べるために用意された正解データを指す。

Forward Renderingで用いられた多くの機能(functions)はDeferred RenderingやForward+でも引き続き利用されている。たとえば、頂点シェーダはほぼそのまま用いられるし、ライティングやシェーディングの計算方法はあらゆるレンダリングテクニックで再利用されている。

### Vertex Shader

頂点シェーダは色んなレンダリングテクニックで共通している。本実験では、静的なジオメトリのみを扱い、異なる頂点シェーダが必要になる骨格(skeletal)アニメーションや地形(terrain)は扱わない。

#### 入力データ

```hlsl
struct VSInput {
    float3 position : POSITION;
    float3 tangent : TANGENT;
    float3 binormal : BONORMAL;
    float3 normal : NORMAL;
    float2 texcood : TEXCOORD;
};
```

`VSInput`はアプリケーションから頂点シェーダへ送られるデータの構造を定義する。それぞれ頂点に対して、`position`は位置、`normal`は法線、`tangent`は接線、`binormal`従法線、`texcood`はテクスチャ座標が格納される。従法線は法線と接線の外積として求められるため、通常は必要はない。

#### 変換行列

```hlsl
cbuffer PerObject : register(b0) {
    float4x4 MVP;
    float4x4 MV;
}
```

ベクトルはその基準点を決める座標系に属している。例えば、アプリケーションから送られる頂点の位置ベクトルは、オブジェクト独自の基準点を採用したobject空間に属する。一方で、ラスタライザはclip空間における頂点の位置ベクトルを要求するため、頂点シェーダは頂点の位置ベクトルをobject空間からclip空間へ変換しなければならない。この変換は事前計された行列として与えられ、定数バッファなどに格納される。

#### 出力データ

```hlsl
struct VSOutput {
    float4 position : SV_Position;
    float3 texcood : TEXCOORD;
    float3 position_v : POSITION_V;
    float3 tangent_v : TANGENT;
    float3 binormal_v : BINORMAL;
    float3 normal_v : NORMAL;
};
```

`VertexShaderOutput`は頂点シェーダの出力及び後段のシェーダの入力に対応したデータ構造である。接尾の`v`はそのベクトルがview空間に属することを示している。
頂点シェーダの出力としての`SV_Position`セマンティックは、clip空間における頂点の位置を示す4次元ベクトルを表す。

#### シェーダコード

```hlsl
VSOutput vsmain(VSInput input) {
    VSOutput output;
    output.position = mul(MVP, float4(input.position, 1.f));
    output.texcoord = input.texcoord;
    output.position_v = mul(MV, float4(input.position, 1.f)).xyz;
    output.tangent_v = mul((float3x3)MV, input.tangent);
    output.binormal_v = mul((float3x3)MV, input.binormal);
    output.normal_v = mul((float3x3)MV, input.normal);
    return output;
}
```

頂点シェーダでは、基本的には入力データを変換するのみを行う。

### Pixel Shader

ピクセルシェーダでは、ピクセル1つに対するすべてのライティング及びシェーディング計算を行う。

#### Material

```hlsl
struct Material {
    float4 global_ambient; // すべてのオブジェクトに適用される環境光の寄与
    float4 ambient_color; // 環境色
    float4 emissive_color; // 発光色
    float4 diffuse_color; // 拡散色
    float4 specular_color; // 鏡面色
    float4 reflectance; // 環境マッピングで用いる反射率
    float opacity; // 不透明度
    float specular_power; // 鏡面反射の強さ。shininess
    float index_of_refraction; // 環境マッピングで用いる屈折率
    bool has_ambient_texture;
    bool has_emissive_texture;
    bool has_diffuse_texture;
    bool has_specular_texture;
    bool has_specular_power_texture;
    bool has_normal_texture;
    bool has_bump_texture;
    bool has_opacity_texture;
    float bump_intensity; // バンプマップの高さの倍率
    float specular_scale; // テクスチャから読み取ったspecular_power値の倍率
    float alpha_threshold; // ピクセルを破棄(discard)するアルファ値のしきい値
    float2 _padding;
}

cbuffer Material : register(b2) {
    Material MATERIAL;
}
```

`Material`はシェーディングを行うために必要なデータの構造を定義する。いくつかの値はテクスチャとして持つこともできるため、その切り替えフラグも一緒に格納してある。

#### Textures

テクスチャはマテリアルに付随するため、各テクスチャを使うかどうかはモデルを作るアーティストの判断に委ねられる。

```hlsl
Texture2D AMBIENT_TEXTURE : register(t0);
Texture2D EMISSIVE_TEXTURE : register(t1);
Texture2D DIFFUSE_TEXTURE : register(t2);
Texture2D SPECULAR_TEXTURE : register(t3);
Texture2D SPECULAR_POWER_TEXTURE : register(t4);
Texture2D NORMAL_TEXTURE : register(t5);
Texture2D BUMP_TEXTURE : register(t6);
Texture2D OPACITY_TEXTURE : register(t7);
```

#### Lights

```hlsl
struct Light {
    float4 position_w; // world空間における位置
    float4 direction_w; // world空間における方向
    float4 position_v; // view空間における位置
    float4 direction_v; // view空間における方向
    float4 color; // 拡散光の色及び鏡面光の色
    float spotlight_angle; // スポットライトの照射角度 * 0.5
    float range; // 発する光が届く最大距離
    float intensity; // 光源の強さ
    bool enabled; // 有効か
    bool selected; // エディタが選択しているか
    uint type; // 光源の形
    float2 _padding;
}

StructuredBuffer<Light> LIGHTS : registerr(t8);
```

`Light`はライティングを行うために必要なデータの構造を定義する。ここでは、スポットライト、ポイントライト、ディレクショナルライトの3つを別構造に分けずに1つの構造で管理する。
ライトタイプによっては必要のないパラメータもあるが、ここでは取り回しの良さを優先してすべて用意する。
通常のライトモデルでは拡散色と鏡面色の2つの色を持つが、ここでは簡略化のため統合している。

ここでは`Light`の配列をStructuredBufferとして渡している。StructuredBufferは定数バッファと比べてサイズ制限がゆるく、より多くのライトを取り扱うことができる。

#### シェーダコード

```hlsl
[earlydepthstencil]
float4 psmain(VSOutput input) : SV_Target {
    Material material = MATERIAL;

    // 拡散色
    float4 diffuse = material.diffuse_color;
    if (material.has_diffuse_texture) {
        float4 diffuse_tex = DIFFUSE_TEXTURE.Sample(SAMPLER, input.texcoord);
        if (any(diffuse.rgb)) {
            // materialのdiffuse_colorが0でなければ、テクスチャの値とブレンドする。
            diffuse *= diffuse_tex;
        } else {
            // materialのdiffuse_colorが0であれば、テクスチャの値に差し替える。
            diffuse = diffuse_tex;
        }
    }

    // 不透明度(アルファ値)
    float alpha = diffuse.a;
    if (material.has_opacity_texture) {
        // テクスチャを持っているならば、テクスチャの値に差し替える。
        alpha = OPACITY_TEXTURE.Sample(SAMPLER, input.texcoord).r;
    }

    // 環境色
    float4 ambient = material.ambient_color;
    if (material.has_ambient_texture) {
        float4 ambient_tex = AMBIENT_TEXTURE.Sample(SAMPLER, input.texcoord);
        if (any(ambient.rgb)) {
            // materialのambient_colorが0でなければ、テクスチャの値とブレンドする。
            ambient *= ambient_tex;
        } else {
            // ambient_colorが0であれば、テクスチャの値に差し替える。
            ambient = ambient_tex;
        }
    }
    ambient *= material.global_ambient;

    // 発光色
    float4 emissive = material.emissive_color;
    if (material.has_emissive_texture) {
        float4 emissive_tex = EMISSIVE_TEXTURE.Sample(SAMPLER, input.texcoord);
        if (any(emissive.rgb)) {
            // materialのemissive_colorが0でなければ、テクスチャの値とブレンドする。
            emissive *= emissive_tex;
        } else {
            // emissive_colorが0であれば、テクスチャの値に差し替える。
            emissive = emissive_tex;
        }
    }

    // Specular Power
    float specular_power;
    if (material.has_specular_power_texture) {
        specular_power = SPECULAR_POWER_TEXTURE.Sample(SAMPLER, input.texcoord).r * material .specular_scale;
    } else {
        specular_power = material.specular_power;
    }

    // 法線
    float4 normal;
    if (material.has_normal_texture) {
        // 法線マッピング
        // [0, 1]に圧縮して保存されていたtangent空間に属する法線を[-1, +1]に伸長する。
        float3 tex = NORMAL_TEXTURE.Sample(SAMPLER, input.texcoord).rgb;
        float3 normal_t = 2.f * tex - 1.f;

        // 法線をtangent空間からview空間へ変換する
        float3x3 tbn = float3x3(normalize(input.tanget_v),
                                normalize(input.binormal_v),
                                normalize(input.normal_v));
        normal = normalize(float4(mul(normal_t, tbn), 0.f));
    } else if (material.has_bump_texture) {
        // バンプマッピング
        // 高さの勾配からtangent空間の法線を求める
        float3 height = BUMP_TEXTURE.Sample(SAMPLER, input.texcoord).r;
        float3 height_u = BUMP_TEXTURE.Sample(SAMPLER, input.texcoord, int2(1, 0)).r;
        float3 height_v = BUMP_TEXTURE.Sample(SAMPLER, input.texcoord, int2(0, 1)).r;
        float3 p = {0.f, 0.f, height};
        float3 pu = {1.f, 0.f, height_u};
        float3 pv = {0.f, 1.f, height_v};
        float3 normal_t = cross(normalize(pu - p), normalize(pv - p));

        // 法線をtangent空間からview空間へ変換する
        float3x3 tbn = float3x3(normalize(input.tanget_v),
                                -normalize(input.binormal_v),
                                normalize(input.normal_v));
        normal = normalize(float4(mul(normal_t, tbn), 0.f));
    } else {
        // モデルの法線をそのまま利用する
        normal = normalize(float4(input.normal_v, 0.f));
    }

    // ライティング
    float4 eye = {0.f, 0.f, 0.f, 1.f}; // view空間でのカメラ位置
    float4 light_diffuse = 0.f;
    float4 light_specular = 0.f;
    for (int i = 0; i < NUM_LIGHTS; ++i) {
        Light light = LIGHTS[i];
        if (!light.enabled) continue;
        switch (light.type) {
            case DIRECTIONAL_LIGHT: {
                float4 l = normalize(-light.direction_v); // 光源方向
                float4 v = normalize(eye - input.positon_v); // 視線方向
                float4 r = normalize(reflect(-l, n)); // 反射方向
                light_diffuse += light.intensity * CalcDiffuse(...);
                light_specular += light.intensity * CalcSpecular(...);
                break;
            }
            case POINT_LIGHT: {
                // 範囲外なら計算を省略する
                if (length(light.position_v - input.position_v) > light.range) continue;

                float4 l = normalize(light.position_v - input.position_v); // 光源方向
                float4 v = normalize(eye - input.positon_v); // 視線方向
                float4 r = normalize(reflect(-l, n)); // 反射方向
                float attn = CalcAttenuation(...); // 減衰係数
                light_diffuse += light.intensity * attn * CalcDiffuse(...);
                light_specular += light.intensity * attn * CalcSpecular(...);
                break;
            }
            case SPOT_LIGHT: {
                // 範囲外なら計算を省略する
                if (length(light.position_v - input.position_v) > light.range) continue;

                float4 l = normalize(light.position_v - input.position_v); // 光源方向
                float4 v = normalize(eye - input.positon_v); // 視線方向
                float4 r = normalize(reflect(-l, n)); // 反射方向
                float attn = CalcAttenuation(...); // 減衰係数
                float spot_intensity = CalcSpotCone(...); // 角度による強度
                light_diffuse += light.intensity * attn * spot_intensity * CalcDiffuse(...);
                light_specular += light.intensity * attn * spot_intensity * CalcSpecular(...);
                break;
            }
        }
    }

    // 鏡面色
    float4 specular = 0.f;
    if (spucular_power > 1.f) { // spucular_powerが小さいときは鏡面色を使わない
        specular = material.specular_color;
        if (material.has_specular_texture) {
            float4 specular_tex = SPECULAR_TEXTURE.Sample(SAMPLER, input.texcoord);
            if (any(specular.rgb)) {
                // materialのspecular_colorが0でなければ、テクスチャの値とブレンドする。
                sperular *= specular_tex;
            } else {
                // materialのspecular_colorが0であれば、テクスチャの値に差し替える。
                sperular = sperular_tex;
            }
        }
    }

    // 合成
    return float4(
            (ambient + emissive +
                (diffuse * light_diffuse) +
                (specular * light_specular)).rgb,
            alpha * material.opacity);
}
```

`earlydepthstencil`アトリビュートは、ピクセルシェーダを起動する前に深度ステンシルテストを行うことを明示する。これを有効にすると、不合格ピクセルの計算処理を省略できるので、パフォーマンスが改善することがあるが、ピクセルシェーダで深度値を変更できなくなる。

##### Normal Mapping

法線マッピングは、法線マップを用いてモデルに細かなディテールを付与する技法である。法線マップにはtangent空間に属する法線ベクトルが格納されている。

tangent空間からview空間への変換行列`tbn`は以下のように考えられる。
前提として、tangent空間は接線、従法線、法線がそれぞれXYZ座標軸となる直交座標系である。view空間も直交座標系である。view空間とtangent空間はともに拡大縮小を含まない。法線は平行移動を考慮しない。
つまり、tangent空間からview空間への法線の変換は座標系を回転させる変換行列として求められる。したがって、回転行列の逆行列はその転置と同等であるため、`tbn`はview空間からtangent空間へ座標軸を回転させる行列の転置として求められる。

#### Bump Mapping

バンプマッピングは、高さマップ(height map)に格納された高さ情報をもとに計算された法線を用いて、モデルに細かなディテールを付与する技法である。高さの勾配からtangent空間の法線が求まるため、法線マッピングと同様に、view空間へ変換しなければならない。

#### Lighting

`CalcDiffuse(...)`および`CalcSpecular(...)`は光源の寄与を計算する。
`CalcAttenuation(...)`は光源との距離から光量の減衰率を計算する。
`CalcSpotCone(...)`は照射角度を定義するための光源強度の減衰率を計算する。

## Deferred Rendering

Deferred RenderingはGバッファパス、ライティングぱすの2パスと透明パスからなる。

### G-buffer pass

Gバッファパスでは、Gバッファとなるテクスチャにジオメトリ情報をレンダリングする。
レンダリングする情報は、深度ステンシル、拡散色、鏡面色、法線などがあるが、その量はパフォーマンスに直結するため、パッキングするなりしてできる限り切り詰めたほうが良い。

#### G-buffer layout

深度ステンシルは通常のレンダリングパイプラインで書き込みを行う。

Light accumulationバッファはライティングパスが最終結果を格納するバッファである。本実験ではGバッファに含めている。

拡散色と鏡面色はライティングされていないマテリアルの色をそのまま受け渡す。Deferred Renderingで扱うオブジェクトはすべて不透明であるため、アルファ値は常に1であると仮定できる。そうすると、本来アルファ値を格納するスペースに別の値を格納することもできる。

法線をview空間で扱う場合、法線のz値が正負一方しか現れないことを利用して、Gバッファパスではxy要素のみを格納してライティングパスで外積によりz要素を復元するという手段を取ることができる。[^view_space_normal]

[^view_space_normal]: ベクトル計算は属する空間によって若干の利点欠点が存在する。
view空間で行う場合、その座標系が視点に依存するため、背面が必要ないならば法線のz要素はxy要素から復元できるが、視点が頻繁に更新されることで、座標系を揃えるための実行時計算を必要とする。
一方、world空間で行う場合、法線のz要素を省略できない代わりに、その座標系が変化しないことでworld空間のベクトルデータを事前計算できる。

#### Pixel Shader

```hlsl
struct PSOutput {
    float4 light_accumulation : SV_Target0;
    float4 diffuse : SV_Target1;
    float4 specular : SV_Target2;
    float4 normal_v : SV_Target3;
}

PSOutput ps_geometry(VSOutput input) {
    PSOutput output;

    // ...

    output.light_accumulation = (ambient + emissive);
    output.diffuse = diffuse;
    output.specular = float4(specular.rgb, log2(specular_power) / 10.5f);
    output.normal_v = normal;
}
```

Gバッファパスのピクセルシェーダは、ライティングを行わないこと以外はForward Renderingと同じように計算を行い、Gバッファに格納する要素を出力する。

### Lighting pass (Guerrilla)

まずは、Guerrilla GamesのKillzone2が採用しているDeferred Renderingを見てゆく。Michiel van der Leeuwの発表によればライティングパスは以下の3工程から成る。

1. ステンシルバッファを0クリアする
2. 最も遠いライトの境界(boundary)より手前にあるピクセルに印をつける
3. ライトボリュームの内側にある照らされているピクセル(lit pixels)を数える
4. 照らされているピクセルの陰影付けを行う

#### ピクセルの印付け

ライトボリュームの背面より手前にあるピクセルに対して、対応するステンシルバッファの値を1にする。このときのパイプラインステートは以下の構成に従う。

- 頂点シェーダのみバインドする。
    - ピクセルシェーダをバインドせず、すべてのピクセルを破棄する。
- 深度ステンシルバッファのみバインドする。
    - カラーバッファはバインドしない。
- ラスタライザステートは以下に従う。
    - 前面カリングを行う。
- 深度ステンシルステートは以下に従う。
    - 深度テストを有効にする。
    - 深度書き込みを無効にする。
    - 深度比較はGREATER_EQUALにする。
    - ステンシルテストを有効にする。
    - ステンシル参照値を1にする。
    - ステンシル比較はALWAYSにする。
    - ステンシル処理は深度テストに合格したときREPLACEする。

この構成でライトボリュームを表現したジオメトリをレンダリングすると、Gバッファパスでレンダリングされたピクセルがライトの中か手前にある場合にステンシル値が1になる。

#### ピクセルの数え上げ

印を付けられたピクセルのうち、ライトボリュームの前面より奥にあれば、そのピクセルはライトの影響範囲内にあることが分かる。このときのパイプラインステートは以下の構成に従う。

- 頂点シェーダのみバインドする。
    - ピクセルシェーダをバインドせず、すべてのピクセルを破棄する。
- 深度ステンシルバッファのみバインドする。
    - カラーバッファはバインドしない。
- ラスタライザステートは以下に従う。
    - 背面カリングを行う。
- 深度ステンシルステートは以下に従う。
    - 深度テストを有効にする。
    - 深度書き込みを無効にする。
    - 深度比較はLESS_EQUALにする。
    - ステンシルテストを有効にする。
    - ステンシル参照値を1にする。
    - ステンシル比較はEQUALにする。
    - ステンシル処理はKEEPにする。

この構成でライトボリュームを表現したジオメトリをレンダリングすると、陰影付けを行うべきピクセル数がオクルージョンクエリにより取得できる。この数をしきい値にかけることで、シェーディング処理を省略することもできる。同様の考えで、光が当たらない範囲には影も落ちないと判断して、シャドウマップの生成を省略することもできる。

#### 照らされているピクセルの陰影付け

ピクセルの数え上げを行ったときの構成は、シェーディングを行うべきピクセルを抽出することができる。そこで、実際にピクセルシェーダでライティング計算を行い、その寄与を加算ブレンドしてlight accumulationバッファに出力すれば、最終結果を計算することができる。そのときのパイプラインステートには以下の構成を追加する。

- ライティング処理を行うピクセルシェーダをバインドする。
- カラーバッファにlight accumulationバッファをバインドする。
- ブレンドステートは以下に従う。
    - ブレンド処理を有効にする。
    - source factorをONEにする。
    - destination factorをONEにする。
    - ブレンド処理はADDにする。

### Lighting Pass (My Implementation)

Michielの発表で説明される方法での問題点として、GPUクエリの結果を待つためにCPUがストールし得ることが挙げられる。これは前のフレームでの結果を利用することで回避することができる。
本実験のようにシャドウマッピングを行わないなら、オクルージョンクエリを行う必要はないためそもそも問題にならない。

そのほかの問題点として、視点がライトボリュームの内側に入ったとき照らされているピクセルを正しく見つけることができないことが挙げられる。これはライトボリュームの前面が視錐台に入らず描画されないために起こる。そこで、Michielの方法を逆転することでこの問題を解決する。その工程は以下の3工程から成る。

1. ステンシルバッファを1にクリアする
2. 最も近いライトの境界より手前にあるピクセルに付けられた印を取り除く
3. 最も遠いライトの境界より手前にあるピクセルに対して陰影付けを行う

#### 印の除去

ライトボリュームの前面より手前にあるピクセルに対して、対応するステンシルバッファの値を0にする。このときのパイプラインステートは以下の構成に従う。

- 頂点シェーダのみバインドする。
    - ピクセルシェーダをバインドせず、すべてのピクセルを破棄する。
- 深度ステンシルバッファのみバインドする。
    - カラーバッファはバインドしない。
- ラスタライザステートは以下に従う。
    - 背面カリングを行う。
- 深度ステンシルステートは以下に従う。
    - 深度テストを有効にする。
    - 深度書き込みを無効にする。
    - 深度比較はGREATERにする。
    - ステンシルテストを有効にする。
    - ステンシル比較はALWAYSにする。
    - ステンシル処理は深度テストに合格したときDECR_SATする。

DECR_SATはデクリメントされた値を0にクランプする処理である。
このやり方ならば、ライトボリュームが視錐台に入らない場合には印の除去が行われないだけになるため、結果としてライト範囲のピクセルに対して印を付けたままにすることができる。

#### ピクセルの陰影付け

印を付けられたピクセルのうち、ライトボリュームの背面より手前にあれば、そのピクセルはライトの影響範囲内にあることが分かる。このときのパイプラインステートは以下の構成に従う。

- 頂点シェーダとピクセルシェーダをバインドする。
- light accumulationバッファと深度ステンシルバッファをバインドする。
- ラスタライザステートは以下に従う。
    - 前面カリングを行う。
    - 深度クリッピングを無効にする。
- 深度ステンシルステートは以下に従う。
    - 深度テストを有効にする。
    - 深度書き込みを無効にする。
    - 深度比較はGREATER_EQUALにする。
    - ステンシルテストを有効にする。
    - ステンシル参照値を1にする。
    - ステンシル比較はEQUALにする。
    - ステンシル処理はKEEPにする。
- ブレンドステートは以下に従う。
    - ブレンド処理を有効にする。
    - source factorをONEにする。
    - destination factorをONEにする。
    - ブレンド処理はADDにする。

深度クリッピングを無効化することで、光源が遠クリッピング面(far clipping plane)より遠くにあっても正しくシェーディング計算を行うことができる。


#### ピクセルシェーダコード

ピクセルシェーダはシェーディングを行うフェーズにのみバインドされる。ピクセルシェーダで行うことは、Gバッファから読み出したデータを使うこと以外はForward Renderingで用いたものとほぼ変わらない。本実験ではview空間で計算を行うので、対象ピクセルのview空間での位置を計算する必要がある。

view空間での位置を再構築するには、スクリーン空間での位置と深度値を用いる。`screen_to_view`関数はDirectX環境下[^directx_coordinate]でスクリーン位置をview空間の位置に変換する。

[^directx_coordinate]: DirectXにおけるスクリーン空間は左上を原点に右及び下を正方向としている。また、DirectXにおけるview空間は左手座標系である。

```hlsl
cbuffer ScreenToViewParams : register(b3) {
    float4x4 INV_P; // 射影行列の逆行列
    float2 SCREEN_DIMENSIONS; // スクリーンサイズ
}

float4 clip_to_view(float4 position_c) {
    float4 position_v = mul(INV_P, position_c);
    return position_v / position_v.w;
}

float4 screen_to_view(float2 position_s, float depth) {
    float2 texcoord = position_s / SCREEN_DIMENSIONS;
    float4 position_c = float4(float2(texcood.x, 1.f - texcoord.y) * 2.f - 1.f, depth, 1.f);
    return clip_to_view(position_c);
}

cbuffer LightIndexBuffer : register(b4) {
    uint LIGHT_INDEX;
}
```

ピクセルシェーダでは`SV_Position`セマンティックを用いることでスクリーン座標を受け取ることができる。

深度値を含むGバッファの要素は、バインドされたテクスチャからサンプラを介さずに最も詳細なMIPMAPレベルの値を読み込む。

`LIGHT_INDEX`は`LIGHTS`中の処理対象である光源を指すインデックスを示す。

シェーディングを行う段階では、ライトの有効無効やその範囲内外の判定はライトボリュームを描画するかしないかで判定できるので、Forward Renderingのときのようにシェーダ内で判定する必要はない。

### Transparent Pass

透明パスはForward Renderingのときとまったく同じやり方を使う。

## Forward+

Forward+は、スクリーン空間におけるある領域にライトがどれだけ占めているかを先に定めることにより、通常のForward Renderingを改良するものである。シェーディングではこのときに定めたライトのみを考えれば良くなる。
Forward+はライトカリング、不透明パス、透明パスの3パスからなる。

### Grid Frustums

スクリーンは正方形のタイルに分割される。このタイルの集まりをライトグリッド(light grid)と呼ぶ。タイルのサイズはコンピュートシェーダのスレッドグループのサイズと関連するため、8x8(64 threads)、16x16(256 threads)、32x32(1024 threads)のいずれかになる。

タイルを占めるライトを見つけ出すには、そのタイルからなる視錐台でカリングする必要がある。このカリング用視錐台はview空間に属するため、事前計算が可能であり、グリッド数やタイルサイズが変化したとき以外は再計算を行う必要がない。

視錐台は6つの面の集合として表現される。上下左右の面について、視点からタイルの四隅へのベクトル2本の外積を法線とし、原点からの距離を0とした面として表すことができる。
near面とfar面は、z方向を法線とし、それぞれのclipping面までの距離を原点からの距離とする面として表すことができる。

### Light Culling

ライトカリングパスでは、求めたgrid frustumsを用いて実際にライトのカリングを行う。生成されるライトリストはその影響範囲の違いにより、不透明オブジェクト用と透明オブジェクト用に分けられる。ライトリストはシーンに変化があったときに更新しなければならないため、ライトカリングパスは基本的に毎フレーム行われる。

ライトカリングの基本的なアルゴリズムは以下の通り。

1. 各タイルに対してview空間における深度の最大最小値を計算する。
2. ライトをカリングして、light index listに記録する。
3. light index listをグローバルメモリにコピーする。

#### 深度の最大最小値の計算

まずは、タイルごとの深値の最大最小値を計算する。これはカリング用視錐台のnearとfarとして用いられる。ただし、ここで求めた最大最小値は不透明オブジェクトだけを考えた場合の値である。透明オブジェクトを考えた場合、その深度値は深度バッファに書き込まれないため、near面から不透明オブジェクトでの最大値までを範囲とする。

#### ライトリストのデータ構造

ライトカリングの結果を格納するため、タイルを占めるライトのインデックスを格納するlight index listと、タイルがlight index listに格納したインデックスの範囲を指すオフセットと格納したライト数を持つlight gridを導入する。
例えばタイルサイズを16x16とすると、1280x720の画面サイズなら80x45=3600個のタイルが必要になる。1タイルに最大で200個のライトが必要になると仮定すると、3600x200=720000個のインデックスを格納する領域が必要になる。インデックスが4Bだとすると、light index listひとつに約3MBが必要になる。[^light_index_list]

[^light_index_list]: ~~light index listは図だと隙間のないリストになっていてappendしたかのように見えてしまうが、実際には固定長ブロックのリストとして扱われる一次元配列に格納されるため、未使用領域が間に挟まる。隙間を埋めたいなら、コンパクト化すればよい。(参考実装:https://github.com/bcrusco/Forward-Plus-Renderer/blob/master/Forward-Plus/Forward-Plus/source/shaders/light_culling.comp.glsl)~~ 訂正:図の通りappendしてました。

#### 視錐台カリング

視錐台カリングは処理するライトボリュームの形状によって実装が異なる。

##### 視錐台 対 球

球は中心点と半径で表される。視錐台は6つの面を境界として持つため、球が視錐台の中にいることを判定したいときは、球が視錐台の境界面すべての内側にいることを確かめれば良い。

球が面の内側にいることを判定するには、球の中心点と面との符号付き距離が半径以下[^frustum_sign]であることを確かめれば良い。点と面との距離$l$は以下で求められる。

[^frustum_sign]: 法線の向きで符号が異なるので注意する

\[
l = \boldsymbol{p}_{\text{sphere}} \cdot \boldsymbol{n}_\text{plane} - d_\text{plane}
\]

near面とfar面の場合、view空間では常にz方向を向いているので、その判定はz値のみを考えれば良い。


##### 視錐台 対 円錐(cone)

Realtime Collision Detection p.164-165(日本語訳版)を参照。

円錐は、頂点の位置$T$、尖端から底面への方向$\boldsymbol d$、尖端から底面までの高さ$h$、底面の半径$r$からなる。円錐が面の内側にいることを判定するには、以下の2つのいずれかが真であることを確かめれば良い。

- 頂点$T$が面の内側にある
- 底面上の点$Q$が面の内側にある（底面の円が面と接触している）

$Q$は$\boldsymbol n$方向に見たときに最も手前にある底面の円周上の点を表し、$Q = T + h\boldsymbol{d} + r\boldsymbol{m}$で求める。このときの$\boldsymbol m$は面に最も近い円周上の点への方向を表し、$\boldsymbol m = (\boldsymbol{n} \times \boldsymbol{d}) \times \boldsymbol{d}$で求める。
面と底面が並行になるとき、$Q$は底面中のいずれかの点であれば良い。とはいえ、このときは$\boldsymbol{n} \times \boldsymbol{d} = 0$となるため、この判定式でも問題なく処理できる。

##### シェーダコード

ライトカリングはコンピュートシェーダで行う。

```hlsl
#ifndef BLOCK_SIZE
#define BLOCK_SIZE 16
#endif

// 球
struct Sphere {
    float3 c; // 中心点
    float r;  // 半径
};

// 円錐
struct Cone {
    float3 t; // 頂点
    float h;  // 底面までの高さ
    float3 d; // 底面への方向
    float r;  // 底面の半径
};

// 面
struct Plane {
    float3 n; // 法線
    float d;  // 原点からの距離
};

// 視錐台の側面
struct Frustum {
    Plane planes[4];
};

// 入力引数型
struct CSInput {
    uint3 group_id : SV_GroupID;
    uint3 group_thread_id : SV_GroupThreadID;
    uint3 dispatch_thread_id : SV_DispatchThreadID;
    uint group_index : SV_GroupIndex;
};

// 追加情報
cbuffer DispatchParams : register(b4) {
    uint3 NUM_THREAD_GROUPS;
    uint3 NUM_THREADS; // 破棄されるスレッドも含めた総数
}

// 入力データ
Texture2D DEPTH_TEXTURE : register(t3);
StructuredBuffer<Frustum> FRUSTUMS : register(t9);

// 出力データ
RWStructuredBuffer<uint> OPAQUE_LIGHT_INDEX_COUNTER : register(u1); // 現時点でのライトインデックスの数
RWStructuredBuffer<uint> TRANSP_LIGHT_INDEX_COUNTER : register(u2);
RWStructuredBuffer<uint> OPAQUE_LIGHT_INDEX_LIST : register(u3); // 最終結果を格納するライトインデックスの配列
RWStructuredBuffer<uint> TRANSP_LIGHT_INDEX_LIST : register(u4);
RWTexture2D<uint2> OPAQUE_LIGHT_GRID : register(u5); // ライトグリッド
RWTexture2D<uint2> TRANSP_LIGHT_GRID : register(u6);

// タイル内共有データ
groupshared uint MIN_DEPTH; // uint化された最小深度値
groupshared uint MAX_DEPTH; // uint化された最大深度値
groupshared Frustum GROUP_FRUSTUM; // グループが使う視錐台
groupshared uint OPAQUE_LIGHT_COUNT; // グループが処理した可視ライトの数
groupshared uint OPAQUE_LIGHT_INDEX_START; // グループが予約したグローバルなライトインデックスリストの開始位置
groupshared uint OPAQUE_LIGHT_LIST[1024]; // グループが処理した可視ライトの配列
groupshared uint TRANSP_LIGHT_COUNT;
groupshared uint TRANSP_LIGHT_INDEX_START;
groupshared uint TRANSP_LIGHT_LIST[1024];

[numthreads(BLOCK_SIZE, BLOCK_SIZE, 1)]
void csmain(CSInput input) {
    float depth = DEPTH_TEXTURE.Load(int3(input.dispatch_thread_id.xy, 0)).r;
    uint depth_as_uint = asuint(depth);

    // グループの0番スレッドが初期化を担当する
    if (input.group_index == 0) {
        MIN_DEPTH = 0xffffffff;
        MAX_DEPTH = 0;
        OPAQUE_LIGHT_COUNT = 0;
        TRANSP_LIGHT_COUNT = 0;
        GROUP_FRUSTUM = FRUSTUMS[input.group_id.x + (input.group_id.y * NUM_THREAD_GROUPS)];
    }

    GroupMemoryBarrierWithGroupSync(); // 0番スレッドが初期化を終えるまでグループは待機する

    // タイル内の深度の最大最小値を求める
    InterlockedMin(MIN_DEPTH, depth_as_uint);
    InterlockedMax(MAX_DEPTH, depth_as_uint);

    GroupMemoryBarrierWithGroupSync(); // タイル内の深度の最大最小値が決定するまで待機

    // view空間でのz値を計算する
    float min_depth = asfloat(MIN_DEPTH);
    float max_depth = asfloat(MAX_DEPTH);
    float min_depth_v = clip_to_view(float4(0.f, 0.f, min_depth, 1.f)).z;
    float max_depth_v = clip_to_view(float4(0.f, 0.f, max_depth, 1.f)).z;
    float near_v = clip_to_view(float4(0.f, 0.f, 0.f, 1.f)).z;
    Plane min_plane = {float3(0.f, 0.f, -1.f), -min_depth_v}; // 右手系の場合

    // ライトをカリングする
    // 各スレッドはgroup_index、group_index+256、group_index+512、…番目のライトを処理する
    for (uint i = input.group_index; i < NUM_LIGHTS; i += BLOCK_SIZE * BLOCK_SIZE) {
        if (LIGHTS.enabled) {
            Light light = LIGHTS[i];
            switch (light.type) {
                case POINT_LIGHT: {
                    Sphere sphere = {light.position_v.xyz, light.range};

                    // ライトボリュームが視錐台の内側にあれば、ライトリストに追加する
                    if (sphere_inside_frustum_without_near_plane(...)) { // 共通部分だけ先に計算する
                        // 透明オブジェクトはnear面と判定を行う
                        if (sphere_inside_near_plane(...)) {
                            append_transparent_light(i);
                        }

                        // 不透明オブジェクトはmin_depth面と判定を行う
                        if (sphere_inside_min_depth_plane(...)) {
                            append_opaque_light(i);
                        }
                    }
                    break;
                }
                case SPOT_LIGHT: {
                    float radius = tan(light.spotlight_angle) * light.range;
                    Cone cone = {light.position_v.xyz, light.range, light.direction_v, radius};

                    // ライトボリュームが視錐台の内側にあれば、ライトリストに追加する
                    if (cone_inside_frustum_without_near_plane(...)) { // 共通部分だけ先に計算する
                        // 透明オブジェクトはnear面と判定を行う
                        if (cone_inside_near_plane(...)) {
                            append_transparent_light(i);
                        }

                        // 不透明オブジェクトはmin_depth面と判定を行う
                        if (cone_inside_min_depth_plane(...)) {
                            append_opaque_light(i);
                        }
                    }
                    break;
                }
                case DIRECTIONAL_LIGHT: {
                    append_transparent_light(i);
                    append_opaque_light(i);
                    break;
                }
            }
        }
    }

    GroupMemoryBarrierWithGroupSync(); // グループがカリング計算を終えるまで待つ

    // グループの0番スレッドが完了処理を担当する
    if (input.group_index == 0) {
        // LIGHT_INDEX_COUNTERを始点にLIGHT_COUNT個のLIGHT_LISTの領域を予約する。
        InterlockedAdd(OPAQUE_LIGHT_INDEX_COUNTER[0], OPAQUE_LIGHT_COUNT, OPAQUE_LIGHT_INDEX_START);
        InterlockedAdd(TRANSP_LIGHT_INDEX_COUNTER[0], TRANSP_LIGHT_COUNT, TRANSP_LIGHT_INDEX_START);

        // ライトグリッドを更新する
        OPAQUE_LIGHT_GRID[input.group_id.xy] = uint2(OPAQUE_LIGHT_INDEX_START, OPAQUE_LIGHT_COUNT);
        TRANSP_LIGHT_GRID[input.group_id.xy] = uint2(TRANSP_LIGHT_INDEX_START, TRANSP_LIGHT_COUNT);
    }

    GroupMemoryBarrierWithGroupSync(); // 0番スレッドが完了処理を終えるまでグループを待機させる

    // グループ共有に保存したライトリストをUAVバッファにコピーする
    for (i = input.group_index; i < OPAQUE_LIGHT_COUNT; i += BLOCK_SIZE * BLOCK_SIZE) {
        OPAQUE_LIGHT_INDEX_LIST[OPAQUE_LIGHT_INDEX_START + i] = OPAQUE_LIGHT_LIST[i];
    }
    for (i = input.group_index; i < TRANSP_LIGHT_COUNT; i += BLOCK_SIZE * BLOCK_SIZE) {
        TRANSP_LIGHT_INDEX_LIST[TRANSP_LIGHT_INDEX_START + i] = TRANSP_LIGHT_LIST[i];
    }
}
```

シェーダモデル5.0は浮動小数点型のアトミック演算をサポートしてないため、アトミック演算が必要な`MIN_DEPTH`と`MAX_DEPTH`の計算には`float`型ではなく`uint`型を採用している。

`SV_GroupIndex`セマンティックはそのスレッドが所属するグループにおけるスレッド番号を受け取る。

ディレクショナルライトのライトボリュームはスクリーン全体なので、問答無用でライトリストにぶち込む。

### シェーディング

Forward+におけるシェーディングは、ライトカリングによって生成したライトインデックスリストを用いる事以外は通常のForward Renderingとほぼ変わらない。

```hlsl
StructuredBuffer<uint> LIGHT_INDEX_LIST : register(t9);
Texture2D<uint2> LIGHT_GRID : register(t10);

[earlydepthstencil]
float4 psmain(VSOutput input) : SV_Target {
    uint2 tile_index = uint2(floor(input.position.xy / BLOCK_SIZE));
    uint2 light_grid = LIGHT_GRID[tile_index].xy;
    uint start_offset = light_grid.x;
    uint light_count = light_grid.y;

    float3 final_color = 0.f;
    float final_alpha = 1.f;
    for (uint i = 0; i < light_count; ++i) {
        uint light_index = LIGHT_INDEX_LIST[start_offset + i];
        Light light = LIGHTS[light_index];

        <<ライティング処理>>
    }

    <<シェーディング処理>>

    return float4(final_color, final_alpha);
}
```

## 実験設定とパフォーマンス結果

NVIDIA GeForce GTX 680を用いて1280x720の解像度でレンダリングした。シーンはCrytek Sponzaを用いた。シーンにはランダムに光源を配置し、そのライト半径が大きい場合(35から40単位)と小さい(1から2単位)の2パターンを用意した。

Forward Renderingでは、処理時間がライト数に対して指数関数的に増加していた。この環境では、ライト半径が大きい場合は64個に、ライト半径が小さい場合は128個になると全体の処理時間が16.6msラインを上回った。

Deferred Renderingでは、透明オブジェクトは原理上Forward Renderingと同等であるため、処理時間に差は現れないが、不透明オブジェクトの処理時間にいくらかの改善が見られた。この改善は深度ステンシルテストにより冗長なライティング計算を排除したことによるものと思われる。この環境では、ライト半径が大きい場合は128個に、ライト半径が小さい場合は512個になると全体の処理時間が16.6msラインを上回った。しかし、この結果は透明オブジェクトによるところが大きく、不透明オブジェクトだけ見れば、2048個程度でも16.6msラインを下回ることを確認できた。

Forward+では、ライト半径が大きい場合は、1タイル平均200個の上限に触れない程度のライト数において、ライトカリングパスはいずれの場合でも1msを上回らなかった。この場合、全体の処理時間はライト数が128個になると16.6msラインを上回った。
ライト半径が小さい場合は、ライティング計算はライト数が10000個を越えても5ms程度に収まった。しかし、ライトカリングパスが1024個を越えた当たりから急激な増加に転じた。全体としては8192個になると16.6msラインを上回った。

3者の傾向として、ライト数が少ない場合はいずれも16.6msラインを下回るが、数が増えるに従って、Forward Rendering、Deferred Rendering、Forward+の順で増加に転じた。ライト半径が大きい場合、Deferred RenderingとForward+に大きな差は見られなかった。ライト半径が小さい場合、特にForward+の計算時間の増加率が他2者に比べて明らかに小さく、128個あたりからそれが顕著になる。ただし、この結果は透明オブジェクトによるものと考えられるため、不透明オブジェクトに限れば、Deferred RenderingとForward+の差はあまりないと思われる。しかし、Deferred Renderingはライトボリュームをレンダリングするために大量のドローコールを発行するため、ドローコール量に敏感に反応するシステムではパフォーマンスが低下する恐れがある。

それぞれの追加コストとして、Deferred RenderingはGバッファが必要であり、Forward+はライトリストのためのバッファが必要である。ライト数の少ない場合には３者に差がないことを考慮すると、メモリ容量に余裕のないシステムでライト数を多くする必要が無い場合は、Forward Renderingが最良の選択になり得る。

## 今後の検討課題

### 共通

ライトの構造をタイプ別に分けることで、最小限のデータのみを要求することができるため、パフォーマンスの向上が期待できる。

### Forward Rendering

本実験では行っていなかった、ライトのカリングや深度プリパスなどの最適化の導入する。

### Deferred Rendering

本実験では、Gバッファを取扱いやすさを基準に選定したが、より攻めた最適化を行うこともできる。例えば法線は、16ビットや10ビットでも十分表現できるだろうし、view空間ならz値はxy値から実行時に復元できるため保存しておく必要がなくなる。

また、本実験ではのような、ディレクショナルライトをスクリーン全体をライトボリュームとする方法は何度も重複して画面全体の描画が行われるため効率的ではない。そこで、環境光と同じようにGバッファパスの段階でlight accumulationバッファに書き出すことができれば、この問題を緩和することができるかもしれない。

この技法でも、Gバッファパス前に深度プリパスを行うことで最低幾できる余地がある。

シャドウマッピングを考えたとき、Deferred Renderingならライティングパスにその光源のシャドウマップをバインドするだけでよくなるため、オンメモリなシャドウマップの総数を削減できるかもしれない。ただし、Gバッファパスにディレクショナルライトの計算を移した場合、そのときに使うすべての光源のシャドウマップを一度に用意しなければならなくなるため注意が必要である。

### Tiled Forward Rendering

パフォーマンス結果にも現れた通り、ライトカリングパスはボトルネックになっている。つまり、ライトカリングパスの改善が全体の改善にクリティカルに効くことを示している。

想定されるライトカリングパスの改善点として、

- タイルベースのカリングを行う前に、カメラの視錐台でカリングを行う。
- sparseな八分木(octree)でライトリストを構成する。
    - DirectX12で登場したVolume Tiled Resourcesを用いる。
- 判定の正確性を向上させる。
    - 「面の内側」という判定方法だとfalse positiveが起こり得る。

などが挙げられる。

GDC2015のGareth Thomasの発表では、錐台をAABBで近似することにより判定処理の複雑化を抑えながら正確性を向上させる方法を提案している。
また、タイル内の深度の不連続性(depth discontinuities)[^depth_discontinuities]が大きいと、何もない空間のライトも集めてしまうため、無駄な処理が多くなってしまうという問題に対しては、錐台を2つに分割してそれぞれの最大最小値を求める方法を提案している。この方法ではカリングを2回行う必要があるが、中間にある何もない空間を切り落とすことができるため、結果としてパフォーマンスが改善するという。

[^depth_discontinuities]: 近傍での深度値の変化量を示す度合い。深度の不連続性が大きいとすると、近傍で深度値が大きく変化することを表すため、それだけオブジェクトの存在しない空間が大きいという意味になる。

さらに興味深いパフォーマンス最適化手法として、OlssonらによるClustered Shadingがある。この手法はリアルタイムレベルで100万個の光源を処理できるとしている。

他に使えそうな空間分割アルゴリズムとして、BSPやSpase Voxel Octreeなどがある。

## 結論

<<いろいろなまとめ>>
