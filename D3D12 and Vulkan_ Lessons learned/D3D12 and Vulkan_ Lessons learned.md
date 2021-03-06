---
title: 'D3D12 and Vulkan: Lessons learned [@Chajdas2016]'
numberSections: false
bibliography: bibliography.bib
---
# バリア制御(Barrier control)

- バリアはD3D12/Vulkan世代の新しいコンセプト。
- 残念ながら、**みんな** 間違っている。
- 2つの失敗ケース
    - 多すぎ(too many) or 広すぎ(too broad): **パフォーマンスの低下**
    - バリア欠落: **データ破壊(corruptions)**
- D3D11ドライバはその傘の下でうまくこれをこなしている。

# つまるところのバリアとは？(What's a barrier, anyway?)

- レンダターゲットをテクスチャにするとき。
    - decompressionが必要になるかもしれない。

- UAVをリソースにするとき。
    - 誤って行えば、コストがかかる。 -- flushやアイドル待ち
    - 正しく行えば、トランジションが実質タダに。

# バリアが欠落していると(Missing barriers)

- フォーマット問題。 -- GPUやドライバ特有のcorruption。
- 同期問題。 -- 時間依存のcorruption。

# サブリソース(Subresources)

- 個別に追跡する必要がある。
    - ダウンサンプリング。
    - シャドウマップアトラス。
- すべてのサブリソースをトランジションするなら、１つ１つやらずに`D3D12_RECOURCE_BARRIER_ALL_SUBRESOURCES`を使う。

# Placedリソースと初期ステート(Placed resoruces & initial states)

- placedリソースなどとして作成したレンダターゲットは使う前に **必ず** クリアする。
- Go into **clear state directly**, ランダムなステートやトランジションで始めない。

# 不必要なトランジション(Unnecessary transitions)

- 間違った型へのトランジション。
    - 普通はないが、時折起こる。
    - バリデーションレイヤで必ず確認する。

- 読み込みから読み込みへのトランジション。
    - 例えば、インデックスバッファからシェーダリソースへ。
    - 今後に必要なすべてのステートにまとめる。
        - UAV -> IB -> SRVなら、UAV -> IB|SRVにまとめる。

# 高価なトランジション(Costly transitions)

- `COMMON`はコピー/表示のためであり、一般的な"全部入り"ステートではない。
- 通常にシェーダアクセスがしたいなら、
    - D3D12: `PS_RESOURCE | NON_PS_RESOURCE`
    - Vulkan: `VK_ACCESS_SHADER_READ_BIT`

# バリア制御 - 最悪のケース#1(Barrier control - Worst case #1)

- 最悪のケースのバリアシステム - バリア多すぎ。
    - マテリアルシステムが間違いを犯す。
    - ステージ毎に行うと、最大ダメージ。

- "late binding"かドロー毎リソースを固定する。

~~~ {.c .numberLines}
for (stage in stages)
    for (resource in resources)
        RsourceBarrier(1, &resource.Barrier)
~~~

- 理想的なフロー: 書き込み -> バリア -> ドロー -> ドロー -> ドロー -> ドロー -> ドロー
- マテリアル/ステージ毎アンチパターン
    - ステージ毎リソース毎にバリア1つ。
    - バリアがコマンドリストのあらゆる場所に散らばる。
    - 書き込み -> バリア -> バリア -> バリア -> バリア -> ドロー -> バリア -> バリア -> バリア -> ドロー -> バリア -> ...
- 最悪のケースでは、ことあるごとに複数のアイドル待機が起こる。

# バリア制御 - 最悪のケース#2(Barrier control - Worst case #2)

- "Base state"、または、冗長なトランジション。
- リストアに続きターゲットステートに遷移する。

# おもしろバリア(Funny barriers)

- ResourceBarrier(0, nullptr)
    - なにもしない。
    - ステートのトラッキングが間違ったことをしていることを示す。
- 以前のステートが次のステートと同じ。
    - 考えている以上のことが起こる、ことはない。
