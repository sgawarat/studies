# Volumetric Rendering

真空でない空間における光の作用を考慮したレンダリングについて

## 理論

### Beer-Lambertの法則

- 光が散乱によって減衰するときの振る舞いを定義する法則

$$
T(\boldsymbol{x}, \boldsymbol{y}) = \exp \left( -\int_0^y \sigma_t(\boldsymbol{x} - s \boldsymbol{\omega}) ds \right)
$$

- 位置$\boldsymbol{x}$から位置$\boldsymbol{y}$までの間を方向$\boldsymbol{\omega}$に向かって関与媒質中を通る光の透過率を求める
- 光は関与媒質中を通るとき、距離に対して指数関数的に減衰するということがわかる

### 位相関数

- 散乱した光の分布を表す関数
  - ある位置において、ある方向から入射する光が散乱して、ある方向へ出射する確率を求める
  - $f_p$で表されることが多い

#### Henyey-Greensteinの位相関数

- 異方的な散乱を求めるときによく使われる
- Zonal球面調和関数への展開が自明である
  - 4次のZonal SHでは$(1, g, g^2, g^3)$となる
- 多くの計算を事前に行うことができる

$$
p(\theta) = \frac{1}{4\pi} \frac{1 - g^2}{(1 + g^2 - 2g \cos \theta)^\frac{3}{2}}
$$

- $g \in [-1, 1]$は異方性を表すパラメータ
  - $-1$で後方散乱に、$0$で等方散乱に、$1$で前方散乱になる
- $\theta$は光線と視線のなす角

### Volume Rendering Equation

- 表面で反射した光が真空でない空間を通るときの放射輝度を求める方程式
  - 最も近い表面が$\boldsymbol{z} = \boldsymbol{x} - z\boldsymbol{\omega}$にあるとしてRadiative Transfer Equationを書き直したもの

$$
L(\boldsymbol{x}, \boldsymbol{\omega}) = \int_0^z T(\boldsymbol{x}, \boldsymbol{y}) \left[ \sigma_a(\boldsymbol{y}) L_e(\boldsymbol{y}, \boldsymbol{\omega}) + \sigma_s(\boldsymbol{y}) L_s(\boldsymbol{y}, \boldsymbol{\omega}) \right] dy + T(\boldsymbol{x}, \boldsymbol{z}) L(\boldsymbol{z}, \boldsymbol{\omega}), \\
T(\boldsymbol{x}, \boldsymbol{y}) = e^{-\int_0^y \sigma_t(\boldsymbol{x} - s\boldsymbol{\omega}) ds}, \\
L_s(\boldsymbol{x}, \boldsymbol{\omega}) = \int_{S^2} f_p(\boldsymbol{x}, \boldsymbol{\omega}, \boldsymbol{\omega}') L_i(\boldsymbol{x}, \boldsymbol{\omega}') d\boldsymbol{\omega}'
$$

- $T(\boldsymbol{x}, \boldsymbol{y})$：位置$\boldsymbol{x}$から位置$\boldsymbol{y} = \boldsymbol{x} - y\boldsymbol{\omega}$までの間の透過率
  - ランベルト・ベールの法則[Lambert-Beer law]として知られている
  - $\int_0^y \sigma_t(\boldsymbol{x} - s\boldsymbol{\omega}) ds$の部分は光学的厚さ[optical thickness]と呼ばれ、$\tau(y)$で表される
  - $\sigma_t = \sigma_a + \sigma_s$は消散係数[extinction coefficient]とか減衰係数[attenuation coefficient]と呼ばれる
- $L_e(\boldsymbol{y}, \boldsymbol{\omega})$：放射による光の放射輝度
- $L_s(\boldsymbol{y}, \boldsymbol{\omega})$：位置$\boldsymbol{y}$で方向$\boldsymbol{\omega}$に散乱した光の放射輝度
- $L(\boldsymbol{z}, \boldsymbol{\omega})$：表面で反射した光の放射輝度

#### 吸収[absorption]

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = -\sigma_a(\boldsymbol{x}) L(\boldsymbol{x}, \boldsymbol{\omega})
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、どれだけの光が媒質中で消失するかを示す
  - このときの放射輝度は減少する
  - $\sigma_a$は吸収係数[absorption coefficient]と呼ばれる

#### out-scattering

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = -\sigma_s(\boldsymbol{x})L(\boldsymbol{x}, \boldsymbol{\omega})
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、どれだけの光が関与媒質と作用して方向$\boldsymbol{\omega}$から逸れるかを示す
  - このときの放射輝度は減少する
  - $\sigma_s$は散乱係数[scattering coefficient]と呼ばれる

#### in-scattering

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = \sigma_s(\boldsymbol{x}) \int_{S^2} f_p(\boldsymbol{x}, \boldsymbol{\omega}, \boldsymbol{\omega}') L_i(\boldsymbol{x}, \boldsymbol{\omega}') d\boldsymbol{\omega}'
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、周囲の光が関与媒質と作用して方向$\boldsymbol{\omega}$にどれだけ合流するかを示す
  - このときの放射輝度は増加する

#### 放射[emission]

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = \sigma_a(\boldsymbol{x}) L_e(\boldsymbol{x}, \boldsymbol{\omega})
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、関与媒質内部で発生した光が方向$\mathbf{\omega}_i$にどれだけ向かうかを示す
  - このときの放射輝度は増加する

## 実践

### 実装にあたって

- $L$は放射光$L_e$と散乱光$L_s$を方向$\boldsymbol{\omega}$に沿って積分する
- 透過率$T$は散乱係数$\sigma_t(\boldsymbol{x})$を方向$\boldsymbol{\omega}$に沿って積分する
- 散乱光$L_s$は入射光を全方向で積分する
  - この入射光を求めるためにまた積分が必要になるかもしれない
