---
titile: Optimizing the Graphics Pipeline with Compute [@Wihlidal2016]
bibliography: bibliography.bib
numberSections: false
---
# 略語(Acronyms)

- ここで示される最適化やアルゴリズムはAMDのGCNハードウェアに特化している。
    - コンソールやハイエンドのAMD PCで有効。

|||
|-:|:-|
|VGT|Vertex Grouper / Tessellator|
|PA|Primitive Assembly|
|CP|Command Processor|
|IA|Input Assembly|
|SE|Shader Engine|
|CU|Compute Unit|
|LDS|Local Data Share|
|HTILE|Hi-Z Depth Compression|
|GCN|Graphics Core Next|
|SGPR|Scalar General-Purpose Register|
|VGPR|Vector General-Purpose Register|
|ALU|Arithmetic Logic Unit|
|SPI|Shader Processor Interpolator|

# Peak Triangle Rate

- Shader Engineは1クロックに三角形を1つ発行(issue)できる。
    - XB1とPS4は2つのSEを持つ。
    - より新しいAMD GPUは4つのSEを持つ。
- Compute Unitは1クロックにFMAを64回実行できる。
    - XB1は12個のCUを持つ。
    - PS4は18個のCUを持つ。
    - より新しいAMD GPUは64個のCUを持つ。
- FMAは1クロックに浮動小数点演算を2回行う。
    - 掛け算と足し算を1回ずつ。
- それぞれをかけ合わせると、サイクルあたりの実行可能なALU処理数が計算できる。
    - XB1では、12CU * 64ALU * 2FLOPs = 1536 [ALU ops / cycle]
    - PS4では、18CU * 64ALU * 2FLOPs = 2304 [ALU ops / cycle]
    - より新しいAMD GPUでは、64CU * 64ALU * 2FLOPs = 8192 [ALU ops / cycle]
- 動作するWaveにより4倍になるが各ALUは4クロックかかるので、双方が打ち消されてこの形になる。
- サイクルあたりの実行可能なALU処理数をSE数で割れば、トライアングルあたりの実行可能なALU処理数が得られる。
    - XB1では、1536 [ALU ops / cycle] / 2 SE = 768 [ALU ops / triangle]
    - PS4では、2304 [ALU ops / cycle] / 2 SE = 1017 [ALU ops / triangle]
    - より新しいAMD GPUでは、8192 [ALU ops / cycle] / 4 SE = 2048 [ALU ops / triangle]
- トライアングルあたりの実行可能なALU処理数をサイクルあたりのALU処理数で割れば、トライアングルをカリングしなければならない命令数の上限を得られる。
    - XB1では、768 [ALU ops / triangle] / 2 [ALU ops / cycle] = 384 [cycle / triangle]
    - PS4では、1017 [ALU ops / triangle] / 2 [ALU ops / cycle] = 508 [cycle / triangle]
    - より新しいAMD GPUでは、2048 [ALU ops / triangle] / 2 [ALU ops / cycle] = 1024 [cycle / triangle]

# Motivation --- Death By 1000 Draws

- DirectX 12は**数百万の描画を約束する**。
    - オーバーヘッドが少なくなることで、CPUパフォーマンスの大幅な改善が期待できる。
    - そのパワーの管理は開発者に委ねられる。
    - コンソールハードウェアは対象が固定である。
- GPUは未だ**小さな描画に喉を詰まらせる(chokes on tiny draws)**。
    - ベースパスの後半にGPUがかろうじて使われているのをよく見る。
    - そこには小さくて細かいオブジェクトや遠くのオブジェクトが大量に存在する --- そのほとんどはHi-Zカリングされる。
    - **空の頂点Wavefront(empty vertex wavefront)**を走らせる必要があることが効率を失わせている。
- **描画が増えるということは必ずしも良いことではない**。

トライアングルがピクセルに対して小さすぎると結果として何も描かれないことになるため、後段のピクセルシェーダに仕事がなく頂点シェーダだけが空回りしている状態になってしまう。

# Motivation --- Primitive Rate

- コンソール上でのサイクルあたりのプリミティブが2に近くなると仮定するのは楽観的すぎる。
    - パイプラインにはラスタライザ以外にもボトルネックが起こり得る。
    - XB1上で実際に測定したところ、通常のレンダリングによるクロックあたりのトライアングル数は**0.9**となった。
        - プリミティブレートはこだわる(bound)[誤訳?]ところではないので、この値は実際には極めて健全である。
