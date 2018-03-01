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

**問題**: SIMDごとのリソースはWavefronts間で共有される

SIMDは(異なるシェーダの)Wavefrontsを実行する

- 占有率は以下によって制限される
    - レジスタ数
    - LDSの量
    - 他の制限が適用されるかも…
- Wavefrontsはキャッシュを求めて競合する

# <font color='red'>🚩リソース競合</font> --- AMD[<font color='red'>🚩 Resource Contention</font> - AMD]

- ベクタレジスタ(VGPR)数に目を配る

|GCN VGPR数|24以下|28|32|36|40|48|64|84|128以下|128超|
|-|-|-|-|-|-|-|-|-|-|-|
|最大SIMDあたりのWave数|10|9|8|7|6|5|4|3|2|1|

- キャッシュスラッシングに気を付けて！
    - ダミーLDSを割り当てることで占有率を制限してみる

# <font color='green'>ハードウェア詳細</font> --- NVIDIA[<font color='green'>Hardware Details</font> - NVIDIA]

- コンピュートはSM全体に幅優先でスケジュールされる[Compute scheduled breadth first over SMs]
- コンピュートワークロードはグラフィクス以上の優先度を持つ
    - ドライバはSMの分散をヒューリスティックに制御する

# <font color='red'>🚩</font><font color='green'>デスクリプタヒープ</font> --- NVIDIA[<font color='red'>🚩</font><font color='green'>Descriptor Heap</font> - NVIDIA]

**問題**: ハードウェアはひとつしか持てない --- アプリケーションは*たくさん*生成できる

デスクリプタヒープの切り替えは(現在のハードウェアでは)ハザードになり得る

- GPUはヒープを切り替える前にワークを空にし[drain]なければならない
- CBV/SRV/UAV **と**サンプラヒープに適用する
- (冗長な変更はフィルタリングされる)
- D3D: CLごとに`SetDescriptorHeap`を呼ばなければならない

# <font color='red'>🚩</font><font color='green'>デスクリプタヒープ</font> --- NVIDIA[<font color='red'>🚩</font><font color='green'>Descriptor Heap</font> - NVIDIA]

デスクリプタ(すべてのヒープ)の総数がプールサイズより小さいとハザードを回避する

プールサイズ(<font color='green'>Kepler+</font>)

- CBV/UAV/SRV = 1048576
- サンプラ = 2048個 + 2032個(静的) + 16個(ドライバ所有)
- **注意せよ[NB]** [1048575|4095] -> [0xFFFFF|0xFFF] -> (32ビットにパックされる)

# <font color='red'>🚩</font>同期[<font color='red'>🚩</font> Synchronization]

考慮すべきGPU同期モデル

- 撃ちっぱなし[fire-and-forget]
- ハンドシェイク

CPUもやることがある

- ExecuteCommandLists(ECLs)はGPUワークを*スケジュール*する
- ***CPU***でのECLs間の隙間は***GPU***に翻訳できる

# <font color='red'>🚩</font>撃ちっぱなし(同期)[<font color='red'>🚩</font> Fire-and-Forget (Sync.)]

- ワークの開始はフェンスを介して同期される

# <font color='red'>🚩</font>撃ちっぱなし(同期)[<font color='red'>🚩</font> Fire-and-Forget (Sync.)]

- ワークの開始はフェンスを介して同期される
- しかし、いくつかのワークロードはフレームごとに変化する
- この変化が望ましくないワークペアリングを引き起こす
- 悪いペアリングはパフォーマンスに影響を及ぼすので、全体フレーム時間に影響を及ぼす

# <font color='red'>🚩</font>CPUレイテンシー(同期)[<font color='red'>🚩</font> CPU Latency (Sync.)]

- 同様のシチュエーション --- ここではCPUが役割を果たす

# <font color='red'>🚩</font>CPUレイテンシー(同期)[<font color='red'>🚩</font> CPU Latency (Sync.)]

