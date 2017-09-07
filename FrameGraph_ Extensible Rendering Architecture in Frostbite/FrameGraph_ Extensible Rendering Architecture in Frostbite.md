---
title: 'FrameGraph: Extensible Rendering Architecture in Frostbite [@ODonnell2017]'
bibliography: bibliography.bib
numberSections: false
---

# 2007年と2017年におけるFrostbiteの比較(Frostbite 2007 vs 20017)

- 2007年
    - DICEの次世代エンジン。
    - 以下を下地に作られた。
        - Xbox 360
        - PlayStation 3
        - マルチコアPC
        - シェーダモデル3.0のDirectX9とDirect3D 10
    - 今後のDICEのゲームで使うために。
- 2017年
    - The EAのエンジン。
    - 以下のために発展(evolve)・拡大(scale up)した。
        - Xbox One
        - PlayStation 4
        - マルチコアPC
        - DirectX 12
    - 約15のEAのゲームで使われる。
        - もうバトルフィールドのエンジンではない。
        - 使われるジャンルはRPG、レース、スポーツ、アクションなど多岐にわたる。

# 2007年のレンダリングシステムのあらすじ(Rendering system overview '07)

![](assets/rendering system overview 2007.png)

# 2017年のレンダリングシステムのあらすじ(Rendering system overview '17)

![](assets/rendering system overview 2017.png)

- このグラフはエンジンを完璧にあらわしているわけではない。システムの広がりを図式化しただけ。
- 複雑に絡み合ったシステムが無数に存在する以外は基本的に同じ。
- このトークではWorld Rendererとレンダリング機能についての話をする。

# レンダリングシステムのあらすじ(簡略化バージョン)(Rendering system overview(simplified))

![](assets/rendering system overview simplified.png)

# WorldRenderer

- すべてのレンダリングを指揮する。
    - **コード駆動**アーキテクチャ。
    - メインのワールドジオメトリ(Shadeing System経由)。
    - ライティング、ポストプロセス(Render Context経由)。
    - すべてのビューとレンダパスについて知っている。
- 設定とリソースのシステム間マーシャリング(表現の変換)を行う。
- リソース(レンダターゲットやバッファ)の割り当てを行う。

# バトルフィールド4のレンダリングパス(Battlefield 4 rendering passes (Features))

![](assets/battlefield 4 rendering passes.png)

- 上記は数年前にバトルフィールド4用にFrostbiteに蓄えたレンダリングパスである。
- 現在のパイプラインはPBR化されており、パスはさらに増えて複雑になっている。

# WorldRendererの課題(WorldRenderer challenges)

- 明示的な即時モードレンダリング。
- 明示的なリソース管理。
    - オーダーメイドによる職人技的な手作りのESRAM管理。(Bespoke, artisanal hand-crafted)
    - 様々なゲームチームによる複数の実装。
- レンダリングシステムとの強固な結合。
- 制限された拡張性。
- カスタマイズのためにフォーク/分岐が必須。
- 4k SLOCから15k SLOCへの組織的な成長。
    - 2k SLOC超の関数。
    - 維持、拡張、統合が高価。

# モジュール式WorldRendererの目標(Modular WorldRenderer goals)

- フレーム全体の高レベルな知識を得る。
- **拡張性**を改善する。
    - 分離された構成可能なコードモジュール。
    - 自動的なリソース管理。
- よりより可視化と診断。

- 蓄積した技術的負債の対処と拡張性と保守性の改善を行うため2016年に主要な部分を再構築した。
- 明示的なパスとリソースのミクロな管理なし。
- エンジンコード内のモノリシックな関数のハックなし。
- メモリ割り当てと別名付け(aliasing)の子守(baby-sitting)なし。

# 新アーキテクチャの要素(New architectural components)

![](assets/new architectural components.png)

- **Frame Graph**
    - **レンダパス**と**リソース**の高レベル表現。
    - フレームの完全な知識。
- **Transient Resource System**
    - リソース割り当て。
    - メモリ別名付け。

# Frame Graphの目標(Frame Graph goals)

TODO

# References
