# DirectX 12 Optimization Techniques in Capcom's RE ENGINE

## RE ENGINEの最適化[RE ENGINE Optimization]

## アジェンダ[Agenda]

- 最適化
    - コンソール最適化のPCへの適用
    - DirectX 12のための最適化
- Tips

## 内製エンジンの背景[Background of in-house engine]

- RE ENGINE
    - カプコンの内製エンジン
    - コンソールとPCを対象とする
- 発売済み
    - バイオハザード7（RE7）
    - バイオハザード RE:2（RE2）
    - デビルメイクライ5（DMC5）

## 内製エンジンの背景[Background of in-house engine]

- RE ENGINEは"中間描画コマンド[intermediate drawing command]"を使う
    - プラットフォーム非依存のコマンド
    - プログラマがプラットフォームを知らなくても描画コマンドを書ける
- マルチプラットフォーム開発に役立つ
- マルチスレッドで描画コマンドを生成できる
- これら"中間描画コマンド"は生成後にソートされ、APIコマンドに翻訳される
- 描画順は優先度変数を用いて制御される（符号なし64ビット整数）
- ユーザーの裁量[discretion]でバッチ処理を行うことができる
- UAVOverlapやAsyncDispatchの同期タイミングを制御するのに役立つ

## RE ENGINEにおけるDirectX 12の実装[Implementation of DirectX 12 in RE ENGINE]

- RE7の製作[production]中に試行[trials]を始めたが、実装はしなかった
- RE2とDMC5はDirectX 12を実装している

## 最適化[Optimization]

- コンソール最適化のPCへの適用
    - MultiDrawを用いたオクルージョンカリング
    - UAVOverlap
    - Wave Intrinsics
    - 深度境界テスト
- DirectX 12に対する最適化
    - リソースバリアの削減
    - バッファ更新
    - RootSignature
    - メモリ管理

## 前後の比較[Comparison of before and after]

- 24%のフレーム時間の節約！

## コンソール最適化のPCへの適用[Adaptation of console optimizations to PC]

## テスト環境[Testing environment]

- RE2（2/15のパッチ）
- 1080p
- 主にRadeon RX480、一部Radeon R9 Fury X
- Radeon GPU Profiler 1.3.1.70、OCAT、PIX for Windows

## MultiDraw

- DirectX 12では、我々はExecuteIndirectを使う
    - 一度に複数の描画コマンドの実行を可能にする
    - メッシュ描画のオーバーヘッドを削減することを目標とする
- DirectX 11では、MultiDrawはAGSやNVAPIによってサポートされる

## 何か改善した？[Any improvements?]

- オーバーヘッド的には期待してたほどの改善はなかった
- ExecuteIndirectはGPUベースのオクルージョンカリングを実装するのに役立った

## GPUベースのオクルージョンカリングOFF[GPU-based occlusion culling OFF]

## GPUベースのオクルージョンカリングON[GPU-based occlusion culling ON]

## 参考までに[FYI]

- 取り得る2つの解法：ExecuteIndirectとPredicationコマンド
- ExecuteIndirect
    - 4バイトのアライメント
    - CountBufferでIndirectArgumentの実行数を制御する
- Predicationコマンド
    - 8バイトのアライメント - コンソールと非互換

## データ構造 - VisibleBuffer[Data structures - VisibleBuffer]

- "VisibleBuffer"を用いて管理される可視性
    - 実際には、RE ENGINEにおけるCountBufferそのもの
    - ByteAddressBuffer
    - 要素数はシーンにおけるメッシュの最大数に等しい
    - 各要素はメッシュごとの可視性を持つ
        - 可視なら0xffff、不可視なら0x0000

## データ構造 - メッシュデータ[Data structures - Mesh data]

- StructuredBuffer
    - AABB - CPUで生成またはGPUで生成
    - VisibleBufferのバイトオフセット
    - IndirectArgumentのバイトオフセット

## 可視性テスト[Visibility test]

- EarlyZで描画する
    - [earlydepthstencil]アトリビュート！
    - VisibleBufferに0xffffを格納する
- Wave単位で同じアドレスへ書き込むのを最小化する[@Drobot16]

```c
[earlydepthstencil]
void PS_Culltest(OccludeeOutput I) {
  uint hash = WaveCompactValue(I.outputAddress);
  [branch]
  if (hash == 0) {
    RWCountBuffer.Store(I.outputAddress, 0xffff);
  }
}
```

## 可視性テストの結果を適用する[Apply visibility test result]

