---
title: CRYENGINE3 Graphics Gems [@Sousa2013]
bibliography: bibliography.bib
numberSections: false
---
# アンチエイリアシング \\ ディファードMSAAのレビュー[ANTIALIASING \\ DEFERRED MSAA REVIEW]

- 問題: マルチパス＋マルチサンプルRTからの読み書き
    - DX10.1で`SV_SampleIndex`と`SV_Coverage`が導入された。
    - ピクセル/サンプル頻度パスに対してマルチパス経由で解決できる[@Thibieroz2011]。
- `SV_SampleIndex`
    - サブサンプル *ごとに* ピクセルシェーダ実行を強制して、現時点で実行されたサブサンプルのインデックスを提供する。
    - インデックスはマルチサンプルRTからサブサンプルをフェッチするのに使える。例:`FooMS.Load(UnnormScreenCoord, nSampleIndex)`
- `SV_Coverage`
    - ラスタステージ中にどのサブサンプルがカバーしていたかをピクセルシェーダに示す。
    - カスタムカバレッジマスクでサブサンプルのカバレッジを修正できる。
- DX11.0のコンピュートタイルベースディファードシェーディング/ライティングのMSAAはより単純になる。
    - サブサンプルでタグ付けされたMSAAを通したループ[Loop through MSAA tagged sub-samples]

# ディファードMSAA \\ 用心せよ！[DEFERRED MSAA \\ HEADS UP!]

- 単純な理論、面倒な実践
    - 少なくとも複雑なディファードレンダラを伴う
- 非MSAAフレンドリーなコードは高速に蓄積する。
    - MSAAを考慮しないで新しいテクニックが加わったため、定期的に崩れる。
    - まだイケるとしても…これらはビジュアルアーティファクトを引き起こすので、頻繁に非MSAAフレンドリーなテクニックを特定して修正する必要があるだろう。
    - 例えば、白/黒のアウトライン、または、AAまったくなし。
- 前もって行おう。ディファードMSAAをサポートするためのレンダラを改良することは[some work]である。
    - そして、とても[finiky]である。

# ディファードMSAA \\ カスタム解決＆サンプル毎マスク[DEFERRED MSAA \\ CUSTOM RESOLVE & PER-SAMPLE MASK]

- Gバッファの後にはカスタムMSAA解決を処理する。
    - ライティング/他のMSAA依存パスのようなピクセル頻度パスに対して、サンプル0を事前解決する。
    - 同じパスで、サブサンプルマスクを生成する(サンプルの類似性を比較し、不一致ならばマークする)。
        - MSAAを必要としない領域を無駄に処理してしまうことになるので、デフォルトの`SV_Coverage`を避ける。

# ディファードMSAA \\ ステンシルバッチング[@Sousa2013][DEFERRED MSAA \\ STENCIL BATCHING [SOUSA13]]

- 通常のステンシルバッファの使い方でサンプル毎ステンシルマスクをバッチ処理する。
    - ステンシルバッファから1ビットを予約する。
    - サブサンプルマスクで更新する。
    - 単一のピクセルだけではなくピクセルクアッド[pixel-quad]全体をタグ付けする。→ステンシルカリング効率が改善する。
    - サンプル毎ビットのオーバーライドを避けるためにステンシル読み/書きビットマスクを使う。
        - `StencilWriteMask = 0x7f`
    - ステンシルクリアが起こるたび復元する。
- 極端にステンシルを使うため可能ではない？
    - clip/discardを使う。
    - サンプル毎マスクに対する追加のテクスチャ読み込みからの余分なオーバーヘッドもある。

# ディファードMSAA \\ ピクセルおよびサンプル頻度パス[DEFERRED MSAA \\ PIXEL AND SAMPLE FREQUENCY PASSES]

- ピクセル頻度パス
    - *ピクセル毎* 領域に対する予約済みビットにステンシル読み込みマスクをセットする(`~0x80`)。
    - 解決前(非マルチサンプル)ターゲットSRVをバインドする。
    - 普段通りのレンダパス
