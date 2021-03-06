---
title: Practice DirectX 12 - Programming Model and Hardware Capabilities [@Thomas2016]
---
# Practice DirectX 12

# アジェンダ[Agenda]

- DX12ベストプラクティス
- DX12ハードウェア能力
- 質問

# 期待されること[Expectations]

- DX12は誰のため？
    - 最大限のGPU&CPUパフォーマンスを達成することを目的とする
    - エンジニアリング時間を投資できる
    - 万人向けではない！

# エンジンの懸念事項[Engine Considerations]

- IHV固有パスの必要性
    - できなければDX11を使う
- アプリケーションはドライバやランタイムの部分を置き換える
    - ひとつのコードがすべてのコンソールで正常に動作するとは思えないし、PCでも似たようなもの
    - アーキテクチャ固有パスを検討する
- <font color="Green">NVIDIA</font>と<font color="Red">AMD</font>固有のものに注意

# ワークサブミッション[Work Submission]

- マルチスレッディング
- コマンドリスト
- バンドル
- コマンドキュー

# マルチスレッディング[Multi-Threading]

- DX11ドライバ:
    - レンダリングスレッド(プロデューサー)
    - ドライバスレッド(コンシューマー)
- DX12ドライバ:
    - ワーカースレッドをスピンしない
    - コマンドリストインターフェイスを介して直接的にコマンドバッファを構築する
- すべてのコアに渡ってエンジンがスケールすることを確認する
    - タスクグラフアーキテクチャがもっとも上手く動作する
    - コマンドリストをサブミットする1つのレンダリングスレッド
    - 並列にコマンドリストを構築する複数のワーカースレッド

# コマンドリスト[Command Lists]

- コマンドリストは他がサブミットされてても構築できる
    - サブミッションやプレゼント中にアイドリングしない
    - コマンドリストの再利用が許可されているが、アプリケーションは同時利用をやめる責任がある
- あまり多くのコマンドリストにワークを分割しすぎない
- 目星(フレーム毎):
    - 15〜30のコマンドリスト
    - 5〜10の`ExecuteCommandLists`呼び出し

# コマンドリストその2[Command Lists #2]

- 各`ExecuteCommandLists`は固定のCPUオーバーヘッドを持つ
    - この呼び出しの内部でフラッシュが誘発する
    - なので、コマンドリストをバッチ処理する
- 各`ExecuteCommandLists`に少なくとも200us、できれば500usのGPUワークを配置してみる
- OSのスケジューリングレイテンシーを隠蔽するのに十分なワークをサブミットする
    - `ExecuteCommandLists`呼び出しが少ないと、OSのスケジューラーが新しいものをサブミットできるようになるよりも早くに完了する

# コマンドリストその3[Command Lists #3]

- 例:
    - 十分でないワークがサブミットされる場合何が起こる？
        - 強調した`ExecuteCommandLists`は実行するのに約20usかかる
        - OSはやって来るワークをスケジュールするのに約60usかかる
        - == 40usのアイドル時間

# バンドル[Bundles]

- そのフレーム内で早期にワークをサブミットする良い方法
- GPUにおいてバンドルで本質的に速くなることはない
    - これらを賢く使おう！
- コマンドリストの呼び出しから状態を継承する --- 長所を生かす
    - ただし、引き継いだ状態のすり合わせはCPUかGPUのコストがかかるかも
- 良好なCPUブーストをもたらす
    - <font color="Green">NVIDIA:</font> 5つ以上の同じドロー/ディスパッチを繰り返すなら、バンドルを使う
    - <font color="Red">AMD:</font> CPU側でもがき苦しんでいる場合にのみバンドルを用いる

# マルチエンジン[Multi-Engine]

- 3Dキュー
- コンピュートキュー
- コピーキュー

# コンピュートキューその1[Compute Queue #1]

- 細心の注意を払って用いること！
    - 正しく用いれば、現時点で最大10%の得が見込める
- これがパーフォーマンス的に得であることを常にチェックする
    - 非同期でないコンピュートパスを維持する
    - 雑にスケジュールされたコンピュートタスクは最終的にロスになり得る
- ハイパースレッディングって覚えてる？　あれと似たようなルールが適用される
    - 2つのデータヘビーなテクニックは、例えば、キャッシュのような、リソースを抑圧する
- ペアリングに適したテクニックがGPUを十分に活用しない場合、まずは「なぜ利用率がクソなのか」を問おう
    - 非同期コンピュートに移行する *前に* まずコンピュートジョブを最適化する