- メッシュごとの描画に適用する
- MaxCommandCountを用いて描画数を指定する
    - VisibleBufferをCountBufferとする
        - CountBufferの0xffff：描画が有効（countはMaxCommandCount）
        - CountBufferの0：描画が無効

```c
void ExecuteIndirect(
    ID3D12CommandSignature* pCommandSignature,
    UINT                    MaxCommandCount,
    ID3D12Resource*         pArgumentBuffer,
    UINT64                  ArgumentBufferOffset,
    ID3D12Resource*         pCountBuffer,
    UINT64                  CountBufferOffset
);
```

## PIX上の結果[Result on PIX]

## メッシュ毎オクルージョンカリングOFF[Per mesh occlusion culling OFF]

## メッシュ毎オクルージョンカリングON[Per mesh occlusion culling ON]

## メッシュ毎オクルージョンカリングON[Per mesh occlusion culling ON]

緑色のOccluder平面

## メッシュ毎オクルージョンカリングOFF[Per mesh occlusion culling OFF]

## メッシュ毎オクルージョンカリングON[Per mesh occlusion culling ON]

## メッシュ毎オクルージョンカリングON[Per mesh occlusion culling ON]

期待されるほどにジオメトリがカリングされていない

## 改善の余地は？[Room for improvement?]

- 小物やキャラのメッシュに対して効果的
    - カリング手法はより小さなAABBユニットに対して効果的
- 大きなメッシュには効果は薄い
    - 大きなメッシュは常に見えている
    - より良い結果のためにメッシュを細かく[finely]分割する必要がある

## 大きなメッシュの自動分割[Automatic division of large mesh]

- 1バッチあたり256トライアングルに分ける
- 各バッチは連続するIndirectArgumentから成る
    - バッチごとにAABBを生成する

## 大量の極小描画コマンドによる問題[Issues with many micro drawing command]

- ほぼすべての描画が768インデックス以下に減る
- バッチ量が多いとパフォーマンス低下を引き起こす
    - ハードウェアに依存する
- IndirectArgumentsが連続する場合はコマンドをマージする

## メッシュ分割OFF[Mesh division OFF]

## メッシュ分割ON[Mesh division ON]

## メッシュ分割OFF[Mesh division OFF]

## メッシュ分割ON[Mesh division ON]

## 部分的Zプリパス[Partial Z-prepass]

- できるだけ少ないフラグメントシェーダを走らせるため
- すべてのメッシュでのZプリパスは高価
    - コストが利点を上回り得る
- カメラに近いメッシュのみにZプリパスを制限する
    - 自動分割モデルを再利用する

## 各手法の比較[Comparison of each method]

オクルージョンカリングとGバッファの時間[duration]（マイクロ秒）

## GPUベースのオクルージョンカリングの比較[Comparison of GPU-based occlusion culling]

- この時点ではパフォーマンスは上昇していない

## UAVOverlap

- DirectX 12では、依存性のないシェーダは並列に実行できる
- UAVバリアは曖昧な依存性がある
    - 読むのか書くのかはっきりしない
    - 各バッチが別個の場所に書き込む場合、並列に実行され得る
        - WAW（書き込み後書き込み）ハザードが回避可能であれば

## UAVOverlap

- コンピュートシェーダのディスパッチごとに制御可能なUAV同期
    - UAVの同期を無効化することで並列実行を可能とした
    - DirectX 11では、AGSやNVAPIを用いて同等の機能を導入することができる

```c
void dispatch(u32 threadGroupX, u32 threadGroupY, u32 threadGroupZ, bool uavResourceSyncDisable = false);
void dispatchIndirect(Buffer& buffer, u32 alignedOffsetForArgs, bool uavResourceSyncDisable = false);
```

## 比較：UAVオーバーラップ[]Comparison : UAV Overlap

- 全体的なパフォーマンス改善

## Wave Intrinsics

- シェーダのスカラ化は並列に動作するスレッドの比率を改善し得る
- ライティング、GPUベースのオクルージョンカリング、SSR…で使われる
- スカラ化については、[@Sousa16]を参照
- Wave Intrinsicsは不必要な同期を取り除くことでスカラ化の効率を改善する
- DirectX 11およびDirectX 12にてサポートされる
    - シェーダモデル5.1ではAGS Intrinsicを用いて
    - シェーダモデル6.0で使うこともできる

## 比較：Wave Intrinsics[Comparison : Wave Intrinsics]

- 全体的なパフォーマンス改善

## 深度境界テスト[Depth Bounds Test]

- 特定の深度範囲に深度をクランプする
    - 無関係な[extraneous]ピクセルシェーダを排除するために主に使われる
    - DirectX 12（Creators Update）とDirectX 11.3で使える
        - DirectX 11はAGSやNVAPIで
