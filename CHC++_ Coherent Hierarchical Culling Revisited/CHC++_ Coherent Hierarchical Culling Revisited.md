---
title: >
CHC++: Coherent Hierarchical Culling Revisited [@Mattausch2008]
#bibliography: bibliography.bib
---
# はじめに {#sec:1}

オクルージョンクエリは複合的なシーンをレンダリングするのにかかる時間を減らすために重要な技術である。いわゆるハードウェアオクルージョンクエリの可用性は可視性の実行時測定[runtime determination]を魅力的にしてきた。ハードウェアオクルージョンクエリはグラフィクスハードウェアが単純なプロキシオブジェクトの可視性状態を素早く報告できるメカニズムである。しかしながら、ハードウェアオクルージョンクエリが使用できるようになるのは、例えばCoherent Hierarchical Culling(CHC)アルゴリズム[@BWPP04]にあるような、時間的一貫性[temporal coherence]を活用することによってのみである。なぜなら、これはCPUをストールしたりGPUを飢餓状態[starvation]にしたりすること回避するためである。

CHCアルゴリズムは密に遮蔽された[densely occluded]シーンで上手く機能するが、ハードウェアオクルージョンクエリのオーバーヘッドはいくつかの状況において単純な視錐台カリング(VFC)にさえ後れを取る[fall behind]ようにしてしまう。これは、オクルージョンの賢い統計モデルとハードウェア較正[calibration]ステップに基づいてクエリ数を減少させるNear Optimal Hierarchical Culling(NOHC)と呼ばれるアルゴリズムを提供する、Guthe et al.[@GBK06]によって認知された。しかしながら、Guthe et al.によって定義された最適条件でさえ更なる単純化の源を活用することによって依然として改善できる。