# コンピュートキューその2[Compute Queue #2]

- 良いペアリング
    - グラフィクス: シャドウレンダリング(I/Oに制限される)
    - コンピュート: ライトカリング(ALUヘビー)
- 悪いペアリング
    - グラフィクス: Gバッファ(帯域に制限される)
    - コンピュート: SSAO(帯域に制限される)

(テクニックペアリングは1対1にしなければならないことはない)

# コンピュートキューその3[Compute Queue #3]

- 無制限にスケジューリングを行うと、残念なテクニックペアリングが行われることがある
    - 利点
        - 実装が単純
    - 欠点
        - フレーム間の非決定論
        - ペアリング制御の不足

# コンピュートキューその4[Compute Queue #4]

- フェンスの賢い使用を通じた非同期コンピュートタスクの明示的なスケジューリングのほうが良い
    - 利点
        - フレーム間の決定論
        - アプリケーションがテクニックペアリング全体を制御する！
    - 欠点
        - 実装するのに多少長めの時間がかかる

# コピーキュー[Copy Queue]

- バックグラウンドタスクにコピーキューを用いる
    - グラフィクスキューはグラフィクスのことに専念させる
- PCIe上でリソースを転送するためにコピーキューを使う
    - マルチGPUでは非同期転送のために必須
- コピーキューの完了までスピンしないようにする
    - 前もって転送を計画する
- <font color="Green">NVIDIA:</font> 深度＋ステンシルのリソースをコピーするときは気を付ける --- 深度のみのコピーは低速パスに当たるかも

# ハードウェアステート[Hardware State]

- パイプラインステートオブジェクト(PSO)
- ルートシグネチャテーブル(RST)

# パイプラインステートオブジェクトその1[Pipeline State Objects #1]

- 未使用のフィールドには賢明で矛盾のない既定を用いる
- ドライバはPSOコンパイルをスレッド化することを許可されていない
    - ワーカースレッドを用いてPSOを生成する
    - コンパイルは数百ミリ秒かかる

# パイプラインステートオブジェクトその2[Pipeline State Objects #2]

- 似たPSOを同じスレッドでコンパイルする
    - 例えば、異なるブレンドステートを持つ同じVS/PS
    - ステートがシェーダに影響を与えないなら、シェーダコンパイルを再利用するだろう
    - 同じシェーダをコンパイルする同時のワーカースレッドは最初のコンパイルの結果を待つだろう

# ルートシグネチャテーブルその1[Root Signature Tables #1]

- RSTを小さくし続ける
    - 複数のRSTを使う
    - すべてを統べる一つのRSTは存在しない…
- 頻繁に変更するスロットを一番目に置く
- ドローコールごとに1つのスロットを変更することを目指す
- 最小のステージセットになるようリソース可視性を制限する
    - 必要なければ`D3D12_SHADER_VISIBILITY_ALL`は使わない
    - `DENY_*_SHADER_ROOT_ACCESS`フラグを用いる
- RSTでは境界チェックが行われないことに注意！
- ルートシグネチャの変更後にリソースバインディングを未定義のままにしない

# ルートシグネチャテーブルその2[Root Signature Tables #2]

- <font color="Red">AMD:</font> ドローごとに変更する定数とCBVのみRSTに入れるべき
- <font color="Red">AMD:</font> ドローごとに1つ以上のCBVを変更するなら、テーブルに入れるほうが恐らく良い
- <font color="Green">NVIDIA:</font> すべての定数とCBVをRSTに配置する
    - RST内の定数とCBVはシェーダを速くする
    - ルート定数はCBVを生成する必要がない == CPU処理がより少ない

# メモリ管理[Memory Management]

- コマンドアロケータ
- リソース
- レジデンシー

# コマンドアロケータ[Command Allocators]

- 「記録スレッド数×バッファリングされるフレーム数＋バンドル用の追加プール」を目指す
    - 数百のアロケータがあるなら、それは間違い
- アロケータは成長するのみ
    - アロケータからメモリを回収することは一切できない
    - コマンドリストに割り当てたままにする方が好ましい
- 可能性のあるサイズによるプールアロケータ

# リソース --- 選択肢は？[Resources - Options?]

|タイプ|物理ページ|仮想アドレス|
|-|-|-|
|Committed|✅|✅|
|Heap|✅|❌|
|Placed|❌|✅|
|Reserved|❌|✅|

# Committedリソース[Committed Resources]