- RE ENGINEでは、デカールとライトシャフトに対して使われる

## デカール[Decals]

- 深度テストに失敗したピクセルで行う
- うまくいくと[preferably]完全に遮蔽されるときに処理を省略する
    - 深度境界テストを使って解決される

## デカールでの深度境テストの比較[Comparison of Depth Bounds Test for decals]

## コンソール最適化の比較[Console optimization comparison]

## DirectX 12に対する最適化[Optimization for DirectX 12]

## 最適化[Optimization]

- コンソールの最適化手法をPCへフィードバックする
    - MultiDraw
    - UAVOverlap
    - Wave Intrinsics
    - 深度境界テスト
- DirectX 12での最適化
    - リソースバリアの削減
    - バッファ更新
    - ルートシグネチャ
    - メモリ管理

## リソースバリアの削減[Reduction of resource barriers]

## 最適化なしのリソースバリア[Resource barrier without optimization]

- 最適化のない元々のビルドでは、バッチにリソースバリアを挿入していた
- すぐさま、描画コマンドの実行前に、現在のバッチで必要になるリソースバリアへ遷移する

## リソースバリア[Resoruce barriers]

- 大量のリソースバリア
    - 理由のひとつは、GPUベースのオクルージョンカリングがそれほどパフォーマンスを改善しなかったこと

## リソースバリア[Resoruce barriers]

- 多くのリソースバリアを伴う区間は効率的に処理できていない

## リソースバリアを減らす[Reducing resource barriers]

- リソースごとにサブリソースを考えることで最適化する
- すべての中間描画コマンドから最適なリソースバリアを手動で生成するのは困難
- 難しさ
    - GPUパフォーマンスを最大にすること
    - バグなしを維持すること

## コマンド解析のためのプリパスを追加する[Add pre-pass for command analysis]

- 自動でリソースバリアの一を計算する
    - 中間描画コマンドを解析する
- 中間描画コマンドは優先度でソートされる
    - リソースごとに時間別で[chronologically]描画コマンドの使い方を追跡できる
- 依存関係を持つバッチを解析すると、優先順位をずらすことでGPUの効率を容易に改善できる

## リソースバリアの圧縮[Resource barrier compaction]

- 前にある[precursor]リソースバリアを探す

## リソースバリアの圧縮[Resource barrier compaction]

- 前にある[precursor]リソースバリアを探す

## リソースバリアの圧縮[Resource barrier compaction]

- 前にある[precursor]リソースバリアを探す
    - できればバンドルする

## 利点/欠点[Advantage / Disadvantage]

- 利点
    - 内部実装やキャッシングを意識しなくて良くなる
    - 不必要なリソースバリアを減らす
- 欠点
    - コマンドをパースする時間が必要になる
        - PCはメチャ早です！

## 比較：リソースバリアの削減[Comparison : Resource barrier reduction]

## まだ足りない？[Still not enough?]

- バッファの更新にまだ非効率な区間がある

## まだ足りない？[Still not enough?]

- DMA転送中にドライバによって引き起こされる大量のリソースバリア

## 何が起こっていた？[What was going on?]

- グラフィクスキューでのバッファ更新
- CopyBufferRegion
    - GPUパーティクルバッファの更新
    - スキニング行列の更新
- CopyBufferRegionはDMA転送として実行される

## 何が起こっていた？[What was going on?]

- DMA転送が行われるとき、強力なキャッシュのフラッシュが行われた
    - L1キャッシュ、L2キャッシュ、Kキャッシュ
        - リソースバリアのバッチ処理は影響を与えない
- 取り得る解法
    - フレームあたり更新1回のみであれば、コピーキューで更新する
    - コンピュートシェーダを使って更新する
- 我々はコンピュートシェーダを使った

## コンピュートシェーダベースの更新[Compute shader based update]

```c
StructuredBuffer<uint> fastCopySoruce;
RWStructuredBuffer<uint> fastCopyTarget;

[numthreads(256, 1, 1)]
void CS_FastCopy(uint groupID : SV_GroupID, uint threadID : SV_GroupThreadID) {
  fastCopyTarget[(groupID.x * 2 + 0) * 256 + threadID.x] = fastCopySource[(groupID.x * 2 + 0) * 256 + threadID.x];
  fastCopyTarget[(groupID.x * 2 + 1) * 256 + threadID.x] = fastCopySource[(groupID.x * 2 + 1) * 256 + threadID.x];
}
```

