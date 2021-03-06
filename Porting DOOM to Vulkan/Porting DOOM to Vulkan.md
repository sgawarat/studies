---
title: Porting DOOM to Vulkan [@Gneiting2016]
bibliography: bibliography.bib
numberSections: false
---
# idTech 6

- OpenGL、Vulkan、PS4、XB1に対応。
- DOOMや今後のid Softwareのタイトルで使われる。
- すべてのプラットフォームで60Hz以上で動作する。
- HLSLに似たシェーダシンタックスを持つ。
    - ビルド時にPSSL/HLSL/GLSLに翻訳される。

# CPU

- 並列なコマンドバッファ生成。
    - フレームごとに複数の"コンテキスト"に分割する。
    - 各コンテキストは自分のコマンドバッファを持つ。
    - 各コンテキストごとにコマンドバッファを埋める複数のジョブを実行する。
    - フレームの最後のジョブはコマンドバッファをGPUに提出(submit)する。
- OpenGLはスレッド1つで順次実行する。
    - いくつかのシーン準備処理は依然としてジョブ内にある。

# GPU

- Clustered Forward Shadingで描画する。いくつかはDeferredで。
- ほとんどのジオメトリで同じシェーダを使う。
    - 同じテクスチャの集まりも使う。(virtual texturing)
    - ステート変更はほとんどない。
- ポストプロセスは広範囲に及ぶ。
    - DoF、テンポラルAA、SSDO、モーションブラー、など。
- 大量の非同期コンピュート。
    - DXTエンコード、パーティクルとポストプロセス。

# Porting to Vulkan

- 初期バージョンは2015年に始まった。
    - Vulkanバックエンドコードのほとんどが書かれた。
    - はじめてトライアングルがレンダリングできた。
- 2016年3月の後半に再び拾い上げた。
- ゲームのローンチのときにはほとんどが動作していた。
    - RenderDocが助けてくれた。現在ではさらに良くなっている。
- 小さな問題がリリースを遅らせた。
    - ドライバの問題。
    - スワップチェーンが驚くほどうまくいかない。

- 当時はバリデーションレイヤは信用ならなかった。
- 間違って報告されるエラー(false errors)が多発。
- バリデーションコードを自分で書かなければならなかった。
- 現在ではバリデーションレイヤは良くなっている。
- 依然としてデバッグのための自家用バリデーションを持つのは良いこと。

# Shaders

- GLSLトランスレータはすでにある。
    - ただし、OpenGLは名前でバインドする。
    - Vulkanはパイプライン生成するときのバインディングIDを使う。
- 可能ならばAMDの拡張を使う。
    - すべてのシェーダのバリアントに対応する。
    - AMD_shader_ballotとAMD_gcn_shader。

- 正規化されたクリップ空間は上下逆さま。
    - シェーダジェネレータは各頂点シェーダの最後にY軸の符号を入れ替える。
    - これを修正する拡張いれてくれない？[^1]
    - プラットフォームの差異は時間の無駄。
- Zの範囲は[0, 1]が良い。

[^1]: で　き　ま　し　た　。(AMD_negative_viewport_height)

# Pipelines & States

- 古いスタイルのAPIのような抽象化レイヤを用いる。
- ステートフルAPIのエミュレートとステートの追跡を行う必要がある。
- パイプライン、レンダパス、フレームバッファのステートにハッシュテーブルを使う。
    - 考えていたよりオーバーヘッドが小さい。
- シザー、ビューポート、ステンシル、デプスバイアスは動的ステートで。
- ゲーム全体でグラフィクスパイプラインの総計は350程度しかない。

- パイプライン生成は高価な処理。
    - 実行時にルックアップしそこなうのは許容されない。
    - あるパイプラインではコンパイルするのに100ミリ秒以上かかる。
- 解決策
    - ゲームをプレイして、ステートをディスクにシリアライズする。
    - スタートアップにパイプラインをコンパイルするジョブをローンチする。
    - かなりロバスト。パイプラインを取り損なってもプレイヤーに対してストールを引き起こすだけ。

# Descriptor Sets

- プレイ中はVulkanオブジェクトを破壊しない。
    - ジオメトリは静的にロードされる。
    - テクスチャは仮想化される。
- デスクリプタのハッシュテーブルでやり過ごす。
- 各組み合わせごとにひとつの大きなデスクリプタセットを使う。
- Vulkanハンドルが破壊されたら、完全なテーブルをフラッシュする。
    - レベルのロード＆アンロード、など。
- 通常で約3000から4000のデスクリプタセットを使う。