- サンプル頻度パス
    - *サンプル毎* 領域に対する予約済みビットにステンシル読み込みマスクをセットする(`0x80`)。
    - マルチサンプルターゲットSRVをバインドする。
    - `SV_SampleIndex`経由で現在のサンプルをインデックス付けする。
    - 普段通りのレンダパス

# ディファードMSAA \\ アルファテストSSAA[DEFERRED MSAA \\ ALPHA TEST SSAA]

- アルファテストにはアドホックな解決策が必須となる。
    - デフォルトの`SV_Coverage`はトライアングルのエッジにのみ適用する。
- あなた自身のサブサンプルカバレッジマスクを生成する。
    - 例えば、現在のサブサンプルがアルファテストを使うかどうかを確認してビットをセットする。

```hlsl
static const float2 vMSAAOffsets[2] = {float2(0.25, 0.25), float2(-0.25, -0.25)};
const float2 vDDX = ddx(vTexCoord.xy);
const float2 vDDY = ddy(vTexCoord.xy);
[unroll] for (int s = 0; s < nSampleCount; ++s) {
    float2 vTexOffset = vMSAAOffsets[s].x * vDDX + vMSAAOffsets[s].y * vDDY;
    float fAlpha = tex2D(DiffuseSmp, vTexCoord + vTexOffset).w;
    uCoverageMask |= ((fAlpha - fAlphaRef) >= 0) ? (uint(0x1) << i) : 0;
}
```

# ディファードMSAA \\ パフォーマンスショートカット[DEFERRED MSAA \\ PERFORMANCE SHORTCUTS]

- ディファードカスケード太陽シャドウマップ
    - ピクセル頻度で通常はシャドウをレンダリングする。
    - ディファードシェーディングの合成パス中にバイラテラルアップスケールする。

# ディファードMSAA \\ パフォーマンスショートカット (2)[DEFERRED MSAA \\ PERFORMANCE SHORTCUTS (2)]

- 深度にアクセスする不透明でないテクニック(例、ソフトパーティクル)
    - サンプル毎頻度を経由して対処するオススメは現実世界のシナリオではかなり低速である。
    - 最大深度を使うとほとんどのケースでちゃんと動作し、N倍速い。

# ディファードMSAA \\ パフォーマンスショートカット (3)[DEFERRED MSAA \\ PERFORMANCE SHORTCUTS (3)]

- 多くのゲームは以下も行っている。
    - アルファテストスーパーサンプリングをスキップする。
        - 代わりにAlpha to Coverageを使う、または、アルファテストAAを使わない。
    - 不透明のみをMSAAありでレンダリングする。
        - その後、MSAAなしで透明をレンダリングする。
        - HDRレンダリングを仮定すると: トーンマッピングは暗黙的に行われ、解決後の結果はハイコントラストの領域で詳細が失われることに注意する。

# ディファードMSAA \\ MSAAの馴染みやすさ[DEFERRED MSAA \\ MSAA FRIENDLINESS]

- これらに注意しよう。
    - MSAAが目立って働かない、または、明るい/暗いシルエットが目立つ。

# ディファードMSAA \\ 要約[DEFERRED MSAA \\ RECAP]

- マルチサンプルRTにアクセスする、および/または、レンダリングする？
    - その後、正しいサブサンプルに対するアクセスと出力について注意する必要がある。
- 一般に、帯域幅を最小化するよう常に努力する。
    - そのままのディファードライティングを避ける。
        - 完全なディファード、ハイブリッド、ディファードをまるまるスキップ、などを推奨する。
    - ディファードの場合、薄いGバッファを推奨する。
        - Gバッファに対する追加の各ターゲットはエクスポートレートのオーバーヘッドを負う[@Thibieroz2011]。
        - NV/AND(GCN): $ExportCost = \sum_i^N Cost(RT_i)$、AND(旧世代): $ExportCost = (RT数) \times (最遅RT)$
        - 大きな[fat]フォーマットはGCNにおいてバイリニアフィルタリングモードに対するサンプリングコストを半分のレートにする[@Thibieroz2013]。
        - ライティング/いくつかのHDRポストプロセスに対して: 殆どの場合、32ビットのR11G11B10Fで十分。