## 定数バッファの更新の最適化[Optimization of constant buffer update]

- アップロードヒープを介してすべての定数バッファを更新する
    - 同じ定数バッファへの更新はリソースバリアとCopyBufferRegion（DMA転送）が必要
    - アップロードヒープへ新しい値を格納し、アップロードヒープのオフセットアドレスを取得する
- 定数バッファのみを使われるシェーダは参照オフセットアドレスのみが必要
    - リソースバリアとCopyBufferRegionはもはや必要ない

## CopyBufferRegionの削減の比較[CopyBufferRegion reduction comparison]

## 各手法の比較[Comparision of each method]

## Root Signature

- DirectX 12はDX11やコンソールと似たRootSignatureを用いる
- シェーダのビルド時ではなく、実行時に決定される
    - IHVごとにカスタマイズされた最適化を提供するため
    - AMDでは、テーブルとしてRootParamaterを使う
    - NVIDIAでは、定数バッファのアクセスを最適化するためにRootParamaterを使う

## メモリ管理[Memory management]

- 最初の実装では、メモリのEvictは約50%のメモリ使用率で始めていた
    - とても保守的
- ゲームプレイ中に多くのスパイクが発生した
    - バイオハザードRE:2では、キャラクターが動くたびにスパイクを引き起こす部屋ごとにローディングや除去[disposal]を制御する
    - ポーズメニュー用UIのロードでさえも発生した

## メモリ管理[Memory management]

- メモリが使い切られる[exhausted]までEvictしない
    - マイクロEvictsを抑制するため
- メモリ使用率が90%を超えたとき、参照されないメモリがEvictされる

## すべての最適化の後の比較[Comparison after all optimizations]

- 24%のフレーム時間の節約！

## DirectX11 / DirectX 12の比較[Comparison of DirectX 11 / DirectX 12]

- ゲーム中のバイオハザードRE:2をプロファイルする

## 今後の課題[Future works]

- 非同期コンピュート
    - コンソールでは使われている
    - 実装はPCと互換性がない
- シェーダモデル6.0
    - いくつかのテストや試行は終えた
    - 安定性を確認するための時間が足りない

## 最適化の再掲[Optimization recap]

- コンソール由来の最適化は有用であるにも関わらず[although]、単独では[by itself]適さない[inadequate]かもしれない
- リソースバリアのさくげんは重要
    - パフォーマンスに多大な影響を与える！
    - 他の最適化手法の効率性がリソースバリアによって影響を受ける可能性がある
- メモリ管理を徐々に行うのではなく一度にすべて行ったとき、ページングのスパイクは減少した
    - ゲームデザインによりけり
    - 90%程度の使用率でもうまく機能した

## Tips

## Pre-bake PipelineStateObject

- 実行時のPipelineStateObjectの生成は遅い
- 前もってPipelineStateObjectを事前に生成[pre-bake]できればなお良い

## Pre-bake PipelineStateObject

- 最終パッケージの前にPSOをpre-bakeする
    - エンジン上で生成されるアセットに含まれる

        ```c

        ```

- さいしょはRTV、DSV、インデックスのストライドは含めない
- 最終パッケージにPSOをpre-bakeするために集めた情報を使う
    - エンドユーザーに対してmuch smoother

## 実行時にPipelineStateObjectを読み込む[Load PipelineStateObjet at runtime]

- アセットのロード中にバックグラウンドでコンパイルする
    - コンピュートシェーダ：別のスレッドで即座に生成する
    - その他のシェーダ：収集した情報の中にあれば生成する
- しかし、PipelineStateObjectのビルドが前もって完了しなければ、CPUはブロックされる

## 品質保証（QA）[Quality Assurance (QA)]

- PCバージョンの品質保証はGPUのクラッシュに頻繁に悩まされる
    - CPU、GPU、ディスプレイ、などのような様々な要因
- しかし、GPUのクラッシュをデバッグするのにクラッシュダンプは役に立たなかった
    - トレースする方法がない
    - RE ENGINEはコマンドリストをリプレイする関数用意していない…今は
- DirectX 12では、WriteBufferImmediateを使う
    - 描画コマンドごとに実行するシェーダ名をバッファに読み戻す
    - クラッシュした時点で動作していたシェーダの名前を知ることができる
    - DirectX 11では、AGSが同様の関数としてBreadcrumbBufferをサポートする

## 謝辞[Acknowledgments]

- RE ENGINE開発チームの貢献とIHVのサポートに大きな感謝を
    - 多数のバグがドライバチームによって修正されました！

## ご質問は？[Questions?]

## 参考文献[References]
