---
title: Vulkan Fast Paths [@Sellers2016b]
bibliography: bibliography.bib
numberSections: false
---
# Vulkan Fast Paths: Binding Model

# GCN Terms --- Background on the hardware

- SGPR - 32ビットのスカラー一般用途レジスタ(Scalar General Purpose Register)。
- VGPR - 32ビットのベクトル一般用途レジスタ(Scalar General Purpose Register)。
- K$ - スカラーデータキャッシュ(Scalar Data Cache)。
- CU - コンピュートユニット(Compute Unit)。
    - 各CUはクロックあたり64のFMA命令のスループットを持つ。
- Wave - 足並みを揃えて(in lock-step)実行する64のシェーダ呼び出し。
- 以下の命令はCUにおいて異なるwaveで同時に複数発行(multi-issue)を行うことができる。
    - SMEM - スカラーメモリ命令(K$へのアクセス、動的に統一(uniform)アドレスを経由したバッファアクセス)。
    - SALU - スカラー算術命令(イメージやバッファのアクセス)。
    - VMEM - ベクトルメモリ命令。
    - VALU - ベクトル算術命令。

# Descriptor Types --- How they map to GCN hardware

- GCNの16バイトサンプラデスクリプタにマップする。
    - `VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER`(サンプラの部分だけ)
    - `VK_DESCRIPTOR_TYPE_SAMPLER`
- GCNの16バイトバッファデスクリプタにマップする。
    - `VK_DESCRIPTOR_TYPE_STORAGE_BUFFER`
    - `VK_DESCRIPTOR_TYPE_STORAGE_BUFFER_DYNAMIC`
    - `VK_DESCRIPTOR_TYPE_STORAGE_TEXEL_BUFFER`
    - `VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER`
    - `VK_DESCRIPTOR_TYPE_UNIFORM_BUFFER_DYNAMIC`
    - `VK_DESCRIPTOR_TYPE_UNIFORM_TEXEL_BUFFER`
- GCNの32バイトイメージデスクリプタにマップされる(後の世代のGDC[^GDC]はすべてのイメージデスクリプタで32バイトを使う)。
    - `VK_DESCRIPTOR_TYPE_COMBINED_IMAGE_SAMPLER`(イメージの部分だけ)
    - `VK_DESCRIPTOR_TYPE_IMAGE_ATTACHMENT`
    - `VK_DESCRIPTOR_TYPE_SAMPLED_IMAGE`
    - `VK_DESCRIPTOR_TYPE_STORAGE_IMAGE`

[^GDC]: GCNのtypo？

# USER-DATA SGPRs --- GDC[^GDC] hardware background for Vulkan descriptor model

- シェーダは2つのレジスタセットへのアクセス権限を持つ。
    - SGPRはひとつのwaveにおけるすべての呼び出しに対して動的に一定(uniform)である。
        - 定数やデスクリプタの例として使われる、
    - VGPRはひとつのwaveにおいてすべての呼び出しに対して唯一(unique)である。
- GCNはwaveの開始に先立って最大16個のSGPRを先読みする能力を持つ。
    - USER-DATA SGPRとして知られている。
    - GPUのコマンドバッファにUSER-DATAを設定するため、いくらかのオーバーヘッドがwaveの開始直前に発生する。
        - なので、必要以上に使わない。
    - 少量のUSER-DATA SGPRがドライバによって内部的に使われる。
        - シェーダステージや使っている機能により変化し、今後のドライバやハードウェアで変わり得る。
    - その残りがVulkanのバインディングモデルに使われる。
        - Push Constants
        - 32ビットのデスクリプタセットのポインタ(デスクリプタセットは下位32ビットアドレス空間に維持される)。
        - `UNIFORM_BUFFER_DYNAMIC`か`STORAGE_BUFFER_DYNAMIC`(それぞれが4つのUSER-DATA SGPRを使う)。

# USER-DATA SGPRs: and spilling --- GDC[^GDC] hardware background for Vulkan descriptor model

- USER-DATA SGPRが埋まりきると、ドライバはそのオーバーフロー分をドライバが管理するバッファに書き込む。
    - こぼれ(spilling)が起こると、コスト増加につながる(シェーダでの間接参照など)。
- USER-DATA SGPRが埋まる優先順位はドライバ依存(最適化やドライバのリビジョン次第で変わり得る)。
    - 現在の優先順位は一般的に、
        - 初めに、Push Constants。
        - 続いて、0からNまでのデスクリプタセット。
            - 先に、動的デスクリプタ。
            - 次いで、デスクリプタセットのポインタ。
- 起こり得るUSER-DATA SGPRの使用例。
    - 4つをドライバ内部で使用。
    - 2つを32ビットのPush Constantsに。
    - 4つを0番の動的デスクリプタに。
    - 1つを0番のデスクリプタセットポインタに。
    - 1つを1番のデスクリプタセットポインタに。
- 一般的なおみやげ(takeaway)
    - こぼさないためにPush Constants、バインドされたデスクリプタセット、動的デスクリプタの総数を十分に少なくしてみよう。

# Constant and Descriptor loads on GCN --- Hardware background

- TODO

# References