# アンチエイリアシング＋4K解像度 \\ 一体全体MSAAは必要なのだろうか？[ANTIALIASING + 4K RESOLUTIONS \\ WILL WE NEED MSAA AT ALL?]

- 恐らくここでクリエイティブを発揮し始めることができる。

# アンチエイリアシング \\ 優秀(で高速)なAAの探求[ANTIALIASING \\ THE QUEST FOR BETTER (AND FAST) AA]

- 2011年: 代替AAモード(と組み合わせに名前を付ける)ブームの年
    - FXAA、MLAA、SMAA、SRAA、DEAA、GBAA、DLAA、などのAA
    - "リアルタイムアンチエイリアシングのためのフィルタリングアプローチ"[@Jimenez2011]
- シェーディングアンチエイリアシング
    - "ミップマッピング法線マップ"[@Toksvig2004]
    - "壮観な[spectacular]スペキュラ: LEANとCLEANスペキュラハイライト"[@Baker2011]
    - "手堅い[rock-solid]シェーディング: ディテールを犠牲にしない画像安定化"[@Hill2012]

# テンポラルSSAA \\ SMAA 2TX/4Xのレビュー[@Jimenez2011; @Sousa2011][TEMPORAL SSAA \\ SMAA 2TX/4X REVIEW [JIMENEZ11][SOUSA11]]

- モーフォロジカルAA＋MSAA＋テンポラルSSAAの組み合わせ
    - コスト/クオリティのトレードオフの釣り合いが取られ、テクニックが互いに補い合う。
    - テンポラル要素は2つのサブピクセルバッファを使う。
    - 各フレームは2xSSAAのサブピクセルジッターを加える。
    - 前フレームを再投影し、速度の長さの重み付けを経由して現フレームと前フレームをブレンドする。
    - 画像のシャープネス＋合理的なテンポラル安定性を保持する。

$$
w = 0.5 \max \left( 0, 1 - K \sqrt{\left| \|v_c\| - \|v_p\| \right|} \right) \\
c = (1 - w) c_c + w c_p
$$

# テンポラルAA \\ 一般的なロバスト性の欠陥[TEMPORAL AA \\ COMMON ROBUSTNESS FLAWS]

- 不透明ジオメトリ情報に依存している。
    - シグナル(色)変化も透明も扱うことができない。
    - 正しい結果のため、すべての不透明ジオメトリは速度を出力しなければならない。
- 異常な[pathological]ケース
    - アルファブレンドされた表面(例えば、パーティクル)、ライティング/シャドウ/反射/UVアニメーション/など
    - AA解決前の、散乱や同様のポストプロセス
- 気の散るエラーになり得る
    - 例えば、透明、ライティング、シャドウの"ゴースト"
    - 散乱や同様のポストプロセス(例、ブルーム)によりシルエットが現れる。
- マルチGPU
    - 最も単純な解法: リソース同期を強制する。
    - NVIDIAはNVAPI経由でリソースを強制同期するためのドライバヒントを開示している。これはNVIDIAのTXAAで使われた解法である。
        - ハードウェアベンダへのメモ: すべてのベンダがこのようなものを開示すればそれは素晴らしいことだろう(機能的に一般化されたマルチGPUのAPIでも良い)。

# SMAA 1TX \\ よりロバストなテンポラルAA[SMAA 1TX \\ A MORE ROBUST TEMPORAL AA]

