# Leveraging Asynchronous Queues for Concurrent Execution

コンカレンシ（とそれを破壊するもの）を理解することは、モダンなGPUで最適化を行うときに極めて重要である。DirectX 12やVulkanのようなモダンなAPIは非同期的にタスクをスケジュールする能力[ability]を提供する。これは[which]、比較的少ない努力でより高いGPU使用率を可能にし得る。

## コンカレンシが重要である理由[Why concurrency is important]

レンダリングは呆れるほどに[embarrassingly]並列なタスクである。メッシュにおけるすべての三角形は並列に変形させる[get transformed]ことができ、オーバーラップしていない三角形は並列にラスタライズさせることができる。その結果として[consequentially]、GPUは大量の処理を並列に行うよう設計される。例えば、Radeon Fury X GPUは64個のコンピュートユニット（CU）から成り、その各々が4つのSingle-Instruction-Multiple-Dataユニット（SIMD）から成り、各SIMDが、我々が"wavefront"と呼ぶ、64スレッドのブロックを実行する。メモリアクセスのレイテンシはシェーダの実行において著しい[significant]ストールを引き起こし得るため、最大10個のwavefrontがこのレイテンシを隠すために同時に各SIMDにスケジュールされ得る。

**実行中[in flight]のwavefrontの実際の数がしばしば理論上の最大値より小さいことにはいくつか[several]の理由がある。これに対する最も一般的な理由は：**

- シェーダは大量のVector General Purpose Registers（VGPR）を用いる。例えば、シェーダが128個以上のVGPRを用いる場合、SIMDあたり1つのwavefrontしかスケジュールさせることができない。（この理由やシェーダが実行できるwavefront数の計算方法の詳細についてはGPR使用率を最適化するためのCodeXLの使い方の記事を参照ください）
- LDSの要件[requirements]：シェーダがスレッドグループあたり32KiBのLDSと64個のスレッドを用いる場合、これはCUあたり2つのwavefrontしか同時にスケジュールすることができないことを意味する。
- コンピュートシェーダが十分なwavefrontをspawnしない場合、または、大量のlow geometryのドローコールがスクリーン上のごくわずかのピクセルしかカバーしない場合、すべてのCUを飽和させるのに十分なwavefrontを生成するためにスケジュールされる処理が十分に存在しないかもしれない。
- すべてのフレームは正しいレンダリングを保証するための同期ポイントやバリアを含んでいる。これは[which]、GPUをアイドル状態にさせる。

**非同期コンピュートは、ともすれば使われないままになってしまうであろう[that would otherwise be left on the table]、これらのGPUリソースを活用する[tap into]ために使うことができる。**

以下の2つの画像はフレームの典型的な部分でのRadeon RX480 GPUのシェーダエンジンで起こることを可視化したスクリーンショットである。グラフは我々がゲームの最適化ポテンシャルを見極めるためにAMD内部で使うツールによって生成される。

画像の上部はあるCUの様々な部分の使用率を示す。

下部は様々なシェーダタイプが起動するwavefront数を示す。

1つ目の画像は約0.25msのG-Bufferレンダリングを示す。上部では、GPU、特にexport unitがとてもビジーなように見える。しかし、大事なこととして、完全に飽和している要素がCU内にはないことに注意したい。

(image1)

2つ目の画像は0.5msの深度のみのレンダリングを示す。左半分では、PSは使われておらず、これが[which]結果として非常に低いCU使用率となっている。中央近くでは、いくつか[some]のPS wavesがspawnされており、おそらくはアルファテストによる半透明ジオメトリのレンダリングに起因する（が、これらのグラフには可視化されていない）。右端[rightmost]の4分の1では、spawnされるwaves総数が0に落ちている区間がいくつか[few]ある。これは次のドローコールでテクスチャとして使われるレンダターゲットに起因する可能性があると思われる。つまり[so]、GPUは前のタスクが終わるのを待たなければならない。

(image2)

## GPU使用率を高めてパフォーマンス改善[Improved performance through higher GPU utilization]

