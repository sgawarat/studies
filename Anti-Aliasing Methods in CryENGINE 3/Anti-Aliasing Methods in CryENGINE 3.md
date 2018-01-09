---
title: Anti-Aliasing Methods in CryENGINE 3 [@Sousa2011b]
bibliography: bibliography.bib
numberSections: false
---
# CryENGINE 3のAA要件[CryENGINE 3 AA Requirements]

- 直交で一般的なソリューション
    - プラットフォーム毎のAAソリューションではない
- HDR/ディファードテクニックでうまく働く。
- サブピクセルの正確さが我々にとって重要である。
    - Schimering^[shimmeringの誤字？]はCrysis1と2のレベルでは最も不快なものであった。
    - Crysisは、アルファテストされる/極小のサブピクセルのディテールのような、エイリアスするアセットを大量[imensively]^[immensivelyのtypo？]に持っている。
    - HDRはライティングコントラスト/色変化の大きな範囲でさらに悪くする。
- 低メモリフットプリント
- ローエンドGPUで2ms以下のコスト
    - Every ms counts for consoles

# 今世代ハードウェアでのMSAAの悩み[MSAA Troubles for this HW Generation]

- メモリが必要
    - 2x、4x、など
- マルチプラットフォーム＋昔ながらでないレンダリング[@Sousa2011]
    - PS3ではFP16をサポートしていない。(アルファブレンディングパス用)
    - X360の10MBのEDRAM＋タイリング＋解決コストオーバーヘッド
    - アルファテストAAはATOCが必要
- トーンマッピングはサブサンプルごとに処理されるべき。
    - でなければ、高コントラストの領域で目立つ間違った結果になる。
    - 昔のプラットフォームには高価すぎる。

多数のワークアラウンドと追加された複雑さ
各々のソリューション、ワークアラウンド、最適化を持つ各プラットフォーム
パフォーマンスコスト

現世代では、こんな状況でやっていける見込みはない。

最小のGバッファ
    RGBA8 ワールド空間の背面の法線＋光沢度[glossiness]
    リードバックD24S8 深度＋屋内エリアのタグ付け用ステンシル
ディファードライティングバッファ
    Xbox360: パフォーマンス/メモリのためにRGB10A2に解決される2つのRGB10F
    PS3: 不透明ではRGBKにエンコードされるRGBA8、透明ではFP16
    PC: FP16

X360: 28MB; PS3 31.5MB; PC: 720pで49MB、1080pで110MBなど

# テンポラルアンチエイリアシング(別名、モーションブラー)[Temporal Anti-Aliasing (aka Motion Blur)]

- スクリーン空間の速度ベクトルに沿った方向ブラー[@Green2003]
    - ピクセルまたは頂点あたりの前/現スクリーン空間の位置からのデルタ
    - イメージ空間のモーションブラー
- 主な利点: 移動中の目立つエイリアシングが少ない。

Crysis1やKillZone2以来、人気を増している。
スクリーン空間の速度ベクトルに沿った方向ブラーとして近似される。
    ピクセルまたは頂点あたりの前/現スクリーン空間の位置からのデルタ
追加ボーナス: 素晴らしい映画的なルック

すべてのプラットフォームで使える。

# Aバッファ SSAA[@Haeberli1990][A-Buffer SSAA[Haeberli90]]

- サブピクセルジッターをカメラ錐台に加える。
- ブルートフォース: 複数回シーンをレンダリングする。
    - N個のサブサンプル ⇔ N回のシーンレンダリング
- ロバストで最高品質
    - SSAA以外の用途でも(TSSAA/DOF/ソフト・シャドウ)。
- 基本の概念が我々のテクニックで使われている。
- 問題: 複数回シーンをレンダリングする余裕が(まだ)取れない。
    - それでも、リファレンス/マーケティングクオリティのショットとしては素晴らしい

あるいは、高解像度でレンダリングしてダウンスケールする通常のアプローチ

# フレームを越えてAバッファ SSAAを分散する[Distribute A-Buffer SSAA Overframes]

- 60fpsで動作する？
    - 毎フレームにカメラ錐台にサブピクセルジッターを加える。
    - 前/現フレームを格納して、線形ブレンドする。
    - 光速の2xSSAA: 現在のコンソールで約0.5ms
    - 2フレーム ⇔ 2xSSAA、4フレーム ⇔ 4xSSAA、など
- しかし、コンソールで60fpsに達することはほとんどない。
    - fpsが低いと、結果として極めて目立つ画像ゴーストになる。

ごく一部の昔の60fpsのゲームで使われた(DMC4？)けど、だれが最初だっけ？

60fpsでもゴーストは目立つ。

# アーティファクトの最小化[Minimizing Artifacts]

- ブレンディングの改善: 再投影
    - 前フレームのサブサンプルターゲットからの速度ベクトルのフェッチ
    - TAAでのものと厳密に同じ(ただし、シングルタップ)
