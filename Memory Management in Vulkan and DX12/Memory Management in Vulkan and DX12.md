---
title: Memory Management in Vulkan and DX12 [@Sawicki2018]
---
# Memory Management in Vulkan and DX12

# アジェンダ

- はじめに
- メモリタイプ
- Tips & Tricks
- ライブラリ
- おわりに

# はじめに

# 挑戦

- 前世代のAPI(OpenGL、DirectX11)は自動的にメモリを管理する
    - リソース(例えば、テクスチャや定数バッファ)を生成すると、補助記憶は自動的に割り当てられる
    - `ID3D11Texture2D* pTexture; pD3D11Device->CreateTexture2D(&desc, nullptr, &pTexture);`
- 新しいAPI(Vulkan、DirectX12)はより低レベルであり、明示的なメモリ管理を必要とする

# 挑戦

それは今やあなたが責任を負うところである

# 利点

- 明示的なメモリ管理は以下を可能にする
    - メモリのより良い管理
    - 特定プラットフォームに対するより良い最適化
    - 一時リソースの別名付け(重複)

# メモリタイプ

# メモリタイプ: NVIDIA

例: NVIDIA GeForce GTX 1080 Ti

- ビデオメモリ: D3D12_MEMORY_POOL_L1
    - ヒープ0 (Size = 10.87GiB, Flags = DEVICE_LOCAL)
        - タイプ7～8 (Flags = DEVICE_LOCAL)
- システムメモリ: D3D12_MEMORY_POOL_L0
    - ヒープ1 (Size = 16GiB, Flags = 0)
        - タイプ0～6 (Flags = 0)
        - タイプ9 (Flags = HOST_VISIBLE | HOST_COHERENT)
        - タイプ10 (Flags = HOST_VISIBLE | HOST_COHERENT | HOST_CACHED)

# メモリタイプ: Intel

例: Intel Iris Plus Graphics 640

- ユニファイドメモリ
    - ヒープ0 (Size = 3.57GiB, Flags = DEVICE_LOCAL)
        - タイプ0 (Flags = DEVICE_LOCAL | HOST_VISIBLE | HOST_COHERENT)
        - タイプ1 (Flags = DEVICE_LOCAL | HOST_VISIBLE | HOST_COHERENT | HOST_CACHED)

# メモリタイプ: AMD

例: AMD Radeon RX "Vega"

- ビデオメモリ
    - ヒープ0 (Size = 7.75GiB, Flags = DEVICE_LOCAL)
        - タイプ0 (Flags = DEVICE_LOCAL)
    - ヒープ2 (Size = 256MiB, Flags = DEVICE_LOCAL)
        - タイプ2 (Flags = DEVICE_LOCAL | HOST_VISIBLE | HOST_COHERENT)
- システムメモリ
    - ヒープ1 (Size = 16GiB, Flags = 0)
        - タイプ1 (Flags = HOST_VISIBLE | HOST_COHERENT)
        - タイプ3 (Flags = HOST_VISIBLE | HOST_COHERENT | HOST_CACHED)

# DEVICE_LOCAL

D3D12_HEAP_TYPE_DEFAULT

- **ビデオメモリ**。GPUからの高速アクセス
- CPUからの直接アクセスなし --- マッピングできない

# DEVICE_LOCAL

- GPUによって定期的に読み書きされるリソースに良い
- CPUによって一度だけ(不変)、または、不定期にアップロードされ、GPUによって定期的に読まれるリソースに良い

# HOST_VISIBLE

D3D12_HEAP_TYPE_UPLOAD

- **システムメモリ**。CPUへアクセスできる --- マッピング可能
- キャッシュされない。書き込みはwrite-combining^[複数の書き込み命令が一度の書き込み操作にまとめて行われること]される
- GPUからアクセスできるが遅い
    - PCIeバスを介し、読み込みはGPUでキャッシュされる

# HOST_VISIBLE