- 同様のシチュエーション --- ここではCPUが役割を果たす
- ゲームはECLs間のCPUでのレイテンシーを導入する
- レイテンシーはGPUに翻訳できる
- 望ましくないワークペアリングなどを引き起こす…

# <font color='red'>🚩</font>ハンドシェイク(同期)[<font color='red'>🚩</font> Handshake (Sync.)]

- ワークペアリングの開始と終了を同期する
- ペアリング決定論を保証する
- いくつかの非同期の機会を捕らえ損なうかも(ハードウェア管理可能)
- あなたのコードをfuture proofにする！

# <font color='red'>🚩</font>同期 --- アドバイス[<font color='red'>🚩</font> Synchronization - Advice]

CPUは純真潔白ではない、目を配ろう

2つのGPU同期モデル

- 撃ちっぱなし :(
    - 反対: 非決定的なregimeペアリング
    - 賛成: 少ない同期 == より即時的なパフォーマンス(ベストケースのシナリオ)
- ハンドシェイク :)
    - 反対: 追加の同期がパフォーマンスを低下させるかも
    - 賛成: regimeペアリング決定論(いつでも)

決定論(と同様に正確性)のために同期する

# <font color='red'>🚩</font>非同期税[<font color='red'>🚩</font> Async. Tax]

非同期コンピュートに関するオーバーヘッドコスト

- 定量化: [AC無効化時(ms)] / [直列化したACの有効化時(ms)] %
    - グラフィクスAPIを介して手動でシリアライズする
- *ACゲインを簡単にノックアウトできる！*

# <font color='red'>🚩</font>非同期税 --- *根本的な原因*[<font color='red'>🚩</font> Async. Tax - *Root Cause*]

**CPU**:

- 非同期タスクを組織化/スケジュールする追加のCPUワーク
- 同期/ExecuteCommandListsのオーバーヘッド

**GPU**:

- 同期のオーバーヘッド
- ACのオン/オフの間で使われるシェーダが異なる
- 追加のバリア(キュー間同期)

# <font color='red'>🚩</font>非同期税 --- *アドバイス*[<font color='red'>🚩</font> Async. Tax - *Advice*]

最初に: CPUまたはGPUがボトルネックかどうかを決定する (GPUView)

**CPU**:

- フレームあたりのAPI呼び出しを数えて、ACのオン/オフの差を比較する
- スレッド毎のプロファイリングを通して差異を計測する

**GPU**:

- ACのオン/オフのシェーダのGPUコストを比較する
- 寄与部分の差異を調査する[Inspect difference contributors]

# ツール[Tools]

- APIタイムスタンプ: 非同期コンピュートを有効化/無効化する時間
- GPUView: (次スライドへ続く)

# GPU Viewその1[GPU View #1]

- 3D、コンピュート、コピーを使用
- フレーム境界 @ Flipキューパケット
- フレーム毎のグラフィクスにオーバーラップするコンピュート

# GPU Viewその2 --- マーカー[GPU View #2 - Markers]

**注意せよ** Ctrl+eで開く

**説明**

- *Time*: GPUの正確な時間
- *DataSize*: Dataのバイトサイズ
- *Data*: PIXBegin/EndEventでemitされたイベント名
    - バイト配列 -> ASCII/Unicode
    - 手動ステップ :(

# GPU Viewその3 --- イベント[GPU View #3 - Events]

**CPUタイムライン**:

- `ID3D12Fence::Signal`
    - DxKrnl --- SignalSynchronizationObjectFromCpu
- `ID3D12Fence::Wait`
    - DxKrnl --- WaitSynchronizationObjectFromCpu

**GPUタイムライン**:

- `ID3D12CommandQueue::Signal`
    - DxKrnl --- SignalSynchronizationObjectFromGpu
- `ID3D12CommandQueue::Wait`
    - DxKrnl --- WaitSynchronizationObjectFromGpu

# ありごとうございました[Thanks \\0]

ご質問は？
