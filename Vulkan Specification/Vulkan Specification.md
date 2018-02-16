---
title: Vulkan 1.0.68 - A Specification
---
# はじめに {id="sec:1"}

## 1.1

## 1.2

## 用語法 {id="sec:1.3"}

この文書におけるキーワード **しなければならない[must]**、**要求されている[required]**、**推奨される[recommend]**、**してもよい[may]**、**選択できる[optional]** はRFC 2119に示される通りに解釈されるものとする^[訳注:日本語訳に際し、この文書では[RFC 2119JA](https://www.ipa.go.jp/security/rfc/RFC2119JA.html)に準拠する。]。

[http://www.ietf.org/rfc/rfc2119.txt](http://www.ietf.org/rfc/rfc2119.txt)

しなければならない[must]
: 単独で使われるとき、この語、または、**要求されている[required]** はその定義が仕様の絶対的な要請事項であることを意味する。**not** が続くとき("**してはならない[must not]**")、このフレーズはその定義が仕様の絶対的な禁止事項であること意味する。

する必要がある[should]
: 単独で使われるとき、この語は、特定の状況において特定の項目を無視する妥当な理由が存在するかもしれないが、異なる進路を取る前にその完全な意義を理解して慎重に検討しなければならない、ということを意味する。**not** が続くとき("**しないほうがよい[should not]**")、特定の挙動が許容できる、または、非常に有用であるときにこのフレーズは特定の状況において妥当な理由が存在するかもしれないが、このラベルで述べられるいかなる挙動を実装する前にその完全な意義を理解して、その場合分けを慎重に検討 **する必要がある**、ということを意味する。文法的に適切な場合、**推奨される[recommend]**、または、**推奨[recommendation]** が **する必要がある** の代わりに使われるかもしれない。

してもよい[may]
: この語、または、形容詞 **選択できる[optional]** は項目が真に選択的であることを意味する。あるベンダーは、特定の市場が要求している、または、他のベンダーはその項目を省略するかもしれないのに対し、そのベンダーは同様の項目がその製品を改良すると思っていることを理由に、その項目を含むことを選んでもよい。特定の選択肢を含まない実装は、恐らく機能性に劣るだろうが、その選択肢を含む他の実装と相互運用する準備をしておかなければならない。同様に、特定の選択肢を含む実装はその選択肢を含まない他の実装と相互運用する準備をしておかなければならない(もちろん、その選択肢が提供する機能は除く)。

追加の用語 **できる[can]**、**できない[cannot]** は以下のように解釈されるものとする。

できる[can]
: この語は、述べられる特定の挙動がアプリケーションに対して妥当な選択であることを意味し、実装の挙動を指して使われることは一切ない。

できない[cannot]
: この語は述べられる特定の挙動がアプリケーションによって達成不可能であることを意味する。例えば、エントリーポイントが存在しなければ、シェーダコードは処理を表現する能力がない。

> 注意
> この仕様書で使われるように、**できない** と **してはならない** の間には重要な区別がある。**できない** はアプリケーションがAPIを通して表現することも達成することも文字通りできないことを意味する。一方で、**してはならない** は、アプリケーションがAPIを通して表現する能力がないこと、実装のために処理の結果が未定義であり回復不可能となり得ることを意味する。

## 1.4

# 基礎 {id="sec:2"}

## アーキテクチャモデル {id="sec:2.1"}

Vulkanは以下の特性を持つCPU、GPU、その他ハードウェアアクセラレータのアーキテクチャのために設計され、そのAPIも同様に記述される。

- これらのバイトサイズの粒度でアドレス指定可能な、8、16、32、64ビットの符号ありと符号なしの二の補数の整数に対する実行時サポート
- [浮動小数点計算](#sec:2.8.1)の節における範囲と精度の制約を満足する、32、64ビットの浮動小数点型に対する実行時サポート
- これらの型の表現とエンディアンはそのホストと物理デバイスに対して同一で **なければならない**。

> 注意
> Vulkanにおける様々なデータの型と構造はホストと物理デバイスのメモリの間を相互に対応 **してもよい** ので、ホストやデバイスのアーキテクチャは両方とも、ポータブルで高性能なアプリケーションを記述するために効率的にそのようなデータへアクセスでき **なければならない**。

## 実行モデル {id="sec:2.2"}

この節はVulkanシステムの実行モデルの概要を述べる。

Vulkanは1つ以上の *デバイス* を露出する。このそれぞれが1つ以上の *キュー* を露出する。これは互いに非同期的に処理を行っ **てもよい**。デバイスによってサポートされる一連のキューは *ファミリー* に分けられる。各ファミリーは1種以上の機能性をサポートし、同様の特徴を持つ複数のキューを含ん **でもよい**。単一のファミリー内のキューは互いに *互換性がある* と見なし、キューのとあるファミリーに対して生成される処理はそのファミリー内のいずれのキューにおいて実行 **できる**。この仕様はキューがサポート **してもよい**、グラフィクス、コンピュート、トランスファー、スパースメモリ管理の4種の機能性を定義する。

> 注意
> 単一のデバイスは、?[multiple members of one or more of those families]を報告するのではなく、もしくはそれと同様に、複数の同様のキューファミリーを報告 **してもよい**。これは、これらのファミリーのメンバーが同様の能力を持つ一方で、これらが直接的に互いへの互換性が *ない* ことを示す。

デバイスメモリはアプリケーションによって明示的に管理される。各デバイスは、異なるメモリエリアを表すために、1つ以上のヒープを公表 **してもよい**。メモリヒープはデバイスローカルかホストローカルのいずれかであるが、デバイスからは常に可視である。メモリヒープについての更なる詳細はそのヒープで利用可能なメモリタイプを経由して公開される。実装で使えるように **してもよい** メモリエリアの例は以下を含める。

- *device local* はデバイスへ物理的に接続されるメモリである。
- *device local*、*host visible* はホストから可視であるデバイスローカルなメモリである。
- *host local*、*host visible* はホストに対してローカルであり、デバイスとホストから可視である。

他のアーキテクチャでは、いかなる目的に対して用いることが **できる** 単一のヒープのみが存在 **してもよい**。

VulkanアプリケーションはVulkanライブラリの呼び出しを経由して発行されるデバイスコマンドを記録したコマンドバッファのサブミッションを通して一連のデバイスを制御する。コマンドバッファの中身は基礎となるハードウェアに特有であり、アプリケーションから不透明である。一度構築されれば、コマンドバッファは実行するためにキューへ一度だけ又は何度もサブミット **できる**。複数のコマンドバッファはアプリケーション内で複数のスレッドを用いることで並列に作ることが **できる**。

異なるキューにサブミットされたコマンドバッファは互いに関して並列に、または、順序を外れて[out of order]さえも実行 **してもよい**。単一のキューにサブミットされたコマンドバッファは、[同期の章](#sec:6)でさらに述べられるように、[サブミッション順序](#sec:6.2.submission_order)を尊重する。デバイスによるコマンドバッファの実行はホストの実行に対して非同期でもある。一度コマンドバッファがキューにサブミットされれば、即座にアプリケーションへ制御を返し **てもよい**。デバイスとホストの間や異なるキューの間の同期はアプリケーションの責任である。

### キュー操作 {id="sec:2.2.1"}

Vulkanのキューはデバイスの実行エンジンへのインターフェイスを提供する。これらの実行エンジンに対するコマンドは実行する前にコマンドバッファに記録される。その後、これらのコマンドバッファは多くの *バッチ* における実行に対する *キューサブミッション* コマンドによってキューに提出される。一度キューにサブミットされれば、これらのコマンドは実行を開始し、アプリケーションの更なる介入なしに完了するだろうが、この実行順は多くの[暗黙的および明示的な順序制約](#sec:6)に依存している。

作業は一般的に`vkQueue*`(例えば、[vkQueueSubmit](#sec:5.5.vkQueueSubmit)、[vkQueueBindSparse](#sec:28.7.6.vkQueueBindSparse))の形を取るキューサブミッションコマンドを用いてキューにサブミットされ、作業を開始する前に待機するセマフォのリストと作業を完了したと一度だけ合図を送るセマフォのリストを任意に取る。作業それ自体は、セマフォの合図や待機と同様に、すべて *キュー操作* である。

異なるキューでのキュー操作は暗黙的な順序制約を持たず、いかなる順序で実行 **してもよい**。キューの間の明示的な順序制約は[セマフォ](#sec:6.4)や[フェンス](#sec:6.3)によって表現 **できる**。

単一のキューへのコマンドバッファサブミッションは[サブミッション順序](#sec:6.2.submission_order)やその他の[暗黙的な順序保証](#sec:6.2)を尊重するが、そうでなければ、重ね合わせたり順序を外れて実行 **してもよい**。その他のバッチや単一のキューに対するキューサブミッションのタイプ(例えば、[スパースメモリバインディング](#sec:28.7.6.sparse_memory_binding))は他のいかなるキューサブミッションまたはバッチに対する暗黙的な順序制約を持たない。キューサブミッションと個々のバッチとの間の追加の明示的な順序制約は[セマフォ](#sec:6.4)や[フェンス](#sec:6.3)によって表現できる。

フェンスまたはセマフォが合図を送る前に、以前にサブミットされたいかなるキュー操作も実行を完了し、これらのキュー操作からのメモリ書き込みが将来のキュー操作で[利用可能](#sec:6.1.availability)であることが保証される。合図が送られたセマフォまたはフェンスを待機することは利用可能である以前の書き込みが後続のコマンドから[可視](#sec:6.1.visibility)でもあることを保証する。

プライマリとセカンダリのコマンドバッファの間、および、同じまたは異なるバッチやサブミッションのプライマリコマンドバッファの間の両方のコマンドバッファ境界は追加のいかなる順序制約も導入しない。言い換えれば、いかなるセマフォまたはフェンスの間の(セカンダリコマンドバッファの実行を含めることが **できる**)一連のコマンドバッファのサブミットは、各境界で現在の状態が[リセット](#sec:5.reset)されることを除いて、それらがすべて単一のプライマリコマンドバッファに記録されたかのように、記録されたコマンドを実行する。明示的な順序制約は[明示的な同期プリミティブ](#sec:6)によって表現 **できる**。

> 注意
> 実装はキューにサブミットされた作業の実行を重ね合わせるための著しい自由度を持つ。そして、これはVulkanデバイスにおける深いパイプライン処理や並列性のために普通のことである。

コマンドバッファに記録されるコマンドはアクションを処理する(描画、ディスパッチ、クリア、コピー、クエリ/タイムスタンプ操作、サブパス開始/終了操作)、ステートを設定する(パイプライン、デスクリプタセット、バッファのバインド、動的ステートの設定、push constants、レンダパス/サブパスのステートの設定)、同期を処理する(イベントの設定/待機、パイプラインバリア、レンダパス/サブパスの依存関係)、のいずれかを行う。いくつかのコマンドはこれらタスクのひとつかそれ以上を処理する。ステートを設定するコマンドはコマンドバッファの *現在の状態* を更新する。アクションを処理するいくつかのコマンド(例えば、描画/ディスパッチ)はコマンドバッファの開始から累積的に設定された現在の状態に基づいて行う。アクションコマンドを処理するのに関係する作業はしばしば重ね合わせたり並べ替えたりできるが、そうすることが各アクションコマンドによって用いられるステートを変更 **してはならない**。一般に、アクションコマンドは、フレームバッファアタッチメントを変更したり、バッファやイメージのメモリを読み書きしたり、クエリプールに書き込んだりするコマンドである。

同期コマンドは2つの一連のアクションコマンドの間の明示的な[実行およびメモリの依存関係](#sec:6.1)を導入する。ここでは、後者のコマンド一式が前者のコマンド一式に依存する。これらの依存関係は前者におけるとある[パイプラインステージ](#sec:6.2.1)の実行の後に後者におけるとあるパイプラインステージの実行が起こること、および、とあるパイプラインステージによって処理される[メモリアクセス](#sec:6.7.2)の効果が順番に起こるの両方を強制し、そして、互いに可視である。明示的な依存関係または[暗黙的な順序保証](#sec:6.2)によって強制されないとき、アクションコマンドは実行を重ね合わせたり順序を外れて実行 **してもよく**、互いのメモリアクセスの副次的効果を確認 **しなくてもよい**。

デバイスはホストに関して非同期にキュー操作を実行する。制御は、キューへのコマンドバッファサブミッションに従い、アプリケーションへ即座に返される。アプリケーションは必要に応じてホストとデバイスの間の作業を同期 **しなければならない**。

## オブジェクトモデル {id="sec:2.3"}

Vulkanにおけるデバイス、キュー、その他のエンティティはVulkanオブジェクトによって表される。APIレベルでは、すべてのオブジェクトはハンドルによって参照される。ハンドルにはディスパッチ可能[dispatchable]とディスパッチ不能[non-dispatchable]の2つクラスが存在する。*ディスパッチ可能* なハンドル型は不透明型へのポインタである。このポインタはAPIコマンドを傍受する一環としてレイヤーによって使われ **てもよい**。そしてそれ故に、各APIコマンドはその第1引数としてディスパッチ可能型を取る。ディスパッチ可能型の各オブジェクトはその寿命の間に唯一のハンドル値を持た **なければならない**。

*ディスパッチ不能* なハンドル型は実装依存の意味を持つ64ビット整数型であり、ソフトウェア構造へのポインタではなくハンドルにオブジェクト情報を直接エンコード **してもよい**。ディスパッチ不能型のオブジェクトは型の内部に、または、型にわたって唯一のハンドル値を持た **なくてもよい**。ハンドル値が唯一でないならば、そのようなハンドルのひとつを破壊しても他の型の同一ハンドルが不正になるようなことがあっ **てはならず**、同じ型の同一ハンドルでは、そのハンドル値の生成された回数が破壊された回数よりも多いならば、不正になるようなことがあっ **てはならない**。

ある`VkDevice`から生成または割り当てられた(すなわち、第1引数として`VkDevice`を伴う)すべてのオブジェクトはそのデバイスに対してプライベートであり、他のデバイスで用いられ **てはならない**。

### オブジェクト寿命 {id="sec:2.3.1"}

オブジェクトは`vkCreate*`や`vkAllocate*`コマンドによってそれぞれ生成または割り当てられる。一度オブジェクトが生成または割り当てられれば、その"構造"は不変であると見なされるが、あるオブジェクト型の中身は依然として自由に変更できる。オブジェクトは`vkDestroy*`や`vkFree*`コマンドによってそれぞれ破壊または解放される。

(生成ではなく)割り当てられるオブジェクトは既存のプールオブジェクトまたはメモリヒープからリソースを取り、解放されるときにリソースをそのプールまたはヒープに返す。オブジェクトの生成や破壊は一般に実行時の間に低頻度で発生することが期待される一方で、割り当てや解放は高頻度で発生することが **できる**。プールオブジェクトは割り当てや解放のパフォーマンス改善に適応するのに役立つ。

Vulkanオブジェクトの寿命を追跡することと使用中に破壊しないことのはアプリケーションの責任である。

<a id="sec:2.3.1.ownership_temporarily_acquired" />アプリケーション所有のメモリの所有権は渡されたVulkanコマンドによって即座に取得される。そのようなメモリの所有権はそのコマンドの存続期間の終了時にアプリケーションへ返却 **しなければならない** ため、アプリケーションは、このメモリを取得したすべてのコマンドが返った後すぐに、そのメモリを変更したり開放したりすることが **できる**。

以下のオブジェクト型は、Vulkanコマンドに渡され、生成するのに使われるオブジェクトによってそれ以上にアクセスされないとき、消費される。これらは渡されるいかなるAPIコマンドの存続期間中にも破壊され **てはならない**。

- `VkShaderModule`
- `VkPipelineCache`

他のオブジェクトを生成するために引数として渡される`VkRenderPass`オブジェクトは渡されたコマンドの存続期間の後でそのオブジェクトによってそれ以上にアクセスされない。コマンドバッファで使われる`VkRenderPass`は以下に示される規定に従う。

`VkPipelineLayout`オブジェクトはそれを使うコマンドバッファが記録状態にある間に破壊 **してはならない**。

`VkDescriptorSetLayout`オブジェクトはこのレイアウトを用いて割り当てられたデスクリプタセットで操作するコマンドによってアクセスされ **てもよく**、これらのデスクリプタセットはデスクリプタセットレイアウトが破壊された後に[vkUpdateDescriptorSets](#sec:13.2.4.vkUpdateDescriptorSets)によって更新 **してはならない**。そうでなければ、デスクリプタセットレイアウトはAPIコマンドによって使用中でないいずれかのときに破壊 **できる**。

アプリケーションは他のいかなるVulkanオブジェクト型を、そのオブジェクトがデバイスによって(コマンドバッファの実行などを介して)使い終わるまで、破壊 **してはならない**。

以下のVulkanオブジェクトはそのオブジェクトを使用しているいずれかのコマンドバッファが[待機状態](#sec:5.1)にある間に破壊 **してはならない**。

- `VkEvent`
- `VkQueryPool`
- `VkBuffer`
- `VkBufferView`
- `VkImage`
- `VkImageView`
- `VkPipeline`
- `VkSampler`
- `VkDescriptorPool`
- `VkFramebuffer`
- `VkRenderPass`
- `VkCommandBuffer`
- `VkCommandPool`
- `VkDeviceMemory`
- `VkDescriptorSet`

これらのオブジェクトを破壊しようとすると、[記録または実行状態](#sec:5.1)にあるいずれかのコマンドバッファが動き出し、これらのオブジェクトを使っていると、[不正状態](#sec:5.1)に遷移する。

以下のVulkanオブジェクトはいずれかのキューがそのオブジェクトを使うコマンドを実行している間に破壊 **してはならない**。

- `VkFence`
- `VkSemaphore`
- `VkCommandBuffer`
- `VkCommandPool`

一般に、(コマンドバッファのリセットのような)他のオブジェクトをもう使わないような方法で破壊またはリセットされることを除いたいかなる方法で解放されたオブジェクトを使うオブジェクトがそれ以上に使われない場合に限り、解放されようとしているオブジェクトが他のオブジェクトの使用(例えば、ビューによるリソースの使用、デスクリプタセットによるビューの使用、コマンドバッファによるオブジェクトの使用、リソースへのメモリ割り当てのバインディング)に関与していても、オブジェクトは任意の順序で破壊または解放 **できる**。そのオブジェクトがリセットされていたならば、解放されたオブジェクトを一切用いていないかのように使うことが **できる**。これの例外はオブジェクト間に親子関係が存在するときである。この場合、アプリケーションは、親が破壊されるときにその子供を解放すると明示的に定義される(例えば、下に定義されるような、プールオブジェクトで)ときを除いて、その子オブジェクトを破壊する前に親オブジェクトを破壊 **してはならない**。

`VkCommandPool`オブジェクトは`VkCommandBuffer`オブジェクトの親である。`VkDescriptorPool`オブジェクトは`VkDescriptorSet`オブジェクトの親である。`VkDevice`オブジェクトは多くのオブジェクト型の親である(それらの生成への引数として`VkDevice`を取るすべて)。

以下のVulkanオブジェクトは破壊 **できる** ときに対して固有の制限を持つ。

- `VkQueue`オブジェクトは明示的に破壊 **できない**。代わりに、それらが取得[retrieve]される`VkDevice`オブジェクトが破壊されるときに暗黙的に破壊される。
- プールオブジェクトを破壊すると、そのプールから割り当てられたすべてのオブジェクトが暗黙的に解放される。具体的には、`VkCommandPool`を破壊すると、それから割り当てられたすべての`VkCommandBuffer`が解放され、`VkDescriptorPool`を破壊すると、それから割り当てられたすべての`VkDescriptorSet`が解放される。
- `VkDevice`オブジェクトはそれらから取得されたすべての`VkQueue`オブジェクトがアイドルであり、それらから生成されたすべてのオブジェクトが破壊されたときに破壊 **できる**。これは以下のオブジェクトを含む。
    - `VkFence`
    - `VkSemaphore`
    - `VkEvent`
    - `VkQueryPool`
    - `VkBuffer`
    - `VkBufferView`
    - `VkImage`
    - `VkImageView`
    - `VkShaderModule`
    - `VkPipelineCache`
    - `VkPipeline`
    - `VkPipelineLayout`
    - `VkSampler`
    - `VkDescriptorSetLayout`
    - `VkDescriptorPool`
    - `VkFramebuffer`
    - `VkRenderPass`
    - `VkCommandPool`
    - `VkCommandBuffer`
    - `VkDeviceMemory`
- `VkPhysicalDevice`オブジェクトは明示的に破壊 **できない**。代わりに、それらが取得[retrieve]される`VkInstance`オブジェクトが破壊されるときに暗黙的に破壊される。
- `VkInstance`は、その`VkPhysicalDevice`オブジェクトのいずれかから生成されたすべての`VkDevice`オブジェクトが破壊されれば、破壊 **できる**。

## 2.4

## コマンドの構文と存続期間 {id="sec:2.5"}

この仕様書はVulkanコマンドをC99の構文を用いる関数または手続きとして記述する。C++やJavaScriptのような他の言語に対する言語バインディングはより厳密な引数渡しやオブジェクト指向インターフェイスを可能に **してもよい**。

Vulkanは、以下に述べられる場合を除いて、適切なときにスカラパラメータの基本型や文書の他の場所で標準Cの型を用いる。

`VkBool32`は、Cが十分に移植性のあるビルトインのブーリアン型を持たないので、ブーリアンの`True`および`False`の値を表す。

```c
typedef uint32_t VkBool32;
```

`VK_TRUE`はブーリアンの **真** (整数の1)の値を表し、`VK_FALSE`はブーリアンの **偽** (整数の0)の値を表す。

Vulkan実装から`VkBool32`で返されるすべての値は`VK_TRUE`か`VK_FALSE`のいずれかであるだろう。

アプリケーションは`VkBool32`が期待されるところのVulkan実装に`VK_TRUE`と`VK_FALSE`以外のいかなる値も渡し
 **てはならない**。

`VkDeviceSize`はデバイスメモリのサイズとオフセットの値を表す。

```c
typedef uint64_t VkDeviceSize;
```

Vulkanオブジェクトを生成するコマンドは`vkCreate*`の形であり、オブジェクトを生成するのに必要な引数とともに`Vk*CreateInfo`構造体を取る。これらのVulkanオブジェクトは`vkDestroy*`の形のコマンドで破壊される。

Vulkanオブジェクトを生成または破壊する各コマンドへの最後の入力引数は`pAllocator`である。`pAllocator`引数は与えられたオブジェクトに対する割り当てがアプリケーションによって提供されるコールバックへ委任されるような非`NULL`値に設定 **できる**。更なる詳細は[メモリ割り当て](#sec:10.1.memory_allocation)の章を参照のこと。

プールオブジェクトによって所有されるVulkanオブジェクトを割り当てるコマンドは`vkAllocate*`の形であり、`Vk*AllocateInfo`構造体を取る。これらのVulkanオブジェクトは`vkFree*`の形のコマンドで解放される。これらのオブジェクトはアロケータを取らない。ホストメモリが必要とされるならば、これらの親のプールが生成されたときに指定されたアロケータを用いるだろう。

コマンドは`vkCmd*`の形のAPIコマンドの呼び出しによってコマンドバッファに記録される。そのようなコマンドは、プライマリおよび/またはセカンダリコマンドバッファの中、レンダパスの内外、サポートされるクエリタイプのひとつまたはそれ以上の中といった、使用 **できる** ところでそれぞれ異なる制限を持っ **てもよい**。これらの制限はそのようなコマンドそれぞれの定義とともに文書化される。

Vulkanコマンドの *存続期間[duration]* はコマンドの呼び出しから呼び出し元へのリターンまでの時間間隔を指す。

### 取得された結果の寿命 {id="sec:2.5.1"}

情報は`vkGet*`や`vkEnumerate*`の形のコマンドで実装から取得される。

個々のコマンドで指定されない場合を除き、その結果は *不変[invariant]* である。すなわち、引数が有効のままである限り、同じ引数による同じコマンドの呼び出しによって再び取得されるときでもこれらは変わらないままだろう。

## スレッディングの挙動 {id="sec:2.6"}

Vulkanは複数のホストスレッドで用いられるときにスケール可能なパフォーマンスを提供することが意図される。すべてのコマンドは複数スレッドから並行に呼び出されることをサポートするが、ある引数または引数の構成要素は *外部同期[externally synchronized]* されることが定義されている。これは、与えられた時間にそのような引数を用いているスレッドが1つ以上存在しないことを呼び出し元が保証 **しなければならない** ことを意味する。

より正確に言うと、VulkanコマンドはVulkanオブジェクトを表現するソフトウェア構造を更新するために単純な格納領域を用いる。外部同期されるとして宣言された引数はホストがコマンド実行している時間にも更新されるそのソフトウェア構造を持っ **てもよい**。2つのコマンドが同じオブジェクトで操作し、少なくともコマンドのひとつが外部同期されるオブジェクトを宣言するならば、呼び出し元は、コマンドが同時に実行しないことだけでなく、(必要であれば)2つのコマンドが適切なメモリバリアによって分けられることも保証 **しなければならない**。

> 注意
> メモリバリアは、多くの開発者が慣れているx86/x64プログラミングのものより弱い順序付けがされる、ARMのCPUアーキテクチャに特に関係する。幸いにも、(pthreadライブラリのような)ほとんどの高レベル同期プリミティブは相互排他の一部としてメモリバリアを処理するので、これらのプリミティブを介してVulkanオブジェクトを相互排他することは望ましい効果を持つだろう。

同様に、アプリケーションはVulkanコマンドによって[一時的に取得される所有権](#sec:2.3.1.ownership_temporarily_acquired)を持つアプリケーション所有のメモリのいかなる潜在的なデータハザードも回避 **しなければならない**。アプリケーション所有のメモリの所有権はコマンドによって取得されたままである一方で、実装は任意の時にそのメモリを読み込ん **でもよく**、任意の時に`const`修飾されていないメモリに書き込ん **でもよい**。`const`修飾されていないアプリケーション所有のメモリを参照している引数は仕様書では *外部同期* されるとして明示的に印付けされない。

多くのオブジェクト型は *不変[immutable]* である。これは、一度生成されればオブジェクトは変更 **できない** ことを意味する。これらのオブジェクト型は、他のスレッドで使用中に破壊 **してはならない** ことを除いて、外部同期を一切必要としない。とある特殊な場合では、可変[mutable]なオブジェクトパラメータは外部同期を必要としないかのように内部で同期される。この一例は`vkCreateGraphicsPipelines`や`vkCreateComputePipelines`における`VkPipelineCache`の使用である。ここでは、そんな重量級コマンドの周りの外部同期は実用的ではないだろう。その実装はこの例においてキャッシュを内部的に同期 **しなければならず**、そのコマンドの周りにより粒度の細かいミューテックスの形で行うことを可能に **してもよい**。外部同期されるとしてラベリングされていないいずれかコマンド引数はコマンドによって変わらないか、内部的に同期されているかのどちらかである。加えて、コマンドの引数に関連するあるオブジェクト(例えば、コマンドプールやデスクリプタプール)はコマンドによって影響を受け **てもよく**、また、外部同期され **なければならない**。これらの暗黙の引数は以下で述べられるように文書化される。

外部同期されるコマンドの引数は以下にリスト化される。

- [vkDestroyInstance](#sec:X.X.vkDestroyInstance)における引数`instance`
- [vkDestroyDevice](#sec:X.X.vkDestroyDevice)における引数`device`
- [vkQueueSubmit](#sec:X.X.vkQueueSubmit)における引数`queue`
- [vkQueueSubmit](#sec:X.X.vkQueueSubmit)における引数`fence`
- [vkFreeMemory](#sec:X.X.vkFreeMemory)における引数`memory`
- [vkMapMemory](#sec:X.X.vkMapMemory)における引数`memory`
- [vkUnmapMemory](#sec:X.X.vkUnmapMemory)における引数`memory`
- [vkBindBufferMemory](#sec:X.X.vkBindBufferMemory)における引数`buffer`
- [vkBindImageMemory](#sec:X.X.vkBindImageMemory)における引数`image`
- [vkQueueBindSparse](#sec:X.X.vkQueueBindSparse)における引数`queue`
- [vkQueueBindSparse](#sec:X.X.vkQueueBindSparse)における引数`fence`
- [vkDestroyFence](#sec:X.X.vkDestroyFence)における引数`fence`
- [vkDestroySemaphore](#sec:X.X.vkDestroySemaphore)における引数`semaphore`
- [vkDestroyEvent](#sec:X.X.vkDestroyEvent)における引数`event`
- [vkSetEvent](#sec:X.X.vkSetEvent)における引数`event`
- [vkResetEvent](#sec:X.X.vkResetEvent)における引数`event`
- [vkDestroyQueryPool](#sec:X.X.vkDestroyQueryPool)における引数`queryPool`
- [vkDestroyBuffer](#sec:X.X.vkDestroyBuffer)における引数`buffer`
- [vkDestroyBufferView](#sec:X.X.vkDestroyBufferView)における引数`bufferView`
- [vkDestroyImage](#sec:X.X.vkDestroyImage)における引数`image`
- [vkDestroyImageView](#sec:X.X.vkDestroyImageView)における引数`imageView`
- [vkDestroyShaderModule](#sec:X.X.vkDestroyShaderModule)における引数`shaderModule`
- [vkDestroyPipelineCache](#sec:X.X.vkDestroyPipelineCache)における引数`pipelineCache`
- [vkMergePipelineCaches](#sec:X.X.vkMergePipelineCaches)における引数`dstCache`
- [vkDestroyPipeline](#sec:X.X.vkDestroyPipeline)における引数`pipeline`
- [vkDestroyPipelineLayout](#sec:X.X.vkDestroyPipelineLayout)における引数`pipelineLayout`
- [vkDestroySampler](#sec:X.X.vkDestroySampler)における引数`sampler`
- [vkDestroyDescriptorSetLayout](#sec:X.X.vkDestroyDescriptorSetLayout)における引数`descriptorSetLayout`
- [vkDestroyDescriptorPool](#sec:X.X.vkDestroyDescriptorPool)における引数`descriptorPool`
- [vkResetDescriptorPool](#sec:X.X.vkResetDescriptorPool)における引数`descriptorPool`
- [vkAllocateDescriptorSets](#sec:X.X.vkAllocateDescriptorSets)における引数`pAllocateInfo`にある`descriptorPool`
- [vkFreeDescriptorSets](#sec:X.X.vkFreeDescriptorSets)における引数`descriptorPool`
- [vkDestroyFramebuffer](#sec:X.X.vkDestroyFramebuffer)における引数`framebuffer`
- [vkDestroyRenderPass](#sec:X.X.vkDestroyRenderPass)における引数`renderPass`
- [vkDestroyCommandPool](#sec:X.X.vkDestroyCommandPool)における引数`commandPool`
- [vkResetCommandPool](#sec:X.X.vkResetCommandPool)における引数`commandPool`
- [vkAllocateCommandBuffers](#sec:X.X.vkAllocateCommandBuffers)における引数`pAllocateInfo`にある`commandPool`
- [vkFreeCommandBuffers](#sec:X.X.vkFreeCommandBuffers)における引数`commandPool`
- [vkBeginCommandBuffer](#sec:X.X.vkBeginCommandBuffer)における引数`commandBuffer`
- [vkEndCommandBuffer](#sec:X.X.vkEndCommandBuffer)における引数`commandBuffer`
- [vkResetCommandBuffer](#sec:X.X.vkResetCommandBuffer)における引数`commandBuffer`
- [vkCmdBindPipeline](#sec:X.X.vkCmdBindPipeline)における引数`commandBuffer`
- [vkCmdSetViewport](#sec:X.X.vkCmdSetViewport)における引数`commandBuffer`
- [vkCmdSetScissor](#sec:X.X.vkCmdSetScissor)における引数`commandBuffer`
- [vkCmdSetLineWidth](#sec:X.X.vkCmdSetLineWidth)における引数`commandBuffer`
- [vkCmdSetDepthBias](#sec:X.X.vkCmdSetDepthBias)における引数`commandBuffer`
- [vkCmdSetBlendConstants](#sec:X.X.vkCmdSetBlendConstants)における引数`commandBuffer`
- [vkCmdSetDepthBounds](#sec:X.X.vkCmdSetDepthBounds)における引数`commandBuffer`
- [vkCmdSetStencilCompareMask](#sec:X.X.vkCmdSetStencilCompareMask)における引数`commandBuffer`
- [vkCmdSetStencilWriteMask](#sec:X.X.vkCmdSetStencilWriteMask)における引数`commandBuffer`
- [vkCmdSetStencilReference](#sec:X.X.vkCmdSetStencilReference)における引数`commandBuffer`
- [vkCmdBindDescriptorSets](#sec:X.X.vkCmdBindDescriptorSets)における引数`commandBuffer`
- [vkCmdBindIndexBuffer](#sec:X.X.vkCmdBindIndexBuffer)における引数`commandBuffer`
- [vkCmdBindVertexBuffers](#sec:X.X.vkCmdBindVertexBuffers)における引数`commandBuffer`
- [vkCmdDraw](#sec:X.X.vkCmdDraw)における引数`commandBuffer`
- [vkCmdDrawIndexed](#sec:X.X.vkCmdDrawIndexed)における引数`commandBuffer`
- [vkCmdDrawIndirect](#sec:X.X.vkCmdDrawIndirect)における引数`commandBuffer`
- [vkCmdDrawIndexedIndirect](#sec:X.X.vkCmdDrawIndexedIndirect)における引数`commandBuffer`
- [vkCmdDispatch](#sec:X.X.vkCmdDispatch)における引数`commandBuffer`
- [vkCmdDispatchIndirect](#sec:X.X.vkCmdDispatchIndirect)における引数`commandBuffer`
- [vkCmdCopyBuffer](#sec:X.X.vkCmdCopyBuffer)における引数`commandBuffer`
- [vkCmdCopyImage](#sec:X.X.vkCmdCopyImage)における引数`commandBuffer`
- [vkCmdBlitImage](#sec:X.X.vkCmdBlitImage)における引数`commandBuffer`
- [vkCmdCopyBufferToImage](#sec:X.X.vkCmdCopyBufferToImage)における引数`commandBuffer`
- [vkCmdCopyImageToBuffer](#sec:X.X.vkCmdCopyImageToBuffer)における引数`commandBuffer`
- [vkCmdUpdateBuffer](#sec:X.X.vkCmdUpdateBuffer)における引数`commandBuffer`
- [vkCmdFillBuffer](#sec:X.X.vkCmdFillBuffer)における引数`commandBuffer`
- [vkCmdClearColorImage](#sec:X.X.vkCmdClearColorImage)における引数`commandBuffer`
- [vkCmdClearDepthStencilImage](#sec:X.X.vkCmdClearDepthStencilImage)における引数`commandBuffer`
- [vkCmdClearAttachments](#sec:X.X.vkCmdClearAttachments)における引数`commandBuffer`
- [vkCmdResolveImage](#sec:X.X.vkCmdResolveImage)における引数`commandBuffer`
- [vkCmdSetEvent](#sec:X.X.vkCmdSetEvent)における引数`commandBuffer`
- [vkCmdResetEvent](#sec:X.X.vkCmdResetEvent)における引数`commandBuffer`
- [vkCmdWaitEvents](#sec:X.X.vkCmdWaitEvents)における引数`commandBuffer`
- [vkCmdPipelineBarrier](#sec:X.X.vkCmdPipelineBarrier)における引数`commandBuffer`
- [vkCmdBeginQuery](#sec:X.X.vkCmdBeginQuery)における引数`commandBuffer`
- [vkCmdEndQuery](#sec:X.X.vkCmdEndQuery)における引数`commandBuffer`
- [vkCmdResetQueryPool](#sec:X.X.vkCmdResetQueryPool)における引数`commandBuffer`
- [vkCmdWriteTimestamp](#sec:X.X.vkCmdWriteTimestamp)における引数`commandBuffer`
- [vkCmdCopyQueryPoolResults](#sec:X.X.vkCmdCopyQueryPoolResults)における引数`commandBuffer`
- [vkCmdPushConstants](#sec:X.X.vkCmdPushConstants)における引数`commandBuffer`
- [vkCmdBeginRenderPass](#sec:X.X.vkCmdBeginRenderPass)における引数`commandBuffer`
- [vkCmdNextSubpass](#sec:X.X.vkCmdNextSubpass)における引数`commandBuffer`
- [vkCmdEndRenderPass](#sec:X.X.vkCmdEndRenderPass)における引数`commandBuffer`
- [vkCmdExecuteCommands](#sec:X.X.vkCmdExecuteCommands)における引数`commandBuffer`

コマンドが外部同期される引数をその中身に持つユーザーの割り当てたリストを取ることが **できる** いくつかの例も存在する。これらの場合、その呼び出し元は多くても1つのスレッドが与えられた時間にリスト内の与えられた要素を使っていることを保証 **しなければならない**。これらの引数は以下にリスト化される。

- [vkQueueSubmit](#sec:X.X.vkQueueSubmit)における引数`pSubmits`の各要素のメンバー`pWaitSemaphores`の各要素
- [vkQueueSubmit](#sec:X.X.vkQueueSubmit)における引数`pSubmits`の各要素のメンバー`pSignalSemaphores`の各要素
- [vkQueueBindSparse](#sec:X.X.vkQueueBindSparse)における引数`pBindInfo`の各要素のメンバー`pWaitSemaphores`の各要素
- [vkQueueBindSparse](#sec:X.X.vkQueueBindSparse)における引数`pBindInfo`の各要素のメンバー`pSignalSemaphores`の各要素
- [vkQueueBindSparse](#sec:X.X.vkQueueBindSparse)における引数`pBindInfo`の各要素のメンバー`pBufferBinds`の各要素のメンバー`buffer`
- [vkQueueBindSparse](#sec:X.X.vkQueueBindSparse)における引数`pBindInfo`の各要素のメンバー`pImageOpaqueBinds`の各要素のメンバー`image`
- [vkQueueBindSparse](#sec:X.X.vkQueueBindSparse)における引数`pBindInfo`の各要素のメンバー`pImageBinds`の各要素のメンバー`image`
- [vkResetFences](#sec:X.X.vkResetFences)における引数`pFences`の各要素
- [vkFreeDescriptorSets](#sec:X.X.vkFreeDescriptorSets)における引数`pDescriptorSets`の各要素
- [vkUpdateDescriptorSets](#sec:X.X.vkUpdateDescriptorSets)における引数`pDescriptorWrites`の各要素のメンバー`dstSet`
- [vkUpdateDescriptorSets](#sec:X.X.vkUpdateDescriptorSets)における引数`pDescriptorCopies`の各要素のメンバー`dstSet`
- [vkFreeCommandBuffers](#sec:X.X.vkFreeCommandBuffers)における引数`pCommandBuffers`の各要素

加えて、外部同期される必要があるいくつかの暗黙の引数がある。例えば、外部同期される必要があるすべての引数`commandBuffer`はそのコマンドバッファの生成時に渡された`commandPool`もまた外部同期される必要があることを暗に示す。暗黙の引数とそれらに関連するオブジェクトはいかにリスト化される。

- [vkDeviceWaitIdle](#sec:X.X.vkDeviceWaitIdle)における`device`から生成されたすべて`VkQueue`オブジェクト
- [vkResetDescriptorPool](#sec:X.X.vkResetDescriptorPool)における`descriptorPool`から割り当てられたいずれかの`VkDescriptorSet`オブジェクト
- [vkBeginCommandBuffer](#sec:X.X.vkBeginCommandBuffer)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkEndCommandBuffer](#sec:X.X.vkEndCommandBuffer)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdBindPipeline](#sec:X.X.vkCmdBindPipeline)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetViewport](#sec:X.X.vkCmdSetViewport)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetScissor](#sec:X.X.vkCmdSetScissor)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetLineWidth](#sec:X.X.vkCmdSetLineWidth)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetDepthBias](#sec:X.X.vkCmdSetDepthBias)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetBlendConstants](#sec:X.X.vkCmdSetBlendConstants)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetDepthBounds](#sec:X.X.vkCmdSetDepthBounds)における`commandBuffer`を割り当てた`VkCommandPool`
[vkCmdSetStencilCompareMask](#sec:X.X.vkCmdSetStencilCompareMask)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetStencilWriteMask](#sec:X.X.vkCmdSetStencilWriteMask)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetStencilReference](#sec:X.X.vkCmdSetStencilReference)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdBindDescriptorSets](#sec:X.X.vkCmdBindDescriptorSets)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdBindIndexBuffer](#sec:X.X.vkCmdBindIndexBuffer)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdBindVertexBuffers](#sec:X.X.vkCmdBindVertexBuffers)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdDraw](#sec:X.X.vkCmdDraw)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdDrawIndexed](#sec:X.X.vkCmdDrawIndexed)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdDrawIndirect](#sec:X.X.vkCmdDrawIndirect)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdDrawIndexedIndirect](#sec:X.X.vkCmdDrawIndexedIndirect)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdDispatch](#sec:X.X.vkCmdDispatch)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdDispatchIndirect](#sec:X.X.vkCmdDispatchIndirect)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdCopyBuffer](#sec:X.X.vkCmdCopyBuffer)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdCopyImage](#sec:X.X.vkCmdCopyImage)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdBlitImage](#sec:X.X.vkCmdBlitImage)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdCopyBufferToImage](#sec:X.X.vkCmdCopyBufferToImage)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdCopyImageToBuffer](#sec:X.X.vkCmdCopyImageToBuffer)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdUpdateBuffer](#sec:X.X.vkCmdUpdateBuffer)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdFillBuffer](#sec:X.X.vkCmdFillBuffer)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdClearColorImage](#sec:X.X.vkCmdClearColorImage)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdClearDepthStencilImage](#sec:X.X.vkCmdClearDepthStencilImage)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdClearAttachments](#sec:X.X.vkCmdClearAttachments)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdResolveImage](#sec:X.X.vkCmdResolveImage)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdSetEvent](#sec:X.X.vkCmdSetEvent)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdResetEvent](#sec:X.X.vkCmdResetEvent)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdWaitEvents](#sec:X.X.vkCmdWaitEvents)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdPipelineBarrier](#sec:X.X.vkCmdPipelineBarrier)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdBeginQuery](#sec:X.X.vkCmdBeginQuery)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdEndQuery](#sec:X.X.vkCmdEndQuery)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdResetQueryPool](#sec:X.X.vkCmdResetQueryPool)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdWriteTimestamp](#sec:X.X.vkCmdWriteTimestamp)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdCopyQueryPoolResults](#sec:X.X.vkCmdCopyQueryPoolResults)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdPushConstants](#sec:X.X.vkCmdPushConstants)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdBeginRenderPass](#sec:X.X.vkCmdBeginRenderPass)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdNextSubpass](#sec:X.X.vkCmdNextSubpass)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdEndRenderPass](#sec:X.X.vkCmdEndRenderPass)における`commandBuffer`を割り当てた`VkCommandPool`
- [vkCmdExecuteCommands](#sec:X.X.vkCmdExecuteCommands)における`commandBuffer`を割り当てた`VkCommandPool`

## エラー{id="sec:2.7"}

Vulkanは層状のAPIである。最下層は、この仕様書で定義されるような、コアとなるVulkanレイヤーである。アプリケーションはデバッグ、検証、その他の目的のためにコアの上に追加のレイヤーを用いる **できる**。

Vulkanのコアとなる原則のひとつはコマンドバッファの構築とサブミットが高効率である **必要がある** ことである。故に、コアレイヤーにおける状態のエラーの確認と検証は最小限であり、より厳格な検証はレイヤーの使用を通じて有効化 **できる**。

コアレイヤーはアプリケーションが正しくAPIを使っていることを仮定する。仕様書の他の場所で文書化されるような場合を除き、間違ってAPIを使用するアプリケーションへのコアレイヤーの挙動は未定義であり、プログラムの終了を含ん **でもよい**。しかし、実装はアプリケーションによる間違った使い方がオペレーティングシステム、Vulkan実装、システム内の他のVulkanクライアントアプリケーションの保全性に影響を与えないことを保証 **しなければならない**。特に、あるプロセスからのメモリが他のプロセスかが可視に **できる** かどうかについてオペレーティングシステムによって行われる保証は **いかなるメモリ割り当て** に対してもVulkan実装によって違反され **てはならない**。アプリケーションの特定の機能や拡張(例えば、[堅牢なバッファアクセス](#sec:30.1.robust_buffer_access))の使用によって明示的に指示されない限り、Vulkan実装はOSによって提供されるこれらを越えて追加のセキュリティまたは保全性の保証を行うことを **要求されていない**。

> 注意
> 例えば、オペレーティングシステムがすべてのメモリ割り当てにおけるデータが新しく割り当てられたときにゼロに設定されることを保証するならば、Vulkan実装はそれが制御する割り当てに対して同じ保証を行わ **なければならない**(例えば、[VkDeviceMemory](#sec:10.2.VkDeviceMemory))。

アプリケーションは「[機能、制限、フォーマット](#sec:30)」章において述べられるように、`robustBufferAccess`機能を有効化することでより強力な堅牢性保証を要求することが **できる**。

正しいAPIの使い方の検証は検証レイヤーに委ねられる。アプリケーションは、エラーを捕捉して排除するのに役立つため、有効化された検証レイヤーとともに開発される **必要がある**。一度検証されれば、リリースされたアプリケーションはデフォルトで検証レイヤーを有効化 **しないほうがよい**。

### 有効な使い方 {id="sec:2.7.1"}

有効な使い方はアプリケーションにおいてうまく定義された実行時の挙動を達成するために満たさ **なければならない** 一連の条件を定義する。これらの条件はVulkanの状態およびその使い方が条件によって制約される引数やオブジェクトにのみ依存する。

いくつかの有効な使い方の条件は実行時の制限や機能の可用性との依存関係を持つ。これらの制限と機能のためのVulkanのサポートされる最小値、または、他の既知の値のいくつかのサブセットに対するこれらの条件を検証することが可能である。

有効な使い方の条件は(エラーコードのリターンを含む)うまく定義された挙動が存在する条件を網羅しない。

有効な使い方の条件はその条件についての完璧な情報がアプリケーションの実行中に知られているであろうコマンドや構造に適用される **必要がある**。これは検証レイヤーやlinterがそれらの指定される時点でこれらのステートメントに対して直接書かれることが **できる** のようなものである。

> 注意
> これは有効な使い方のステートメントに対するいくつかの明確でない場所を引き起こす。例えば、構造に対する有効な値は呼び出すコマンドにおける別の値に依存するかもしれない。この場合、その構造自体はそれが無効である構造から有効性を定めることができないとしてこの有効な使い方を参照しないだろう --- 代わりにこの有効な使い方は呼び出すコマンドに付随するだろう。
>
> もうひとつの例は描画状態である --- その状態を設定するものは独立であり、ドローコール間の正当に無効な状態構成を引き起こすことができる。つまり、有効な使い方のステートメントはすべての状態が有効である必要がある場所に付随する - その描画コマンドで。

有効な使い方の条件は以下のそれらが適用される各コマンドや構造に"有効な使い方[Valid Usage]"とラベル付けされたブロックに述べられる。

### 暗黙の有効な使い方 {id="sec:2.7.2"}

いくつかの有効な使い方の条件は、特定のコマンドや構造に対して明示的に示されていない限り、APIにおけるすべてのコマンドと構造に適用される。これらの条件は *暗黙的* と見なされ、それらが適用される各コマンドや構造に"有効な使い方(暗黙的)"とラベル付けされたブロックに述べられる。暗黙の有効な使い方の条件は以下にその詳細を述べる。

#### オブジェクトハンドルに対する有効な使い方

TODO

## 2.8

### 2.8.1 {id="sec:2.8.1"}

# 3

# 4

# 5

<a id="sec:5.reset" />Each command buffer manages ...

# 6 {id="sec:6"}

## 6.1

<a id="sec:6.1.availability" /><a id="sec:6.1.visibility" />Two additional types ...

## 6.2

<a id="sec:6.2.submission_order">*提出順序*</a>
