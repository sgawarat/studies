# Volumetric Rendering

真空でない空間における光の作用を考慮したレンダリングについて

## 表記法

- $\boldsymbol{x}$：位置を表すベクトル
- $\boldsymbol{\omega}$：方向を表すベクトル
- $f_p(\boldsymbol{x}, \boldsymbol{\omega}, \boldsymbol{\omega}')$：位相関数
  - 地点$\boldsymbol{x}$において方向$\boldsymbol{\omega}$から入射する光が関与媒質中で散乱して方向$\boldsymbol{\omega}'$へ出射する確率

## Volume Rendering Equeation

- 真空でない空間を通る光の放射輝度を求める方程式
- $d$だけ離れた表面で$\boldsymbol{\omega}$の方向に反射した光が$\boldsymbol{x}$の位置に到達したときの放射輝度を求める
  - 光はその道中で関与媒質と相互作用する

$$
L(\boldsymbol{x}, \boldsymbol{\omega}) = T(\boldsymbol{x}, \boldsymbol{\omega}, d) L_d(\boldsymbol{x} + d\boldsymbol{\omega}, \boldsymbol{\omega}) + \int_0^d T(\boldsymbol{x}, \boldsymbol{\omega}, t) \left[ \sigma_a(\boldsymbol{x}) L_e(\boldsymbol{x} + t\boldsymbol{\omega}, \boldsymbol{\omega}) + \sigma_s(\boldsymbol{x}) L_s(\boldsymbol{x} + t\boldsymbol{\omega}, \boldsymbol{\omega}) \right] dt, \\
T(\boldsymbol{x}, \boldsymbol{\omega}, t) = \exp \left( -\int_0^t \sigma_t(\boldsymbol{x} + s\boldsymbol{\omega}) ds \right), \\
L_s(\boldsymbol{x}, \boldsymbol{\omega}) = \int_{S^2} f_p(\boldsymbol{x}, \boldsymbol{\omega}, \boldsymbol{\omega}') L(\boldsymbol{x}, \boldsymbol{\omega}') d\boldsymbol{\omega}'
$$

- $T$：位置$\boldsymbol{x}$から方向$\boldsymbol{\omega}$に$t$だけ進む間の透過率の関数
- $L_d$：位置$\boldsymbol{x}$から方向$\boldsymbol{\omega}$に$t$だけ進んだ所にある表面から出射する放射輝度の関数
- $L_e$：放射による光の放射輝度の関数
- $L_s$：位置$\boldsymbol{x}$に集まった光が散乱して方向$\boldsymbol{\omega}$に合流する際の放射輝度の関数

### 吸収[absorption]

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = -\sigma_a(\boldsymbol{x}) L(\boldsymbol{x}, \boldsymbol{\omega})
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、どれだけの光が媒質中で消失するかを示す
  - このときの放射輝度は減少するのでマイナスが付く
  - $\sigma_a$は吸収係数[absorption coefficient]と呼ぶ

### out-scattering

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = -\sigma_s(\boldsymbol{x})L(\boldsymbol{x}, \boldsymbol{\omega})
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、どれだけの光が関与媒質と作用して方向$\boldsymbol{\omega}$から逸れるかを示す
  - このときの放射輝度は減少するのでマイナスが付く
  - $\sigma_s$は散乱係数[scattering coefficient]と呼ぶ

#### 減衰[attenuation]

- 吸収とout-scatteringを合わせて消散[extinction]とか減衰[attenuation]と呼ぶ
  - $\sigma_t = \sigma_a + \sigma_s$を減衰係数[attenuation coefficient]と呼ぶ
- 減衰係数に対する散乱係数の割合を散乱アルベド[scattering albedo]と呼ぶ
  - $\Lambda = \sigma_s / \sigma_t$

### in-scattering

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = \sigma_s(\boldsymbol{x}) \int_{S^2} f_p(\boldsymbol{x}, \boldsymbol{\omega}, \boldsymbol{\omega}') L(\boldsymbol{x}, \boldsymbol{\omega}') d\boldsymbol{\omega}'
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、周囲の光が関与媒質と作用して方向$\boldsymbol{\omega}$にどれだけ合流するかを示す
  - このときの放射輝度は増加するのでマイナスが付かない
- 地点$\boldsymbol{x}$に集まる光のうち、方向$\boldsymbol{\omega}$に散乱した光の放射輝度の総和を求めている

### 放射[emission]

$$
(\boldsymbol{\omega} \cdot \nabla)L(\boldsymbol{x}, \boldsymbol{\omega}) = \sigma_a(\boldsymbol{x}) L_e(\boldsymbol{x}, \boldsymbol{\omega})
$$

- 光が関与媒質中を微小距離だけ$\boldsymbol{\omega}$の方向に進むとき、関与媒質内部で発生した光が方向$\mathbf{\omega}_i$にどれだけ向かうかを示す
  - このときの放射輝度は増加するのでマイナスが付かない

# 参考文献

- [ボリュームレンダリング方程式1 - memoRANDOM](https://rayspace.xyz/CG/contents/VLTE1/)
- [ボリュームレンダリング入門](https://www.slideshare.net/OtsuHisanari/introduction-to-volume-rendering)
- [Physically Based and Unified Volumetric Rendering in Frostbite](https://www.slideshare.net/DICEStudio/physically-based-and-unified-volumetric-rendering-in-frostbite)
- [Production Volume Rendering](http://graphics.pixar.com/library/ProductionVolumeRendering/paper.pdf)
