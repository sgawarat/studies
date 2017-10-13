---
title: Render graphs and Vulkan - a deep dive [@Themaister2017]
bibliography: bibliography.bib
numberSections: false
codeBlockCaptions: true
---
# Introduction

VulkanやD3D12といったモダンなグラフィクスAPIはエンジン開発者に新たな挑戦をもたらす。これらAPIによりCPUオーバーヘッドは劇的に減少した一方で、ドライバの"良い"経路に当たっているときはGPUパフォーマンスに関するギャップを埋めることは難しくなるのは明らかであり、我々はGPU束縛である。OpenGLやD3D11のドライバは(はっきりと)あらゆる種類のトリックを使ってでもGPUパフォーマンスを改善するためならいかなる苦労も惜しまない。開発者として我々がこれに払うコストは予測不可能なパフォーマンスであり、より高いオーバーヘッドである。我々は柔軟性、パフォーマンス、使いやすさのバランスを取るこれらのAPIのために素晴らしいレンダリングバックエンドを構築する方法を考え出す中にあるため、グラフィクスバックエンドを書くことは再び面白くなっている。

先週、私は自分の見解としてのVulkanレンダリングエンジンであるGraniteというサイドプロジェクトをリリースした。そのようなプロジェクトは界隈にたくさんあり、それぞれにはそれぞれの利点があるが、私は特にレンダグラフ実装を議論したい。

レンダグラフ実装はYuriy O'DonnellのGDC2017でのプレゼンテーション[@ODonnell2017]に端を発する。このトークはD3D12が中心になっているが、私はVulkanで実装した。

(注記:個々ではレンダグラフとフレームグラフは同じことを意味する。また、Vulkanで言及することはD3D12にも当てはまるかもしれない…多分。)

# The problem

レンダグラフはモダンAPIにおけるひどく迷惑な問題を根本的に解決する。どうやって手動による同期を扱うのだろうか？すぐに分かる代替案を見ていこう。

## Just-in-time synchronization

最も素直なアプローチは基本的には直前に同期を行うことである。テクスチャにレンダリングしようとしたり、リソースをバインドしたり、似たようなことをするたび、自分自身に「このリソースは同期する必要がある保留中の仕事があるか？」と問いかける必要がある。そうであれば、その直前でなんとかしてそれを扱う必要がある。読み込みが1000回以上あっても書き込みが一度しかなかったりするため、この種の追跡は非常に辛いものになる。マルチスレッディングは非常に辛いものになる。2つのスレッドが1つのバリアを見つけた場合どうなるだろうか？あるスレッドが"勝つ"必要があるので、これを扱うために大量の役に立たないスレッド間同期に悩まされることになる。

追跡する必要があるのは実行にとどまらず、Vulkanにおいてイメージレイアウトやメモリアクセスの問題もある。特定の方法でリソースを使うには固有のイメージレイアウトが必要になる(または、単にGENERALでもよいが、この場合フレームバッファの圧縮が使えなくなる)。

本質的に、我々が欲しいものがジャストインタイムでの自動的な同期であるならば、基本的にはOpenGLやD3D11に立ち返れば良い。ドライバはすでに死ぬほど最適化されているのに、どうして中途半端に再実装したいと望むのだろうか？

## Fully explicit synchronization

別の側面から見れば、我々の選択するAPI抽象化は自動的な同期の一切を取り除くため、アプリケーションはすべての同期点を手動で扱う必要がある。間違いを犯したときは、"面白い"デバッグ講義の準備をしておいてください。

単純なアプリケーションならこれでうまく行くが、この道を下り始めてしまえば、どれだけ厄介なことになっているかにすぐに気付くことになる。一般的に、レンダリングパイプラインはブロックに区分けされているだろう。あるモジュールにフォワードやディファード、クールな某といったレンダラがあったり、他のモジュールにポストプロセスパスが散財していたり、再投影ステップのコピペが持ち込まれていたりして、あちらこちらに新しいテクニックを追加するので、同期戦略を練り直さなければならなくなる。そうして、モノが腐っていく。

なぜこんなことが起こるのか？