- コンセプト: シグナル変化のみを追跡して、ジオメトリ情報に依存しない。
    - より高い時間的安定性に対して: TXAAと同様に、複数フレームをアキュムレーションバッファに累積する[@Lottes2012]。
    - アキュムレーションバッファを再投影する。
    - 重み付け: アキュムレーションバッファの色を現フレームの近傍色の大きさの範囲にマップする[@Malan2012]。高/低頻度の領域に対して異なる重みを与える(シャープネス維持のため)。

$$
c_{max} = \max(c_{TL}, c_{TR}, c_{BL}, c_{BR}) \\
c_{min} = \min(c_{TL}, c_{TR}, c_{BL}, c_{BR}) \\
c_{acc} = \text{clamp}(c_{acc}, c_{min}, c_{max}) \\
w_k = |(C_{TL} + c_{TR} + c_{BL} + c_{BR}) * 0.25 - c_M| \\
w_{rgb} = \text{clamp} \left( \frac{1}{K_{lowfweq} * (1 - w_k) + K_{hifreq} * w_k}, 0, 1 \right) \\
c = c_M * (1 - w_{rgb}) + c_{acc} * w_{rgb}
$$

```hlsl
float3 cM = tex2D(tex0, tc.xy);
float3 cAcc = tex2D(tex0, reproj_tc.xy);

float3 cTL = tex2D(tex0, tc0.xy);
float3 cTR = tex2D(tex0, tc0.zw);
float3 cBL = tex2D(tex0, tc1.xy);
float3 cBR = tex2D(tex0, tc1.zw);

float3 cMax = max(cTL, max(cTR, max(cBL, cBR)));
float3 cMin = min(cTL, min(cTR, min(cBL, cBR)));

float3 wk = abs((cTL + cTR + cBL + cBR) * 0.25 - cM);

return lerp(cM, clamp(cAcc, cMin, cMax), saturate(rcp(lerp(k1, kh, wk))));
```

# 被写界深度[DEPTH OF FIELD]

# 被写界深度 \\ もっともらしいDOF: パラメータ化[DEPTH OF FIELD \\ PLAUSIBLE DOF: PARAMETERIZATION]

- アーティストフレンドリーなパラメータはゲームのDOFが間違って見える傾向にある理由のひとつである。
    - "フォーカス範囲[focus range]"と"ブラー量[blur amount]"などのような一般的な制御方法には物理的な意味合いがあまりない。
    - 錯乱円[CoC]は主にF値、焦点距離、フォーカス距離[focal distance]に依存する。後ろの2つは直接FOVに影響する。
    - もっとボケが欲しい場合、焦点距離を最大にして開口[aperture]を広げる必要がある。これは適切なフレーミング[framing]のために対象物に近づいたり離れたりすることを意味する。
        - ゲームアーティスト/プログラマーがDOFについて考える一般的な方法ではない。

# 被写界深度 \\ 焦点距離[DEPTH OF FIELD \\ FOCAL LENGTH]

# 被写界深度 \\ F値[DEPTH OF FIELD \\ F-STOPS]

# 被写界深度 \\ フォーカス距離[DEPTH OF FIELD \\ FOCAL DISTANCE]

# 被写界深度 \\ もっともらしいDOF: ボケ[DEPTH OF FIELD \\ PLAUSIBLE DOF: BOKEH]

- 写真において焦点外の領域は一般に"ボケ"と呼ばれる(日本語でブラーのこと)。
- ボケ形状はカメラの開口サイズ(F値)と絞り羽根の枚数に直接関係を持つ。
    - 大きな開口＝"円形"ボケ、小さな開口＝多角形ボケ
        - 多角形ボケの見た目は絞り羽根の枚数に依存する。
        - 羽根の枚数はレンズの特徴で変化する。
    - 大きな開口＝入る光が多い、小さな開口＝入る光が少ない
        - 夜景の撮影では、円形ボケとモーションブラーがしばしば多くなることに気付くかもしれない。