- 動的なuniformはリングバッファに書き込む。
- アトミックを使ってリングバッファからスレッドセーフに割り当てる。
    - シンプルさのために、256バイトにアライメントした割り当てを行う。
- UNIFORM_BUFFER_DYNAMICにバインドする。
    - オフセットは`vkCmdBindDescriptorSets`の引数としてセットする。
- スキニングデータにもUNIFORM_BUFFER_DYNAMICを使う。
    - 焼きこまれた範囲が問題になる。
    - すべてを64kBの範囲に収めてやり過ごす。
    - 代わりに、さらにおおくのデスクリプタセットを使う方法もあった。

# Multithreading

- たいていはコンソールからの素直なポーティング。
- イメージレイアウトが問題になる。
- コンテキストごとにダブルバッファリングしたコマンドバッファを持つ。
- ステートのハッシュテーブルにリードライトロックを使う。
    - ハズレを引かなければ、絶対にブロックしない。

# Image layouts & barriers

- イメージレイアウトはすごく頭を悩ませる。
    - フレームごとに25個以上のバリア。
    - 何百ものレイアウト変更。
- できるだけ多くのバリアを組み合わせる。
- 最後のイメージステートを知るのは難しい。
    - コードでは新しいステートのみを特定する。
- 並列化すると完璧に自動追跡するのは無理。

- 各コンテキスト/コマンドバッファの内部で自動追跡する。
- イメージはコマンドバッファをまたいではあまり使わない。
- フレーム開始時: 追跡の失敗を修正するためにコマンドバッファの開始のステートをセットする。
- フレーム終了時:
    - 遷移を行い、次のフレームステートを決定する。
    - イメージ遷移を検証する。
- 今は`vkCmdSetEvent`や`vkCmdWaitEvents`は使っていない。

# Memory

- シンプルなブロックアロケータ。
    - 最大128MBの部分に分ける。
    - 割り当てが成功するまでより小さい割り当てを試す。
    - または、VRAMの割り当てに失敗したら、システムメモリにフォールバックする。
    - リサイズ可能なイメージは個別に割り当ている。
- NVIDIAでは2GBに強制されるという問題がある。
    - いまでは多くが修正されている。
    - 可能ならばNV_dedicated_allocationを使う。

- すべてのアップロードは共通のマネージャを通して行う。
- ホストのステージングメモリはダブルバッファリングする。
- 各ステージングバッファは以下に関連している。
    - コマンドバッファ。
    - フェンス。
- バッファがいっぱいになったら、コマンドバッファの終わりにフェンスを書き込み提出する。
- グラフィクスを提出するまえにhost visibleの範囲をフラッシュする。

# Synchronization

- どこでもダブルバッファリングしている。
    - CPUでコマンドバッファのフェンスを待つ。
    - レイテンシーを最小化する。
- GPUViewはともだち。
    - OpenGL/DX11のときよりさらに便利。
- スワップチェインはトリッキー。
    - AcquireとPresentが常に一致しているかを確かめる。
    - Acquireはできるだけあとで行う(ストールを避けるため)。

# Asynchronous Compute

- 無駄なGPUのアイドル時間を活用するのに役立つ。
    - 例: シャドウパスやデプスパスの間。
- GPUパーティクルやポストプロセス。
- ポストプロセスは次フレームの初めとオーバーラップする。
    - AMDではコンピュートキューからプレゼントする。
    - NVIDIAはドライバサポート待ち。
- レンダターゲットにはSHARING_MODE_CONCURRENTを使う。
    - 注意して行うこと。たぶん遅くなる。

# Results

- パフォーマンスの向上に非常に満足している。
- AMDではあるシーンでGPUの限界の60%から70%を達成した。
    - 非同期やintrinsicsなしでもOpenGLより速い。
- NVIDIAのGPUでも同じくらい。
- レンダリングのCPU制限はほぼなくなった。
    - パワーセーブモードで60Hz以上が出ているという報告もある。
- 大きな可能性がある。

# Future Work

- フレームの開始時にイメージバリアとイメージレイアウトを準備する。
- ハッシュを取り除いて、ステートのハイレベルコードを認識するようにする。
- ゲーム中にどのパイプラインが使われているかを正確に知る。
- レンダパス(とサブパスとレイアウト遷移)をもっとうまく使う。

- バリアを分ける。(`vkCmdSetEvent`/`vkCmdWaitEvents`)
- コマンドバッファの再利用。(例: ディファードパスやポストプロセス)
- さらなる非同期コンピュート。
- 非同期転送。

# References