死ぬほど単純なポストプロセスパスの疑似コードをいくつか書いて考えてみよう。

~~~c
// 私が最後にこのイメージから読み出したのはいつだったか？たしかポストチェインの後の最後のフレーム…
// write-after-readハザードを回避したい。
// イメージ全体を書き込もうとしている。
// そうであれば、UNDEFINEDから以前のコンテンツの"破棄"へ遷移したほうがよさそうだ…
// 理想では前のフレームからVkEventの追跡をきちんと行っていたいが、さすがに面倒だ…
// このレンダターゲットはどこから割り当てられた？
BeginRenderPass(RT = BloomTheresholdBuffer)

// このイメージはおそらく前のフレームで書き込まれているだろうが、誰も知らないだろう。
BindTexture(HDR)

DrawMyQuad()
EndRenderPass()
~~~

この種の問題は一般的には大きく幅のあるパイプラインバリアで解決される。パイプラインバリアは**グローバルな**同期問題について**局所的に**推理させるが、常に最適な方法とはならない。

~~~c
// 安全になるようにすべてのフラグメントの実行が完了するのを待つために、これはwrite-after-readとHDRレンダパスの面倒を見ていることになる…
// 非同期コンピュートで使われないと仮定すれば…ふむ、これならうまく行きそうだ。

PipelineBarrier(FRAGMENT -> FRAGMENT,
    RT layout: UNDEFINED -> COLOR_ATTACHMENT_OPTIMAL,
    RT srcAccess: 0 (write-after-read)
    RT dstAccess: COLOR_ATTACHMENT_WRITE_BIT,
    HDR layout: COLOR_ATTACHMENT_OPTIMAL -> SHADER_READ_ONLY,
    HDR srcAccess: COLOR_ATTACHMENT_WRITE_BIT,
    HDR dstAccess: SHADER_READ_BIT)

BeginRenderPass(...)
~~~

そうして、我々はそれが以前のパスであると仮定したのでHDRイメージを遷移したが、今後異なるパスを遷移の間に追加するかもしれない。…つまり、依然としてイメージレイアウトの追跡を続ける必要がある。まぁ、世界の終わりでもなければ。

`FRAGMENT -> FRAGMENT`のワークロードを扱うだけだとすると、これはそれほどひどいことではなく、なんだかんだ言ってレンダパス間で起こるオーバーラップはそれほどない。コンピュートをごちゃまぜにし始めると、頭がどうにかなりそうになる。なぜなら、このようなパイプラインバリアをあちこちにポンポンと置いていくことは**できない**ので、効率的な実行オーバーラップを達成するためにフレームについての非局所的な知識が必要になる。加えて、異なるキューで非同期コンピュートを行うため、セマフォが必要になることもあるかもしれない。

# Render graph implementation

