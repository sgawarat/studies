---
title: Vulkan 1.0.68 - A Specification
---
# はじめに {id="sec:1"}

## 1.1 {id="sec:1.1"}

## 1.2 {id="sec:1.2"}

## 用語法 {id="sec:1.3"}

この文書におけるキーワード**しなければならない[must]**、**要求されている[required]**、**推奨される[recommend]**、**してもよい[may]**、**選択できる[optional]**はRFC 2119に示される通りに解釈されるものとする^[訳注:日本語訳に際し、この文書では[RFC 2119JA](https://www.ipa.go.jp/security/rfc/RFC2119JA.html)に準拠する。]。

[http://www.ietf.org/rfc/rfc2119.txt](http://www.ietf.org/rfc/rfc2119.txt)

しなければならない[must]
: 単独で使われるとき、この語、または、**要求されている[required]**はその定義が仕様の絶対的な要請事項であることを意味する。**not**が続くとき("**してはならない[must not]**")、このフレーズはその定義が仕様の絶対的な禁止事項であること意味する。

する必要がある[should]
: 単独で使われるとき、この語は、特定の状況において特定の項目を無視する妥当な理由が存在するかもしれないが、異なる進路を取る前にその完全な意義を理解して慎重に検討しなければならない、ということを意味する。**not**が続くとき("**しないほうがよい[should not]**")、特定の挙動が許容できる、または、非常に有用であるときにこのフレーズは特定の状況において妥当な理由が存在するかもしれないが、このラベルで述べられるいかなる挙動を実装する前にその完全な意義を理解して、その場合分けを慎重に検討**する必要がある**、ということを意味する。文法的に適切な場合、**推奨される[recommend]**、または、**推奨[recommendation]**が**する必要がある**の代わりに使われるかもしれない。

してもよい[may]
: この語、または、形容詞**選択できる[optional]**は項目が真に選択的であることを意味する。あるベンダーは、特定の市場が要求している、または、他のベンダーはその項目を省略するかもしれないのに対し、そのベンダーは同様の項目がその製品を改良すると思っていることを理由に、その項目を含むことを選んでもよい。特定の選択肢を含まない実装は、恐らく機能性に劣るだろうが、その選択肢を含む他の実装と相互運用する準備をしておかなければならない。同様に、特定の選択肢を含む実装はその選択肢を含まない他の実装と相互運用する準備をしておかなければならない(もちろん、その選択肢が提供する機能は除く)。

追加の用語**できる[can]**、**できない[cannot]**は以下のように解釈されるものとする。

できる[can]
: この語は、述べられる特定の挙動がアプリケーションに対して妥当な選択であることを意味し、実装の挙動を指して使われることは一切ない。

できない[cannot]
: この語は述べられる特定の挙動がアプリケーションによって達成不可能であることを意味する。例えば、エントリーポイントが存在しなければ、シェーダコードは処理を表現する能力がない。

!!! note
    この仕様書で使われるように、**できない**と**してはならない**の間には重要な区別がある。**できない**はアプリケーションがAPIを通して表現することも達成することも文字通りできないことを意味する。一方で、**してはならない**は、アプリケーションがAPIを通して表現する能力がないこと、実装のために処理の結果が未定義であり回復不可能となり得ることを意味する。

## 1.4 {id="sec:1.4"}

# 基礎 {id="sec:2"}

## アーキテクチャモデル {id="sec:2.1"}

Vulkanは以下の特性を持つCPU、GPU、その他ハードウェアアクセラレータのアーキテクチャのために設計され、そのAPIも同様に記述される。

- これらのバイトサイズの粒度でアドレス指定可能な、8、16、32、64ビットの符号ありと符号なしの二の補数の整数に対する実行時サポート
- [浮動小数点計算](#sec:2.8.1)の節における範囲と精度の制約を満足する、32、64ビットの浮動小数点型に対する実行時サポート
- これらの型の表現とエンディアンはそのホストと物理デバイスに対して同一で**なければならない**。

!!! note
    Vulkanにおける様々なデータの型と構造はホストと物理デバイスのメモリの間を相互に対応**してもよい**ので、ホストやデバイスのアーキテクチャは両方とも、ポータブルで高性能なアプリケーションを記述するために効率的にそのようなデータへアクセスでき**なければならない**。

## 実行モデル {id="sec:2.2"}

この節はVulkanシステムの実行モデルの概要を述べる。

Vulkanは1つ以上の*デバイス*を露出する。このそれぞれが1つ以上の*キュー*を露出する。これは互いに非同期的に処理を行っ**てもよい**。デバイスによってサポートされる一連のキューは*ファミリー*に分けられる。各ファミリーは1種以上の機能性をサポートし、同様の特徴を持つ複数のキューを含ん**でもよい**。単一のファミリー内のキューは互いに*互換性がある*と見なし、キューのとあるファミリーに対して生成される処理はそのファミリー内のいずれのキューにおいて実行**できる**。この仕様はキューがサポート**してもよい**、グラフィクス、コンピュート、トランスファー、スパースメモリ管理の4種の機能性を定義する。

!!! note
    単一のデバイスは、?[multiple members of one or more of those families]を報告するのではなく、もしくはそれと同様に、複数の同様のキューファミリーを報告**してもよい**。これは、これらのファミリーのメンバーが同様の能力を持つ一方で、これらが直接的に互いへの互換性が*ない*ことを示す。

デバイスメモリはアプリケーションによって明示的に管理される。各デバイスは、異なるメモリエリアを表すために、1つ以上のヒープを公表**してもよい**。メモリヒープはデバイスローカルかホストローカルのいずれかであるが、デバイスからは常に可視である。メモリヒープについての更なる詳細はそのヒープで利用可能なメモリタイプを経由して公開される。実装で使えるように**してもよい**メモリエリアの例は以下を含める。

- *device local*はデバイスへ物理的に接続されるメモリである。
- *device local*、*host visible*はホストから可視であるデバイスローカルなメモリである。
- *host local*、*host visible*はホストに対してローカルであり、デバイスとホストから可視である。

他のアーキテクチャでは、いかなる目的に対して用いることが**できる**単一のヒープのみが存在**してもよい**。

VulkanアプリケーションはVulkanライブラリの呼び出しを経由して発行されるデバイスコマンドを記録したコマンドバッファのサブミッションを通して一連のデバイスを制御する。コマンドバッファの中身は基礎となるハードウェアに特有であり、アプリケーションから不透明である。一度構築されれば、コマンドバッファは実行するためにキューへ一度だけ又は何度もサブミット**できる**。複数のコマンドバッファはアプリケーション内で複数のスレッドを用いることで並列に作ることが**できる**。

異なるキューにサブミットされたコマンドバッファは互いに関して並列に、または、順序を外れて[out of order]さえも実行**してもよい**。単一のキューにサブミットされたコマンドバッファは、[同期の章](#sec:6)でさらに述べられるように、[サブミッション順序](#sec:6.2.submission_order)を尊重する。デバイスによるコマンドバッファの実行はホストの実行に対して非同期でもある。一度コマンドバッファがキューにサブミットされれば、即座にアプリケーションへ制御を返し**てもよい**。デバイスとホストの間や異なるキューの間の同期はアプリケーションの責任である。

### キュー操作 {id="sec:2.2.1"}

Vulkanのキューはデバイスの実行エンジンへのインターフェイスを提供する。これらの実行エンジンに対するコマンドは実行する前にコマンドバッファに記録される。その後、これらのコマンドバッファは多くの*バッチ*における実行に対する*キューサブミッション*コマンドによってキューに提出される。一度キューにサブミットされれば、これらのコマンドは実行を開始し、アプリケーションの更なる介入なしに完了するだろうが、この実行順は多くの[暗黙的および明示的な順序制約](#sec:6)に依存している。

作業は一般的に`vkQueue*`(例えば、[vkQueueSubmit](#sec:5.5.vkQueueSubmit)、[vkQueueBindSparse](#sec:28.7.6.vkQueueBindSparse))の形を取るキューサブミッションコマンドを用いてキューにサブミットされ、作業を開始する前に待機するセマフォのリストと作業を完了したと一度だけシグナルを送るセマフォのリストを任意に取る。作業それ自体は、セマフォのシグナル送信や待機と同様に、すべて*キュー操作*である。

異なるキューでのキュー操作は暗黙的な順序制約を持たず、いかなる順序で実行**してもよい**。キューの間の明示的な順序制約は[セマフォ](#sec:6.4)や[フェンス](#sec:6.3)によって表現**できる**。

単一のキューへのコマンドバッファサブミッションは[サブミッション順序](#sec:6.2.submission_order)やその他の[暗黙的な順序保証](#sec:6.2)を尊重するが、そうでなければ、重ね合わせたり順序を外れて実行**してもよい**。その他のバッチや単一のキューに対するキューサブミッションのタイプ(例えば、[スパースメモリバインディング](#sec:28.7.6.sparse_memory_binding))は他のいかなるキューサブミッションまたはバッチに対する暗黙的な順序制約を持たない。キューサブミッションと個々のバッチとの間の追加の明示的な順序制約は[セマフォ](#sec:6.4)や[フェンス](#sec:6.3)によって表現できる。

フェンスまたはセマフォがシグナル状態になる前に、以前にサブミットされたいかなるキュー操作も実行を完了し、これらのキュー操作からのメモリ書き込みが将来のキュー操作で[可用](#sec:6.1.availability)であることが保証される。シグナル状態のセマフォまたはフェンスを待機することは可用である以前の書き込みが後続のコマンドから[可視](#sec:6.1.visibility)でもあることを保証する。

プライマリとセカンダリのコマンドバッファの間、および、同じまたは異なるバッチやサブミッションのプライマリコマンドバッファの間の両方のコマンドバッファ境界は追加のいかなる順序制約も導入しない。言い換えれば、いかなるセマフォまたはフェンスの間の(セカンダリコマンドバッファの実行を含めることが**できる**)一連のコマンドバッファのサブミットは、各境界で現在の状態が[リセット](#sec:5.reset)されることを除いて、それらがすべて単一のプライマリコマンドバッファに記録されたかのように、記録されたコマンドを実行する。明示的な順序制約は[明示的な同期プリミティブ](#sec:6)によって表現**できる**。

!!! note
    実装はキューにサブミットされた作業の実行を重ね合わせるための著しい自由度を持つ。そして、これはVulkanデバイスにおける深いパイプライン処理や並列性のために普通のことである。

コマンドバッファに記録されるコマンドはアクションを処理する(描画、ディスパッチ、クリア、コピー、クエリ/タイムスタンプ操作、サブパス開始/終了操作)、ステートを設定する(パイプライン、デスクリプタセット、バッファのバインド、動的ステートの設定、push constants、レンダパス/サブパスのステートの設定)、同期を処理する(イベントの設定/待機、パイプラインバリア、レンダパス/サブパスの依存関係)、のいずれかを行う。いくつかのコマンドはこれらタスクのひとつかそれ以上を処理する。ステートを設定するコマンドはコマンドバッファの*現在の状態*を更新する。アクションを処理するいくつかのコマンド(例えば、描画/ディスパッチ)はコマンドバッファの開始から累積的に設定された現在の状態に基づいて行う。アクションコマンドを処理するのに関係する作業はしばしば重ね合わせたり並べ替えたりできるが、そうすることが各アクションコマンドによって用いられるステートを変更**してはならない**。一般に、アクションコマンドは、フレームバッファアタッチメントを変更したり、バッファやイメージのメモリを読み書きしたり、クエリプールに書き込んだりするコマンドである。

同期コマンドは2つの一連のアクションコマンドの間の明示的な[実行およびメモリ依存性](#sec:6.1)を導入する。ここでは、後者のコマンド一式が前者のコマンド一式に依存する。これらの依存関係は前者におけるとある[パイプラインステージ](#sec:6.2.1)の実行の後に後者におけるとあるパイプラインステージの実行が起こること、および、とあるパイプラインステージによって処理される[メモリアクセス](#sec:6.7.2)の効果が順番に起こるの両方を強制し、そして、互いに可視である。明示的な依存関係または[暗黙的な順序保証](#sec:6.2)によって強制されないとき、アクションコマンドは実行を重ね合わせたり順序を外れて実行**してもよく**、互いのメモリアクセスの副次的効果を確認しなく**てもよい**。

デバイスはホストに関して非同期にキュー操作を実行する。制御は、キューへのコマンドバッファサブミッションに従い、アプリケーションへ即座に返される。アプリケーションは必要に応じてホストとデバイスの間の作業を同期**しなければならない**。

## オブジェクトモデル {id="sec:2.3"}

Vulkanにおけるデバイス、キュー、その他のエンティティはVulkanオブジェクトによって表される。APIレベルでは、すべてのオブジェクトはハンドルによって参照される。ハンドルにはディスパッチ可能[dispatchable]とディスパッチ不能[non-dispatchable]の2つクラスが存在する。*ディスパッチ可能*なハンドル型は不透明型へのポインタである。このポインタはAPIコマンドを傍受する一環としてレイヤーによって使われ**てもよい**。そしてそれ故に、各APIコマンドはその第1引数としてディスパッチ可能型を取る。ディスパッチ可能型の各オブジェクトはその寿命の間に唯一のハンドル値を持た**なければならない**。

*ディスパッチ不能*なハンドル型は実装依存の意味を持つ64ビット整数型であり、ソフトウェア構造へのポインタではなくハンドルにオブジェクト情報を直接エンコード**してもよい**。ディスパッチ不能型のオブジェクトは型の内部に、または、型にわたって唯一のハンドル値を持たなく**てもよい**。ハンドル値が唯一でないならば、そのようなハンドルのひとつを破壊しても他の型の同一ハンドルが無効になるようなことがあっ**てはならず**、同じ型の同一ハンドルでは、そのハンドル値の生成された回数が破壊された回数よりも多いならば、無効になるようなことがあっ**てはならない**。

ある`VkDevice`から生成または割り当てられた(すなわち、第1引数として`VkDevice`を伴う)すべてのオブジェクトはそのデバイスに対してプライベートであり、他のデバイスで用いられ**てはならない**。

### オブジェクト寿命 {id="sec:2.3.1"}

オブジェクトは`vkCreate*`や`vkAllocate*`コマンドによってそれぞれ生成または割り当てられる。一度オブジェクトが生成または割り当てられれば、その"構造"は不変であると見なされるが、あるオブジェクト型の中身は依然として自由に変更できる。オブジェクトは`vkDestroy*`や`vkFree*`コマンドによってそれぞれ破壊または解放される。

(生成ではなく)割り当てられるオブジェクトは既存のプールオブジェクトまたはメモリヒープからリソースを取り、解放されるときにリソースをそのプールまたはヒープに返す。オブジェクトの生成や破壊は一般に実行時の間に低頻度で発生することが期待される一方で、割り当てや解放は高頻度で発生することが**できる**。プールオブジェクトは割り当てや解放のパフォーマンス改善に適応するのに役立つ。

Vulkanオブジェクトの寿命を追跡することと使用中に破壊しないことのはアプリケーションの責任である。

<a id="sec:2.3.1.ownership_temporarily_acquired" />アプリケーション所有のメモリの所有権は渡されたVulkanコマンドによって即座に取得される。そのようなメモリの所有権はそのコマンドの存続期間の終了時にアプリケーションへ返却**しなければならない**ため、アプリケーションは、このメモリを取得したすべてのコマンドが返った後すぐに、そのメモリを変更したり開放したりすることが**できる**。

以下のオブジェクト型は、Vulkanコマンドに渡され、生成するのに使われるオブジェクトによってそれ以上にアクセスされないとき、消費される。これらは渡されるいかなるAPIコマンドの存続期間中にも破壊され**てはならない**。

- `VkShaderModule`
- `VkPipelineCache`

他のオブジェクトを生成するために引数として渡される`VkRenderPass`オブジェクトは渡されたコマンドの存続期間の後でそのオブジェクトによってそれ以上にアクセスされない。コマンドバッファで使われる`VkRenderPass`は以下に示される規定に従う。

`VkPipelineLayout`オブジェクトはそれを使うコマンドバッファが記録状態にある間に破壊**してはならない**。

`VkDescriptorSetLayout`オブジェクトはこのレイアウトを用いて割り当てられたデスクリプタセットで操作するコマンドによってアクセスされ**てもよく**、これらのデスクリプタセットはデスクリプタセットレイアウトが破壊された後に[vkUpdateDescriptorSets](#sec:13.2.4.vkUpdateDescriptorSets)によって更新**してはならない**。そうでなければ、デスクリプタセットレイアウトはAPIコマンドによって使用中でないいずれかのときに破壊**できる**。

アプリケーションは他のいかなるVulkanオブジェクト型を、そのオブジェクトがデバイスによって(コマンドバッファの実行などを介して)使い終わるまで、破壊**してはならない**。

以下のVulkanオブジェクトはそのオブジェクトを使用しているいずれかのコマンドバッファが[待機状態](#sec:5.1)にある間に破壊**してはならない**。

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

これらのオブジェクトを破壊しようとすると、[記録または実行可能状態](#sec:5.1)にあるいずれかのコマンドバッファが動き出し、これらのオブジェクトを使っていると、[無効状態](#sec:5.1)に遷移する。

以下のVulkanオブジェクトはいずれかのキューがそのオブジェクトを使うコマンドを実行している間に破壊**してはならない**。

- `VkFence`
- `VkSemaphore`
- `VkCommandBuffer`
- `VkCommandPool`

一般に、(コマンドバッファのリセットのような)他のオブジェクトをもう使わないような方法で破壊またはリセットされることを除いたいかなる方法で解放されたオブジェクトを使うオブジェクトがそれ以上に使われない場合に限り、解放されようとしているオブジェクトが他のオブジェクトの使用(例えば、ビューによるリソースの使用、デスクリプタセットによるビューの使用、コマンドバッファによるオブジェクトの使用、リソースへのメモリ割り当てのバインディング)に関与していても、オブジェクトは任意の順序で破壊または解放**できる**。そのオブジェクトがリセットされていたならば、解放されたオブジェクトを一切用いていないかのように使うことが**できる**。これの例外はオブジェクト間に親子関係が存在するときである。この場合、アプリケーションは、親が破壊されるときにその子供を解放すると明示的に定義される(例えば、下に定義されるような、プールオブジェクトで)ときを除いて、その子オブジェクトを破壊する前に親オブジェクトを破壊**してはならない**。

`VkCommandPool`オブジェクトは`VkCommandBuffer`オブジェクトの親である。`VkDescriptorPool`オブジェクトは`VkDescriptorSet`オブジェクトの親である。`VkDevice`オブジェクトは多くのオブジェクト型の親である(それらの生成への引数として`VkDevice`を取るすべて)。

以下のVulkanオブジェクトは破壊**できる**ときに対して固有の制限を持つ。

- `VkQueue`オブジェクトは明示的に破壊**できない**。代わりに、それらが取得[retrieve]される`VkDevice`オブジェクトが破壊されるときに暗黙的に破壊される。
- プールオブジェクトを破壊すると、そのプールから割り当てられたすべてのオブジェクトが暗黙的に解放される。具体的には、`VkCommandPool`を破壊すると、それから割り当てられたすべての`VkCommandBuffer`が解放され、`VkDescriptorPool`を破壊すると、それから割り当てられたすべての`VkDescriptorSet`が解放される。
- `VkDevice`オブジェクトはそれらから取得されたすべての`VkQueue`オブジェクトがアイドルであり、それらから生成されたすべてのオブジェクトが破壊されたときに破壊**できる**。これは以下のオブジェクトを含む。
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
- `VkPhysicalDevice`オブジェクトは明示的に破壊**できない**。代わりに、それらが取得[retrieve]される`VkInstance`オブジェクトが破壊されるときに暗黙的に破壊される。
- `VkInstance`は、その`VkPhysicalDevice`オブジェクトのいずれかから生成されたすべての`VkDevice`オブジェクトが破壊されれば、破壊**できる**。

## 2.4 {id="sec:2.4"}

## コマンドの構文と存続期間 {id="sec:2.5"}

この仕様書はVulkanコマンドをC99の構文を用いる関数または手続きとして記述する。C++やJavaScriptのような他の言語に対する言語バインディングはより厳密な引数渡しやオブジェクト指向インターフェイスを可能に**してもよい**。

Vulkanは、以下に述べられる場合を除いて、適切なときにスカラパラメータの基本型や文書の他の場所で標準Cの型を用いる。

`VkBool32`は、Cが十分に移植性のあるビルトインのブーリアン型を持たないので、ブーリアンの`True`および`False`の値を表す。

```c
typedef uint32_t VkBool32;
```

`VK_TRUE`はブーリアンの**真**(整数の1)の値を表し、`VK_FALSE`はブーリアンの**偽**(整数の0)の値を表す。

Vulkan実装から`VkBool32`で返されるすべての値は`VK_TRUE`か`VK_FALSE`のいずれかであるだろう。

アプリケーションは`VkBool32`が期待されるところのVulkan実装に`VK_TRUE`と`VK_FALSE`以外のいかなる値も渡し
**てはならない**。

`VkDeviceSize`はデバイスメモリのサイズとオフセットの値を表す。

```c
typedef uint64_t VkDeviceSize;
```

Vulkanオブジェクトを生成するコマンドは`vkCreate*`の形であり、オブジェクトを生成するのに必要な引数とともに`Vk*CreateInfo`構造体を取る。これらのVulkanオブジェクトは`vkDestroy*`の形のコマンドで破壊される。

Vulkanオブジェクトを生成または破壊する各コマンドへの最後の入力引数は`pAllocator`である。`pAllocator`引数は与えられたオブジェクトに対する割り当てがアプリケーションによって提供されるコールバックへ委任されるような非`NULL`値に設定**できる**。更なる詳細は[メモリ割り当て](#sec:10.1.memory_allocation)の章を参照のこと。

プールオブジェクトによって所有されるVulkanオブジェクトを割り当てるコマンドは`vkAllocate*`の形であり、`Vk*AllocateInfo`構造体を取る。これらのVulkanオブジェクトは`vkFree*`の形のコマンドで解放される。これらのオブジェクトはアロケータを取らない。ホストメモリが必要とされるならば、これらの親のプールが生成されたときに指定されたアロケータを用いるだろう。

コマンドは`vkCmd*`の形のAPIコマンドの呼び出しによってコマンドバッファに記録される。そのようなコマンドは、プライマリおよび/またはセカンダリコマンドバッファの中、レンダパスの内外、サポートされるクエリタイプのひとつまたはそれ以上の中といった、使用**できる**ところでそれぞれ異なる制限を持っ**てもよい**。これらの制限はそのようなコマンドそれぞれの定義とともに文書化される。

Vulkanコマンドの*存続期間[duration]*はコマンドの呼び出しから呼び出し元へのリターンまでの時間間隔を指す。

### 取得された結果の寿命 {id="sec:2.5.1"}

情報は`vkGet*`や`vkEnumerate*`の形のコマンドで実装から取得される。

個々のコマンドで指定されない場合を除き、その結果は*不変[invariant]*である。すなわち、引数が有効のままである限り、同じ引数による同じコマンドの呼び出しによって再び取得されるときでもこれらは変わらないままだろう。

## スレッディングの挙動 {id="sec:2.6"}

Vulkanは複数のホストスレッドで用いられるときにスケール可能なパフォーマンスを提供することが意図される。すべてのコマンドは複数スレッドから並行に呼び出されることをサポートするが、ある引数または引数の構成要素は*外部同期[externally synchronized]*されることが定義されている。これは、与えられた時間にそのような引数を用いているスレッドが1つ以上存在しないことを呼び出し元が保証**しなければならない**ことを意味する。

より正確に言うと、VulkanコマンドはVulkanオブジェクトを表現するソフトウェア構造を更新するために単純な格納領域を用いる。外部同期されるとして宣言された引数はホストがコマンド実行している時間にも更新されるそのソフトウェア構造を持っ**てもよい**。2つのコマンドが同じオブジェクトで操作し、少なくともコマンドのひとつが外部同期されるオブジェクトを宣言するならば、呼び出し元は、コマンドが同時に実行しないことだけでなく、(必要であれば)2つのコマンドが適切なメモリバリアによって分けられることも保証**しなければならない**。

!!! note
    メモリバリアは、多くの開発者が慣れているx86/x64プログラミングのものより弱い順序付けがされる、ARMのCPUアーキテクチャに特に関係する。幸いにも、(pthreadライブラリのような)ほとんどの高レベル同期プリミティブは相互排他の一部としてメモリバリアを処理するので、これらのプリミティブを介してVulkanオブジェクトを相互排他することは望ましい効果を持つだろう。

同様に、アプリケーションはVulkanコマンドによって[一時的に取得される所有権](#sec:2.3.1.ownership_temporarily_acquired)を持つアプリケーション所有のメモリのいかなる潜在的なデータハザードも回避**しなければならない**。アプリケーション所有のメモリの所有権はコマンドによって取得されたままである一方で、実装は任意の時にそのメモリを読み込ん**でもよく**、任意の時に`const`修飾されていないメモリに書き込ん**でもよい**。`const`修飾されていないアプリケーション所有のメモリを参照している引数は仕様書では*外部同期*されるとして明示的に印付けされない。

多くのオブジェクト型は*不変[immutable]*である。これは、一度生成されればオブジェクトは変更**できない**ことを意味する。これらのオブジェクト型は、他のスレッドで使用中に破壊**してはならない**ことを除いて、外部同期を一切必要としない。とある特殊な場合では、可変[mutable]なオブジェクトパラメータは外部同期を必要としないかのように内部で同期される。この一例は`vkCreateGraphicsPipelines`や`vkCreateComputePipelines`における`VkPipelineCache`の使用である。ここでは、そんな重量級コマンドの周りの外部同期は実用的ではないだろう。その実装はこの例においてキャッシュを内部的に同期**しなければならず**、そのコマンドの周りにより粒度の細かいミューテックスの形で行うことを可能に**してもよい**。外部同期されるとしてラベリングされていないいずれかコマンド引数はコマンドによって変わらないか、内部的に同期されているかのどちらかである。加えて、コマンドの引数に関連するあるオブジェクト(例えば、コマンドプールやデスクリプタプール)はコマンドによって影響を受け**てもよく**、また、外部同期され**なければならない**。これらの暗黙の引数は以下で述べられるように文書化される。

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

コマンドが外部同期される引数をその中身に持つユーザーの割り当てたリストを取ることが**できる**いくつかの例も存在する。これらの場合、その呼び出し元は多くても1つのスレッドが与えられた時間にリスト内の与えられた要素を使っていることを保証**しなければならない**。これらの引数は以下にリスト化される。

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

Vulkanは層状のAPIである。最下層は、この仕様書で定義されるような、コアとなるVulkanレイヤーである。アプリケーションはデバッグ、検証、その他の目的のためにコアの上に追加のレイヤーを用いる**できる**。

Vulkanのコアとなる原則のひとつはコマンドバッファの構築とサブミットが高効率である**必要がある**ことである。故に、コアレイヤーにおける状態のエラーの確認と検証は最小限であり、より厳格な検証はレイヤーの使用を通じて有効化**できる**。

コアレイヤーはアプリケーションが正しくAPIを使っていることを仮定する。仕様書の他の場所で文書化されるような場合を除き、間違ってAPIを使用するアプリケーションへのコアレイヤーの挙動は未定義であり、プログラムの終了を含ん**でもよい**。しかし、実装はアプリケーションによる間違った使い方がオペレーティングシステム、Vulkan実装、システム内の他のVulkanクライアントアプリケーションの保全性に影響を与えないことを保証**しなければならない**。特に、あるプロセスからのメモリが他のプロセスかが可視に**できる**かどうかについてオペレーティングシステムによって行われる保証は**いかなるメモリ割り当て**に対してもVulkan実装によって違反され**てはならない**。アプリケーションの特定の機能や拡張(例えば、[堅牢なバッファアクセス](#sec:30.1.robust_buffer_access))の使用によって明示的に指示されない限り、Vulkan実装はOSによって提供されるこれらを越えて追加のセキュリティまたは保全性の保証を行うことを**要求されていない**。

!!! note
    例えば、オペレーティングシステムがすべてのメモリ割り当てにおけるデータが新しく割り当てられたときにゼロに設定されることを保証するならば、Vulkan実装はそれが制御する割り当てに対して同じ保証を行わ**なければならない**(例えば、[VkDeviceMemory](#sec:10.2.VkDeviceMemory))。

アプリケーションは「[機能、制限、フォーマット](#sec:30)」章において述べられるように、`robustBufferAccess`機能を有効化することでより強力な堅牢性保証を要求することが**できる**。

正しいAPIの使い方の検証は検証レイヤーに委ねられる。アプリケーションは、エラーを捕捉して排除するのに役立つため、有効化された検証レイヤーとともに開発される**必要がある**。一度検証されれば、リリースされたアプリケーションはデフォルトで検証レイヤーを有効化**しないほうがよい**。

### 有効な使い方 {id="sec:2.7.1"}

有効な使い方はアプリケーションにおいてうまく定義された実行時の挙動を達成するために満たさ**なければならない**一連の条件を定義する。これらの条件はVulkanの状態およびその使い方が条件によって制約される引数やオブジェクトにのみ依存する。

いくつかの有効な使い方の条件は実行時の制限や機能の可用性との依存関係を持つ。これらの制限と機能のためのVulkanのサポートされる最小値、または、他の既知の値のいくつかのサブセットに対するこれらの条件を検証することが可能である。

有効な使い方の条件は(エラーコードのリターンを含む)うまく定義された挙動が存在する条件を取り扱わない。

有効な使い方の条件はその条件についての完璧な情報がアプリケーションの実行中に知られているであろうコマンドや構造に適用される**必要がある**。これは検証レイヤーやlinterがそれらの指定される時点でこれらのステートメントに対して直接書かれることが**できる**のようなものである。

!!! note
    これは有効な使い方のステートメントに対するいくつかの明確でない場所を引き起こす。例えば、構造に対する有効な値は呼び出すコマンドにおける別の値に依存するかもしれない。この場合、その構造自体はそれが無効である構造から有効性を定めることができないとしてこの有効な使い方を参照しないだろう --- 代わりにこの有効な使い方は呼び出すコマンドに付随するだろう。
    <!-- -->
    もうひとつの例は描画状態である --- その状態を設定するものは独立であり、ドローコール間の正当に無効な状態構成を引き起こすことができる。つまり、有効な使い方のステートメントはすべての状態が有効である必要がある場所に付随する - その描画コマンドで。

有効な使い方の条件は以下のそれらが適用される各コマンドや構造に"有効な使い方[Valid Usage]"とラベル付けされたブロックに述べられる。

### 暗黙の有効な使い方 {id="sec:2.7.2"}

いくつかの有効な使い方の条件は、特定のコマンドや構造に対して明示的に示されていない限り、APIにおけるすべてのコマンドと構造に適用される。これらの条件は*暗黙的*と見なされ、それらが適用される各コマンドや構造に"有効な使い方(暗黙的)[Valid Usage (Implicit)]"とラベル付けされたブロックに述べられる。暗黙の有効な使い方の条件は以下にその詳細を述べる。

#### オブジェクトハンドルに対する有効な使い方

オブジェクトハンドルであるコマンドのいずれかの入力引数は、そうでないと指定されない限り、有効なオブジェクトで**なければならない**。オブジェクトハンドルは以下であれば有効である。

- 以前の成功したAPI呼び出しによって生成または割り当てられた。そのような呼び出しは仕様書に記載されている。
- 以前のAPI呼び出しによって消去または解放されなかった。そのような呼び出しは仕様書に記載されている。
- 生成や実行の一部としてそのオブジェクトに使われるいずれかのオブジェクトもまた有効で**なければならない**。

予約された値[VK_NULL_HANDLE](#sec:D.VK_NULL_HANDLE)と`NULL`は、*仕様書に明記される[explicitly called out in the specification]*とき、それぞれ有効なディスパッチ不能ハンドルとディスパッチ可能ハンドルの代わりに使用**できる**。正常にオブジェクトを生成するいずれかのコマンドはこれらの値を返し**てはならない**。これらの値を`vkDestroy*`や`vkFree*`コマンドに渡すことは有効であり、そのときはこれらの値を淡々と無視するだろう。

#### ポインタに対する有効な使い方

ポインタであるいずれかの引数は有効な使い方のステートメントによって明記される場合に限り*有効なポインタ[valid pointer]*で**なければならない**。

ポインタは、それがコマンドによって期待される数値や型の値を含むメモリを指し、(例えば、配列の要素や構造体のメンバのような)ポインタを通してアクセスされるすべての基本型がホストプロセッサのアライメント要件を満足するならば、"有効"である。

#### 文字列に対する有効な使い方

`char`へのポインタであるいずれかの引数は、null文字で終端された値の有限の列で**なければならず**、もしくは、*仕様書に明記される*ならば、`NULL`にすることが**できる**。

#### 列挙型に対する有効な使い方

列挙型のいずれかの引数はその型の有効な列挙子[enumerant^[訳注:ラテン語由来の語である"enamerate"を形容詞化および名詞化するための語尾"-ant"に変化させたもの。]]で**なければならない**。列挙子は以下の場合に有効である。

- 列挙子は列挙型の一部として定義される。
- 列挙子は、`_BEGIN_RANGE`、`_END_RANGE`、`_RANGE_SIZE`、`_MAX_ENUM`[^special_values_for_enum]が語尾に付いた、列挙型のために定義された特別な値のひとつでない。

[^special_values_for_enum]: これらの特殊なトークンの意味はVulkanの仕様書において公開されない。これらはAPIの一部ではなく、アプリケーションで使われないようにする**必要がある**。これらの元々意図された使い方はVulkan実装による内部消費のためであった。この使い方は将来的にもはやサポートされないだろうが、後方互換性の理由のために保持されるだろう。

クエリコマンドから返される、またはそうでなければ、Vulkanからアプリケーションに出力される、いかなる列挙型も予約された値を持っ**てはならない**。予約された値はその列挙型に対するいずれの拡張によっても定義されない値である。

!!! note
    この言葉は、ドライバ内部だけが知っている拡張や、アプリケーションの情報なしに拡張を有効化するレイヤーのような"隠された"場合を、いかなる拡張によっても定義されない値の返却を許可しないことで、融通することを意図している。

#### フラグに対する有効な使い方

フラグの集まりは`VkFlags`型を用いるビットマスクによって表される。

```c
typedef uint32_t VkFlags;
```

ビットマスクは選択肢をコンパクトに表すために多くのコマンドと構造に渡されるが、`VkFlags`はAPIにおいて直接使われない。代わりに、`VkFlags`の別名である`Vk*Flags`型が使われる。その名前は、この型に対して有効である対応する`Vk*FlagBits`と一致する。これらの別名は仕様書の付録「[フラグ型](#sec:X.flag_types)」において述べられる。

APIにおいて入力として使われる引数のメンバー`Vk*Flags`はいずれもビットフラグの有効な組み合わせで**なければならない**。有効な組み合わせはゼロか有効なビットフラグのビット単位ORのどちらかである。ビットフラグは以下の場合に有効である。

- そのビットフラグは`Vk*FlagBits`の一部として定義される。そこでは、ビット型はフラグ型を取り、語尾の`Flags`を`FlagBits`に置き換えることによって得られる。例えば、[VkColorComponentFlags](#sec:X.X.VkColorComponentFlags)型のフラグ値は[VkColorComponentFlagBits](#sec:X.X.VkColorComponentFlagBits)によって定義されるビットフラグのみを含ま**なければならない**。
- そのフラグはそれが使われている文脈に応じて許可される。例えば、いくつかの場合、あるビットフラグやビットフラグの組み合わせは相互排他的である。

クエリコマンドから返される、またはそうでなければ、Vulkanからアプリケーションに出力される、いずれかのメンバーまたは引数の`Vk*Flags`はその対応する`Vk*FlagBits`型に定義されていないビットフラグを含ん**でもよい**。アプリケーションはこれらの明示されていないビットの状態に依存**できない**。

#### 構造タイプに対する有効な使い方

メンバー`sType`を含む構造体であるいかなる引数も構造体の型に一致する有効な[VkStructureType](#sec:X.X.VkStructureType)値である`sType`の値を持た**なければならない**。

Vulkan APIによってサポートされる構造タイプは以下を含める。

```c
typedef enum VkStructureType {
    VK_STRUCTURE_TYPE_APPLICATION_INFO = 0,
    VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO = 1,
    VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO = 2,
    VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO = 3,
    VK_STRUCTURE_TYPE_SUBMIT_INFO = 4,
    VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO = 5,
    VK_STRUCTURE_TYPE_MAPPED_MEMORY_RANGE = 6,
    VK_STRUCTURE_TYPE_BIND_SPARSE_INFO = 7,
    VK_STRUCTURE_TYPE_FENCE_CREATE_INFO = 8,
    VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO = 9,
    VK_STRUCTURE_TYPE_EVENT_CREATE_INFO = 10,
    VK_STRUCTURE_TYPE_QUERY_POOL_CREATE_INFO = 11,
    VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO = 12,
    VK_STRUCTURE_TYPE_BUFFER_VIEW_CREATE_INFO = 13,
    VK_STRUCTURE_TYPE_IMAGE_CREATE_INFO = 14,
    VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO = 15,
    VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO = 16,
    VK_STRUCTURE_TYPE_PIPELINE_CACHE_CREATE_INFO = 17,
    VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO = 18,
    VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO = 19,
    VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO = 20,
    VK_STRUCTURE_TYPE_PIPELINE_TESSELLATION_STATE_CREATE_INFO = 21,
    VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO = 22,
    VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO = 23,
    VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO = 24,
    VK_STRUCTURE_TYPE_PIPELINE_DEPTH_STENCIL_STATE_CREATE_INFO = 25,
    VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO = 26,
    VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO = 27,
    VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO = 28,
    VK_STRUCTURE_TYPE_COMPUTE_PIPELINE_CREATE_INFO = 29,
    VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO = 30,
    VK_STRUCTURE_TYPE_SAMPLER_CREATE_INFO = 31,
    VK_STRUCTURE_TYPE_DESCRIPTOR_SET_LAYOUT_CREATE_INFO = 32,
    VK_STRUCTURE_TYPE_DESCRIPTOR_POOL_CREATE_INFO = 33,
    VK_STRUCTURE_TYPE_DESCRIPTOR_SET_ALLOCATE_INFO = 34,
    VK_STRUCTURE_TYPE_WRITE_DESCRIPTOR_SET = 35,
    VK_STRUCTURE_TYPE_COPY_DESCRIPTOR_SET = 36,
    VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO = 37,
    VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO = 38,
    VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO = 39,
    VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO = 40,
    VK_STRUCTURE_TYPE_COMMAND_BUFFER_INHERITANCE_INFO = 41,
    VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO = 42,
    VK_STRUCTURE_TYPE_RENDER_PASS_BEGIN_INFO = 43,
    VK_STRUCTURE_TYPE_BUFFER_MEMORY_BARRIER = 44,
    VK_STRUCTURE_TYPE_IMAGE_MEMORY_BARRIER = 45,
    VK_STRUCTURE_TYPE_MEMORY_BARRIER = 46,
    VK_STRUCTURE_TYPE_LOADER_INSTANCE_CREATE_INFO = 47,
    VK_STRUCTURE_TYPE_LOADER_DEVICE_CREATE_INFO = 48,
} VkStructureType;
```

各値は一致する名前を持つ`sType`メンバーを含む特定の構造体に対応する。一般的な規定として、各`VkStructureType`の値の名前は、その構造体の名前を取り、先頭の`Vk`を取り除き、各頭文字の前に`_`を付け、結果の文字列全体を大文字に変換し、その前に`VK_STRUCTURE_TYPE_`を付けることで得られる。例えば、構造体`VkImageCreateInfo`は`VK_STRUCTURE_TYPE_IMAGE_CREATE_INFO`の`VkStructureType`に対応し、故に、そのメンバー`sType`はAPIに渡されるときにそれと等しく**なければならない**。

`VK_STRUCTURE_TYPE_LOADER_INSTANCE_CREATE_INFO`と`VK_STRUCTURE_TYPE_LOADER_DEVICE_CREATE_INFO`はローダーによる内部使用のために予約され、この仕様書において対応するVulkan構造体を持たない。

#### 構造ポインタチェインに対する有効な使い方

メンバー`void*pNext`を含む構造体であるいかなる引数も、`NULL`である、または、文書「[Vulkan Documentation and Extensions]()」における"Extension Interactions"の章で述べられるように`sType`と`pNext`を含む、拡張によって定義される有効な構造対を指す、のいずれかである`pNext`の値を持た**なければならない**。ポインタ`pNext`によって接続される一連の構造体は*`pNext`チェイン[pNext chain]*と呼ばれる。その拡張が実装によってサポートされるならば、それは有効化され**なければならない**。

有効な構造体の各型は`pNext`チェイン中に1回以上現れ**てはならない**。

実装のいずれかの構成要素(ローダー、有効化されたレイヤー、ドライバ)は、その構成要素によってサポートされる拡張によって定義されない`sType`値を持つチェインにおけるいかなる構造体も(`sType`と`pNext`メンバーを読むこと以外に)処理をせず、読み飛ばさ**なければならない**。

拡張の構造体はベースとなるVulkan仕様書では述べられず、これらの拡張が組み込まれるレイヤーの仕様書か別のベンダーが提供する文書に記載される。

#### ネストした構造に対する有効な使い方

上記の条件は、コマンドへの直接の引数として、または、それら自体が他の構造体のメンバーとして、コマンドへの入力として提供される構造体のメンバーにも再帰的に適用される。

各コマンドの有効な使い方の詳細はそれらの個々の節で取り扱われる。

### リターンコード {id="sec:2.7.3"}

コアとなるVulkan APIは間違った使い方を捕捉するように設計されていないが、いくつかの状況はリターンコードが必要とする。Vulkanにおけるコマンドは2つのカテゴリーの内のひとつにあるリターンコードを介してそれらのステータスを返す。

- 正常完了コードはコマンドが成功またはステータス情報を伝える必要があるときに返される。すべての正常完了コードは非負数である。
- 実行時エラーコードはコマンドが実行時のみに検出される可能性がある失敗を伝える必要があるときに返される。すべての実行時エラーコードは負数である。

Vulkanにおけるすべてのリターンコードは[VkResult](#sec:X.X.VkResult)の返り値を介して報告される。可能性のある値は以下である。

```c
typedef enum VkResult {
    VK_SUCCESS = 0,
    VK_NOT_READY = 1,
    VK_TIMEOUT = 2,
    VK_EVENT_SET = 3,
    VK_EVENT_RESET = 4,
    VK_INCOMPLETE = 5,
    VK_ERROR_OUT_OF_HOST_MEMORY = -1,
    VK_ERROR_OUT_OF_DEVICE_MEMORY = -2,
    VK_ERROR_INITIALIZATION_FAILED = -3,
    VK_ERROR_DEVICE_LOST = -4,
    VK_ERROR_MEMORY_MAP_FAILED = -5,
    VK_ERROR_LAYER_NOT_PRESENT = -6,
    VK_ERROR_EXTENSION_NOT_PRESENT = -7,
    VK_ERROR_FEATURE_NOT_PRESENT = -8,
    VK_ERROR_INCOMPATIBLE_DRIVER = -9,
    VK_ERROR_TOO_MANY_OBJECTS = -10,
    VK_ERROR_FORMAT_NOT_SUPPORTED = -11,
    VK_ERROR_FRAGMENTED_POOL = -12,
} VkResult;
```

成功コード

- `VK_SUCCESS` コマンドが正常に完了した。
- `VK_NOT_READY` フェンスやクエリがまだ完了していなかった。
- `VK_TIMEOUT` 待機操作が指定の時間で完了しなかった。
- `VK_EVENT_SET` イベントがシグナル状態である。
- `VK_EVENT_RESET` イベントが非シグナル状態である。
- `VK_INCOMPLETE` 返す配列が結果に対して小さすぎた。

エラーコード

- `VK_ERROR_OUT_OF_HOST_MEMORY` ホストメモリの割り当てが失敗した。
- `VK_ERROR_OUT_OF_DEVICE_MEMORY` デバイスメモリの割り当てが失敗した。
- `VK_ERROR_INITIALIZATION_FAILED` オブジェクトの初期化が実装固有の理由により完了できなかった。
- `VK_ERROR_DEVICE_LOST` 論理または物理デバイスがロストした。[ロストしたデバイス](#sec:4.2.3)を参照。
- `VK_ERROR_MEMORY_MAP_FAILED` メモリオブジェクトのマッピングが失敗した。
- `VK_ERROR_LAYER_NOT_PRESENT` 要求されたレイヤーが存在していない、または、ロードできなかった。
- `VK_ERROR_EXTENSION_NOT_PRESENT` 要求された拡張がサポートされていない。
- `VK_ERROR_FEATURE_NOT_PRESENT` 要求された機能がサポートされていない。
- `VK_ERROR_INCOMPATIBLE_DRIVER` 要求されたVulkanバージョンがドライバによってサポートされない、または、実装固有の理由により互換性がない。
- `VK_ERROR_TOO_MANY_OBJECTS` 多すぎるその型のオブジェクトがすでに生成されていた。
- `VK_ERROR_FORMAT_NOT_SUPPORTED` 要求されたフォーマットがこのデバイスではサポートされない。
- `VK_ERROR_FRAGMENTED_POOL` プールの割り当て処理がプールのメモリのフラグメンテーションのために失敗した。これはホストまたはデバイスメモリを割り当てる試みが新しい割り当てに適応するために行われなかった場合にのみ返され**なければならない**。

コマンドが実行時エラーを返すならば、出力引数が`sType`と`pNext`フィールドを持つ構造体である場合を除いて、出力引数が未定義の内容を持つと指定されない限り、これらのフィールドは変更されないだろう。`pNext`からチェインされたいずれかの構造もまた、`sType`と`pNext`が変更されないであろうことを除いて、未定義の内容を持つ。

メモリ不足エラーはいかなる現時点で存在しているVulkanオブジェクトにも害を与えない。すでに正常に生成されていたオブジェクトは依然としてアプリケーションによって用いることが**できる**。

パフォーマンス重視のコマンドは一般にリターンコードを返さない。そのようなコマンドで実行時エラーが発生するならば、実装は特定の時点までエラー報告を遅らせるだろう。コマンドバッファに記録されるコマンド(`vkCmd*`)では、実行時エラーは`vkEndCommandBuffer`によって報告される。

## 2.8 {id="sec:2.8"}

### 2.8.1 {id="sec:2.8.1"}

# 初期化 {id="sec:3"}

# デバイスとキュー {id="sec:4"}

### ロストしたデバイス {id="sec:4.2.3"}

論理デバイスは、ハードウェアエラー、実行タイムアウト、電力管理イベント、プラットフォーム固有のイベントのために*ロスト*になっ**てもよい**。これは保留および今後のコマンド実行を失敗させたり、ハードウェアリソースを破損させ**てもよい**。これが起こるとき、あるコマンドは`VK_ERROR_DEVICE_LOST`を返すだろう(そのようなコマンドのリストのための[エラーコード](#sec:2.7.3.error_codes)を参照)。いずれかのそのようなイベントのあとに、論理デバイスは*ロスト*したと見なされる。こうなると非ロスト状態に論理デバイスをリセットすることは不可能であるが、ロスト状態は論理デバイス(`VkDevice`)に特有であり、対応する物理デバイス(`VkPhysicalDevice`)は影響を受けなく**てもよい**。いくつかの場合、物理デバイスもロスト**してもよい**ので、新しい論理デバイスを生成しようとすると、失敗して`VK_ERROR_DEVICE_LOST`を返すだろう。これは通常は基礎にあるハードウェアかそれのホストとの接続による問題を示している。物理デバイスがロストしなかったならば、新しい論理デバイスは物理デバイスから正常に生成され、それは非ロスト状態に**なければならない**。

!!! note
    論理デバイスのロストは回復可能であっ**てもよい**が、物理デバイスのロストの場合、追加の影響を受けない物理デバイスがシステムに存在しない限り、アプリケーションが回復することができるであろう可能性は低い。そのエラーは主に情報提供であり、恐らくそれらのハードウェアに障害が発生したこと、または、物理的に切断されたことのみをユーザーに通知することを意図しており、さらに調査される**必要がある**。多くの場合、物理デバイスのロストはオペレーティングシステムのクラッシュのような他のより深刻な問題を引き起こ**してもよい**。この場合、Vulkan APIを介して報告しなく**てもよい**。

!!! note
    アプリケーションによって引き起こされる未定義の挙動はデバイスをロストさせ**てもよい**。しかし、そのような未定義の挙動はプロセスへ回復不可能な損害を引き起こし**てもよく**、故に、`VkPhysicalDevice`や`VkInstance`を含むAPIオブジェクトが依然として有効であること、または、そのエラーが回復可能であることを保証しない。

デバイスがロストされるとき、その子オブジェクトは暗黙的に破壊されず、それらのハンドルは依然として有効である。これらのオブジェクトは依然としてこれらの親やそのデバイスが破壊**できる**前に破壊される**しなければならない**([オブジェクト寿命](#sec:2.3.1)の節を参照)。[vkMapMemory](#sec:X.X.vkMapMemory)を用いてマップされたデバイスメモリに対応するホストのアドレス空間は依然として有効であり、これらのマップされた領域へのホストメモリアクセスは依然として有効であるが、その内容は未定義である。デバイスと子オブジェクトでAPIコマンドを呼び出すことは依然として合法である。

一度デバイスがロストされれば、コマンド実行は失敗**してもよく**、[VkResult](#)を返すコマンドは`VK_ERROR_DEVICE_LOST`を返し**てもよい**。実行時エラーを許可しないコマンドは有効な使い方では依然として正しく操作し、適用可能ならば、有効なデータを返す。

デバイス実行に対して無期限に待機するコマンド(すなわち、最大の`timeout`を持つ[vkDeviceWaitIdle](#)、[vkQueueWaitIdle](#)、[vkWaitForFences](#)や、`flags`に`VK_QUERY_RESULT_WAIT_BIT`を持つ[vkGetQueryPoolResults](#))はデバイスロストの場合には有限時間で、`VK_SUCCESS`か`VK_ERROR_DEVICE_LOST`のいずれかを返さ**なければならない**。`VK_ERROR_DEVICE_LOST`を返し**てもよい**いずれかのコマンドに対して、コマンドバッファが[保留状態](#)にあるかどうか、または、リソースがデバイスによって使用中であると見なせるかどうかを決定する目的では、返り値`VK_ERROR_DEVICE_LOST`は`VK_SUCCESS`と等価である。

# コマンドバッファ {id="sec:5"}

コマンドバッファは実行のためにデバイスキューに順次サブミット**できる**コマンドを記録するオブジェクトである。コマンドバッファには、セカンダリコマンドバッファを実行**でき**、キューにサブミットされる*プライマリコマンドバッファ*と、プライマリコマンドバッファによって実行され、直接キューにサブミットされない*セカンダリコマンドバッファ*の、2つのレベルがある。

コマンドバッファは`VkCommandBuffer`ハンドルによって表される。

```c
VK_DEFINE_HANDLE(VkCommandBuffer)
```

記録されるコマンドはパイプラインやデスクリプタセットをバインドするコマンド、動的ステートを変更するコマンド、(グラフィクスレンダリングのために)描画するコマンド、(コンピュートのために)ディスパッチするコマンド、(プライマリコマンドバッファでのみ)セカンダリコマンドバッファを実行するコマンド、バッファやイメージをコピーするコマンド、その他のコマンドが含まれる。

<a id="sec:5.reset" />各コマンドバッファは他のコマンドバッファから独立して状態を管理する。プライマリとセカンダリコマンドバッファをまたいだ、または、セカンダリコマンドバッファ間の状態の継承は存在しない。コマンドバッファが記録を始めるとき、そのコマンドバッファ内のすべての状態は未定義である。セカンダリコマンドバッファはプライマリコマンドバッファで実行するために記録されるとき、そのセカンダリコマンドバッファはプライマリコマンドバッファから何も引き継がず、プライマリコマンドバッファのすべての状態はセカンダリコマンドバッファの実行が記録されたのち未定義である。この規定には例外がひとつある --- プライマリコマンドバッファがレンダパスインスタンスの内部にあるならば、そのレンダパスとサブパスの状態は実行しているセカンダリコマンドバッファによって妨げられない。コマンドバッファの状態が未定義であるときにはいつでも、アプリケーションは、描画やディスパッチのようないずれかに依存しているコマンドが記録される前に、そのコマンドバッファ上ですべての関連する状態を設定**しなければならず**、そうしなければ、そのコマンドバッファを実行するときの挙動は未定義である。

明示的な同期がなく、その他に特に指定がない限り、コマンドバッファを介してキューにサブミットされる様々なコマンドは互いに対して任意の順序で、または/および、並行に実行**してもよい**。また、これらのコマンドのメモリの副次的効果は明示的なメモリ依存性を持たずに他のコマンドから直接的に可視でなく**てもよい**。これはコマンドバッファ内、与えられたキューにサブミットされるコマンドバッファ間において真である。コマンド間の[暗黙的](#)および明示的同期に関する情報は[同期の章](#)を参照のこと。

## コマンドバッファのライフサイクル {id="sec:5.1"}

各コマンドバッファは常に以下の状態のひとつにある。

初期[Initial]
: コマンドバッファが[最初に割り当てられる](#)とき、それは*初期状態*にある。いくつかのコマンドはひとつのコマンドバッファまたは一連のコマンドバッファを*リセット*して、実行可能、記録、無効のいずれかの状態からこの状態に戻すことができる。初期状態にあるコマンドバッファは記録状態に移行させること、および、解放されることのみが**できる**。

記録[Recording]
: [vkBeginCommandBuffer](#)はそのコマンドバッファの状態を初期状態から*記録状態*に変更する。一度コマンドバッファが記録状態になれば、`vkCmd*`コマンドがそのコマンドバッファに記録するために使用**できる**。

実行可能[Executable]
: [vkEndCommandBuffer](#)はコマンドバッファの記録を終わらせ、記録状態から*実行可能状態*に移行させる。実行可能なコマンドバッファは[サブミット](#)、リセット、または、[他のコマンドバッファに記録させる](#)ことが**できる**。

保留[Pending]
: コマンドバッファの[キューサブミッション](#)はコマンドバッファの状態を実行可能状態から*保留状態*に変更する。保留状態にある間、アプリケーションはいかなる方法でもコマンドバッファを修正しようと**してはならない**--- デバイスはそれに記録されたコマンドを処理してい**てもよい**。一度コマンドバッファの実行が完了すれば、そのコマンドバッファは実行可能状態に戻る。[同期](#)コマンドはいつこれが発生するかを検出するのに使われる**必要がある**。

無効[Invalid]
: コマンドバッファに記録されたコマンドで使われたリソースの修正や消去のようないくつかの操作はコマンドバッファの状態を*無効状態*に遷移させるだろう。無効状態にあるコマンドバッファは、リセットして*記録状態*に移行させること、または、解放することのみ行うことが**できる**。

コマンドバッファ上で操作するいずれかの与えられたコマンドは、そのコマンドの有効な使い方の制約にて詳しく述べられる、コマンドバッファがい**なければならない**状態に関するそれ自身の要件を持つ。

コマンドバッファのリセットは以前に記録されたコマンドを破棄し、コマンドバッファを初期状態にする操作である。リセットは[vkResetCommandBuffer](#)や[vkResetCommandPool](#)の結果、または、(加えてコマンドバッファを記録状態にする)[vkBeginCommandBuffer](#)の一部として発生する。

[セカンダリコマンドバッファ](#)は[vkCmdExecuteCommands](#)を介してプライマリコマンドバッファに記録**できる**。これは2つのコマンドバッファのライフサイクルを共に部分的に紐付ける --- プライマリがキューにサブミットされるならば、プライマリとそれに記録されたいずれかのセカンダリの両方は保留状態に移行する。一度プライマリの実行が完了すれば、それに記録されるいずれかのセカンダリもそうなり、一度各コマンドバッファのすべての実行が完了すれば、それらは実行可能状態に移行する。セカンダリがいずれか他の状態に移行している間に他のコマンドバッファに記録されるならば、そのプライマリは無効状態に移行する。いずれか他の状態に移行しているプライマリはセカンダリの状態に影響を与えない。プライマリコマンドバッファのリセットおよび解放はそれに記録されたセカンダリコマンドバッファとのつながりを取り除く。

## 5.2 {id="sec:5.2"}

## 5.3 {id="sec:5.3"}

## 5.4 {id="sec:5.4"}

## 5.5 {id="sec:5.5"}

## キューの前進性 {id="sec:5.6"}

アプリケーションはコマンドバッファのサブミッションがいずれのキューに対するアプリケーションによる後続のいかなる操作もなしに完了するであろうことを保証**しなければならない**。`vkQueueSubmit`の呼び出しのあと、キューされたすべてのセマフォ待機ごとに、異なるセマフォ待機によって消費されないであろうそのセマフォの先立つシグナルが存在**しなければならない**。

サブミッション内のコマンドバッファはそのキューにおいて先行するコマンドによってシグナル状態にならないであろうイベントを待つ[vkCmdWaitEvents](#)コマンドを含めることが**できる**。そのようなイベントは[vkSetEvent](#)を用いてアプリケーションによってシグナル状態にされ**なければならず**、それらを待つ`vkCmdWaitEvents`コマンドはレンダパスインスタンスの内部にあっ**てはならない**。実装は、デバイスの他のクライアントの進捗に干渉するのを避けるために、コマンドバッファが待とうとする長さの制限を持っ**てもよい**。イベントがこれらの制限内でシグナル状態にならないならば、結果は未定義であり、デバイスロストを含ん**でもよい**。

## 5.7 {id="sec:5.7"}

# 同期とキャッシュ制御 {id="sec:6"}

リソースへのアクセスの同期はVulkanでは主にアプリケーションの責任である。ホストに関するコマンドやその他のデバイスでのコマンドの実行の順序はいくつかの暗黙の保証があり、明示的に指定される必要がある。メモリキャッシュと他の最適化もまた明示的に管理され、システムを通るデータフローがほぼアプリケーションの制御下にあることを必要とする。

コマンド間にいくつかの暗黙の保証が存在する一方、5つの明示的な同期メカニズムがVulkanにより公開されている。

[フェンス](#)
: フェンスはデバイスでのいくつかのタスクの実行が完了したことをホストと通信するのに使うことが**できる**。

[セマフォ](#)
: セマフォは複数のキューをまたいだリソースを制御するのに使うことが**できる**。

[イベント](#)
: イベントは、ホストかコマンドバッファのいずれかによってシグナル状態に**できる**、および、コマンドバッファ内で待機したりホスト上でクエリしたり**できる**、粒度の細かい同期プリミティブを提供する。

[パイプラインバリア](#)
: パイプラインバリアもまたコマンドバッファ内での同機制御を提供するが、シグナルと待機の操作に分かれておらず、単一の点で行われる。

[レンダパス](#)
: レンダパスは、この章におけるその概念に依存した、ほとんどのレンダリングタスクに対して便利な同期フレームワークを提供する。アプリケーションが他の同期プリミティブを用いる必要があろう多くの場合はレンダパスの一部としてより効率的に表現**できる**。

## 実行およびメモリ依存性　{id="sec:6.1"}

ある*操作*とはホスト、デバイス、表示エンジンのような外部エンティティで実行される任意の作業のかたまりである。同期コマンドはコマンドの2つの*同期スコープ[synchronization scopes]*によって定義される2つの一連の操作の間の明示的な*実行依存性[execution dependencies]*と*メモリ依存性[memory dependencies]*を導入する。

同期スコープは同期コマンドが他のどの操作と実行依存性を生成することができるかを定義する。同期コマンドの同期スコープの中にない操作タイプは結果の依存関係に含まれないだろう。例えば、多くの同期コマンドに対して、その同期スコープは特定の[パイプラインステージ](#)で実行する単なる操作に制限**でき**、他のパイプラインステージが依存関係から除外されることを許可する。他のスコープ化の選択肢は、特定のコマンドに依存することで、可能である。

*実行依存性*は、2つの一連の操作に対して、前者が後者より*前に発生[happens-before]*<!---->**しなければならない**ことの保証である。ある操作がもうひとつの操作より前に発想するならば、先の操作は後の操作が開始される前に完了**しなければならない**。より正確に言えば以下の通りである。

- $A$、$B$をそれぞれ一連の操作とする
- 同期コマンドを$S$とする
- $A_s$、$B_s$を$S$の同期スコープとする
- $A'$を$A$と$A_s$の交差とする
- $B'$を$B$と$B_s$の交差とする
- 実行するために$A$、$S$、$B$の順にサブミットすると、$A'$と$B'$の間の実行依存性$E$となるだろう
- 実行依存性$E$は$A'$が$B'$より前に発生することを保証する

*実行依存性連鎖[execution dependency chain]*は最初の依存性の$A'$と最後の依存性の$B'$との間の順序関係を形作る実行依存性の列である。実行依存性の連続的な組ごとに、最初の依存性にある$B_s$との交差と最後の依存性にある$A_s$との交差が空でないならば、連鎖が存在する。実行依存性連鎖からの単一実行依存性の形成は実行依存性の記述において以下のように置き換えることで説明できる。

- $S$を実行依存性連鎖を生成する一連の同期コマンドとする
- $A_s$を$S$における初めのコマンドの第1同期スコープとする
- $B_s$を$S$における最後のコマンドの第2同期スコープとする

!!! note
    ひとつの実行依存性は本質的に複数の実行依存性でもある --- ふとつの依存性は各サブセット$A'$と$B'$の間に存在し、実行依存性連鎖にも同様に真である。例えば、そのステージマスクに複数の[パイプラインステージ](#)を持つ同期コマンドは各sourceステージと各destinationステージとの間の1つの依存性を効果的に生成する。これは同期コマンドの依存性の全部を伴わない場合にどれだけの実行連鎖が形作られるかを検討する時について考えるのに役立つ可能性がある。同様に、実行依存性連鎖に隣接するいずれかの一連の依存性はそれ自体で実行依存性連鎖と見なすことが**できる**。

実行依存性単体ではある一連の操作における書き込み結果の値がもうひとつの一連の操作から読み込むことが**できる**ことを保証するのに十分ではない。

<a id="sec:6.1.availability" /><a id="sec:6.1.visibility" />追加の2つの操作タイプはメモリアクセスを制御するのに使われる。*可用性操作[availability operations]*は特定のメモリ書き込みアクセスによって生成された値が今後のアクセスで*可用[available]*になるようにする。可用な値は後続の同じメモリ位置への書き込みが(可用になるか否かに関わらず)発生するかメモリが解放されるまで可用なままである。*可視性操作[visibility operations]*は可用な値が特定のメモリアクセスから*可視*になるようにする。

*メモリ依存性*は以下のような可用性および可視性操作を含む実行依存性である。

- 最初の一連の操作は可用性操作より前に発生する
- その可用性操作は可視性操作より前に発生する
- その可視性操作は次の一連の操作より前に発生する

ひとたび書き込まれた値が特定のメモリアクセスタイプから可視になれば、それらはそのメモリアクセスタイプによって読み書き**できる**。Vulkanにおけるほとんどの同期コマンドはメモリ依存性を定義する。

可用および可視にする指定のメモリアクセスはメモリ依存性の*アクセススコープ*によって定義される。メモリ依存性の最初のアクセススコープにあって$A'$で発生する、いずれかのアクセスタイプは可用である。メモリ依存性の次のアクセススコープにあって$B'$で発生する、いずれかのアクセスタイプはそれを可視にするいずれかの可用な書き込みを持つ。同期コマンドのアクセススコープにないいずれかの操作タイプは結果の依存性に含まれないだろう。

メモリ依存性はメモリアクセスの可用性および可視性と2つの一連の操作の間の実行順序を強制する。これを[実行依存性連鎖](#)の説明に加えると以下のようになる。

- $A'$によって処理されるメモリアクセス一式を$a$とする
- $B'$によって処理されるメモリアクセス一式を$b$とする
- $S$における最初のコマンドの最初のアクセススコープを$a_s$とする
- $S$における最後のコマンドの次のアクセススコープを$b_s$とする
- $a$と$a_s$の交差を$a'$とする
- $b$と$b_s$の交差を$b'$とする
- 実行するために$A$、$S$、$B$の順にサブミットすると、$A'$と$B'$の間のメモリ依存性$m$となるだろう
- メモリ依存性$m$は以下を保証する
    - $a'$におけるメモリ書き込みが可用となること
    - $a'$由来を含む可用なメモリ書き込みが$b'$から可視となること

!!! note
    実行およびメモリ依存性はデータハザードを解決する、すなわち、読み書き操作がうまく定義された順序で発生することを保証するのに使われる。write-after-readハザードは単なるひとつの実行依存性で解決できるが、read-after-writeハザードとwrite-after-writeハザードは適切なメモリ依存性がそれらの間に含まれる必要がある。アプリケーションがこれらのハザードを解決するために依存性を含めないならば、メモリアクセスの実行順序やその結果は未定義である。

### イメージレイアウトの遷移 {id="sec:6.1.1"}

イメージサブリソースは、(例えば、[イメージメモリバリア](#)を用いることによる)[メモリ依存性](#)の一部として、ある[レイアウト](#)からもうひとつへ遷移させることが**できる**。レイアウト遷移があるメモリ依存性で指定されるとき、そのメモリ依存性における可用性操作の後に発生、および、可視性操作の前に発生する。イメージレイアウト遷移はイメージサブリソース範囲にバインドされたすべてのメモリで読み書きアクセスを処理**してもよい**ので、アプリケーションはすべてのメモリ書き込みがレイアウト遷移の実行前に[可用](#)となっていることを保証**しなければならない**。可用なメモリは自動的にレイアウト遷移から可視になり、レイアウト遷移によって処理される書き込みは自動的に可用になる。

レイアウト遷移は特定のイメージサブリソース範囲へ常に適用され、古いレイアウトと新しいレイアウトの両方を指定する。古いレイアウトが新しいレイアウトと一致しないならば、遷移が発生する。古いレイアウトはイメージサブリソース範囲の現在のレイアウトと一致**しなければならない**。ここでの例外として、古いレイアウトは`VK_IMAGE_LAYOUT_UNDEFINED`として常に指定**できる**。ただし、これを行うとイメージサブリソース範囲の中身が無効になる。

!!! note
    古いレイアウトを`VK_IMAGE_LAYOUT_UNDEFINED`に設定することはそのイメージサブリソースの中身が保存される必要がないことを暗に示す。実装は高価なデータ遷移操作の処理を回避するためにこの情報を使用**してもよい**。

!!! note
    アプリケーションはレイアウト遷移が古いレイアウトでイメージにアクセスするすべての操作の後に発生し、新しいレイアウトでイメージにアクセスするであろういかなる操作の前に発生することを保証**しなければならない**。レイアウト遷移は潜在的に読み書き操作であるので、これを保証するために適切なメモリ依存性を定義しないことはデータ競合となるだろう。

イメージレイアウト遷移は[メモリエイリアシング](#)と相互作用する。

### パイプラインステージ {id="sec:6.1.2"}

[アクションコマンド](#)によって処理される作業は、*パイプラインステージ*として知られる論理的に独立な実行単位の列によって処理される、複数の操作で構成される。実行されるパイプラインステージは使われる特定のアクションコマンドとアクションコマンドが記録されたときのコマンドバッファの状態に依存する。[描画コマンド](#)、[ディスパッチコマンド](#)、[コピーコマンド](#)、[クリアコマンド](#)のすべては異なる一連の[パイプラインステージ](#)で実行する。

パイプラインステージ間の操作の実行は、特に[パイプラインステージ順序](#)を含む、[暗黙の順序保証](#)を守ら**なければならない**。そうでなければ、パイプラインステージ間の実行は、実行依存性によって矯正されない限り、他のステージに関して重なり合ったり、順序を外れて実行したり**してもよい**。

同期コマンドのいくつかは、コマンドの[同期スコープ](#)をそれらのステージだけに制限するために、パイプラインステージ引数を含む。これは厳密な実行依存性やアクションコマンドによって処理されるアクセスに対する粒度の細かい制御を可能にする。実装は不必要なストールやキャッシュのフラッシュを回避するためにこれらのパイプラインステージを用いる**必要がある**。

パイプラインステージを指定するために設定できるビットは以下の通りである。

```c
typedef enum VkPipelineStageFlagBits {
    VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT = 0x00000001,
    VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT = 0x00000002,
    VK_PIPELINE_STAGE_VERTEX_INPUT_BIT = 0x00000004,
    VK_PIPELINE_STAGE_VERTEX_SHADER_BIT = 0x00000008,
    VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT = 0x00000010,
    VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT = 0x00000020,
    VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT = 0x00000040,
    VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT = 0x00000080,
    VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT = 0x00000100,
    VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT = 0x00000200,
    VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT = 0x00000400,
    VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT = 0x00000800,
    VK_PIPELINE_STAGE_TRANSFER_BIT = 0x00001000,
    VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT = 0x00002000,
    VK_PIPELINE_STAGE_HOST_BIT = 0x00004000,
    VK_PIPELINE_STAGE_ALL_GRAPHICS_BIT = 0x00008000,
    VK_PIPELINE_STAGE_ALL_COMMANDS_BIT = 0x00010000,
} VkPipelineStageFlagBits;
```

- `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`はキューがコマンドをはじめに受け取るパイプラインステージを指定する。
- `VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT`はDraw/DispatchIndirectのデータ構造が消費されるパイプラインステージを指定する。
- `VK_PIPELINE_STAGE_VERTEX_INPUT_BIT`は頂点バッファとインデックスバッファが消費されるパイプラインステージを指定する。
- `VK_PIPELINE_STAGE_VERTEX_SHADER_BIT`は頂点シェーダステージを指定する。
- `VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT`はテッセレーション制御シェーダステージを指定する。
- `VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT`はテッセレーション評価シェーダステージを指定する。
- `VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT`はジオメトリシェーダステージを指定する。
- `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT`はフラグメントシェーダステージを指定する。
- `VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT`は前期フラグメントテスト(フラグメントシェーディング前の深度ステンシルテスト)が処理されるパイプラインステージを指定する。このステージは深度ステンシルフォーマットを持つフレームバッファアタッチメントに対する[サブパスのロード操作](#)も含める。
- `VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT`は後期フラグメントテスト(フラグメントシェーディング後の深度ステンシルテスト)が処理されるパイプラインステージを指定する。このステージは深度ステンシルフォーマットを持つフレームバッファアタッチメントに対する[サブパスのストア操作](#)も含める。
- `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT`は最終色の値がパイプラインから出力されるブレンディング後のパイプラインステージを指定する。このステージは色フォーマットを持つフレームバッファアタッチメントに対する[サブパスのロードおよびストア操作](#)とマルチサンプル解決操作も含める。
- `VK_PIPELINE_STAGE_TRANSFER_BIT`はコピーコマンドの実行を指定する。これはすべての[コピーコマンド](#)、([vkCmdClearAttachments](#)を除く)[クリアコマンド](#)、[vkCmdCopyQueryPoolResults](#)に起因する操作を含める。
- `VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT`はコンピュートシェーダの実行を指定する。
- `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`はすべてのコマンドによって生成された操作が実行を完了するパイプラインの最終ステージを指定する。
- `VK_PIPELINE_STAGE_HOST_BIT`はホスト上でのデバイスメモリの読み書きの実行を示す疑似ステージを指定する。このステージはコマンドバッファに記録されるいずれのコマンドによっても引き起こされない。
- `VK_PIPELINE_STAGE_ALL_GRAPHICS_BIT`はすべてのグラフィクスパイプラインステージの実行を指定し、以下を論理ORしたものと等価である。
    - `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`
    - `VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT`
    - `VK_PIPELINE_STAGE_VERTEX_INPUT_BIT`
    - `VK_PIPELINE_STAGE_VERTEX_SHADER_BIT`
    - `VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT`
    - `VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT`
    - `VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT`
    - `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT`
    - `VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT`
    - `VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT`
    - `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT`
    - `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`
- `VK_PIPELINE_STAGE_ALL_COMMANDS_BIT`は共に使われるキューでサポートされる他のすべてのパイプラインステージフラグを論理ORしたものと等価である。

!!! note
    destinationステージマスクに`VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`のみを持つ実行依存性はそのステージが続いてサブミットされるコマンドを実行しないようにするのみであるだろう。このステージは実際の実行を処理しないので、これは観察できない --- 実際には、それは後続のコマンドの処理を遅らせない。同様に、sourceステージマスクに`VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`のみを持つ実行依存性は先立つコマンドが完了するのを事実上待機しないだろう。
    <!--  -->
    メモリ依存性を定義するとき、`VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`か`VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`のみを用いると、これらのステージはメモリにアクセスしないので、いずれのアクセスも可用および/または可視に一切しないだろう。
    <!--  -->
    `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`と`VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`は、例えば、キュー間のセマフォ操作のような、要求される実行依存性が他の方法で満たされるときに、レイアウト遷移の達成やキューの所有権操作に役立つ。

```c
typedef VkFlags VkPipelineStageFlags;
```

`VkPipelineStageFlags`は0以上の[VkPipelineStageFlagBits](#)のマスクを設定するためのビットマスク型である。

同期コマンドがsourceステージマスクを含むならば、その最初の[同期スコープ](#)はそのマスクに指定されるパイプラインステージの実行のみを含み、その最初の[アクセススコープ](#)はそのマスクに指定されるパイプラインステージで処理されるメモリアクセスのみを含む。同期コマンドがdestinationステージマスクを含むならば、その次の[同期スコープ](#)はそのマスクに指定されるパイプラインステージの実行のみを含み、その次の[アクセススコープ](#)はそのマスクに指定されるパイプラインステージで処理されるメモリアクセスのみを含む。

!!! note
    コマンドの最初の[同期スコープ](#)における特定のパイプラインステージを含むことは同期スコープにおける[論理的に以前の[logically earlier]](#)パイプラインステージを暗黙的に含む。同様に、次の[同期スコープ](#)は[論理的に以後の[logically later]](#)パイプラインステージを含む。
    <!--  -->
    しかし、その[アクセススコープ](#)はこの方法において影響を受けないことに注意する --- 指定された正確なステージのみが各アクセススコープの一部と見なされる。

あるパイプラインステージは特定の操作一式をサポートするキュー上でのみ利用可能である。以下の表はパイプラインステージフラグごとにどのキュー能力フラグがキューによってサポートされ**なければならない**かをリスト化する。複数フラグが表の2列目に列挙されるとき、そのパイプラインステージは、リストアップされた能力フラグのいずれかをサポートするならば、キュー上でサポートされることを意味する。キュー能力の更なる詳細については[物理デバイス列挙](#)と[キュー](#)を参照。

|パイプラインステージフラグ|要求されるキュー能力フラグ|
|-|-|
|`VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`|要求なし|
|`VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT`|`VK_QUEUE_GRAPHICS_BIT`または`VK_QUEUE_COMPUTE_BIT`|
|`VK_PIPELINE_STAGE_VERTEX_INPUT_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_VERTEX_SHADER_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT`|`VK_QUEUE_COMPUTE_BIT`|
|`VK_PIPELINE_STAGE_TRANSFER_BIT`|`VK_QUEUE_GRAPHICS_BIT`または`VK_QUEUE_COMPUTE_BIT`または`VK_QUEUE_TRANSFER_BIT`|
|`VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`|要求なし|
|`VK_PIPELINE_STAGE_HOST_BIT`|要求なし|
|`VK_PIPELINE_STAGE_ALL_GRAPHICS_BIT`|`VK_QUEUE_GRAPHICS_BIT`|
|`VK_PIPELINE_STAGE_ALL_COMMANDS_BIT`|要求なし|

コマンドの結果として実行するパイプラインステージは指定の順序で実行を論理的に完了し、そのような論理的に以後のパイプラインステージの完了は論理的に以前のステージの完了より前に発生**してはならない**。これは、特定の同期コマンドに対してsourceステージマスクに与えられたステージを含むことは、いずれかの論理的に以前のステージがそのコマンドの$A_s$に含まれることも暗示する、ということを意味する。

!!! note
    実装はすべての同期操作に対するすべてのパイプラインステージでの同期をサポートしなく**てもよい**。実装が同期をサポートしないパイプラインステージがsourceステージマスクに現れるならば、それより論理的に以後のいずれかのステージを最初の同期スコープの代わりに用い**てもよい**。実装が同期をサポートしないパイプラインステージがdestinationステージマスクに現れるならば、それより論理的に以前のいずれかのステージを次の同期スコープの代わりに用い**てもよい**。<br>
    例えば、実装が頂点シェーダ実行の完了後に即座にイベントをシグナル状態にできないならば、代わりに色アタッチメント出力の完了後にイベントをシグナル状態に**してもよい**。<br>
    実装がそのような置き換えを行うならば、実行またはメモリ依存性やイメージまたはバッファメモリバリアのセマンティクスに影響を与えないように**しなければならない**。

パイプラインステージの順序は、グラフィクス、コンピュート、トランスファー、ホストといった、特定のパイプラインに依存する。

グラフィクスパイプラインでは、以下のステージがこの順序で発生する。

- `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`
- `VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT`
- `VK_PIPELINE_STAGE_VERTEX_INPUT_BIT`
- `VK_PIPELINE_STAGE_VERTEX_SHADER_BIT`
- `VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT`
- `VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT`
- `VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT`
- `VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT`
- `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT`
- `VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT`
- `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT`
- `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`

コンピュートパイプラインでは、以下のステージがこの順序で発生する。

- `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`
- `VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT`
- `VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT`
- `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`

トランスファーパイプラインでは、以下のステージがこの順序で発生する。

- `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`
- `VK_PIPELINE_STAGE_TRANSFER_BIT`
- `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT`

ホスト操作では、1つのパイプラインステージのみが発生するので、順序は保証されない。

- `VK_PIPELINE_STAGE_HOST_BIT`

### アクセスタイプ {id="sec:6.1.3"}

Vulkanにおけるメモリはシェーダ呼び出しの内部やパイプラインのいくつかの固定機能ステージ経由でアクセス**できる**。*アクセスタイプ*は、用いられる[デスクリプタタイプ](#)の関数、または、固定機能ステージがメモリにアクセスする方法である。各アクセスタイプは[VkAccessFlagBits](#)におけるビットフラグに対応する。

いくつかの同期コマンドはメモリ依存性の[アクセススコープ](#)をていぎする引数としてアクセスタイプの集合を取る。同期コマンドがsourceアクセスマスクを含むならば、その最初の[アクセススコープ](#)はそのマスクに指定されるアクセスタイプを介するアクセスのみを含む。同様に、同期コマンドがdestinationアクセスマスクを含むならば、その次の[アクセススコープ](#)はそのマスクに指定されるアクセスタイプを介するアクセスのみを含む。

アクセスマスクに設定**できる**アクセスタイプは以下を含む。

```c
typedef enum VkAccessFlagBits {
    VK_ACCESS_INDIRECT_COMMAND_READ_BIT = 0x00000001,
    VK_ACCESS_INDEX_READ_BIT = 0x00000002,
    VK_ACCESS_VERTEX_ATTRIBUTE_READ_BIT = 0x00000004,
    VK_ACCESS_UNIFORM_READ_BIT = 0x00000008,
    VK_ACCESS_INPUT_ATTACHMENT_READ_BIT = 0x00000010,
    VK_ACCESS_SHADER_READ_BIT = 0x00000020,
    VK_ACCESS_SHADER_WRITE_BIT = 0x00000040,
    VK_ACCESS_COLOR_ATTACHMENT_READ_BIT = 0x00000080,
    VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT = 0x00000100,
    VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_READ_BIT = 0x00000200,
    VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT = 0x00000400,
    VK_ACCESS_TRANSFER_READ_BIT = 0x00000800,
    VK_ACCESS_TRANSFER_WRITE_BIT = 0x00001000,
    VK_ACCESS_HOST_READ_BIT = 0x00002000,
    VK_ACCESS_HOST_WRITE_BIT = 0x00004000,
    VK_ACCESS_MEMORY_READ_BIT = 0x00008000,
    VK_ACCESS_MEMORY_WRITE_BIT = 0x00010000,
} VkAccessFlagBits;
```

- `VK_ACCESS_INDIRECT_COMMAND_READ_BIT`はindirect描画およびディスパッチコマンドの一部として読み込まれたindirectコマンド構造への読み込みアクセスを指定する。
- `VK_ACCESS_INDEX_READ_BIT`は[vkCmdBindIndexBuffer](#)によってバインドされた、インデックス付き描画コマンドの一部としてのインデックスバッファへの読み込みアクセスを指定する。
- `VK_ACCESS_VERTEX_ATTRIBUTE_READ_BIT`は[vkCmdBindVertexBuffers](#)によってバインドされた、描画コマンドの一部としての頂点バッファへの読み込みアクセスを指定する。
- `VK_ACCESS_UNIFORM_READ_BIT`は[ユニフォームバッファ](#)への読み込みアクセスを指定する。
- `VK_ACCESS_INPUT_ATTACHMENT_READ_BIT`はフラグメントシェーダ中のレンダパス内部の[入力アタッチメント](#)への読み込みアクセスを指定する。
- `VK_ACCESS_SHADER_READ_BIT`は[ストレージバッファ](#)、[ユニフォームテクセルバッファ](#)、[ストレージテクセルバッファ](#)、[Sampledイメージ](#)、[ストレージイメージ](#)への読み込みアクセスを指定する。
- `VK_ACCESS_SHADER_WRITE_BIT`は[ストレージバッファ](#)、[ストレージテクセルバッファ](#)、[ストレージイメージ](#)への書き込みアクセスを指定する。
- `VK_ACCESS_COLOR_ATTACHMENT_READ_BIT`は、[ブレンディング](#)、[論理操作](#)、ある[サブパスロード操作](#)を介するような、[カラーアタッチメント](#)への読み込みアクセスを指定する。
- `VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT`は、[ブレンディング](#)、[論理操作](#)、ある[サブパスロードおよびストア操作](#)を介するような、[カラーアタッチメント](#)への書き込みアクセスを指定する。
- `VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_READ_BIT`は、[深度またはステンシル操作](#)、ある[サブパスロード操作](#)を介するような、[深度/ステンシルアタッチメント](#)への読み込みアクセスを指定する。
- `VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT`は、[深度またはステンシル操作](#)、ある[サブパスロードおよびストア操作](#)を介するような、[深度/ステンシルアタッチメント](#)への書き込みアクセスを指定する。
- `VK_ACCESS_TRANSFER_READ_BIT`は[コピー](#)操作におけるイメージやバッファへの読み込みアクセスを指定する。
- `VK_ACCESS_TRANSFER_WRITE_BIT`は[コピー](#)や[クリア](#)操作におけるイメージやバッファへの書き込みアクセスを指定する。
- `VK_ACCESS_HOST_READ_BIT`はホスト操作による読み込みアクセスを指定する。このタイプのアクセスはリソースを通さず、メモリで直接処理される。
- `VK_ACCESS_HOST_WRITE_BIT`はホスト操作による書き込みアクセスを指定する。このタイプのアクセスはリソースを通さず、メモリで直接処理される。
- `VK_ACCESS_MEMORY_READ_BIT`は固有でないエンティティを介した読み込みアクセスを指定する。これらのエンティティはVulkanデバイスおよびホストを含むが、Vulkanデバイスの外部にある、または、コアVulkanパイプラインの一部ではないエンティティを含ん**でもよい**。destinationアクセスマスクに含まれるとき、すべての可用な書き込みをVulkanデバイスに知られているエンティティでの今後すべての読み込みアクセスから可視にする。
- `VK_ACCESS_MEMORY_WRITE_BIT`は固有でないエンティティを介した書き込みアクセスを指定する。これらのエンティティはVulkanデバイスおよびホストを含むが、Vulkanデバイスの外部にある、または、コアVulkanパイプラインの一部ではないエンティティを含ん**でもよい**。sourceアクセスマスクに含まれるとき、Vulkanデバイスに知られているエンティティによって処理されるすべての書き込みは可用になる。destinationアクセスマスクに含まれるとき、すべての可用な書き込みをVulkanデバイスに知られているエンティティでの今後すべての書き込みアクセスから可視にする。

あるアクセスタイプはパイプラインステージのサブセットによってのみ処理される。ステージマスクとアクセスマスクの両方を取るいずれかの同期コマンドは[アクセススコープ](#)を定義するために両方を使う --- 指定のステージによって処理される指定のアクセスタイプのみがアクセススコープに含まれる。アプリケーションは、そのタイプのアクセスを処理できる対応するステージマスクにパイプラインステージを含まないならば、同期コマンドにアクセスフラグを指定**してはならない**。以下の表はアクセスフラグごとにどのパイプラインステージがそのアクセスタイプを処理**できる**かをリスト化する。

|アクセスフラグ|サポートするパイプラインステージ|
|-|-|
|VK_ACCESS_INDIRECT_COMMAND_READ_BIT|VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT|
|VK_ACCESS_INDEX_READ_BIT|VK_PIPELINE_STAGE_VERTEX_INPUT_BIT|
|VK_ACCESS_VERTEX_ATTRIBUTE_READ_BIT|VK_PIPELINE_STAGE_VERTEX_INPUT_BIT|
|VK_ACCESS_UNIFORM_READ_BIT|VK_PIPELINE_STAGE_VERTEX_SHADER_BIT、VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT、VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT、VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT、VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT、または、VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT|
|VK_ACCESS_INPUT_ATTACHMENT_READ_BIT|VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT|
|VK_ACCESS_SHADER_READ_BIT|VK_PIPELINE_STAGE_VERTEX_SHADER_BIT、VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT、VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT、VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT、VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT、または、VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT|
|VK_ACCESS_SHADER_WRITE_BIT|VK_PIPELINE_STAGE_VERTEX_SHADER_BIT、VK_PIPELINE_STAGE_TESSELLATION_CONTROL_SHADER_BIT、VK_PIPELINE_STAGE_TESSELLATION_EVALUATION_SHADER_BIT、VK_PIPELINE_STAGE_GEOMETRY_SHADER_BIT、VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT、または、VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT|
|VK_ACCESS_COLOR_ATTACHMENT_READ_BIT|VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT|
|VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT|VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT|
|VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_READ_BIT|VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT、または、VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT|
|VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT|VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT、または、VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT|
|VK_ACCESS_TRANSFER_READ_BIT|VK_PIPELINE_STAGE_TRANSFER_BIT|
|VK_ACCESS_TRANSFER_WRITE_BIT|VK_PIPELINE_STAGE_TRANSFER_BIT|
|VK_ACCESS_HOST_READ_BIT|VK_PIPELINE_STAGE_HOST_BIT|
|VK_ACCESS_HOST_WRITE_BIT|VK_PIPELINE_STAGE_HOST_BIT|
|VK_ACCESS_MEMORY_READ_BIT|該当なし|
|VK_ACCESS_MEMORY_WRITE_BIT|該当なし|

メモリオブジェクトが`VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`特性を持たないならば、ホストからメモリオブジェクトへの書き込みが`VK_ACCESS_HOST_WRITE_BIT`[アクセスタイプ](#)から可視となることを保証するために、[vkFlushMappedMemoryRanges](#)を呼び出さ**なければならない**。ここでは、[同期コマンド](#)によってデバイスに対してさらに可用にすることが**できる**。同様に、`VK_ACCESS_HOST_READ_BIT`[アクセスタイプ](#)から可視である書き込みがホスト操作から可視となることを保証するため、[vkInvalidateMappedMemoryRanges](#)を呼び出さ**なければならない**。

メモリオブジェクトが`VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`特性フラグを持つならば、星ストからメモリオブジェクトへの書き込みは自動的に`VK_ACCESS_HOST_WRITE_BIT`[アクセスタイプ](#)から可視になる。同様に、`VK_ACCESS_HOST_READ_BIT`[アクセスタイプ](#)から可視となった書き込みは自動的にホストから可視となる。

!!! note
    [vkQueueSubmit](#)コマンドは、コマンドが実行される前にフラッシュされたならば、[`VK_ACCESS_HOST_WRITE_BIT`にフラッシュしたホスト書き込みが可用となることを自動的に保証する]ので、ほとんどの場合、このようなケースに対して明示的なメモリバリアは必要とされない。サブミットがホスト書き込みとデバイス読み込みアクセスの間に発生しないようないくつかの状況では、書き込みは明示的なメモリバリアを用いることによって可用となることが**できる**。

```c
typedef VkFlags VkAccessFlags;
```

`VkAccessFlags`は0個以上の[VkAccessFlagBits](#)のマスクを設定するためのビットマスク型である。

# フレームバッファ領域の依存性 {id="sec:6.1.4"}

フレームバッファで操作する、または、それに関連する[パイプラインステージ](#)はまとめて*フレーバッファスペース*パイプラインステージである。これらのステージは以下の通り。

- `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT`
- `VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT`
- `VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT`
- `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT`

これらのパイプラインステージでは、最初の操作一式から次の操作一式への実行およびメモリ依存性は単一の*フレーバッファグローバル*な依存性とするか、複数の*フレーバッファローカル*な依存性に分けるかのいずれかを行うことが**できる**。フレーバッファスペースでないパイプラインステージを持つ依存性はフレーバッファグローバルでもフレームバッファローカルでもない。

*フレームバッファ領域*はフレームバッファ全体のサブセットである一連のサンプル(X、Y、レイヤー、サンプル)座標である。

フレームバッファローカルな依存性の両方の[同期スコープ](#)は(以下で定義されるような)対応するフレームバッファ領域内で処理される操作のみを含む。順序保証はフレームバッファローカルな依存性に対する異なるフレームバッファ領域の間に作られない。

フレームバッファグローバルな依存性の両方の[同期スコープ](#)はすべてのフレームバッファ領域での操作を含む。

最初の同期スコープがN個のサンプルを持つピクセル/フラグメントでの操作を含み、次の同期スコープがM個のサンプルを持つピクセル/フラグメントでの操作を含むならば、NはMと等しくないとすると、最初の同期スコープにおける与えられた(X、Y、レイヤー)座標でのすべてのサンプルを含むフレームバッファ領域は次の同期スコープにおける同じ座標でのすべてのサンプルを含む領域に対応する。言い換えれば、これはピクセル粒度の依存性である。NがMと等しいならば、最初の同期スコープにおける単一の(X、Y、レイヤー、サンプル)座標を含むフレームバッファ領域は次の同期スコープにおける同じ座標での同じサンプルを含む領域に対応する。言い換えれば、これはサンプル粒度の依存性である。

!!! note
    フラグメント呼び出しは特定のグルーピングにおいて動作することを指定されていないので、フレームバッファ領域のサイズは実装依存であり、アプリケーションに知らされず、上記で指定されたものより大きくないことを前提と**しなければならない**。

!!! note
    実質的に、ピクセル対サンプル粒度依存性は、入力アタッチメントがパイプラインの`rasterizationSamples`と異なるサンプル数を持つならば、フラグメントはフレームバッファローカル依存性のみを用いる場合でも入力アタッチメントのピクセルにおけるいかなるサンプルにもアクセス**できる**ことを意味する。入力アタッチメントが同じサンプル数を持つならば、フラグメントはその入力の`SampleMask`でカバーされたサンプルのみにアクセス**できる**(つまり、フラグメント操作はフラグメントがカバーするサンプルごとにフレームバッファローカル依存性の後に発生する)。カバーされていないサンプルにアクセスするには、フレームバッファグローバル依存性が要求される。

同期コマンドが`dependencyFlags`引数を含み、`VK_DEPENDENCY_BY_REGION_BIT`フラグを指定するならば、それは、すべてのフレームバッファ領域に対して、その同期コマンドにおけるフレームバッファスペースパイプラインステージに対するフレームバッファローカル依存性を定義する。`dependecyFlags`引数が何も含まない、または、`VK_DEPENDENCY_BY_REGION_BIT`フラグが指定されていないならば、フレームバッファグローバル依存性はこれらのステージに対して指定される。`VK_DEPENDENCY_BY_REGION_BIT`フラグはフレームバッファスペースでないパイプラインステージの間の依存性にも、フレームバッファスペースと非フレーバッファスペースのパイプラインステージの間の依存性にも影響を与えない。

!!! note
    フレームバッファローカル依存性はほとんどのアーキテクチャに対してより最適である。特にタイルベースのアーキテクチャはフレームバッファ領域を完全にオンチップレジスタに保持することができ、そのような依存性にまたがる外部帯域幅を回避することができる。レンダリングにフレームバッファグローバル依存性を含むことは通常ではすべての実装にメモリや高レベルキャッシュへデータをフラッシュすることを強制するので、局所性最適化の可能性を破壊するだろう。

## 暗黙の同期保証 {id="sec:6.2"}

暗黙の順序保証のいくつかはVulkanによって提供され、コマンドのサブミットされる順序が意味を持ち、一般的の操作における不必要な複雑さを回避することを保証する。

<a id="sec:6.2.submission_order" />*サブミッション順序*はVulkanにおける基礎的順序であり、[アクションおよび同期コマンド](#)が記録され単一のキューにサブミットされる順序に意味を与える。Vulkanにおけるコマンド間の明示的および暗黙的な順序保証はすべてこの順序が意味を持つことを前提に正常に動作する。

与えられたコマンド一式に対するサブミッション順序はそれらがコマンドバッファに記録され、サブミットされた順序に依存する。この順序は以下の通りに決定される。

1. はじめの順序は、最初から最後へ、単一のキューに対して、[vkQueueSubmit](#)コマンドがホストで実行される順序によって決定される。
2. インデックスの小さい方から大きい方へ、[VkSubmitInfo](#)構造体が[vkQueueSubmit](#)の`pSubmits`引数で指定される順序。
3. インデックスの小さい方から大きい方へ、コマンドバッファが[VkSubmitInfo](#)の`pCommandBuffers`メンバで指定される順序。
4. 最初から最後へ、コマンドがホストでコマンドバッファに記録される順序。
    - レンダパスの外側で記録されるコマンドに対して、これは、[vkCmdBeginRenderPass](#)と[vkCmdEndRenderPass](#)コマンドを含む、レンダパスの外側で記録される他のすべてのコマンドを含み、レンダパスの内側のコマンドを直接的には含まない。
    - レンダパスの内側で記録されるコマンドに対して、これは、同じレンダパスインスタンスを区切る[vkCmdBeginRenderPass](#)と[vkCmdEndRenderPass](#)コマンドを含む、同じサブパスの内側で記録される他のすべてのコマンドを含み、他のサブパスに記録されるコマンドを含まない。

コマンドバッファに記録される[アクションおよび同期コマンド](#)は[サブミッション順序](#)で`VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT`パイプラインステージを実行する --- 各コマンドにおいてこのステージ間の暗黙の実行依存性を形成する。

[ステートコマンド](#)はデバイスでいかなる操作も実行しない代わりに、それらはホストで実行するときに記録された順序でコマンドバッファのステートを設定する。[アクションコマンド](#)はそれらがきろくされるときにコマンドバッファの現在のステートを消費し、記録されたステートを一致させるために要求通りにデバイスでステート変更を実行するだろう。

[クエリコマンド](#)、[グラフィクスパイプラインを通過するプリミティブの順序](#)、[イメージメモリバリアの一部としてのイメージレイアウト遷移](#)はサブミッション順序に基づく追加の保証を提供する。

与えられたコマンド内の[パイプラインステージ](#)の実行もまた、単一コマンドにのみ依存する、緩い順序付けを持つ。

## 6.3 {id="sec:6.3"}

# 7 {id="sec:7"}

*レンダパス[render pass]*はアタッチメント、サブパス、サブパス間の依存性の集まりを表現し、アタッチメントがサブパスのコース上で使われる方法を述べる。コマンドバッファにおけるレンダパスの使用を*レンダパスインスタンス*という。

レンダパスは`VkRenderPass`ハンドルによって表される。

```c
VK_DEFINE_NON_DISPATCHABLE_HANDLE(VkRenderPass)
```

*アタッチメントデスクリプション[attachment description]*そのフォーマット、サンプル数、その中身がレンダパスインスタンスごとの開始と終了で扱われる方法を含むアタッチメントの特性を述べる。

*サブパス[subpass]*はレンダパスにおけるアタッチメントのサブセットを読み書きするレンダリングの段階を表す。レンダリングコマンドはレンダパスインスタンスの特定のサブパスに記録される。

*サブパスデスクリプション[subpass description]*サブパスの実行に組み込まれるアタッチメントのサブセットを述べる。各サブパスは*入力アタッチメント[input attachments]*としていくつかのアタッチメントから読み込むこと、*色アタッチメント[color attachments]*または*深度/ステンシルアタッチメント[depth/stencil attachments]*としていくつかへ書き込むこと、*解決アタッチメント[resolve attachments]*へ*マルチサンプル解決操作[multisample resolve operations]*を処理することが**できる**。サブパスデスクリプションはそのサブパスでは読み書きされないがサブパスを通して保存され**なければならない**アタッチメントである*保存アタッチメント[preserve attachments]*を含むことも**できる**。

サブパスはアタッチメントがそのサブパスに対して色、深度/ステンシル、解決、または、入力アタッチメントがある場合に*アタッチメントを用いる*(それぞれ[VkSubpassDescription](#)の`pColorAttachments`、`pDepthStencilAttachment`、`pResolveAttachments`、`pInputAttachments`メンバーに定められる)。サブパスはそのアタッチメントがサブパスによって保存される場合にアタッチメントを使わない。*アタッチメントの最初の使用*はそのアタッチメントを使う最小番号のサブパスにおいてである。同様に、*アタッチメントの最後の使用*はそのアタッチメントを使う最大番号のサブパスにおいてである。

レンダパスにあるサブパスすべては同じ大きさにレンダリングし、1つのサブパスにおけるピクセル(x, y, layer)のフラグメントは同じ(x, y, layer)の場所に以前のサブパスによって書き込まれたアタッチメントの内容のみを読み込むことが**できる**。

!!! note
    前もってサブパスの完全なセットを記述することで、レンダパスはサブパス間のアタッチメントデータの容量や転送を最適化する機会を実装に提供する。
    <!--  -->
    実践では、これは単純なフレームバッファスペース依存性を持つサブパスが単一のタイル化されたレンダリングパスにマージされ**てもよい**ので、レンダパスインスタンスの存続期間中にアタッチメントデータをオンチップに維持**してもよい**。しかし、レンダパスにとって単一のサブパスのみを含むことも極めて一般的である。

*サブパス依存性[subpass dependencies]*はサブパス間の[実行およびメモリ依存性](#)を述べる。

*サブパス依存性連鎖[subpass dependency chain]*はレンダパスにおけるサブパス依存性の列である。ここでは、各サブパス依存性の依存元サブパスはその前の依存性の依存先サブパスに等しい。

サブパスの実行は、実行依存性によって強制されない限り、他のサブパスに関して重なり合ったり順序を外れて実行**してもよい**。各サブパスは同じサブパスに記録されたコマンドやレンダパスを区切る[vkCmdBeginRenderPass](#)と[vkCmdEndRenderPass](#)コマンドの[サブミッション順序](#)のみを尊重し、他のサブパス内のコマンドは含まれない。これは他のほとんどの[暗黙の順序保証](#)に影響を与える。

レンダパスはアタッチメントに固有のイメージビューから独立してサブパスとアタッチメントの構造を述べる。アタッチメントで用いられるであろう固有のイメージビューとその大きさは`VkFramebuffer`オブジェクトに指定される。フレームバッファはそれと互換性のある固有のレンダパスに関して生成される([レンダパス互換性](#)を参照)。まとめて、れんだぱすとフレームバッファは1つ以上のサブパスに対する完全なレンダターゲットステートとサブパス間のアルゴリズム的な依存関係を定義する。

与えられたサブパスのための描画コマンドの様々なパイプラインステージは、[パイプライン順序](#)を尊重しつつ、描画コマンドの内[within]と外[across]の両方で、並行して、および/または、順序を外れて実行**してもよい**。しかし、与えられた(x, y, layer, sample)の場所に対して、あるサンプル毎の操作は[ラスタライゼーション順序](#)で処理される。

# レンダパス生成{id="sec:7.1"}

TODO

# 8 {id="sec:8"}

# 9 {id="sec:9"}

# 10 {id="sec:10"}

## 10.1 {id="sec:10.1"}

## デバイスメモリ {id="sec:10.2"}

*デバイスメモリ*は、例えばデバイスが自然に使用**できる**イメージやバッファオブジェクトの内容のような、デバイスから可視であるメモリである。

物理デバイスのメモリ特性は利用可能なメモリヒープとメモリタイプを述べる。

TODO: `vkGetPhysiclDeviceMemoryProperties`

TODO: `VkPhysicalDeviceMemoryProperties`

構造体`VkPhysicalDeviceMemoryProperties`はこれらのヒープで割り当てられたメモリにアクセスするのに使われることが**できる**その*メモリヒープ*の数と*メモリタイプ*の数を述べる。各ヒープは特定サイズのメモリリソースを述べ、各メモリタイプは与えられたヒープで用いられることが**できる**(例えばホストキャッシュの有無のような)一連のメモリ特性を述べる。特定のメモリタイプを用いるアロケータはメモリタイプのヒープインデックスによって示されるヒープからリソースを消費するだろう。1つ以上のメモリタイプは各ヒープを共有**してもよく**、そのヒープとメモリタイプは、メモリが多種多様な異なる特性で用いられることを可能にしつつ、物理メモリリソースの正確なサイズを知らせるメカニズムを提供する。

メモリヒープ数は`memoryHeapCount`によって与えられ、`VK_MAX_MEMORY_HEAPS`以下である。各ヒープは配列`memoryHeaps`の[VkMemoryHeap](#)型の要素によって述べられる。メモリヒープ全体にわたって利用可能なメモリタイプ数は`memoryTypeCount`によって与えられ、`VK_MAX_MEMORY_TYPES`以下である。各メモリタイプは配列`memoryTypes`の[VkMemoryType]型の要素によって述べられる。

少なくとも1つのヒープは[VkMemoryHeap](#)::`flags`に`VK_MEMORY_HEAP_DEVICE_LOCAL_BIT`を含め**なければならない**。そのすべてが似たようなパフォーマンス的な特徴を持つ複数のヒープがあるならば、これらはすべて`VK_MEMORY_HEAP_DEVICE_LOCAL_BIT`を含め**てもよい**。ユニファイドメモリアーキテクチャ(UMA)システムでは、しばしばホストともデバイスとも同等に"ローカル"であると見なされる単一のメモリヒープのみが存在し、そのような実装はそのヒープをデバイスローカルとして知らせ**なければならない**。

[vkGetPhysiclDeviceMemoryProperties](#)によって返される各メモリタイプは以下の値の内のひとつとなる`propertyFlags`を持た**なければならない**。

- `0`
- `VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`
- `VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_CACHED_BIT`
- `VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_CACHED_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`
- `VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT`
- `VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT | VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`
- `VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT | VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_CACHED_BIT`
- `VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT | VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_CACHED_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`
- `VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT | VK_MEMORY_PROPERTY_LAZILY_ALLOCATED_BIT`

少なくとも1つのメモリタイプは`propertyFlags`に`VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT`と`VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`の両方が設定されてい**なければならない**。少なくとも1つのメモリタイプは`propertyFlags`に`VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT`が設定されてい**なければならない**。

`memoryTypes`に返される要素**X**と**Y**のペアごとに、**X**は以下であるならば**Y**より小さいインデックスの場所に配置され**なければならない**。

- **X**のメンバー`propertyFlags`に返される一連のビットフラグのいずれかは**Y**のメンバー`propertyFlags`に返される一連のビットフラグの厳格なサブセットである。
- または、**X**と**Y**のメンバー`propertyFlags`は等しく、**X**はより良いパフォーマンスを持つメモリヒープに属している(何を良しとするかは実装固有の方法で決定される)。

!!! note
    メンバー`propertyFlags`がサブセット関係にない場合、要素**X**と**Y**の間に順序要件は存在しない。これは潜在的に、1つ以上の方法で同じ一連のメモリタイプに並び替えられるようにする。[すべての許可されたメモリ特性フラグの組み合わせのリスト](#)は要求された順序で書かれることに注目する。しかし、代わりに`VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT`が`VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT`の前にあったならば、そのリストは依然として要求された順序であるだろう。

この順序要件はアプリケーションが単純な探索ループを用いて望んだメモリタイプを選択することを可能にする。

```cpp
// すべての`requiredProperties`を含む`memoryTypeBitsRequirement`にあるメモリを見つける
int32_t findProperties(const VkPhysicalDeviceMemoryProperties* MemoryProperties, uint32_t memoryTypeBitsRequirement, VkMemoryPropertyFlags requiredProperties) {
    const uint32_t memoryCount = pMemoryProperties->memoryTypeCount;
    for (uint32_t memoryIndex = 0; memoryIndex < memoryCount; ++memoryIndex) {
    const uint32_t memoryTypeBits = (1 << memoryIndex);
    const bool isRequiredMemoryType = memoryTypeBitsRequirement & memoryTypeBits;
    const VkMemoryPropertyFlags properties = pMemoryProperties->memoryTypes[memoryIndex].propertyFlags;
    const bool hasRequiredProperties = (properties & requiredProperties) == requiredProperties;
        if (isRequiredMemoryType && hasRequiredProperties) {
            return static_cast<int32_t>(memoryIndex);
        }
    }
    // メモリタイプが見つからなかった
    return -1;
}

// 最適なメモリタイプを見つけようとする、または、存在しなければメモリタイプをフォールバックしてみる
// `device`はデバイス
// `image`はメモリをバインドする必要があるイメージ
// `memoryProperties`はvkGetPhysicalDeviceMemoryPropertiesで取得した値
// `requiredProperties`は示していなければならない特性フラグ
// `optimalProperties`はアプリケーションに好ましい特性フラグ
VkMemoryRequirements memoryRequirements;
vkGetImageMemoryRequirements(device, image, &memoryRequirements);
int32_t memoryType = findProperties(&memoryProperties, memoryRequirements.memoryTypeBits, optimalProperties);
if (memoryType == -1) {
    // 見つからないので、フォールバックしてみる
    memoryType = findProperties(&memoryProperties, memoryRequirements.memoryTypeBits, requiredProperties);
}
```

TODO: `VkMemoryHeap`

TODO: `VkMemoryType`

TODO: `vkAllocateMemory`

TODO: `vkFreeMemory`

解放される時点でメモリオブジェクトがマップされているならば、暗黙的にアンマップされる。

### デバイスメモリオブジェクトへのホストアクセス {id="sec:10.2.1"}

TODO: `vkMapMemory`

`vkMapMemory`はホストアクセス可能なポインタを返す前にデバイスメモリが現時点で使用中であるかどうかを確認しない。アプリケーションは、以前にサブミットされたこの範囲に書き込むコマンドがホストのその範囲への読み書きの前に完了していること、および、以前にサブミットされたその範囲から読み込むコマンドがホストのその領域への書き込みの前に完了していることを保証**しなければならない**(そのような保証を満たすことについての詳細は[ここ](#)を参照)。デバイスメモリが…

TODO: `vkUnmapMemory`

# 11 {id="sec:11"}
