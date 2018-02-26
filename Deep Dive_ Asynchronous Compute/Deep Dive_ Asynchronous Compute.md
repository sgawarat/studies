---
title: >
Deep Dive: Asynchronous Compute
---
# Deep Dive: Asynchronous Compute

# 合同会議[Joint Session]

**<font color='red'>AMD</font>**

- Graphics Core Next (GCN)
- Compute Unit (CU)
- Wavefronts

**<font color='green'>NVIDIA</font>**

- Maxwell、Pascal
- Streaming Multiprocessor (SM)
- Warps

# 用語法[Terminology]

非同期[Asynchronous]: **独立**ではなく、非同期ワークはハードウェアを共有する

ワークペアリング[Work Pairing]: 同時に実行するGPUワークのアイテム

非同期税[Async. Tax]: 非同期コンピュートに関連するオーバーヘッドコスト

# 非同期コンピュート→更なるパフォーマンス[Async Compute -> More Performance]

# キューの基礎[Queue Fundamentals]

- 3種のキュー
    - コピー/DMAキュー
    - コンピュートキュー
    - グラフィクスキュー

すべて非同期的に動作する！

# 一般的なアドバイス[General Advice]

- 常にプロファイルする！
    - パフォーマンスを作ったり打ち破ったりできる
- 非同期でないパスを維持する
    - 非同期のオン/オフをプロファイルする
    - いくつかのハードウェアは非同期をサポートしないだろう
- ハイパースレッディングの仲間？
    - 似たようなルールが適用される
    - 共有ハードウェアリソースの帯域幅調整[throttling]を回避する

# Regime Pairing

- 上手なペアリング
    - グラフィクス
        - シャドウレンダリング(ジオメトリ制限)
    - コンピュート
        - ライトカリング(ALUヘビー)
- 下手なペアリング
    - グラフィクス
        - Gバッファ(帯域幅制限)
    - コンピュート
        - SSAO(帯域幅制限)

(テクニックのペアリングは1対1でなくてもよい)

# <font color='red'>🚩</font> --- 赤い旗[<font color='red'>🚩</font> - Read Flags]

問題/解決策の形式

トピックス

- <font color='red'>リソース競合</font> --- AMD
- <font color='green'>デスクリプタヒープ</font> --- NVIDIA
- 同期モデル
- "非同期コンピュート税"の回避

# <font color='red'>ハードウェアの詳細</font> --- AMD[<font color='red'>Hardware Details</font> - AMD]

- CUあたり4つのSIMD
- SIMDあたり最大10個のWavefrontがスケジュールされる
    - レイテンシーの隠蔽を達成する
    - グラフィクスとコンピュートが同じCUで同時に実行できる
- グラフィクスのワークロードは*通常では*コンピュートより大きい優先度を持つ

# <font color='red'>🚩リソース競合</font> --- AMD[<font color='red'>🚩 Resource Contention</font> - AMD]

TODO