- ボケのカーネルは平坦である。
    - ほぼ同じ光量がすべての方向からカメラの虹彩[iris]に入る。
        - 縁[edge]は影になるかもしれない。これは一般に口径食[vignetting]として知られる。
        - レンズの製造が粗悪だと、無数の光学収差[optical aberration]が引き起こすかもしれない。
    - これはガウシアンブラー、拡散[diffusion]DOF、派生テクニックが間違って/ビジュアル的に不快に見える主な理由である。

# 被写界深度 \\ 最新技術の概要[DEPTH OF FIELD \\ STATE OF THE ART OVERVIEW]

- 散乱ベーステクニック[@Cyril2005; @Sawada2007; @3DMark2011; @Mittring2011; @Sousa2011]
    - クアッド1つかトライアングル1つをレンダリングして、散乱円に基づいてスケールする。
- 単純な実装に良好な結果。欠点:パフォーマンス、特に浅いDOFで
    - 可変な/整合性のないフィルレートヒット。近/遠レイヤの解像度に依存して、開口サイズが5ms以上に達するかもしれない。
    - クアッド生成フェーズは固定のコストが付与される。

# 被写界深度 \\ 最新技術の概要 (2)[DEPTH OF FIELD \\ STATE OF THE ART OVERVIEW (2)]

- 収集ベース: 分割可能(柔軟性のないカーネル) vs. カーネル柔軟性 [@Kawase2009; @Gotanda2009; @White2011; @Andreev2013; @McIntosh2012]

# 被写界深度 \\ もっともらしく効率的なDOF再構築フィルタ[DEPTH OF FIELD \\ A PLAUSIBLE AND EFFICIENT DOF RECONSTRUCTION FILTER]