- ドライバは**あなたが最適なことをやってくれると思っている**し、**いかなる** 試行錯誤(heuristic)も経ないことを常に念頭に置く。

# 未来のために準備する(Get ready for the future)

- すべてのリソースステートを追跡しなければならない**わけではない**。
- 99%のリソースはイミュータブル。
- "トランジション"ポイント--いつパスが終わるかを見つける。
    - バッチバリアはここ。
    - 必要なトランジションのみ。
    - Gバッファ -> シャドウマップ -バッチトランジション-> シェーディング -バッチトランジション-> ポストエフェクト -バッチトランジション->

# バリアのデバッギングtips

- リード/ライトのビットを持つ。
- すべてのトランジションを記録する。
    - grepやスプレッドシートはともだち。
    - トランジション数やタイプなどをチェックする。
- トランジション数は書き込み可能なリソースの数の順にすべき。
    - 9000以上だったら、何かがおかしい。

- 全部バリアするモードを持つ。
    - 最悪のケースと同じような状態。
    - デバッグ用。
- リソースがフレーム毎に少なくとも1度は既知の状態にあることを保証する。
    - 例えば、フレームの終わりや始まり。
    - TAAやシャドウアトラスの破損のような問題を解決するため、すべてを既知のステートに遷移する。

# その先へ(Going forward)

- より良い、イベント的に。
    - トランジションを扱うドライバ時間を与える。
    - D3D12では"Split barrier"
    - vkCmdSetEvent + vkCmdWaitEvents

# まとめ: バリア

- 必要とするリソースすべてをトランジションすることを確かめる。(それ以上はしていないことも確かめる。)
- できる限り最大限特有のステートに移行する。
- 様々なステートを組み合わせることができることを忘れない。

# 発射の制御

- GPUに供給する方法
    - まずなによりも、コマンドリストのサブミット。
    - 次点で、フレーム毎リソースの更新と追跡。

# CPUスレッディング

- 手動によるコア割り当てることで並列化を制限しない。
- タスク/ジョブシステムを使う。
    - すべてのコアを自動的に使ってくれる。
    - 効率的な処理サブミッションとリソース同期のために特別な注意が必要になる。

# 何が起こった？

- スレッドプールが暴走した。
    - CPUタスクは終わりにワークをサブミットした。
    - タスクの境界がCPU/GPU同期ポイントになった。
- タスクが完了したあとにコマンドリストを制御しよう。

# 何が起こった？

- 各フェンスは(多かれ少なかれ)基本的にGPUでアイドル待ちになる。
- ベター:
    - フレーム毎リソースを保護する。
    - "フレーム中間"でコマンドリストの処理を開始できることはまずない。
    - １つのフェンスで多くのリソースを保護する。
- ジョブシステムがこれをできることを確認する。
- できるだけ多くのサブミッションをバッチ処理する。
- いつでもGPUがビジーになるように早めにサブミットする。

# コマンドアロケータ

- コマンドアロケータは"成長のみ"行うよう定義されている。
    - 新しいアロケータに100ドローコールを記録したら、メモリ割り当てが発生すると思われる。
    - リセットして、同じだけのドローコールを記録してもメモリ割り当ては発生しないと思われる。
    - 似たような仕事量のものでコマンドアロケータを再利用してみる。
- アロケータのリサイクルは最悪ケースのサイズに成長させる。
- アロケータ数はおおまかに、スレッド数xバッファするフレーム数xGPU数。
- アロケータ/コマンドリストを再利用し、フレームごとに再生成していないことを確認する。

# それと: レンダパス

- ハイレベルなフレームのグラフを作る。
- Vulkanのレンダパスとサブパスを経由してレンダラのそれを伝える。
- ドライバが最適なスケジュールを選択できるようにする。

- "気にしない"ことを上手に説明できるようにする。
- より詳しい話は"Vulkan Fast Paths"トークを参照。

# デバッギングヒント