*主に[render_graph.hpp](https://github.com/Themaister/Granite/blob/master/renderer/render_graph.hpp)と[render_graph.cpp](https://github.com/Themaister/Granite/blob/master/renderer/render_graph.cpp)を参照している。*

*注記:これは巨大なbrain dump(知識の吐き出し)である。順番にモノを見てゆくので、これを読みながら一緒にコード追ってみてください。*

*注意2:実装では用語として"フラッシュ(flush)"と"無効化(invalidate)"を用いている。これはVulkan仕様の専門用語ではない。Vulkanではそれぞれ"make available"と"make visible"を使っている。フラッシュはキャッシュのフラッシュを指し、無効化はキャッシュの無効化を指す。*

基本のアイデアは"大局的な"レンダグラフを持つことである。モノをレンダリングする必要があるシステムの全要素はこのレンダグラフに登録する必要がある。我々はどのパスがあり、どのリソースが参加し、どのリソースが書き込まれるか、などなどを指定する。これは、アプリケーションのスタートアップに一度か、毎フレームに一度か、でなければ必要な時に行われることがある。主なアイデアはフレーム全体の大局的な知識を形作り、より高いレベルでそれに応じた最適化を可能にすることである。モジュールは、バックエンドAPIが自動でスケジューリングせずに依存性を扱うときに直面する主要な問題を解決するため、全体像を見ることを可能にしつつそれらの入力と出力について局所的に推論することができる。レンダグラフはバリア、レイアウト遷移、セマフォ、スケジューリングなどの面倒を見ることができる。

レンダパスからの出力はある次元を必要とするが、かなり素直である。

イメージ。
```cpp
struct AttachmentInfo {
    SizeClass size_class = SizeClass::SwapchainRelative;
    float size_x = 1.0f;
    float size_y = 1.0f;
    VkFormat format = VK_FORMAT_UNDEFINED;
    std::string size_relative_name;
    unsigned samples = 1;
    unsigned levels = 1;
    unsigned layers = 1;
    bool persistent = true;
};
```

バッファ。
```cpp
struct BufferInfo {
    VkDeviceSize size = 0;
    VkBufferUsageFlags usage = 0;
    bool persistent = true;
};
```

そして、これらのリソースがレンダパスに追加される。
```cpp
// ディファードレンダラのセットアップ

AttachmentInfo emissive, albedo, normal, pbr, depth; // デフォルトはスワップチェーンの大きさ
emissive.format = VK_FORMAT_B10G11R11_UFLOAT_PACK32;
albedo.format = VK_FORMAT_R8G8B8A8_SRGB;
normal.format = VK_FORMAT_A2B10G10R10_UNORM_PACK32;
pbr.format = VK_FORMAT_R8G8_UNORM;
depth.format = device.get_default_depth_stencil_format();

auto& gbuffer = graph.add_pass("gbuffer", VK_PIPELINE_STAGE_ALL_GRAPHICS_BIT);
gbuffer.add_color_output("emissive", emissive);
gbuffer.add_color_output("albedo", albedo);
gbuffer.add_color_output("normal", normal);
gbuffer.add_color_output("pbr", pbr);
gbuffer.set_depth_stencil_output("depth", depth);

auto& lighting = graph.add_pass("lighting", VK_PIPELINE_STAGE_ALL_GRAPHICS_BIT);
lighting.add_color_output("HDR", emissive, "emissive");
lighting.add_attachment_input("albedo");
lighting.add_attachment_input("normal");
lighting.add_attachment_input("pbr"));
lighting.add_attachment_input("depth");
lighting.set_depth_stencil_input("depth");

lighting.add_texture_input("shadow-main"); // 外部の依存性
lighting.add_texture_input("shadow-near");
```

ここではリソースをレンダパスで使えるようする方法が3つ提示されている。

- Write-only。リソースは完全に書き込まれる。`loadOp`は`CLEAR`か`DONT_CARE`。
- Read-write。inputを保存して、その上に書き込む。`loadOp`は`LOAD`。
- Read-only。

コンピュートでも似たような話で、非同期コンピュートで行う適応的な輝度更新処理は以下のように記述する。

```cpp
auto& adapt_pass = graph.add_pass("adapt-luminance", VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT);
adapt_pass.add_storage_output("average-luminance-updated", buffer_info, "average-luminance");
adapt_pass.add_texture_input("bloom-downsample-3");
```

例えば、ここで輝度バッファはRMW(Read-Modify-Write)を得る。

実際の処理を行うために毎フレーム呼ばれる可能性があるコールバックも必要である。G-Bufferでは以下のようになる。

```cpp
gbuffer.set_build_render_pass([this, type](Vulkan::CommandBuffer& cmd) {
    render_main_pass(cmd, cam.get_projection(), cam.get_view());
});

gbuffer.set_get_clear_depth_stencil([](VkClearDepthStencilValue* value) -> bool {
if (value) {
    value->depth = 1.0f;
    value->stencil = 0;
}
return true; // CLEARかDONT_CAREか？
});

gbuffer.set_get_clear_color([](unsigned int render_target_index, VkClearColorValue* value) -> bool {
    if (value) {
        value->float32[0] = 0.0f;
        value->float32[1] = 0.0f;
        value->float32[2] = 0.0f;
        value->float32[3] = 0.0f;
    }
    return true; // CLEARかDONT_CAREか？
});
```

レンダグラフは、リソースを割り当て、これらのコールバックを操って、最終的に適切な順序でGPUにサブミットする責任を持つ。このグラフを終わらせるには、特定のリソースを"バックバッファ"に昇格させる。

```cpp
// これはアドホックなデバッグにとても使いやすい。:)
const char* backbuffer_source = getenv("GRANITE_SURFACE");
graph.set_backbuffer_source(backbuffer_source ? backbuffer_source : "tonemapped");
```

では、実際の実装に入っていこう。

# Time to bake!

いったん構造をセットアップすれば、レンダグラフをベイクする必要がある。これは一連の手順を進み、各々がパズルのピースひとつひとつを埋めてゆく…

## Validate

とても素直で迅速な、レンダパス構造中のデータが理にかなっていることを保証するための正気度チェックである。

ここで面白いことのひとつは、我々が色入力の次元が色出力と一致しているかをチェックできることである。これらが異なれば、そのまま`loadOp = LOAD`をせずに、レンダパスの開始時にスケーリングしたblitを行うことができる。これは、内部解像度のアップスケーリングを行うようなときにとても便利である。このときの`loadOp`は`DONT_CARE`になる。

### Traverse dependency graph

我々はレンダパスの非循環グラフを持つ。これはつまりレンダパスの配列に平坦化する必要がある。我々が作るリストは全てのパスを順々にサブミットすることであった場合に妥当なサブミッション順となる。このサブミッション順は最大限最適化されていないかもしれないが、あとで近づかせる。

ここでのアルゴリズムは素直である。我々はボトムアップにツリーを走査する。再帰を使って、バックバッファに書き込む全てのパスのインデックスをプッシュして、そのらのパスすべてに対して、リソースに対する書き込みをプッシュして…最上位のリーフに達するまで続ける。こうして、パスAがパスBに依存する場合、パスBは常にリスト中でパスAの後に見つかるようになることを保証する。そして、リストを遡って、重複分を取り除く。

パスが他のパスと良い"マージ候補"であるかも登録する。例えば、ライティングパスはGバッファパスからの入力アタッチメントを使い、いくつかのカラー/デプスアタッチメントを共有する…タイルベースのアーキテクチャでは、Vulkanのマルチパス機能を使ったメインメモリを取らずに、実際にこれらのパスをマージする事ができる。なので、後にやるパスの並べ替えのためにこれを気に留めておく。

## Render pass reordering

これはこのプロセスの第1の面白ステップである。理想としては、パス間が最適にオーバーラップしたサブミッション順が欲しい。パスAがあるデータに書き込み、パスBがそれを読み出す場合、"ハードバリア"の数を最小化するためAとBの間に最大限のパスが欲しい。これは最適化のメトリック(距離)となる。

実装されたアルゴリズムはおそらくCPU時間に関して言えばそれほど最適ではないが、仕事は果たしている。まだスケジューリングされていないパスのリストを調べ、以下の3つの判断基準に基づく最適なひとつを見つけ出そうとする。

- 依存性グラフ走査ステップより前に決定されたマージ候補があるか？(スコア:無限)
- すでにスケジューリングされた待つ必要があるパスのリストの中で最後のパスは何か？(スコア:その間にオーバーラップする可能性のあるパス数)
- このパスをスケジューリングすると依存関係が壊れるか？(そうなら、このパスをスキップする)

コードを読むほうがおそらくもっと為になるので、`RenderGraph::reorder_passes()`を参照のこと。

含めるべきもうひとつのsneakyな検討事項は、ライティングパスがいくつかのリソースに依存するが、Gバッファパスがそうでないときである。これは、以下のスケジューリングプロセスを通るため、サブパスのマージ処理を破壊する可能性がある。

- Gバッファパスでスケジューリングされ、依存性を持たない。
- ライティングパスでスケジューリングを試みるが、依存しているシャドウパスをまだスケジューリングしてなかった。

これの泥臭い解決法はマージ候補からマージチェインの第1パスに依存性を引き上げることであった。すると、Gバッファパスはシャドウパスの後にスケジューリングされ、すべてがうまく行く。もっと賢いスケジューリングアルゴリズムはここで助けになるだろうが、私はできるだけ単純にしておきたいと考えている。

# Logical-to-physical resource assignment

TODO

# References