- リソースに一致するのに必要な最小サイズのヒープを割り当てる
- アプリケーションはリソースごとで`MakeResident/Evict`を呼び出さなければならない
- アプリケーションはOSのページングロジックの意のままである
    - `MakeResident`では、OSがどこにリソースを配置するかを決める
    - リターンするまでスタックする

# HeapとPlacedリソース[Heaps & Placed Resources]

- より大きなヒープの生成
    - 10〜100MBのオーダーで
    - Placedリソースを用いてサブアロケートする
- ヒープごとに`MakeResident/Evict`を呼び出す
    - リソースごとではない:)
- これはアプリケーションがアロケーションを追跡し続ける必要がある
    - 同様に、アプリケーションはヒープごとに解放する/使われるメモリの範囲を追跡し続ける必要がある

# レジデンシー[Residency]

- GPUへ/からメモリを`MakeResident/Evict`する
    - CPU+GPUコストは重大なので、`MakeResident`と`UpdateTileMappings`をバッチする
    - 必要なら、複数フレームをまたいで大きな負荷を償却する
    - `Evict`がすぐには何もしないかもしれないことを知っておく
- `MakeResident`は**同期的**である
    - `MakeResident`はリソースがレジデントになるまで返らないだろう
    - OSは動作を止めることがあり、リソースを配置する場所を理解するのにめちゃくちゃ多くの時間をかける。返るまでスタックする
    - ワーカースレッドで呼び出す

# レジデンシーその2[Residency #2]

- ビデオメモリはどれだけある？
    - `IDXGIAdapter3::QueryVideoMemoryInfo(...)`
    - フォアグラウンドアプリケーションはトータルビデオメモリのサブセットを保証される
        - 残りは可変であり、アプリケーションはOSからの予算変更に応じるべき
- アプリケーションは`MakeResident`の失敗をハンドルしなければならない
    - 通常は利用可能なメモリが十分にないことを意味する
    - ただし、十分なメモリがあっても発生し得る(フラグメンテーション)
- 非レジデントな読み込みはページフォールトである！致命的なクラッシュになる可能性が高い
- 十分なメモリがないときは何をする？

# ビデオメモリのオーバーコミットメント[Vidmem Over-commitment]

- システムメモリにオーバーフロー用ヒープを生成し、いくつかのリソースをビデオメモリのヒープから移す
    - ここで、アプリケーションはドライバ/OSと比べて利点がある。ほぼ間違いなく、何をビデオメモリに置き続けるのが一番重要かを知っていることである
- **アイデア**: 2つのインスタンスを走らせてアプリケーションをテストする

# リソース: 実践的Tips[Resources: Practical Tips]

- ターゲットのエイリアシングは大幅なメモリ節約になり得る
    - エイリアシングバリアを用いることを忘れずに！
- CymmittedなRTV/DSVリソースはドライバより好ましい
- <font color="Green">NVIDIA:</font> 読み込みがコヒーレントのときは、構造化バッファではなく定数バッファを用いる。例: Tiled Rendering

# 同期[Synchronization]

- バリア
- フェンス

# バリアその1[Barriers #1]

- リソースバリアがパフォーマンスバリアにならないようにする
- バリアを共にバッチする
- 最小のusageフラグセットを用いる
    - 冗長なフラッシュの回避
- 読み込み間[read-to-read]バリアを避ける
    - 後続のすべての読み込みに対して正しいステートでリソースを取得する
- 可能なときは"分割バリア[split-barriers]"を用いる

# バリアその2[Barriers #2]

- `COPY_SOURCE`は恐らく`SHADER_RESOURCE`より大幅に高価であるだろう
    - コピーキューでのアクセスを可能にする
- バリア数は大まかに書き込まれるサーフェス数の倍にすべき

# フェンス[Fences]

- GPUセマフォ
    - 例、GPUがリソースを使い終わることを確実にする
- 各フェンスは`ExecuteCommandLists`とだいたい同じCPUとGPUコストである
- フェンスが`ExecuteCommandLists`呼び出しあたり1回より細かい粒度でシグナルをトリガーする/進展すると思わない

# その他諸々[Miscellaneous]

- マルチGPU
- スワップチェイン
- Set Stable Power State
- ピクセル対コンピュート

# マルチGPU[Multi GPU]

