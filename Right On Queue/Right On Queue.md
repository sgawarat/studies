---
title: Right On Queue
---
# AMD

# RIGHT ON QUEUE

# 一般的なアドバイス[GENERAL ADVICE]

パフォーマンスの80%はそのハードウェアを理解することに起因する
20%はそのAPIを効率的に使用することに起因する

# Direct3D 12のCPUパフォーマンス[DIRECT3D 12 CPU PERFORMANCE]

- Direct3D 12は低いCPUオーバーヘッドのために設計されている
- マルチスレッド化されたコマンドリスト記録を用いる
- 実行時のヒープの生成/破棄を回避する
- CPU/GPU同期ポイントを回避する

# Direct3D 12のGPUパフォーマンス[DIRECT3D 12 GPU PERFORMANCE]

- Direct3D 11ドライバは過去8年かけて最適化されてきた
- はじめのDirectX 12移植はDirectX 11より大幅に低速である傾向がある
    - すべてのDirectX 12の恩恵を受けられるようエンジンを再設計する
    - 非同期キューはDirectX 11のパフォーマンスに打ち勝つのに役立つ
- アジェンダ
    - 一般的なパフォーマンスのアドバイス
    - デスクリプタセット
    - 複数の非同期キュー
    - バリアの理解
    - メモリ管理のベストプラクティス

# ズバリGCNとは[GCN IN A NUTSHELL]

- ハードウェアは変わっていない -- Direct3D 11のアドバイスが依然として当てはまる
    - 現在のAMDのハードウェアの要約
        - いくつかのCompute Units (CU)
            - FuryXでは64個
        - CUあたり4つのSIMD
        - SIMDあたり最大10個の実行中wave fronts
        - wave frontあたり64個のスレッド
    - VGPR数が多いと、wave front数が制限される可能性がある
        - CodeXLを用いる

# 一般的なパフォーマンスのアドバイス[General PERFORMANCE ADVICE]

<font color="Yellow"><b>ほとんどのパフォーマンスアドバイスは依然として当てはまる</b></font>

- カリング: GPUに必要のない作業を送らない
    - コンピュートトライアングルフィルタリングを検討する
    - 見に行こう: [Optimizing the Graphics Pipeline With Compute](https://www.gdcvault.com/play/1023109/Optimizing-the-Graphics-Pipeline-With)
- ソート: 不必要なオーバーヘッドを回避する
    - パイプライン(やパイプライン内部に使われるPS)でドローをソートする
    - 前から後へレンダリングする
- バッチ、バッチ、バッチ (ｺﾞﾒﾝﾔﾃﾞ)
    - 小さなドローコールはGPUを満たさない

# Direct3D 12のパフォーマンスアドバイス --- プロファイリング[DIRECT3D 12 PERFORMANCE ADVICE - PROFILING]

エンジン内パフォーマンスカウンタを追加する

- `D3D12_QUERY_TYPE_TIMESTAMP`
    - 結果を取得しようとしてストールしないように
- `D3D12_QUERY_DATA_PIPELINE_STATISTICS`
    - `VSInvocations / IAVertices`: 頂点キャッシュ効率
    - `CPrimitives / IAPrimitives`: カリングレート
    - `PSInvocatons / レンダターゲット解像度`: オーバードロー
    - `PSInvocations / CPrimitives`: Geometry bound?
        - 深度のみのレンダリングはPSを使わないことに気に留めておく
        - 深度テストは`PSInvocations`を減少させる

# デスクリプタセット[DESCRIPTOR SETS]

# デスクリプタセット[DESCRIPTOR SETS]

- ルートシグネチャ
    - 最大サイズ: 64 DWORD
    - 以下を含むことができる
        - データ(大量のスペースを要する！)
        - デスクリプタ(2 DWORD)
        - デスクリプタテーブルへのポインタ
    - 単一のデスクリプタヒープを維持する
        - リングバッファとして使う
    - 静的サンプラを使う
        - 最大2032個
        - 64 DWORD制限にカウントしない

# デスクリプタセット[DESCRIPTOR SETS]

- ドローごとに変化する、小さくて、すごく使う定数のみを直接ルートシグネチャに置く
- 更新頻度でデスクリプタテーブルを分ける
    - もっとも変化しやすい要素を最初に置く
- `D3D12_SHADER_VISIBILITY`フラグを用いる
    - マスクではなく
    - 厳密な可視性を設定するためにエントリーを複製する

# デスクリプタセット[DESCRIPTOR SETS]

- 起動時にルートシグネチャはSGPRにコピーされる
    - コンパイル時に定義されたレイアウト
    - 各シェーダステージに必要なもののみ

# デスクリプタセット[DESCRIPTOR SETS]

- 起動時にルートシグネチャはSGPRにコピーされる
    - コンパイル時に定義されたレイアウト
    - 各シェーダステージに必要なもののみ
    - 多すぎるSGPR → ルートシグネチャがローカルメモリにはみ出るだろう
- もっとも頻繁に変更されるエントリーを最初に
- デスクリプタテーブルのはみ出しを回避する！

# 非同期キュー[ASYNC QUEUES]

D3D12 --- 更なるパフォーマンスの解放

# キュータイプ[QUEUE TYPES]

TODO
