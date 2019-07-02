# Volumetric Rendering

真空でない空間における光の作用を考慮したレンダリングについて

## Volume Rendering Equeation

- **関与媒質[participating media]** と光の相互作用も考慮する
  - 吸収[absorption]、散乱[scattering]、放射[emission]が起こる

### 表記法

- $\mathbf{x}$：位置を表すベクトル
- $\mathbf{\omega}$：方向を表すベクトル
- $L_i$：入射する放射輝度を求める関数
  - 光は$\mathbf{x}$の地点から$\mathbf{\omega}$の向きに進む

### 吸収[absorption]

- 光が関与媒質中を微小距離$s$だけ通るとき、どれだけの光が媒質中に吸収されるかを示す
  - $dL_i$は光が媒質中に吸収されることで変化する放射輝度の変化量を表す
  - 放射輝度は減少するのでマイナスが付く
  - $\sigma_a$は吸収係数[absorption coefficient]と呼ぶ
- 関与媒質を通る光は吸収係数に応じて指数関数的に減衰する

$$
dL_i(\mathbf{x}, \mathbf{\omega}) = -\sigma_a(\mathbf{x})L_i(\mathbf{x}, \mathbf{\omega})ds
$$



# 参考文献

- [memoRANDOM](https://rayspace.xyz/CG/contents/VLTE1/)
- [ボリュームレンダリング入門](https://www.slideshare.net/OtsuHisanari/introduction-to-volume-rendering)