- 分割可能な柔軟性のあるフィルタ: 低帯域幅が必須＋異なるボケ形状が可能
    - 第1パス $N^2$タップ(例:7x7)
    - 第2パス 形状を[flood-fill](https://en.wikipedia.org/wiki/Flood_fill)するための$N^2$タップ(例:3x3)
    - R11G11B10F: ダウンスケールしたHDRシーン、R8G8: CoC
    - 半分の解像度で行われる。
    - 遠/近フィールドは同じパスで処理される。
    - アンダーサンプリングを最小化するためにオフセット範囲を制限する。
    - スペックの高いハードウェアではタップ数をより多く取れる。
- 絞りと光学収差のシミュレーション
- 物理ベースCoC

# 被写界深度 \\ レンズのレビュー[DEPTH OF FIELD \\ LENS REVIEW]

- ピンホール"レンズ"
    - レンズのないカメラ
    - 光は像平面に当たる前に単一の小さな開口を通らなければならない。
    - 典型的なリアルタイムレンダリング
- 薄レンズ
    - カメラレンズは有限の大きさを持つ。
    - 光は像平面に当たるまでにレンズを通って屈折する。
    - $F$ = 焦点距離
    - $P$ = フォーカスが当たっている平面[plane in focus]
    - $I$ = 像距離

# 被写界深度 \\ レンズのレビュー (2)[DEPTH OF FIELD \\ LENS REVIEW (2)]

- 薄レンズの式は以下の関係をもたらす。
    - $F$ = 焦点距離(光が集まり始める所)
    - $P$ = フォーカスが当たっている平面(カメラのフォーカス距離)
    - $I$ = 像距離(像がピントが合って投影される所)

$$
\frac{1}{P} + \frac{1}{I} = \frac{1}{F}
$$

- 錯乱円[@Potmesil1982]
    - $f$ = F値
    - $D$ = 物体距離
    - $A$ = 開口の直径

$$
CoC = \left| \frac{FD}{D-F} - \frac{FP}{P-F} \right| \frac{D-F}{fD} \\
f = \frac{F}{A}
$$

- 単純化すると
    - 注: $f$と$F$はカメラ設定からの既知の変数である。
    - シェーダでは単一のMAD命令に落とし込む。

$$
CoC = \left| A \frac{F(P-D)}{D(P-F)}\right|
$$

- カメラのFOV:
    - 一般的なフィルムフォーマット(または、センサー)、35mm/70mm
    - 代わりにFOVから焦点距離を導き出せる。

$$
\theta = 2\text{arctan} \frac{width_{film}}{2F}, F = \frac{0.5width_{film}}{\tan(\theta/2)}
$$

# 被写界深度 \\ サンプリング[DEPTH OF FIELD \\ SAMPLING]

- 同心円マッピング[@Shirley1997]は一様サンプル分布に対して使われる。
    - 単位四角形を単位円にマップする。
    - 四角形は$(a, b) [-1, 1]^2$にマップされ、$a=b$と$a=-b$の直線で4つの領域に分割される。
    - 第1領域は以下で求められる。

$$
r = a \\
\theta = \frac{PIb}{4a}
$$

- サンプルをN角形に変形することによる絞りシミュレーション
    - 修正した通常の多角形の式を経由して

$$
f = \frac{f_{stops} - f_{stops_min}}{f_{stops_max} - f_{stops_min}} \\
\theta = \theta + f\theta_{shutter_max} \\
r_{ngon} = r \left( \frac{\cos(\frac{PI}{n})}{\cos(\theta - \frac{2PI}{n}\text{floor}(\frac{n\theta + PI}{2PI}))} \right)^f
$$

# 被写界深度 \\ サンプリング: 2回目の反復[DEPTH OF FIELD \\ SAMPLING: 2ND ITERATION]

- 最終的な形状をfloodfillするため、[@McIntosh2012]と同じように、二値の和集合[boolean union]で合成する。

# 被写界深度 \\ 分割可能なフィルタパス[DEPTH OF FIELD \\ SEPARABLE FILTER PASSES]

# 被写界深度 \\ リファレンス vs 分割可能なフィルタ[DEPTH OF FIELD \\ REFERENCE VS SEPARABLE FILTER]

# 被写界深度 \\ 動作中の絞りシミュレーション[DEPTH OF FIELD \\ DIAPHRAGM SIMULATION IN ACTION]

# 被写界深度 \\ タイル最大/最小CoC[DEPTH OF FIELD \\ TILE MIN/MAX COC]

- タイル最大/最小CoC
    - CoCターゲットをk回だけダウンスケールする(kはタイル数)。
    - 遠方場[far field]のための最小フラグメントと近接場[near field]のための最大フラグメントを取る。
    - R8G8のストレージ
- 近接/遠方場を同じパスで処理するために使う。
    - 両場のタイル最大/最小CoCを用いる動的分岐
    - 遠/近の間のコストのバランスを取る。
    - 近接場でのscatter as gather近似でも使われる。
- 他のポストプロセスを伴うコストを折りたたむ[fold]ことができる。
    - ブルームのためのHDRシーンのダウンスケールを最初のダウンスケールコストに畳み込まれ、近接/遠方場のHDR入力をR11G11B10Fにパックする --- オールインワンパス。

# 被写界深度 \\ 遠方＋近接場処理[DEPTH OF FIELD \\ FAR + NEAR FIELD PROCESSING]

- 両場は半分の解像度の入力を使う。
    - 注意: ダウンスケールはバイリニアフィルタリングにより誤差のもとになる。
    - ダウンスケールにカスタムのバイリニア(バイラテラル)フィルタを使う。
- 遠方場
    - カーネルサイズをスケールして、遠方CoCでサンプルを重み付けする[@Scheuermann2004]。
    - 遠方CoCでレイヤを事前乗算する[@Gotanda2009]。
        - バイリニア/分割可能フィルタからのブリーディングアーティファクトを軽減する。
- 近接場
    - scatter as gather近似
    - カーネルサイズをスケールして、近接CoCに対するタイル最大CoCでフラグメントを重み付けする。
    - 近接CoCで事前乗算する。
        - 近接場のフラグメントをブラーしたいだけ(安価な部分的な遮蔽の近似)。

# 被写界深度 \\ 最後の合成[DEPTH OF FIELD \\ FINAL COMPOSITE]

- 遠方場: バイラテラルフィルタを経由してアップスケールする。
    - ハーフ解像度CoCから4タップ取って、フル解像度のCoCと比較する。
    - クオリティのためにバイキュービックフィルタを使って重み付けされる[@Sigg2005]。
    - 遠方場CoCはブレンディングに使われる。
- 近接場: 気にせずアップスケールする。
    - ハーフ解像度の近接場CoCがブレンディングに使われる。
    - できるだけにじませる[bleed]ことができる。
    - バイキュービックフィルタを使うことも。
- ブレンディングに注意
    - 線形ブレンディングはよく見えない(signal frequency soup)。
        - Crysisシリーズを含め、色んなゲームで見られる(puts hat of shame)。
    - 単純な対処法: 代わりに非線形のブレンドファクタを使う。

# モーションブラー[MOTION BLUR]

# モーションブラー \\ シャッタースピードとF値のレビュー[MOTION BLUR \\ SHUTTER SPEED AND F-STOPS REVIEW]

- モーションブラーの量はカメラのシャッタースピードとF値の使い方に関連する。
    - 露出が長い(シャッターが遅い)と、より多くの光を受け取る(そして、モーションブラー量が多くなる)。逆もまた然り。
    - F値が小さいと、露出が速くなる(そして、モーションブラーが少なくなる)。逆もまた然り。

# モーションブラー \\ 最新技術の概要[MOTION BLUR \\ STATE OF THE ART OVERVIEW]

- ジオメトリ拡張[expansion]による散乱[@Green2003; @Sawada2007]。
    - 追加のジオメトリパス＋ジオメトリシェーダの使用が必須。

# モーションブラー \\ 最新技術の概要 (2)[MOTION BLUR \\ STATE OF THE ART OVERVIEW (2)]

- scatter as gather[@Sousa2008; @Gotanda2009; @Sousa2011; @McGuire2012]
    - 例、速度の膨張[dilation]、速度ブラー、タイル最大速度。シングルvsマルチパス合成。深度/頂点/オブジェクトIDマスキング。シングルパスDOF＋MB。

# モーションブラー \\ もっともらしいモーションブラーのための再構築フィルタ [@McGuire2012][MOTION BLUR \\ RECONSTRUCTION FILTER FOR PLAUSIBLE MB [MCGUIRE12]]

- タイル最大速度[Tile Max Velocity]とタイル近傍最大速度[Tile Neighbor Max Velocity]
    - 速度バッファをk倍ダウンスケールする(kはタイル数)。
    - 各ステップで速度の最大長を取る。
- モーションブラーパス
    - 早期脱出のためのタイル近傍最大速度
    - 中心速度タップとしてのタイル最大速度
    - 各反復ステップで、フル解像度の$\|V\|$と深度に対して重み付けする。

# モーションブラー \\ 改善された再構築フィルタ[MOTION BLUR \\ AN IMPROVED RECONSTRUCTION FILTER]

- パフォーマンスの品質
    - 内部ループの重み計算を単純化してベクトル化する(最終的にいくつかのMAD命令になる)。
    - 大きい[fat]バッファのサンプリングはGCNハードウェアでバイリニアを伴うとレートが半分になる。(ポイントフィルタリングはフルレートだが、エイリアシングによりよく見えない。)
    - 入力: シーンでR11G11B10F、$\|V\|$と8ビット深度をR8R8ターゲットに焼き込む。
    - 分割可能、2パスにする[@Sousa2008]。

# モーションブラー \\ 改善された再構築フィルタ (2)[MOTION BLUR \\ AN IMPROVED RECONSTRUCTION FILTER (2)]

```hlsl
float2 DepthCmp(float2 z0, float2 z1, float2 fSoftZ) {
    return saturate((1.0f + z0 * fSoftZ) - z1 * fSoftZ);
}

float4 VelCmp(float lensq_xy, float2 vxy) {
    return saturate((1.0f - lensq_xy.xxxx * rcp(vxy.xyxy)) + float4(0.0f, 0.0f, 0.95f, 0.95f));
}

const float2 tc = min_tc + blur_step * s;
const float lensq_xy = abs(min_len_xy + len_xy_step * s);
const float2 vy = tex2Dlod(tex1, float4(tc.xy, 0, 0));  // x = ||v||, y = depth

const float2 cmp_z = DepthCmp(float2(vx.y, vy.y), float2(vy.y, vx.y), 1);
const float4 cmp_v = VelCmp(lensq_xy, float2(vy.x, lensq_vx));
const float w = (dot(cmp_z.xy, cmp_v.xy) + (cmp_v.z * cmp_v.w) * 2);

acc.rgb += tex2Dlod(tex0, float4(tc.xy, 0, 0)) * w;
```

# モーションブラー \\ 改善された再構築フィルタ (3)[MOTION BLUR \\ AN IMPROVED RECONSTRUCTION FILTER (3)]

- Gバッファにオブジェクトの速度を出力する(必要な場合のみ)。
    - 別の義お娶りパスを避ける。
    - リジッド[rigid]ジオメトリ: オブジェクト距離 < 距離しきい値
    - 変形できる[deformable]ジオメトリ: 移動量 > 移動しきい値の場合
    - 移動するジオメトリは最後にレンダリングされる。
    - R8G8フォーマット
- カメラ速度で合成する。
    - 速度はガンマ2.0空間でエンコードされる。
    - 精度はまだ十分ではないが、実践ではそれほど目立たない。

- エンコード

$$
v_{enc} = \sqrt{|v_{xy}|} * \text{sgn}(v_{xy}) * (127.0 / 255.0) + 0.5
$$

- デコード

$$
v_{enc} = (v_{enc} - 127.0 / 255.0) / 255.0
v = (v_{enc} * v_{enc}) * \text{sgn}(v_{enc})
$$

# モーションブラー \\ MBとDOFのどちらが先？[MOTION BLUR \\ MB OR DOF FIRST?]

- 現実世界では、MB/DOFは同時に起こる。
    - 夢の実装: 大きな$N^2$カーネル＋バッチ化されたDOF/MB
    - または、MBクアッド伸縮[stretching]に基づくスプライト
    - フル解像度！10億タップ！FP16！複数レイヤ！:)
