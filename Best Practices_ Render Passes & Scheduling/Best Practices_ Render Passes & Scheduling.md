---
title: >
    Best Practices: Render Passes & Scheduling [@Hector2016]
bibliography: bibliography.bib
numberSections: false
---
# What is a Render Pass?

- Vulkanのユニークな機能。
    - 複数のパスが効率的にスケジュールされることを許可する。
    - タイルベースGPUが処理すべき方法を明示的に呼び出す。
- すべてのGPUに渡る利点。
    - すべてのGPUでスケジューリングに対する利点がある。
    - タイルベースGPLで帯域やメモリを節約できる。
- 移植性のための巨大なenabler。
    - すべてのベンダで例えばDeferred Shadingを行う最善の方法。
    - ベンダ特有のエクステンションを必要としない。(例えば、Pixel Local Storage)

# Efficient Scheduling

- スケジューリングは複雑である。
    - 以前のプレゼンテーションを参照。([video](https://www.youtube.com/watch?v=iZ3J25qsacA))
    - 物事が起こる必要があるとき、正確に考える必要がある。
- 効率的にスケジューリングを行うことは未来についての知識を持つことを意味する。
    - 同期プリミティブは現在と過去を説明する。
    - **非常に**注意深いアプリケーション管理が必要になる。

# Render pass dependencies

- レンダパスは未来の処理を記述する。
    - サブパス間の依存関係。
    - サブパス間の暗黙的な順序は存在しない。
- ドライバはこれらの構造をコンパイルできる。
    - 最適化された依存関係グラフを構築できる。
    - 未来の処理を極めて効率的にスケジューリングできる。
    - @Sellers2016a のトーク。
- レンダパスインスタンスはこのグラフを使う。
    - 描画コマンドを実行するフレームワークとして機能する。

# Additional benefits --- As if that wasn't enough...

- タイルベースGPUは追加のブーストを得る。
    - サブパスをマージできる --- オンチップでGバッファのようなデータを完全に保持しているため。
    - 帯域が必要なくなる！
    - いくつかの直接レンダラはキャッシュのflushを回避できるかもしれない。
    - GB/s単位の節約。
- もしRAMから読み書きする必要がなければ…
    - そもそもアタッチメントの割り当てすらしなくて良くなる。
    - 高解像度で大幅にメモリを節約できる。
    - 例: 1080pのRGBA8アタッチメントひとつで最大8MB。

# Best Practices

- できるだけ少ないレンダパスにできるだけ詰め込む。
    - お互いに依存しないパスも！
    - 例: 複数のシャドウマップ生成パス。
    - ほとんどのアプリケーションはひとつかふたつを必要とするべきである。
- サブパスの依存関係を使う。
    - バリアやイベントの代わりに。
- `initialLayout`と`finalLayout`を使う。
    - 明示的なイメージ遷移の代わりに。

- `loadOp`と`StoreOp`を使う。
    - 気前よく`DONT_CARE`を使う。
    - `vkCmdClearAttachment`や`vkCmdClearImage`の代わりに`CLEAR`を使う。
- MSAAの`ResolveAttachments`を使う。
    - `vkCmdResolveImage`の代わりに。
- `TRANSIENT_ATTACHMENT_BIT`や`LAZILY_ALLOCATED_MEMORY`を使う。
    - いくつかのアーキテクチャではメモリを割り当てる必要がない！

# Conclusion

- レンダパスはすごい。
    - よりすごいことができるように続けてゆく。
- 絶対にこれらを使うべき。
    - 怖くないし、難しくない。約束する。
    - (まぁ、Vulkan以上にはならんでしょ…)
- 質問があれば、質問してね！
    - パネルの間でも、そのあとでも。
    - お気軽にどうぞ。
    - ツイッターでも: \@TobskiHectov

# References
