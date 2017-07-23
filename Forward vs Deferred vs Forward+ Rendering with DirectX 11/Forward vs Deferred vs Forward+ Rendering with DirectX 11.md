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

float4 screen_to_view(float2 position_s, float depth) {
    float2 texcoord = position_s / SCREEN_DIMENSIONS;
    float4 position_c = float4(float2(texcood.x, 1.f - texcoord.y) * 2.f - 1.f, depth, 1.f);

    float4 position_v = mul(INV_P, position_c);
    return position_v / position_v.w;
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

### Grid Frusta

スクリーンは正方形のタイルに分割される。このタイルの集まりをライトグリッド(light grid)と呼ぶ。タイルのサイズはコンピュートシェーダのスレッドグループのサイズと関連するため、8x8(64 threads)、16x16(256 threads)、32x32(1024 threads)のいずれかになる。

タイルを占めるライトを見つけ出すには、そのタイルからなる視錐台でカリングする必要がある。このカリング用視錐台はview空間に属するため、事前計算が可能であり、グリッド数やタイルサイズが変化したとき以外は再計算を行う必要がない。

視錐台は6つの面の集合として表現される。上下左右の面について、視点からタイルの四隅へのベクトル2本の外積を法線とし、原点からの距離を0とした面として表すことができる。
near面とfar面は、z方向を法線とし、それぞれのclipping面までの距離を原点からの距離とする面として表すことができる。

### Light Culling

TODO
