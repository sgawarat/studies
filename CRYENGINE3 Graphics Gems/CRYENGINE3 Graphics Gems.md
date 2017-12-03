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

TODO

# 参考文献[References]