- 変形可能なジオメトリを扱うのが若干高価
    - ピクセル速度をレンダターゲットに出力する。
    - 草木ではその余裕が取れないかも。
- 問題: 遮蔽されていない領域のゴースティング

# アーティファクトの最小化(2)[Minimizing Artifacts (2)]

- $\|V\| > 0$ならば、ブレンディングを無効化する？
    - プレイヤーが動かない場合は非常に稀
    - そして、依然としてカメラの移動中にAAが欲しい。
- 色/エッジのタグ付けを用いた重み付け？
    - サブピクセル/高周波ディテールは結果として目立つシマーになる。
- 再投影範囲のクランプ？ (Cryrsis2で使った)
    - ピクセル重みは再投影限界に比例する。
        - 例: fBlendW = saturate(1 - (fVLen / fVMaxLen))
    - 荒い深度はサブサンプルバッファのアルファチャンネルに格納する。
        - fVLen > fMaxVThresholdかつfCurrD > fPrevDならば、マスクアウトする。
        - 8ビットでは大きな範囲を網羅するのに不十分で、近くに制限される。

# アーティファクトの最小化(3)[Minimizing Artifacts (3)]

- サブサンプルバッファのアルファチャンネルに$\|V\|$を格納する。
    - 重み: abs(fPrevLenV - fCurrLenV) / fVMaxLen

# 例コード[Example Code]

```hlsl
float fDepth = GetLinearDepth(sDepth, tcBase.xy);
float3 vPosWS = WorldViewPos.xyz + IN.vCam.xyz * fDepth;

float4 vPrevPos = mul(mViewProjPrev, float4(vPosWS, 1.0));
vPrevPos /= vPrevPos.w;

float2 vVelocity = vPrevPos.xy - vtcBase.xy;
half4 cObjVelocityParams = tex2D(sObjVelocity, tcBase.xy);
half2 vObjVelocity = DecodeMotionVector(cObjVelocityParams);

vVelocity = cObjVelocityParams.w ? vObjVelocity : vVelocity;
float fVLenSq = dot(vVelocity.xy, vVelocity.xy) + 1e-6f;
vVelocity /= fVlenSq;

half4 cCurr = tex2D(sCurrFrame, tcBase.xy);
half4 cPrev = tex2D(sPrevFrame, tcBase.xy + vVelocity * min(fVLenSq, fVMaxLen));

half fBlendW = 0.5 - 0.5 * saturate(fVlenSq / fVMaxLen);
fBlendW *= saturate(1 - (abs(cCurr.a - cPrev.a) * fVWeightScale));

OUT.Color = lerp(cCurr, cPrev, fBlendW);
```

# 2x Quincunx^[サイコロの五の目のような形] SSAA

- 2つのサブサンプルでクオリティを改善する。
    - サブサンプルのひとつをバイリニアフェッチする。
    - 4xSSAAを"近似"する。

# 分散AバッファSSAA: 注意点[Distributed A-Buffer SSAA: Caveats]

- 時間的に安定でない。
    - 非遮蔽領域でAAがない。
    - 入力信号(色/ライティング)が変化する。ロバストな解決法がまだない。
- アルファブレンディングが問題になる。
    - OITなしでは、初めの一回だけしか正しく扱われない可能性がある。
    - 追加のオーバーヘッド
- マルチGPU
    - 対処するための追加のフレームレイテンシー
    - Cryrsis2では、マルチGPUのときはNVIDIAのFXAAに切り替えた。
        - シマー再び。マルチGPUのユーザーには大不評。

# 今後の課題[Future Work]

- ポストプロセスAAとSSAAの組み合わせ
    - 多分DLAA似た感じ: 水平/垂直のエッジ、タップをブレンド
        - これは少なくとも4つの追加タップを意味する。
    - 非遮蔽領域でのAA

# 分散AバッファSSAA: 現在の成果[Distributed A-Buffer SSAA: Current Results]

- 完璧には程遠い、が:
    - 直交
    - サブピクセルの正確性
        - シェーダアンチエイリアシングボーナス
    - 2x Quincunx SSAA: コンソールで1ms
        - PCだと1080pで0.2ms
        - 2xSSAA+エッジAA: 1.7ms
        - 4xSSAA+エッジAA: 2.2ms
        - 3MBの追加メモリフットプリント

# ボーナス: マーケティングスクリーンショット[Bonus: Marketing Screenshots]

- 常にいくつかインチキ[trickery]がある。
    - CryENGINE2では、高解像度で複数のタイルをレンダリングして、SSAAを得るためダウンサンプリングした。
- CryENGINE3では、大量のサンプルでSSAAを分散した。
    - 無作為なサブピクセルジッター
    - ほぼ完璧なSSAA
    - すべてのCrysis2のマーケティングスクリーンショットはこのバリエーションを使った。