これらの画像で確認できるように、典型的なフレームでは余っている[spare]GPUリソースが多分に存在する。新しいAPIはGPUでスケジュールされるタスク数に対するさらなる制御手段[control]を提供するよう設計される。異なることといえば[One difference is ...]、ほとんどすべてのコールが独立していると暗黙的に仮定され、描画処理[draw operation]が前のそれの結果に依存するときのように、バリアを指定して正しさを保証するのがその開発者次第[it's up to the developer]であるという点である。バリアのバッチ処理を改善するためにworkloadsをシャッフルすることで、アプリケーションはGPU使用率を改善し、フレームごとにバリアで消費されるGPUアイドル時間を減らすことができる。

GPU使用率を改善するもうひとつの方法[additional way]は非同期コンピュートである。そのフレームにてある地点で他のworkloadsとともに逐次的にコンピュートシェーダを実行する代わりに、非同期コンピュートは他の処理との同時実行を可能にする。これは上記の画像に見られる隙間のいくつか[some]を埋め、追加のパフォーマンスをもたらすことができる。

開発者がどのworkloadsが並列に実行され得るかを指定することを可能にするため、新しいAPIはアプリケーションがタスクをスケジュールするための複数のキューを定義することを可能にする。

キューは3つのタイプがある：

- コピーキュー (DirectX 12) / トランスファーキュー (Vulkan)：PCIeバス上でのデータのDMA転送
- コンピュートキュー (DirectX 12およびVulkan)：コンピュートシェーダの実行やデータのコピー、できれば[preferably]ローカルメモリ内で。
- ダイレクトキュー (DirectX 12) / グラフィクスキュー (Vulkan)：このキューはなんでもできる。つまり、レガシーAPIにおけるメインデバイスと似ている。

アプリケーションは同時使用のために複数のキューを生成できる。DirectX 12では、タイプごとに任意の数のキューを生成でき、一方のVulkanでは、ドライバはサポートするキューの数を列挙するだろう。

GCNハードウェア単一のジオメトリフロントエンドから成る。つまり、DirectX 12では複数のダイレクトキューを生成して得られる追加のパフォーマンスはないだろう。ダイレクトキューにスケジュールされるいずれのコマンドリストも同じハードウェアキューにシリアライズされるだろう。GCNハードウェアが複数のコンピュートエンジンをサポートする一方、我々はこれまでに[so far]アプリケーションにて1つ以上のコンピュートキューによる著しいパフォーマンス的なメリットがプロファイルされたのを確認できていない。コマンドリストの実行におけるより直接的な制御方法を有することにより、ハードウェアがサポートする以上のキューを生成しないことは一般に良いプラクティスである。

## タスクグラフをベースにエンジンを作る[Build a task graph based engine]

どのように非同期にスケジュールされる処理を決めるのか。フレームは、各タスクが他のタスクに対する依存性を持つタスクのグラフと見る[be considered]べきである。例えば、複数のシャドウマップは独立して生成でき、これはシャドウマップの入力を使用するVariance Shadow Map (VSM)を生成するコンピュートシェーダを伴う処理フェーズを含むかもしれない。影になる[shadowed]すべての光源を同時に処理するtiled lightingシェーダはすべてのシャドウマップおよびG-Bufferが処理を完了し終えたあとにのみ始めることができる。この場合、VSM生成は他のシャドウマップがレンダリングされている間に実行できるかもしれないし、G-Bufferレンダリングの間にバッチ処理されるかもしれない。

同様に、アンビエントオクルージョンの生成は深度バッファに依存するが、シャドウやtiled lightingとは独立している。なので、通常は非同期コンピュートキューで実行するのに良い候補である。

ゲーム開発者が非同期コンピュートを利用する[take advantage of]最適なシナリオを思い付く[come up with]のを支援した経験として、我々は並列に実行するタスクを手動で指定することはこのプロセスを自動化しようとするより効率的であることを発見した。コンピュートタスクだけが非同期的にスケジュールされるので、実行をオーバーラップさせるタスクの決定におけるより大きな自由度を持つために可能な限り多くのrender workloadsに対してコンピュートパスを実装することを推奨する。

最後に、コンピュートキューに処理を移すとき、アプリケーションは各コマンドリストが十分に大きいことを確認[make sure]すべきである。これは非同期コンピュートによるパフォーマンスゲインがコマンドリストを分割したりフェンスでストールしたりするコストを補う[make up for]ことを可能にするだろう。これは[which]、異なるキューでタスクを同期のに必須の操作[operations]である。

## キューが期待通りに動作しているかを確認する方法[How to check if queues are working as expected]

アプリケーションにおいて非同期キューが期待通りに動作していることを確認[ensure]するためにGPUViewを用いることを推奨する。GPUViewは使われているキューや各キューに入っている処理の数、そして最も重要なことだが、workloadsが実際に互いに対して並列に実行されているかどうかを可視化する。

Windows 10の下では、Windowsがページングに対して使用するため[which]、ほとんどのアプリケーションは最低でも3Dグラフィクスキュー1つとコピーキュー1つを示す[show]。以下の画像では、GPUへデータをアップロードするための追加のコピーキューを用いるアプリケーションのフレーム1つ分を確認できる。このgrabはフレームがレンダリングを始める前にデータのストリーミングや動的な定数バッファのアップロードにコピーキューを使っている開発中のゲームによるものである。そのゲームのこのビルドでは、グラフィクスキューが、レンダリングを始める前に、コピーの完了を待つ必要があった。このgrabでは、コピーキューがコピーの開始前に前のフレームのレンダリングの完了を待っていることを確認することもできる。

(image3)

この場合、アップロードされたデータのダブルバッファリングが実装されていなかったので、コピーキューを使用しても結果としてパーフォーマンス上の有利とはならなかった。データをダブルバッファリング化したあとでは、前のフレームが3Dキューで処理している最中にアップロードが発生し、3Dキューにおける隙間[gap]が取り除かれている[eliminated]。この変更は総フレーム時間のおおよそ10%を節約した。

2つ目の例はコンピュートキューをがっつり[heavy]使っているゲームであるAshes of the Singularityにおけるベンチマークのシーンのフレーム2つ分を示している。

(image4)

非同期コンピュートを使用するとき、異なるキューにあるコマンドキューが並列に実行されるにも関わらず[even though]、それでもなお同じGPUリソースを共有していることを考慮する[taken into account]必要がある。

- リソースがシステムメモリに配置される場合、グラフィクスまたはコンピュートキューからこれらへのアクセスはDMAキューのパフォーマンスに影響を与える[have an impact on]だろう。そしてまた逆も然り。
- ローカルメモリへアクセスするグラフィクスおよびコンピュートキュー（例えば、テクスチャデータのフェッチ、UAVへの書き込み、ラスタライゼーション負荷の高い処理[performing rasterization-heavy tasks]）は帯域制限に起因してお互いに影響を与え合う可能性がある。
- 同じCUを共有するスレッドはGPRやLDSを共有するだろう。なので、使えるリソースのすべてを使うタスクは非同期workloadsが同じCU上で実行されるのを阻害するかもしれない。
- 異なるキューはキャッシュを共有する。複数のキューが同じキャッシュを活用する場合、これは結果としてキャッシュスラッシングとなったりパフォーマンス低下となったりする可能性がある。

上記の理由により、パスごとにボトルネックを推定し、*補完的ボトルネック[complementary bottlenecks]*を持つパスを隣同士に配置することを推奨する。

- LDSやALUを大量に使用するコンピュートシェーダは通常、非同期コンピュートキューにとって良い候補である
- 深度のみのレンダリングパスは通常、その隣でいくつかのコンピュートタスクを走らせるのに良い候補である
- 効率的な非同期コンピュートの使い方に対する一般的な解法として、Nフレーム目のポストプロセッシングとN+1フレーム目のシャドウマップレンダリングをオーバーラップさせることが考えられる
- フレームの[as much]をコンピュートに移植すると、どのタスクが隣同士にスケジュールされ得るかを実験するときに柔軟性が増すだろう
- タスクをサブタスクに分割して互い違いにすること[interleaving]はバリアを減らしたり非同期コンピュートを効率的に使う機会を生み出したりする可能性がある（例えば、"ライトごとにシャドウマップをクリアし、シャドウをレンダリングし、VSMを計算する"のではなく、"すべてのシャドウマップをクリアし、すべてのシャドウマップをレンダリングし、すべてのシャドウマップに対するVSMを計算する"ようにする）

大事なこととして、非同期コンピュートは最適に使われないときにはパフォーマンスを低下させ得ることに注意したい。このケースを回避するため、非同期コンピュートをタスクごとに簡単に有効化・無効化できるようにしておくことが推奨される。これはいずれかのパフォーマンス上の利点を計測したり、アプリケーションが色々なハードウェア上で[on a wide range of hardware]最適に動作することを確かめたり[ensure]することを可能にする。