- 一回のサブミッションですべてのコマンドリストをサブミットするオプションを入れる。
    - タイミングの問題に役立つ。
    - できそうにないなら、フレーム内GPU/CPU同期を作ろう。:(
- いずれかのコマンドリストを待つオプションを入れる。
    - アップロードやリソース同期の助けになる。
    - リソースが壊れたときは、アップロードまえにフラッシュする。

# まとめ: サブミッション

- フレーム毎の粒度でリソースを追跡する。
- フレーム構造を知る。
- 良いCPU使用率を得るためにスレッディングは必須。

# マルチキュー

- D3D12とVulkanは複数のキュータイプがある: コピー、グラフィクス、コンピュート。
    - Vulkanでは、どのキューがどれだけ使えるかを確認する。
    - D3D12では、各種1つずつが利用可能であることが保証されている --- が、スケジューリング保証はない。
- コンピュートキューにはよい使用例がたくさんある。
- コピーキューはそれほど使われていない。 --- もっとできることがあるだろう。

# グラフィクスとコンピュート

- これまでの非同期コンピュートの成果は素晴らしい。
- グラフィクスキューがアイドルの間コンピュート処理を走らせる。
- 1つのコンピュートコマンドリストが1つのフェンスで同期しつつ並列に動作するのはよく見る。
    - それは結構。
    - もっとコンピュートすれば、もっと良くなる。

# 非同期コンピュート

- Pit of success
    - ボトルネックが異なるなら、GPU使用率が最大化する。(例:シャドウマップ(帯域)とSSAO(ALU))
- Pit of no success
    - リソースが競合していると、順次処理したときよりもひどくなる可能性がある。
- Pit of even more success
    - フレームが異なる処理でもオーバーラップさせる。(例:前フレームのポストプロセスと現フレームのGバッファ構築)

# コピーは助けになるのか

- コピーキューは低レイテンシーで低速度だが、ハードウェアが分けられている。
    - GPUローカルコピーではなくPCIe上での転送に最適化されている。
    - PCIeはデータ転送にて最速。
    - 数フレームにかけてデータをストリーミングするようなものが理想的な使い方。
- これまではあまり使われてこなかった。
    - 何故なのか教えて。
    - アダプタ間コピーではコピーキューがベスト。--- 共有スワップチェインの検討もそうだが。

# まとめ: マルチキュー

- GPUを満たすためコンピュートキューを使おう。
- PCIeを飽和させるためコピーキューを使おう。
- 非同期処理をスケジュールする最適な位置を見つけるためフレーム構造を知ろう。

# リソース

- 往々にしてことはうまく行く。
    - アップロードはほとんど問題を起こさないが、コピーキューを確認すること。
    - GPU上の管理はほぼほぼ大丈夫。
    - パッキングは可能な限りタイトにならないときがある。アライメントを確認しよう。
- フレームバッファのような"頻度の高い"リソースはD3D12だと`CreateCommittedResource`を推奨。
- residencyやbudgetによる多くの問題。
    - Dave OldcornとStephan Hodesのトーク"Right on Queue --- Advanced DirectX12 programming"を見よう。
    - たちの悪いトピック。--- ここではカバーしきれない。

# デバッグランタイム＆バリデーションレイヤ

- D3D12とVulkanにはバリデーションレイヤがある。
- ドライバはパフォーマンスによる問題は検証**しない**。
- アプリケーションが**完璧である**ことを前提としている。
- 開発中では、バリデーションで警告やエラーが出ていないことを確かめる。
    - アプリケーションがバリデーションをサポートしていないなら、今すぐサポートを追加しよう。
    - どんなundefined behaviorでもあなたを悩ませる。Vulkanでは特に。
- 仕様の弁護人にならないで。--- 何かが不明瞭だったり疑問だったりときはIHV(独立系ハードウェアベンダー)パートナーに相談してみて。
    - 仕様やバリデーションレイヤは日々進歩している。
    - あらゆるコーナーケースが完全に理解されているわけではない。

# 参考文献