- 反射光$L(\boldsymbol{z}, \boldsymbol{\omega})$は入射光を全方向に積分する
  - この入射光を求めるためにまた積分が必要になるかもしれない

#### 計算の単純化

- 媒質が均質（$\sigma_t(\boldsymbol{x}) = \sigma_t$）であれば、$T(\boldsymbol{x}, \boldsymbol{y}) = e^{-\sigma_t y}$とすることができる
- 単一散乱のみを扱うならば、散乱光$L_s$は光源からの直接光を集めるだけで良い
- 視線に平行な放射輝度マップがあれば、$L$を積分するのに使える


## 事例

### Volumetric Fog: Unified compute shader based solution to atmospheric scattering [@Wronski2014]

- 単一散乱のみを扱う
- 錐台に平行な3Dテクスチャに中間値を格納する
  - サイズは160x90x64または160x90x128
  - 奥行方向は指数関数的に分割される

#### 詳細

1. 太陽光のシャドウマップをダウンサンプリングする
   - 指数シャドウマップの手法を用いて格納する
2. 関与媒質の密度を推定してライティングする
   - プロシージャルな方法で散乱係数を計算してAに格納する
     - 風の影響を表現するためのパーリンノイズ
     - 重い粒子を表現するための鉛直方向の減衰
   - 密度で変化させたライティング結果を合計してRGBに格納する
     - シャドウイングされたメインライト（太陽光または月光）
     - 定数のアンビエント項
     - 点光源
   - このときに用いる位相関数は完全にアートドリブンなもの
     - 物理ベースではない
     - 太陽方向の色とその反対方向の色の2色から成る
3. 散乱の方程式を解く
   - 透過率を計算してAに格納する
   - 散乱光を合計して透過率を適用してRGBに格納する
4. 効果を適用する
   - 3Dテクスチャから透過率と散乱光を取り出す
     - バイリニア補間してサンプルする
   - ピクセル色にかけ合わせる

### Physically Based and Unified Volumetric Rendering in Frostbite [@Hillaire2015]

- 単一散乱[single scattering]のみを扱う
- 錐台に平行な3Dテクスチャに値を格納する
  - ワールド空間から見ると、錐台型のボクセル[frustum voxel; froxel]になる
- タイルベースのライティングで生成されるライトリストを利用して放射輝度を計算する

#### データフロー

1. 関与媒質の特性をV-Bufferにボクセル化する
   - V-Bufferは2つのRGBA16Fテクスチャで構成される
     - 1つ目は散乱(RGB)と消散(A)の値をそれぞれ合計する
       - 消散が波長に非依存なのは、バッファサイズを小さくするため
     - 2つ目は放射(RGB)の値を合計し、位相(A)の値を平均する
2. froxelごとに関与媒質の特性をサンプリングして積分する
   - 1つのRGBA16Fテクスチャに格納する
     - 散乱光(RGB)
       - すべての光源（間接光、太陽、ローカルライト）を積分する
         - 間接光：ボリュームの中心にあるプロブ1つをSHコサインローブとして位相関数を積分する？^[TODO:要検証]
         - 太陽光：Cascaded shadow mapsをサンプリングする
         - ローカルライト：タイルベースライティングのライトリストを使う
     - 消散(A)
       - 時間的に積分するので、非線形な透過率ではなく線形な消散を用いる
3. 視線に沿ってfroxelを積分する
   - froxelごとに視線に沿ってその奥行き$D$だけ積分する？^[TODO:要検証]
     - 散乱光$S$と消散係数$\sigma_t$はfroxel内で一定とする
     - froxel内の放射輝度：$\int_0^D e^{-\sigma_t x} \times S dx = \frac{S - S \times e^{-\sigma_t D}}{\sigma_t}$

#### 時間的な積分

- 視線に沿ってすべてのサンプルを同じオフセットでジッタリングする
- 前フレームの散乱と消散を現在のフレームに再投影する
  - 指数関数的な移動平均を用いて5%だけブレンドする

### The lighting technology of Detroit: Become Human [@Caurant2018]

- [@Wronski2014; @Hillaire2015]の手法を用いる
  - ライトクラスタの奥行きに合わせる
  - チェッカーボードレンダリングを用いる
    - PS4で192x108x64、PS4 Proで240x135x64
  - ブルーノイズでジッタリングしてTAAをかける
  - 直接光とディフューズプロブでライティングする
- ライトリークを防ぐテクニックを追加する
  - タイルごとの最大深度でボクセルの厚さクランプする
  - テクスチャサンプリングにZバイアスを適用する
    - タイル深度の分散 > しきい値

# 参考文献

- [ボリュームレンダリング方程式1 - memoRANDOM](https://rayspace.xyz/CG/contents/VLTE1/)
- [ボリュームレンダリング入門](https://www.slideshare.net/OtsuHisanari/introduction-to-volume-rendering)
- [Physically Based and Unified Volumetric Rendering in Frostbite](https://www.slideshare.net/DICEStudio/physically-based-and-unified-volumetric-rendering-in-frostbite)
- [Production Volume Rendering](http://graphics.pixar.com/library/ProductionVolumeRendering/paper.pdf)
- [Monte Carlo Methods for Volumetric Light Transport Simulation - Disney Research Studios](https://studios.disneyresearch.com/2018/04/16/monte-carlo-methods-for-volumetric-light-transport-simulation/)
- [Volumetric Fog: Unified compute shader-based solution to atomospheric scattering](http://advances.realtimerendering.com/s2014/wronski/bwronski_volumetric_fog_siggraph2014.pdf)
- [The lighting technology of Detroit: Become Human](https://www.gdcvault.com/play/1025339/The-Lighting-Technology-of-Detroit)