- 何か役に立つことをするなら、パイプラインの他の所にこだわるほうが良い。
- 最大レートを得るには、VGTとPAの間の良いバランスとlucky schedulingが必要になる。
    - 例えば、waveはPAを交互に使い(alternate between PAs)、そのPAは独立して働く(operate)ため、異なる2つのwave内の同じ頂点は2回シェーディングされなければならないことがある。
- VGTとPAの間のFIFOの深さにより、頂点がFIFOに入った瞬間から数えて**4096サイクル未満で頂点シェーダのpositionを計算する必要がある**。
    - このため、かかるサイクル数はpositionを計算するより僅かに多くなる。
    - 頂点シェーダに時間がかかると、比例してプリミティブレートが下がる。
- いくつかのゲームはシャドウパスにおいてピークパフォーマンス(95%以上の範囲)に極めて近いところにまで達する。
    - 通常は巨大なトライアングルがあることで他より遅くなる領域がいくつか存在する。
    - 粗い(coarse)ラスタライザはクロックあたりのsuper-tileを1つだけ処理する。
    - 32x32より大きな境界矩形を持つトライアングルを粗いラスタライザ上で処理するには複数サイクルが必要になる。
        - プリミティブレートが下がる。

- サイクルあたりのプリミティブがほぼ2になるベンチマークは以下のような特徴を持つ。
    - 頂点シェーダは何も読み出さない。
    - 頂点シェーダは`SV_Position`のみを書き出す。
    - 頂点シェーダは常に0をpositionとして出力する --- すべてのプリミティブは自ずと明らかにカリングされる。
    - インデックスバッファはすべて0である。
        - つまり、すべての頂点はキャッシュヒットである。
        - キャッシュヒットの頂点はピーク計算に含まれない。
        - これが始めの2[頂点/クロック]をヒットさせずに2[プリミティブ/クロック]を達成することができる唯一の方法である。
    - すべてのインスタンスは64の倍数分の頂点を持つ --- 頂点シェーダのwaveが埋まらない可能性を少なくする。
    - ピクセルシェーダをバインドしない --- パラメータキャッシュを使わない。
- 頂点シェーダ後にストールを起こさせないようにする必要がある。
    - ParamSize <= 4 * PosSize
    - ピクセルの排出が生み出されるよりも速い。
    - シザリングを起こさない。
- PAは頂点シェーダが生成できるより早く仕事を受け取ることができる。
    - テッセレーションがピーク頂点プリミティブスループットを達成するのをしばしば見る。
        - 一度にひとつのSEで。

# Motivation --- Opportunity

- CPUで粗くカリングして、GPUで洗練する(refine)。
    - GPUサブミッションの前にCPU上で粗いカリングを行うことがよくある。
- **CPUとGPUの間のレイテンシが最適化を妨げる**。
    - タイトなlock steppingになり得る。
    - コンソールではCPUリソースは限られており、CPUコアをうまく使えているとは言えない。
- GPGPUサブミッション！
    - 深度を意識したカリング(depth-aware culling)。
        - タイトにしたシャドウ境界 / Sample Distribution Shadow Map。[21]
        - 寄与を持たないシャドウキャスタをカリングする。[4]
        - カラーパスから非表示オブジェクトをカリングする。
    - VRのlate-latchカリング。
        - CPUは保守的な錐台をサブミットして、GPUが洗練する。
    - **トライアングルカリングとクラスタカリング**。
        - このプレゼンテーションでカバーする。

- グラフィクスパイプラインへ直接的に対応する。
    - ハルシェーダの負荷を下げる。
    - テッセレーションパイプライン全体の負荷を下げる。[16][17]
    - プロシージャルな頂点アニメーション(風や布など)を使う。
    - **複数のパスやフレームの間で結果を再利用する**。
- グラフィクスパイプラインへ間接的に対応する。
    - 境界ボリュームの生成。
    - 事前スキニング。
    - ブレンドシェイプ。
    - GPUからGPUの仕事を生成する。[4][13]
    - シーンや可視性の決定。
- 言いたいこと(the mantra)は**描画を普通のデータとして扱う**ことである。
    - 事前に構築できる。
    - キャッシュしたり再利用したりできる。
    - GPUで生成できる。

# Culling Overview

- TODO

# References