![視錐台カリング(VFC)、Coherent Hierarchical Culling (CHC)、Near Optimal Hierarchical Culling (NOHC)、我々の新しいアルゴリズム(CHC++)に対する発電所モデルのウォークスルーのフレーム時間の比較。](){#fig:1}

本論において、我々は以前のオンラインのオクルージョンカリング手法に大幅な改良を加える手法であるCHC++を提案する([@fig:1]を参照)。そのアルゴリズムの核は単純なままであり、較正をまったく必要とせず、ゲームエンジンへと簡単に統合することができる。その手法の主要な貢献は以下である。

- **状態変化[state changes]の削減**。その重要性に関わらず、状態変化の削減は以前のオクルージョンカリング手法によって明示的に扱われなかった。我々の手法はクエリのバッチ処理を用いることによって状態変化の数を最小化するための強力なメカニズムを提供する。結果として、状態変化の総数は1桁以上[more than an order of magnitude]削減される。
- **クエリ数の削減**。クエリ数の削減はハードウェアベースのオクルージョンカリングにおける以前の研究の主要な目標であった。例えば、Guthe et al.[@GBK06]によって提案されるNOHCアルゴリズムは低い遮蔽率を持つビューに対するクエリ数の削減では非常に成功している。我々はクエリ数の更なる削減のための2つの新しい手法を提案する。1つ目の手法は単一のクエリによって階層において多くのノードの可視性を解決し、2つ目の手法はOriented Bounding Boxやk-DOPのような補助データ構造を必要としないクエリに対するよりタイトな境界ボリュームを活用する。結果として、我々はGuthe et al.[@GBK06]によって定義される"最適"なアルゴリズムより大幅に少ないクエリ数を達成する([@fig:2]を参照)。
- **CPUストールの削減**。CHCアルゴリズムはCPUストールの削減では良い仕事を行っているが、あるシナリオにおいて、ストールは依然として発生し、パフォーマンス上のペナルティを引き起こす。我々は待ち時間のさらなる削減をもたらし、それと同時に状態変化の削減に対する我々の手法と上手く統合する単純な修正を提案する。
- **レンダリングされるジオメトリの削減**。よりタイトな境界ボリュームは境界ボリュームによって引き起こされる可視性の過剰推定を削減し、それ故に、可視として分類されるジオメトリ量を削減するだろう。
- **ゲームエンジンとの統合**。ほとんどのゲームエンジンはマテリアルによってソートされている高度に最適化されたレンダリングループを組み入れ[incorporate]、シェーダはレンダリングステートの変更を最小化する順番で処理される。我々の手法はレンダリングエンジンがレンダキューに格納されるプリミティブのバッチ上でそのようなソートを処理できるようにする。加えて、提案される技術はエンジン呼び出し数を大幅に削減する。

![左上:街のシーンにおけるサンプル視点。右上:CHCアルゴリズムによって必要とされる状態変化(状態変化の数 = 階層ノードの異なる色の数)。左下:CHC++アルゴリズムによって必要とされる状態変化。右下:マルチクエリ:すべての可視ノードがたった2つのオクルージョンクエリによって網羅される(異なる色で示される)。](){#fig:2}

# 関連研究 {#sec:2}

こんにちの急速に進化するグラフィクスハードウェアでしばしば発生するCPUに制約された[CPU-limited]アプリケーションでさえ、可視性カリングはグラフィクスドライバやレンダリングAPIにかかる時間を大幅に削減でき、グラフィクスハードウェアをより良く使うことができる。可視性カリングの一般的な概要について、Cohen-Or et al.[@COCSD02]やBittner and Wonka[@BW03]の精緻な調査[thorough surveys]を参照ください。

可視性アルゴリズムは事前処理の工程として処理するものと実行時に処理するものに大まかに分類することができる。事前処理のアルゴリズムは実行時のオーバーヘッドを一切持たないが、その一方で、しばしば実装し辛く、静的なシーンに対してのみ正常に動作する。一方のオンラインのオクルージョンカリングは長々しい事前処理の工程に頼らず、ひとつの点からの可視性を計算するためにより正確となる可能性があり、また、完全に動的なシーンに対して行うことが可能となる。ほとんどのオンラインのカリングアルゴリズムはイメージ空間で動作するため、ラスタライゼーションを用いて地頭的なオクルーダーの融合を可能にする。専用のハードウェアサポートが存在する前は、オンラインのオクルージョンカリングは、Zhang et al.によるHierarchical Occlusion Maps[@ZMHH97]やAila et al.によるdPVSシステム[@AM04]のようないくつかの有名な例外を除いて、実践で用いるにはコストが高すぎるとおおよそ考えられていた。

ハードウェアで高速化された[hardware accelerated]オクルージョンクエリの導入に伴い、オンラインのオクルージョンカリングは多くの人気を獲得した。ハードウェアオクルージョンクエリはフレームバッファを読み戻す必要なしにプロキシジオメトリの可視ピクセル数を返す比較的に軽量な命令である。これらは多種多様なアルゴリズム[@KS01; @HSLM02; @GSYM03; @SBS04; @KS05]に対する分野を開いたが、そのクエリはコストを伴うため、愚直な[naive]実装ではクエリが戻るのを待つことによって引き起こされるGPUおよびCPUのアイドル時間により非常に遅くなる可能性がある。

Coherent Hierarchical Culling[@BWPP04]や後のNear Optimal Hierarchical Culling[@GBK06]は時間的一貫性を利用することでアイドル時間を回避する。これらは次節にてその詳細を論じたい。

# 概要 {#sec:3}

この節では、CHCおよびNOHCアルゴリズムを手短に評し[briefly review]、これらの問題のいくつかを論ずる。その後、我々は新しいCHC++アルゴリズムの主要な構成要素を述べる。

## CHCとその問題 {#sec:3.1}

Coherent Hierarchical Cullingアルゴリズム[@BWPP04]はハードウェアオクルージョンクエリのオーバーヘッドおよびレイテンシを減らすために時間的および空間的一貫性を利用する。前から後ろへの順に階層を横断し、以前に可視であった[previously visible]リーフ^[訳注:木構造の葉ノード]と以前に不可視であった[previously invisible]境界のノードに対してのみクエリを発行する。以前に可視であったリーフは現在のフレームにおいて可視のままであると仮定され、故にこれらは即座にレンダリングされる。これらのノードに対するクエリの結果は次のフレームに対するこれらの分類を更新するのみである。不可視であるノードは不可視のままであると仮定されるが、そのアルゴリズムは可視性の変化を発見するために現在のフレームにおけるクエリ結果を検索する[retrieve]。オリジナルのCHCアルゴリズムの擬似コードに対する図6を参照のこと(印のない部分[unmarked parts])。

クエリ数の削減(クエリは以前に可視であった内側[interior]ノードに関して発行されない)と賢いインターリービングは許容できる量にまでオクルージョンクエリのオーバーヘッドを削減した。そのアルゴリズムは大量の遮蔽を持つシナリオに対して非常に上手く動作する。しかし、ジオメトリのレンダリングが問い合わせ[querying]と比べて安価となるより新しいハードウェアやシーンの多くが可視となる視点では、その手法は伝統的な視錐台カリングより低速にさえなる可能性がある。これは無駄なクエリや不必要な状態変化の結果である。この問題は視錐台カリングより確実に高速であるアルゴリズムを求めているゲーム開発者に対してCHCアルゴリズムを魅力的でなくしている。CHCのもうひとつの問題は高度に最適化されたゲームエンジンのレンダリングループへのその手法の複雑な統合にある。CHCは空間的階層の個々のノードのレンダリングと問い合わせを互い違いにする[interleave]。これはエンジンがマテリアルでのソートを処理することを不可能にし、より多数のエンジンAPI呼び出しを引き起こす。

## NOHCとその問題 {#sec:3.2}

Guthe et al.によって提案されるNear Optimal Hierarchical Cullingアルゴリズム[@GBK06]は無駄なクエリの問題に取り組む。その手法はクエリコストとレンダリングコストを推定するためにグラフィクスハードウェアの較正されたモデルを用いる。単純なスクリーンカバレッジモデルと時間的一貫性を仮定する更なる補正を用いることによってノードの遮蔽を推定する。その遮蔽推定とハードウェアモデルは現時点で処理されているノードに関してオクルージョンクエリを適用するかどうかを決める費用便益[cost/benefit]のヒューリスティクスに用いられる。このヒューリスティックは2、3の規則を伴うクエリに対する洗練された妥当性テストを用いる。

そのアルゴリズムは、かなりのクエリ数、特に可視ノード上に適用されるであろうクエリを節約する。これはCHCで提案される仮定された可視性の最適化が用いられない場合にCHCアルゴリズム全体に大幅な改善をもたらし得る。

NOHCに対する結果は、適切なハードウェア較正により、その手法が視錐台カリングより上手に常に処理されることを示している。これらの論文において、Guthe et al.[@GBK06]もまたオクルージョンクエリに基づいた最適なカリングアルゴリズムを定義した。その最適なアルゴリズムはすべてのカリングされるノードの状態がオクルージョンクエリによって検証されなければならないという前提の下で導出される。そうして、NOHC手法は最適なアルゴリズムに近づく。

我々の論文では、我々はGuthe et al.[@GBK06]によって用いられる最適性の定義が依然として多大な改善の余地を残している[leaves significant room for improvement]ことを示す。実際に、CHC++アルゴリズムはGuthe et al.によって定義される最適条件[optimum]を常に明らかに下回っている。

NOHCはハードウェアパラメータが人為的なレンダリングシナリオを用いる事前処理において測定されるハードウェア較正の工程を必要とする。モデルの正確なパラメータの測定は非常に慎重な[careful]実装を必要とする。しかし、精密に[precisely]実装したとしても、これらの測定は状態変化、パイプライン処理、実際のウォークスルー中のレンダリングとオクルージョンクエリのインターリービングの複合的な処理を反映する必要がないことが判明している。我々の新しい手法はハードウェア較正に頼らず、外部パラメータへの依存度を最小化することを目的とする。実際に、ユーザにはその影響度が十分に予測可能であるいくつかのパラメータを大まかに設定できるようにしてある。

## CHC++の構成要素[building blocks] {#sec:3.3}

新しいCHC++アルゴリズム手法は以前に言及したすべての問題に取り組み、以下の新しい構成要素を含めることによってCHCを拡張する。

**クエリのバッチ処理[batching]のためのキュー**。ノードはクエリされる前に、キューへ追加される。以前に可視であったノードと以前に不可視であったノードを累積するために別々のキューが用いられる。我々は個別のキューではなくクエリのバッチを発行するためにキューを用いる。これは状態変化を1から2桁ほど削減する。クエリのバッチ処理は[@sec:4]で述べたい。

**マルチクエリ**。我々は単一のオクルージョンクエリによってより多くのノードをカバーすることができるマルチキューをコンパイルする([@sec:5.1])。これは以前に不可視であったノードに対するクエリ数を1桁台にまで削減する。

**タイトな境界ボリューム**。我々は明示的な構築を必要とせずにタイトな境界ボリュームを用いる([@sec:6])。これはクエリ数の削減と共にレンダリングされるトライアングル数の削減をもたらす。

本論に示されるすべてのテストに対して、我々は表面積のヒューリスティクス[@MB90]に従って構築される軸並行な境界ボリューム階層(BVH)を用いたことに注意する。しかし、その示される手法は、BVHの特性を明示的に活用するタイトな境界ボリューム最適化を除いて、他の種類の空間的階層[@MBM*01]と互換性がある。

# 状態変化の削減 {#sec:4}

レンダリングステートの変更はレンダリングパイプラインにおいてかなりのコストを占める[constitute]。以前のオクルージョンカリング手法は、クエリの全体数を削減するだけでなく、主にレイテンシを隠蔽しつつGPUを占有し続ける方法においてクエリをスケジューリングすることに焦点を当てていた。しかし、クエリ数が削減されるとしても、残りのクエリのすべては、少なくとも色や深度バッファへの書き込みが無効化され、そして、クエリの後に再び有効化されるような状態変化を潜在的に引き起こす。複合的なシェーダが用いられる場合、この状態変化もまたシェーダのオンオフの切り替えを伴う。

これらのレンダリングステートの変更はクエリそれ自体よりずっと大きなオーバーヘッドを引き起こす。そのオーバーヘッドはハードウェア側(例えば、キャッシュのフラッシュ)にあるかもしれないし、ドライバ側にあるかもしれないし、アプリケーション側にさえあるかもしれない。よって、状態変化の数を許容できる量にまで削減することは非常に望ましい。ゲーム開発者は現在のハードウェア上で許容できる値の参考としてフレームあたりの状態変化数を約200としている[refer to]。

**クエリのバッチ処理**。この問題への我々の解法は、アルゴリズムによって要求されたときに即座にクエリを発行するのではなく、オクルージョンクエリをバッチ処理することに基づいている。レンダリングステートはバッチあたりに一度だけ変更されるため、状態変化の削減は我々が発行するクエリのバッチサイズに直接的に対応する。そのバッチ処理のアルゴリズムは以下の節にて述べられるように個別に可視および不可視ノードを扱う。

## 以前に不可視であったクエリのバッチ処理 {#sec:4.1}

クエリされた不可視ノードは我々が*iキュー[i-queue]*と呼ぶキューに追加される。iキューの中のノードの数がユーザ定義のバッチサイズ*b*に達したとき、我々はクエリのためにレンダリングステートを変更し、iキューの中のノードごとにオクルージョンクエリを発行する。([@sec:5.1]では、いくつかのノードがクエリ数を削減するために1つのオクルージョンクエリに組み合わせることができる方法を見てゆきたい。)

バッチサイズ*b*はレンダステートの変更の削減に強く結び付き[tightly connected]、CHCアルゴリズムよりおおよそ*b*倍少ない状態変化をもたらす。一方で、バッチ処理は不可視ノードに対するクエリ結果の可用性を実質的に遅延させている。これは可視性の変化が後に検出される可能性があり、十分な量の代替の仕事(例えば、可視ノードのレンダリング)が残されていない場合、それらによって生成された[spawned]追跡調査[follow-up]クエリが更なるレイテンシをもたらすだろう。

*b*に対する最適値はシーンのジオメトリ、マテリアルシェーダ、マテリアルソートに関するレンダリングエンジンの能力に依存する。我々のシーンおよびレンダリングエンジンでは、我々は、このパラメータの精密な調整は必要なく、20から80までの値がその手法に追加のレイテンシをもたらすことなくレンダステートの変更のほぼ十分な削減量をもたらすことに気がついた。

## 以前に可視であったクエリのバッチ処理 {#sec:4.2}

CHCアルゴリズムは以前に可視であったノードに対するクエリを発行し、クエリの結果を待たずにノードのジオメトリをレンダリングすること思い出してほしい[recall]。CHCと同様に、我々の提案する手法はその階層を横断している間に以前に可視であったノードのジオメトリをレンダリングする。しかし、クエリは即座には発行されない。代わりに、対応するノードは我々が*vキュー[v-queue]*と呼ぶキューに格納される。

重要な観測結果[observation]は、これらのノードに対するクエリは、これらの結果が次のフレームで使われるのみであろうことから、現在のフレームに不可欠[critical]ではない、ということである。我々は待ち時間を穴埋めするためにvキューからのノードを用いることによってこの観測結果を活用する。横断キューが空であり、未解決クエリの結果が利用できないときはいつでも、我々はvキューからのノードを処理する。結果として、我々は未解決クエリのレイテンシによって操られる以前に可視であったノードに対するクエリの適応的バッチ処理を処理している。フレームの終わりに、以前に不可視であったノードに対するすべてのクエリが処理され終わったとき、その手法はvキューからのすべての処理されていないノードに対する単一の大きなバッチを単に適応する。

vキューからのノードを処理する前に、我々はレンダステートの変更が必要とされているかどうかの確認も行うことに注意する。場合のほとんどで、以前に発行された不可視ノードに対するクエリのバッチによってすでに変更されたためにレンダリングステートを変更する必要は一切ないことが判明している。故に、我々は以前に可視であったノードに対する状態変化を基本的に取り除いた。

有益な副次的効果として、vキューは元のCHCアルゴリズムによって作られた前から後への順序の違反[violations]の効果を削減する。具体的には、以前に隠れていたノードが現在のフレームにおいて以前に可視であったノードを遮蔽する場合、以前に可視であったノードは以前に不可視であったノードがレンダリングされる前にしばしばクエリされるであろうために、この効果は次のフレームで補足のみされるだろう。この問題は多くの可視性の変化が同時に発生するような状況で明らかになる。vキューを用いてクエリを遅延させることはそのような可視性変化を検出され易くするだろう。

## ゲームエンジン統合 {#sec:4.3}

CHC++手法の既存のゲームエンジンへの簡単な統合のために、我々はアルゴリズムに我々が*レンダキュー[render queue]*と呼ぶ追加のキューを使うことを提案する。このキューはレンダリングのためにスケジューリングされたすべてのノードを累積し、クエリのバッチが発行される間近となるときに処理される。レンダキューを処理するとき、エンジンは内部のマテリアルシェーダソートを適用でき、そして、新しい順序でキューに格納されたオブジェクトをレンダリングする。レンダキューのもうひとつの有益な効果はエンジンAPI呼び出しの削減である。これらの呼び出しは非常にコストが高く、故に、これらの削減は、例えば、有名なOGREゲームエンジンで体験したような、大幅な高速化をもたらす。

![CHC++アルゴリズムによって用いられる様々なキュー。CHCアルゴリズムによって用いられなかったキューは青色で強調される。](){#fig:3}

CHC++アルゴリズムによって用いられる様々なキューの概要は[@fig:3]に示される。クエリキューにおける重なり合った[overlaid]ノードは[@sec:5.1]で述べられるであろうマルチクエリに対応する。

# クエリ数の削減 {#sec:5}

最近のオンラインのオクルージョンカリング手法はオクルージョンクエリの数をそのオーバーヘッドを減らすために削減することに焦点を当てていた。具体的に、Guthe et al.[@GBK06]の手法は費用便益のヒューリスティクスに基づくクエリの除去やグラフィクスハードウェアの較正されたモデルに対する洗練されたアプローチを提案した。この節では、我々はGuthe et al.によって最適であると以前に定義された仮説のアルゴリズム*以下[below]*にさえクエリ数を削減することができる2つの新しい手法を提案する。

## 不可視ノードのためのマルチキュー {#sec:5.1}

以前の技術のすべてはテストされる以前に不可視であったプリミティブあたり1つのオクルージョンクエリを用いる(階層内のノード、境界ボリューム、グリッド内のセル)。これらのノードに対するオクルージョンクエリは削減できないと見なされた。

しかし、以下の観測結果は我々が以前に不可視であったノードに対してさえもクエリ数を削減することを可能にする。いくつかの以前に不可視であったシーンの部分が現在のフレームにおいて不可視のままである場合、その部分全体に対する単一のオクルージョンクエリはそれの可視性の状態を検証するのに十分である。そのようなクエリはこのシーンの部分にあるプリミティブのすべての境界ボックスをレンダリングし、プリミティブが遮蔽されたままであれば0を返すだろう。例えば、静的なシーンと静的な視点の極端なケースでは、単一のオクルージョンクエリがシーンのすべての不可視部分で用いられることができるかもしれない。

特定の可視性の一貫性を仮定すれば、我々の新しい技術は不可視のままである可能性が同等に高い以前に不可視であったノードのグループを形成することによってそのようなシーンの部分を識別することを目的としている。我々が*マルチクエリ[multiquery]*と呼ぶ単一のオクルージョンクエリはそのようなグループごとに発行される。マルチクエリが0を返すならば、グループ中のすべてのノードは不可視のままであり、これらの状態は単一のクエリによって更新され続ける。そうでなければ、一貫性はこのグループに対して破壊されてしまい、我々はiキューにそれらを再挿入することですべてのノードに対する個々のクエリを発行する。最初のケースではクエリ数はグループ内のプリミティブ数によって削減されることに注意する。しかし、次のケースではバッチに対するマルチクエリが無駄になってしまった。

我々は適切なノードのグルーピングを見つけるために費用便益のヒューリスティクスに基づいた適応的メカニズムを用いる。計算の極めて重要な[crucial]部分は、次節で述べられる、ノードの可視性分類における一貫性の推定である。実際のヒューリスティクスは[@sec:5.1.2]で述べたい。

### 可視性の一貫性の推定 {#sec:5.1.1}

場合の大多数において、階層内のほとんどのノードに対する可視性には強い一貫性がある。我々の目的はこの一貫性を定量化することである。具体的には、与えられたノードの可視性の分類を知れば、我々はこのノードが次のフレームにおいてその可視性の分類を維持するであろう確率を推定することを目指す。我々の実験はノードの"履歴[history]"との、すなわち、ノードが同じ可視性の分類を維持したフレーム数とのこの値の強い相関があることを示している(我々はこの値を*可視性の持続度[visibility persistence]*と呼ぶ)。非常に長い時間の間に不可視であったノードは不可視であり続ける可能性が高い。そのようなノードは、例えば、カメラが車のエンジンの内部に移動しない限り見えることが一切ないであろう車のエンジンブロックであるかもしれない。それどころか[on the contrary]、低速移動のシナリオでさえ、分類を定期的に変化させる可視境界[visible border]上のいくつかのノードが常にある。それ故に、最近不可視になったノードがまもなく可視になる可能性は極めて高い。

我々は可視性の持続度$i$の関数として望ましい確率を定義し、以前のノードの履歴に基づいてそれを近似する。

$$
P_{keep}(i) \approx \frac{n_{i}^{keep}}{n_{i}^{all}}
$$ {#eq:1}

ここで、$n_{i}^{keep}$は、$i$フレームの間に同じ状態にあり続け、$i+1$番目のフレームでこれらの状態を維持する、すでにテストされたノード数であり、$n_{i}^{all}$は、$i$フレームの間同じ状態にあり続けた、すでにテストされたノードの総数である。[@fig:4]は可視性の持続度$i$に対する確率$p_{keep}$のプロットを示す。計数[counters]$n_{i}^{keep}$と$n_{i}^{all}$はウォークスルーのすべての前のフレーム上で累積される。

最初の数フレームでは、$i$の値が大きいと特に、$p_{keep}(i)$の正確な計算のための計測値が足りない。我々は$p_{keep}(i)$のより高い値へのすでに計算された値の区分定数伝播[piecewise constant propagation]によってこの問題を解決する。

計測によって$p_{keep}(i)$を計算するより単純な代替法として、我々は我々のシーンやウォークスルーで計測した関数にかなり[reasonably]上手く対応する解析的な式を提案する。

$$
p_{keep}(i) \approx 0.99 - 0.7e^{-i}
$$ {#eq:2}

この関数の使用は計測された関数と同様に正確な$p_{keep}$の推定値をもたらさないが、測定された関数の計算を実装することを回避するのに使うことができる。[@fig:4]は2つの異なるシーンに対する解析的な関数と計測された関数を図示する。

![可視性の持続度$i$に依存する$p_{keep}(i)$。[@eq:2]からの解析的な関数はPowerplantとViennaのシーンで計測した関数とほぼ一致することに注意する。](){#fig:4}

### マルチクエリに対する費用便益モデル {#sec:5.1.2}

与えられたバッチ(すなわち、iキュー)における以前に不可視であったノードに対するマルチクエリの最適なサイズを定めることはマルチクエリへのバッチのすべての可能性のある分割の計算を必要とする大局的な最適化問題である。代わりに、我々はマルチクエリごとに費用便益の比を最大化する貪欲なモデルを用いる。

そのコストはマルチクエリ1つあたりに発行されるクエリ数の期待値であり、以下で表される。

$$
C(M) = 1 + p_{fail}(M) * |M|
$$ {#eq:3}

ここで、$p_{fail}(M)$はマルチクエリが失敗する確率であり(可視を返し、この場合、すべてのノードが個々にテストされなければならない)、$|M|$はマルチクエリ内のノード数である。定数$1$はマルチクエリそれ自体のコストを表しているのに対し、$p_{fail}(M) * |M|$は個々のノードに対して追加で発行されるクエリ数の期待値を説明していることに注意する。確率$p_{fail}$は以下のようにマルチクエリにおけるノードの可視性の持続度の値$i_N$から計算される。

$$
p_{fail}(M) = 1 - \prod_{\forall N \in M} p_{keep}(i_N)
$$ {#eq:4}

マルチクエリの有益な点は単純にマルチクエリにおけるノード数、すなわち、$B(M) = |M|$である。

iキューにおけるノードを仮定するならば、貪欲な最適化アルゴリズムは与えられたコストでの利益を最大化する。我々はまず不可視であり続ける確率、すなわち、$p_{keep}(i_N)$に基づいて降順にノードをソートする。その後、キューにおける最初のノードを皮切りに、我々はマルチクエリにノードを追加し、各工程で、費用便益の比としてマルチクエリの値$V$を計算する。

$$
V(M_j) = \frac{B(M_j)}{C(M_j)}
$$ {#eq:5}

$V$は特定の$M_j$に対して最大値に達し、故に、$j$はiキューの先頭にあるノードに対するマルチクエリの最適なサイズに対応することが判明している。ひとたびこの最大値を発見すれば、我々は対応するノードに対するマルチクエリを発行し、iキューが使い果たされるまでその処理を繰り返す。結果として、我々は不可視であり続ける確率の高いノードに対するマルチクエリをより大きく、可視に転ずる可能性の高いノードに対するマルチクエリをより小さくコンパイルする。

## 可視ノードのテストの省略 {#sec:5.2}

元のCHCアルゴリズムは以前に可視であったノード上のクエリ数を削減するための重要な最適化を導入した。可視ノードは$n_{av}$フレームの間に可視であり続けると仮定され、フレーム$n_{av}+1$においてテストされるのみであろう。この最適化は以前に可視であったリーフに対する平均クエリ数を$n_{av}+1$倍[a factor of]に実質的に削減する。

しかし、この単純な手法はクエリが時間的に整列し得る[can be temporally aligned]という問題を持つ。このクエリの整列[alignment]はノードが同じフレームにおいて可視になりがちな状況において問題となる。例えば、多くのノードを同じフレームで可視にしてしまうような、典型的な街のシーンにおいて地面の高さ[ground level]から屋根の高さ[roof level]以上に視点が移動する場合が考えられる。その後、これらのノードのクエリは$n_{av}+1$番目のフレームでスケジューリングされ、故に、クエリのほとんどは再び整列するだろう。フレームあたりの平均クエリ数は依然として削減されるだろうが、その整列は目に付く[observable]フレームレートドロップを引き起こし得る。

我々は小さな乱数$-r_{max} < r < r_{max}$による$n_{av}+1$の無作為化が満足するやり方において問題を解決しないことに気付いた。その問題は、無作為化が小さい場合、クエリが依然として非常に[very much]整列しているかもしれない、ということである。対して、無作為化が大きい場合、クエリの幾つかを処理するのが遅すぎてしまい、故に、可視状態から不可視状態への変化を補足するのが遅すぎてしまうだろう。

我々は最も満足する解法がオクルージョンクエリの*[first invocation]*を無作為化することで達成されることを発見した。ノードが可視に転じた後、我々はクエリが発行されるであろう次のフレームを決定するために乱数$0 < r < n_{av}$を用いる。続けて、そのノードが以前のテストですでに可視であった場合、我々は$n_{av}$によって与えられる通常のサンプリング間隔[interval]を用いる。

我々は様々な$n_{av}$の値で実験を行った。その最適値はシーンそれ自体、検査の一貫性[inspection coherence]、ハードウェアのパラメータ、および、レンダリングエンジンのパラメータに依存する。幸いにも、我々のテストはその依存性がそれほど強くなく、5から10の値がすべてのテストで安全かつ堅牢な選択であったことを示している。

# よりタイトな境界ボリューム {#sec:6}

オクルージョンくえりによってもたらされるオーバーヘッドとは別に、カリングアルゴリズムの成功は空間的階層における境界ボリュームがどれだけタイトに含まれるジオメトリを近似するかに強く依存する。その合致[fit]が十分にタイトでないならば、多くのノードはたとえ含まれるジオメトリが可視でないとしても可視として分類されるだろう。タイトな境界ボリュームを得るための技術はいくつかあり、ほとんどはより複合的な形状で軸並行な境界ボックスを置き換えることによってなされる。これらの手法はほとんどのオクルージョンカリングアルゴリズムに直接的に適用できるだろうが、一方で、これらもまたボリュームの計算と維持のオーバーヘッドを成す[constitute]。これは特に動的なシーンで高価となり得る。

我々は任意の境界ボリューム階層に適用されるハードウェアオクルージョンクエリの観点において*内部ノード[inner nodes]*に対するタイトな境界を定める単純な手法を提案する。特定のノードに対して、我々は特定の深さでのその子供の境界ボリュームの集まりとしてその*タイトな境界ボリューム[tight bounding volume]*を定める([@fig:5]を参照)。

![球として示されるオブジェクトをよりピッタリと[closely]表現するBVHのノードに対するタイトな境界ボリューム。タイトな境界ボリュームは親のボックス(白)ではなく子供の境界ボックス(赤)から成る。](){#fig:5}

境界ボリュームのジオメトリをレンダリングするための最新のAPIを用いるとき(例えば、OpenGLのVertex Buffer Objects)、オクルージョンクエリに対するよりわずかに複合的なジオメトリは実践的にそのオーバーヘッドを増加させないことが判明している。しかし、より小さな境界プリミティブがスクリーン空間において重なるときにタイトな境界ボリュームをラスタライズするためのペナルティがあるかもしれず、故に、オリジナルのノードの境界ボリュームの投影と比べてフィルレートを増加させる。そのようなケースを回避するため、我々はよりタイトな境界の有用性を保証するための単純なテストを用いる。タイトな境界ボリュームのために子ノードを集めるとき、我々は子供の境界ボリュームの表面積の合計が$s_{max}$×親ノードの表面積より大きくないかどうかをテストする(これは特定の視点に依存しないことに注意)。これが正しいならば、我々は横断を終了し、境界表現をこれ以上に精錬しない。我々はノードからの深さがしたいされた最大の深さ$d_{max}$より大きいかどうかによって境界ボリュームの探索を終了する。以下の値は我々のテストにおいて良い結果をもたらした: $d_{max} = 3, s_{max} = 1.4$

タイトな境界ボリュームを定めることは階層の*リーフ*に対しても有利であることに注意する。これはよりわずかに深い階層を構築し、*仮想リーフ[virtual leaves]*として指定された数より少ないトライアングルを含む階層の内部ノード、すなわち、横断中にリーフと見なされる内部ノードを作ることによって容易に達成できる。

結果として、タイトな境界ボリュームはほぼコストなしでいくつかの利点をもたらす。(1)階層の内部ノードの早期[earlier]カリング、(2)可視として別途分類される[otherwise be classified]であろうリーフのカリング、(3)内部ノードの可視性分類の一貫性の増加。第1の特性はクエリ数の削減を引き起こす。第2の特性はレンダリングされるトライアングル数の削減をもたらす。最後に、第3の利点は繰り返される可視性の引き上げ[pull-up]と引き下げ[pull-down]によって引き起こされる内部ノードに対する可視性分類における変化を回避する。

# まとめ上げ {#sec:7}

CHC++アルゴリズムは、いくつかの重要な追加要素[add-ons]を含めて、CHCアルゴリズムの単純さを維持することを目的としている。この節では、我々は完全なCHC++アルゴリズムを要約し、CHCアルゴリズムとのその主な差異を強調する。CHC++アルゴリズムの擬似コードは[@fig:6]に示される。

![CHC++の主な横断ループの擬似コードと選ばれた重要な関数。オリジナルのCHCとの差異は青色で印を付けている。](){#fig:6}

CHCにあるように、我々は階層を横断するために優先度付きキュー[priority queue]を用いる。このキューは処理されるノードの前から後への順序をもたらす。CHCと異なり、新しいアルゴリズムはクエリされるべきノードを格納するための2つの新しいキューを用いる(vキューとiキュー)。これらの2つのキューはレンダリングステートの変更を削減したりマルチクエリをコンパイルしたりするための鍵である。

以前に可視であったノードはCHCの場合と同様に即座にレンダリングされる。これらは現在のフレームにおけるテストのためにスケジューリングされる場合、vキューに配置される。クエリをスケジューリングするためのアルゴリズムは、クエリ数を削減し、フレーム全体で均等に分布させるため、すでに論じた[discussed]時間的にジッタリングされたサンプリングパターンを用いる。vキューに格納されるノードに対するクエリはそれが発生する必要がある場合に待ち時間を埋めるために用いられる。フレームの終わりで、vキューにある残りのノードはクエリの単一バッチを形成する。

iキューは前のフレームにおいて不可視であり続けている処理されたノードを累積する。そのキューの中に十分な数のノードがあるとき、我々は、それらをマルチクエリにコンパイルしつつ、iキューにあるノードに対するオクルージョンクエリのバッチを適用する。

ゲームエンジンにその手法を統合するとき、可視ノードはまずレンダキューに累積される。そして、レンダキューは、iキューからのクエリのバッチがまさに発行されるようとする前に、エンジンによって処理される。

# 結果 {#sec:8}

TODO

# おわりに {#sec:9}

TODO