- リソースのCPU側の(ステージング)コピーに良い --- 転送元として用いられる
- CPUで書き込まれ、GPUで一度だけ読まれるデータ(例えば、定数バッファ)はうまく働く(常に計測！)
    - GPUでのキャッシュが役立つかもしれない
- GPUで読まれる大きなデータ --- 最終手段としてここに置いておく
- GPUで読み書きされるおおきなデータ --- ここにあるべきでは到底ない

# DEVICE_LOCAL + HOST_VISIBLE

- **ビデオメモリの特殊なプール**
- AMDでのみ開示される。256MiB
- GPUからの高速アクセス
- CPUへアクセスできる --- マッピング可能
    - ビデオメモリに直接書き込まれる
    - 書き込みはwrite-combiningされるかもしれない
    - キャッシュされない。そこから読み込めない

# DEVICE_LOCAL + HOST_VISIBLE

- CPUで定期的に更新され(動的)、GPUで読まれるリソースに良い
- CPUとGPUの両方で直接アクセスする --- 明示的な転送を行う必要がない
- DEVICE_LOCALが小さく、oversubscribeされる場合に代替案として用いる

# HOST_VISIBLE + HOST_CACHED

D3D12_HEAP_TYPE_READBACK

- **システムメモリ**
- キャッシュされたCPUの読み書き(書き戻し)
- PCIeを介したGPUアクセス
    - GPU読み込みはCPUキャッシュを覗き見る[snoop]

# HOST_VISIBLE + HOST_CACHED

- GPUで書かれ、CPUで読まれるリソースに良い --- 計算結果
- CPUとGPUの両方で直接アクセスする --- 明示的な転送を行う必要がない
- CPUで無作為に読み込みやアクセスされるリソースに用いる

# メモリタイプ: AMD APU

- AMDの統合グラフィクスは、AMDの外付けGPUのように、様々なメモリタイプを報告する
- 報告されるDEVICE_LOCALヒープは、0Bから数GiBまで、任意の大きさをとり得る

# メモリタイプ: AMD APU

- メモリは実際にはユニファイドである --- すべてのヒープが等しく高速である
- 統合グラフィクスを検出した場合 `VK_PHYSICAL_DEVICE_TYPE_INTEGRATED_GPU`
    - すべてのメモリヒープの大きさを一緒くたに数える
    - 要件を満たすメモリタイプが何であれリソースを配置する

# Tips & Tricks

# サブアロケーション

- リソースごとに別のメモリブロックを割り当てない(DX12: `CreateCommittedResource`)
    - 最大割り当て数の小さな限界値(例、4096)
    - 割り当ては低速である
- カクつき[hitching]を回避するためにゲームプレイ中にはメモリブロックを割り当てたり解放したりしない方が良い
    - 必要なら、バックグラウンドのスレッドで行うことができる

# サブアロケーション

- より大きなブロックを割り当てて、リソース用に範囲をサブアロケートする (DX12: `CreatePlacedResource`)
    - 256MiBが良いデフォルトのブロックサイズである
    - 1GiB以下のヒープでは、より小さいブロックを用いる (例、ヒープサイズ / 8)

# Over-commitment

- 物理ビデオメモリの最大量を超えると何が起こる？
    - ドライバに依存する
        - 割り当てが失敗するかも (`VK_ERROR_OUT_OF_DEVICE_MEMORY`)
        - 割り当てが成功するかも (`VK_SUCCESS`)
            - いくつかのブロックが静かにシステムメモリに移動させられる
    - とはいえ、ブロックはシステムメモリに移されるかもしれない
        - あなたは一人じゃない --- 他のアプリケーションがビデオメモリを使える
        - GPUのシステムメモリに移されたブロックを使うとパフォーマンスが低下する

# Over-commitment --- Vulkan

- メモリヒープのサイズ: VkMemoryHeap::size
- メモリブロックのレジデンシーを手動で制御する既知の方法はない
- 使用済み、または、利用可能なフリーメモリの量を問い合わせる既知の方法はない
    - 推定する必要がある