- だけど、パフォーマンスは依然として重要である(コンソール)。
    - MB前にDOF[DOF->MB]は、合焦で起こるMBのとき、より小さい誤差を引き起こす。
        - これはMBがジオメトリデータに依存したscatter as gather処理であるため
        - その後の他の同じような処理は誤差を引き起こす。そして、逆もまた然り。
        - DOF後にMB[DOF->MB]からの誤差はより目立ちにくい。
    - 逆順[order swap]だと他のポストエフェクトと折りたたむのが難しくなる。
        - 追加のオーバーヘッド

# 最後の備考[FINAL REMARKS]

- 実践的MSAAの詳細
    - すること、しないこと
- SMAA 1TX: 更にロバストなテンポラルAA
    - 4つの追加のテクスチャ処理といくつかのALU
- もっともらしく高性能な[performant]DOF再構築フィルタ
    - 分割可能で柔軟性のあるフィルタ、どんなボケカーネル形状も可能
    - 1stパス: 0.426ms、2ndパス: 0.094ms。再構築フィルタで合計: 0.52ms。(1080p+AMD7970)
- もっともらしいモーションブラーのための改善された再構築フィルタ
    - 分割可能、1stパス: 0.236ms、2ndパス: 0.236ms。再構築フィルタで合計: 0.472ms。(1080p+AMD7970)

# 参考文献[References]