- DirectX 12のAPIに組み込まれた機能
- cross-adapterとlinked-nodeのトレードオフ
    - 詳しいことは[Juha Sjöholmのトーク](https://www.gdcvault.com/play/1023131/Advanced-Graphics-Techniques-Tutorial-Day)を参照
- デバイス間でリソースを明示的に同期する
    - 適切な`CreationNodeMask`を用いる
- PCIeの帯域幅を忘れんといて
    - PCI 3.0 (8x) --- 8GB/s (期待値は約6GB/s)
    - PCI 2.0 (8x) --- 4GB/s (期待値は約3GB/s) ← まだまだ現役…
        - 例えば、4kのHDRバッファを転送すると、すぐに50/100FPSくらいまで下がってしまう

# スワップチェイン[Swap Chains]

- アプリケーションは明示的なバッファローテーションを行わなければならない
    - `IDXGISwapChain3::GetCurrentBackBufferIndex()`
- VSYNCオフを再現するには
    - `SettFullScreenState(TRUE)`
    - ボーダーのないフルスクリーンウィンドウを用いる
    - FLIPスワップチェインモード
- とってもリッチで新しいAPI！

# Set Stable Power State

- `HRESULT ID3D12Device::SetStablePowerState(BOOL Enable);`
    - パフォーマンスを落とす
    - チップ内のGPU構成要素のパフォーマンス比率が変化する
- **やらんといて！(頼んます)**

# ピクセル対コンピュート --- パフォーマンス[Pixel vs Compute - Performance]

- <font color="Green">NVIDIA</font>
    - ピクセルシェーダ
        - 共有メモリがない？
        - スレッドは同時に完了する？
        - 高頻度に定数バッファへアクセスする？
        - 2Dバッファに格納する？
    - コンピュートシェーダ
        - グループ共有メモリを使う？
        - アウトオブオーダーなスレッド完了を期待する？
        - レジスタを大量に使う？
        - 1D/3Dバッファに格納する？
- <font color="Red">AMD</font>
    - ピクセルシェーダ
        - 深度/ステンシルリジェクションから恩恵を受けられる？
        - グラフィクスパイプラインが必要？
        - カラー圧縮を活用したい？
    - コンピュートシェーダ
        - その他なんでも:)
- 以上のこれらのガイドラインからベストなパフォーマンスが得られる
- (非同期コンピュートを用いるパフォーマンス的な利点を検討する)

# ハードウェア機能[Hardware Features]

- Conservative Rasterization
- Volume Tiled Resources
- Rster Ordered Views
- Typed UAV Loads
- Stencil Output

# ハードウェア機能の状況[Hardware Features Stats]

# Conservative Rasterization

- プリミティブが触れるすべてのピクセルを描画する
    - 異なるTiers --- 仕様を参照
- GS通過前に可能なトリック、ただし、比較的遅い
    - [GPU Gems 2](https://developer.nvidia.com/gpugems/GPUGems2/gpugems2_chapter42.html)を参照
- ラスタライゼーションを使って、いくつかのナイスなテクニック実装できる
    - [Hybrid Raytraced Shadows](http://developer.download.nvidia.com/assets/events/GDC15/hybrid_ray_traced_GDC_2015.pdf)を参照

# Ray traced shadows in, "Tom Clancy's The Division", using conservative rasterization

# Volume Tiled Resources

- DX11.2からあるTiledリソースはボリューム/3Dリソースに使える
    - Tier 3のTiledリソース
- タイルは依然として64kB
    - そして、タイルマッピングがCPUから更新させるのに依然として必要
- かなりのメモリ/パフォーマンス的な利点がある
    - [Latency Resistant Sparse Fluid Simulation](https://www.gdcvault.com/play/1021767/Advanced-Visual-Effects-With-DirectX)

# Raster Ordered Views

- 順序のある書き込み
    - クラシックなユースケースはOIT
        - [k+ buffer](https://abasilak.github.io/#i3d2014)を参照
    - プログラマブルブレンディング
        - 固定ハードウェアに縛られない、完全にカスタムなブレンディング
- 気を付けて使おう！タダじゃないので
    - 衝突数を監視する

# Typed UAV Loads

- ついに、APIから32ビット制限がなくなった
- エンジンにコンソール固有パスを取り除けるかも
- UAVからのローディングはSRVからのローディングより遅い
    - SRVは未だにリードオンリーアクセスで使うので

# Stencil Output

- 実装？
    - ステンシルを用いるN変数のアルゴリズム？
        - 以前は、クリア＋Nパス
        - 現在は、シングルパス
- パフォーマンス的な懸念事項
    - 深度出力を用いる場合に匹敵する

# ご質問は？[Questions?]