# Over-commitment --- Vulkan

- VkDeviceMemoryブロックのサイズを合計する
- メモリを占有する暗黙的なリソースについて覚えておく
- 余剰分を解放したままにする (例、DEVICE_LOCALの20%、DEVICE_LOCAL+HOST_VISIBLEの33%)

# Over-commitment --- DX12

- メモリタイプのサイズ: DXGI_ADAPTER_DESC
- プログラムに対する現在の使用量と利用可能な予算: DXGI_QUERY_VIDEO_MEMORY_INFO
- 通知に対して登録できる: IDXGIAdapter3::RegisterVideoMemoryBudgetChangeNotificationEvent

# Over-commitment --- DX12

- ビデオメモリに割り当てたブロックをページイン/アウトできる
    - ID3D12Device::Evict, MakeResident
    - ID3D12Device3::EnqueueMakeResident
- リソースにレジデンシーの優先度を設定できる
    - ID3D12Device1::SetResidencyPriority
- 最小必要めメモリについてDX12に知らせることができる
    - IDXGIAdapter3::SetVideoMemoryReservation

# マッピング

- メモリブロック全体を永続的にマッピングしておくことは通常は大丈夫
    - GPUで用いる前にアンマップする必要がない
- 例外
    - **Vulkan、AMD、Windowsバージョン10以前**: SubmiSubmitかPresentの呼び出しの時にマップされたままでいるDEVICE_LOCAL+HOST_VISIBLEメモリブロックはシステムメモリに移される
    - 大量の大きなメモリブロックをマップしたままにすると、安定性や**デバッグツール**のパフォーマンスに影響する

# 転送

- コピーキューは**PCIe**を介する効率的な転送のために設計されている
    - レンダリングフレームとさえ非同期に、3Dレンダリングと並行して用いる。テクスチャストリーミングに良い
    - バックグラウンドでのGPUメモリのデフラグメンテーションにも用いる
    - データがグラフィクスキューで必要になるずっと前に転送を行う
- **GPUから**(同じ)**GPUへ**のコピーはグラフィクスキューの方がより高速である
    - グラフィクスキューが転送結果を待つ必要がある場合に用いる

# ライブラリ

# Direct3D Residency Starter Library

- [https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12Residency](https://github.com/Microsoft/DirectX-Graphics-Samples/tree/master/Libraries/D3DX12Residency)
- Microsoftのライブラリ
    - MITライセンス
    - 統合しやすい --- 単一のC++ヘッダ
- DX12ヒープ/committedリソースのレジデンシーを管理する
- 本質的にDX11アプリケーションが得たようなのと同じメモリ管理挙動を実装する

# Direct3D Residency Starter Library

- ExecuteCommandListsを呼ぶ代わりに
    - 使うリソースのリストと一緒に実行されるコマンドリストを渡す
    - ライブラリは
        - メモリ予算をDXGIに問い合わせる
        - 最近最も使われなかったリソースに対してEvictを呼び出す
        - 使われているリソースに対してMakeResidentを呼び出す
        - ExecuteCommandListsを呼び出す

# Vulkan Memory Allocator

- [https://gpuopen.com/gaming-product/vulkan-memory-allocator/](https://gpuopen.com/gaming-product/vulkan-memory-allocator/)
- AMDのライブラリ
    - MITライセンス
    - 統合しやすい --- シングルヘッダ
    - (Vulkanと同じスタイルの)Cのインターフェイス、C++での実装
    - よくドキュメント化されている
- すでにいくつかのAAAタイトルで使われている
- 最終バージョン2.0.0がリリースされている

# Vulkan Memory Allocator

- 意図された使用方法に基づいて正しい最適なメモリタイプを選ぶのに役立つ関数
- メモリブロックを割り当てて、取っておいたり、それらの部分をユーザーに戻したりする関数
    - ライブラリは割り当てたメモリブロックやその中の範囲の使用不使用を追跡し続ける
    - アライメントやバッファ/イメージ粒度を尊重する

# Vulkan Memory Allocator

- イメージ/バッファを生成して、それ用のメモリを割り当てて、一緒にバインドする関数 --- オールインワン

# VmaDumpVis.py

- Vulkan Memory AllocatorのJSONダンプを可視化する補助ツール

たった今リリースした！

# おわりに

# おわりに

- 新しいグラフィクスAPI(Vulkan、DirectX 12)はより明示的なメモリ管理を必要とする
    - リソース生成は複数段階の処理である
    - 以前のドライバマジックは今やあなたの制御下にある
- GPU間の菜を取り扱う必要がある
- 良いプラクティスに従うことで、いずれのGPUでも最適なパフォーマンスを達成できる
- このタスクに役立つ可能性があるオープンソースのライブラリがある

# 参考文献

# ありがとうございました

# 免責 & 帰属

# AMD

# Backup

# 専用割り当て

- あるリソースはより大きいブロックからサブアロケートされた領域ではなくそれ自身の専用メモリブロックから恩恵を受けるかも
    - ドライバは追加の最適化を使うかも
- 以下のために使う
    - レンダターゲット、デプスステンシル、UAV
    - 非常に大きいバッファやイメージ(数十MiB)
    - 実行時にリサイズ(解放と再割り当て)が必要かもしれない大きな割り当て

# 専用割り当て

- DX12: ID3D12Device::CreateCommittedResource関数
- Vulkan: VK_KHR_dedicated_allocation拡張

# キャッシュ制御

- Vulkan: HOST_COHERENTフラグを持たないいずれかのメモリタイプは手動のキャッシュ制御を必要とする
    - CPUでの読み込みの前にvkInvalidateMappedMemoryRanges
    - CPUでの書き込みの後にvkFlushMappedMemoryRanges
- 実践では、すべてのPCのGPUベンダ(AMD、Intel、NVIDIA)はHOST_VISIBLEであるすべてのメモリタイプでHOST_COHERENTをサポートする
    - 現在のWindowsPCでこの心配をする必要はない

# エイリアシング

- 異なるリソースをエイリアスすることができる --- それらを同じ、または、重複するメモリ範囲にバインドする
    - メモリを節約する
    - フレームの一部でのみ用いられる一時メモリ(例、レンダターゲット)に良い
    - メモリが異なるリソースで用いられた後、未初期化状態としてリソースを扱う
        - 行うていることを本当に知っていないならば

# その他

- Vulkan: メモリ要件(例:サイズ)は(フォーマット、幅、高さ、ミップレベル、など)同じパラメータで生成されたときでさえ異なるリソース(例:イメージ)で変化し得る
    - 実際に本当に起っているので、その対策を。結果をキャッシュせず、リソースごと問い合わせる
- そうしなければならない場合を除いて、TILING_LINEAR(DX12: LAYOUT_ROW_MAJOR)を持つイメージを使わない
    - TILING_OPTIMAL(DX12: LAYOUT_UNKNOWN)を使う
    - イメージ-バッファ間のコピー

# その他

- VK_IMAGE_LAYOUT_GENERAL(D3D12:D3D12_RESOURCE_STATE_GENERIC_READ)を避ける。常に適切なVK_IMAGE_LAYOUT_\*_OPTIMALにイメージを遷移する
- レンダターゲットテクスチャではVK_SHARING_MODE_CONCURRENTを避ける。DCC圧縮が無効化される
    - VK_SHARING_MODE_EXCLUSIVEのほうが良く、明示的なキューファミリーの所有権転送バリアを行う
- VK_IMAGE_CREATE_MUTABLE_FORMAT_BITを避ける
    - 例えば線形/sRGBとして解釈するために異なるフォーマットを本当に必要とする場合、VK_KHR_image_format_list拡張とともに使う
