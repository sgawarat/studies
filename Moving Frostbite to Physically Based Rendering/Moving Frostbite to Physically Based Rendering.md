---
title: Moving Frostbite to Physically Based Rendering 3.0 [@Lagarde2014]
codeBlockCaptions: true
figureTemplate: 図 \[i\] \[titleDelim\] \[t\]
tableTemplate: 表 \[i\] \[titleDelim\] \[t\]
listingTemplate: リスト \[i\] \[titleDelim\] \[t\]
bibliography: bibliography.bib
---
# まえがき(Introduction) {id="sec:1"}

Frostbiteは'映画的ルック(cinematic look)'を追求してきたことから、物理ベースレンダリングへの移行も自然なことだった。移行作業は[@Drobot2013]、[@Karis2013]、[@Harduin2013]、[@Kojima2013]といったゲーム業界の最新技術を基に行い、既存技術の改良とオープン問題の削り出し(chip away)を試みた。

研究開発では、測定データかレンダリングデータのいずれか適切な方を正確性の"評価基準(ground-truth)"として用いた。しかし、真に物理的に正しい処理は負荷が大きく、リアルタイム性能という制約の中で今日のゲームエンジンがそれを達成できるとは考えにくい。そこで、まともな(decent)近似が可能な所では、品質目標(quality target)に近くなるならば、絶対的な正しさよりも *信憑性(bilievability)* を支持することとした。

PBRは業界用語として一般的になっているが、その意味はゲームエンジンごとに大きな差異がある。Frostbiteではマテリアルとライティングの情報を分離することをその原則の中核のひとつとした。これはシーン中のすべてのオブジェクトが一貫性を持ったビジュアルになることを保証するために重要となる。この考えのもとに、特別なハックをせずに同じライティングをすべてのオブジェクト、すべてのマテリアルレイヤに適用する。プロダクションの観点から見ると、アセットやライティング用のリグを異なる環境で再利用しやすくなり、アーティストの操るパラメータ数が減ることでオーサリングがより直観的になる。しかし、本書で後に述べるが、パフォーマンス上の理由によりコード上ではライティングとマテリアルは密に結合している。

PBRを採用するとなると、レンダラやツールを含めたグラフィクスパイプライン全体に手を入れなければならないことはすぐに理解できる。そのことを踏まえて、本書では普通の文献なら省かれる細かいところも含めて、大規模なプロダクションのエンジンに必要な様々なアップグレードのすべてを網羅した。まずはじめに、[@sec:2]ではground-truthとして利用するリファレンスがPBRの文脈においてどれだけ重要であるかをより詳しく説明する。次に、[@sec:3]ではマテリアルを提示して、光が物質とどのように相互作用するかを調査(review)する。[@sec:4]では光の定義や放出(emit)について説明する。[@sec:5]では輝度(luminance)を最終ピクセル値に変換する方法に触れながらカメラと出力画像に焦点を当てる。最後に、[@sec:6]ではPBRへ移行するさいのスケジュールの立て方やこの期間中に検討したことを見直すことで結論を述べる。

|||
|:-:|:-|
|$\boldsymbol v$|視線ベクトル|
|$\boldsymbol l$|入射光ベクトル|
|$\boldsymbol n$|法線
|$\boldsymbol h$|ハーフベクトル|
|$L$|ライティング関数|
|$f$|BRDF|
|$f_d$|BRDFのディフューズ要素|
|$f_r$|BRDFのスペキュラ要素|
|$\alpha$|マテリアルのラフネス|
|$\alpha_{lin}$|知覚的に線形なマテリアルのラフネス|
|$\cdot$|内積|
|$\langle \cdot \rangle$|クランプされた内積|
|$\lvert \cdot \rvert$|内積の絶対値|
|$\rho$|ディフューズ反射率|
|$\chi^+(a)$|Heaviside関数: $a>0$のとき1、$a<=0$のとき0|
: 数学的な表記法(mathematical notation) {#tbl:1}

# リファレンス(Reference) {id="sec:2"}

## モデルと仮説の検証(Validating models and hypothesis)

![現実世界のライティング(左)とインエンジン(in-engine)ライティング(右)の比較。](assets/Figure1.png){#fig:1}

ビデオゲーム業界は長年に渡りフォトリアルな画像を得ようと試みを重ねてきたが、*フォトリアリズム(photorealism)* では画像を生成する方法やデータについて言及せずに感覚で(定性的に(qualitatively))判断していた。現実世界の振る舞いや性質をシミュレートしようとする *物理ベースレンダリング* ではそれと異なり、数値で(定量的に(quantitatively))判断する。そのため、きちんとしたモデルや正しい仮説を選び出すためにはground-truthとなる良いリファレンスが必要になる。妥当性の判断や正しい選択をする最善の方法は現実世界と比較観察を行うことである。現実世界を観察することで、[@fig:1]に見られるようなハイライト形状、濡れた表面の振る舞い、ライト強度の違いによる差異といった様々なビジュアル要素を迅速に把握することができる。現実世界をマテリアルのリファレンスとする場合には、異なるライティングの振る舞いをキャプチャするために複数の縮尺で撮影することが重要になるが、正確に測定するには膨大な時間がかかることがよくある。MERLなどのデータベースを利用すれば、モデルの性能評価を迅速に行うことができる。本アプローチでは、ライトの強度やフォールオフ、空の明るさ、カメラのエフェクトなどの実データの測定及び評価を試みたが、これらの工程は時間がかかる上、必ずしも簡単にセットアップできるわけではない。

## インエンジン近似の評価(Validating in-engine approximations)

![実装の妥当性を検証するためのインエンジンの結果とオフラインパストレースの結果との比較ツール。](assets/Figure2.png){#fig:2}

MitsubaのようなモダンなPBRパストレーサーは最新のレンダリング技術を実装しており、近似の妥当性を評価する代替手段として十分なほどに現実感のある画像を生成することができる。Frostbiteでは近似の妥当性を迅速に評価できるようにMitsuba用の簡単なエクスポータを作った。このエクスポータはジオメトリ情報、(テクスチャを含まない)定数のマテリアル情報、すべてのライト強度を出力でき、マテリアルモデル、ライトインテグレーション、ライト強度を簡単に確認できる。さらにこのエクスポータはグローバルイルミネーション、アンビエントオクルージョン、反射といったより複雑な現象の正確性を検証することを可能にする。[@fig:2]はエクスポート後に自動的に起動するウィジェットを示す。このウィジェットでは完全に制御された露出の下で境目をスワイプしてピクセル値を比較することが可能になっている。もうひとつ重要なこととして、出力する強度を広い値範囲で保持するためにレンダラは線形なHDRフォーマットであるOpenEXRに最終画像を出力する。

## インエンジンリファレンスモードの評価(Validating in-engine reference mode)

<!-- <div id="fig:3" class="subfigures">
![インエンジン](assets/Figure3a.png){#fig:3a}
![インエンジンリファレンス](assets/Figure3b.png){#fig:3b}
![パストレーサーリファレンス](assets/Figure3c.png){#fig:3c}

左:いくつかのタイプのエリアライトから成るインエンジンシーンのレンダリング。中:エリアライトをGPUによる重点サンプリングによってレンダリングした同じインエンジンシーン。右:評価用パストレーサーでレンダリングされたシーン。
</div> -->
![(a)左:いくつかのタイプのエリアライトから成るインエンジンシーンのレンダリング。(b)中:エリアライトをGPUによる重点サンプリングによってレンダリングした同じインエンジンシーン。(c)右:評価用パストレーサーでレンダリングされたシーン。](assets/Figure3.png){#fig:3}

前節で説明したエクスポータは便利だが、シーンをレンダリングするのに毎度数秒から数分かかってしまう。そこでイテレーションを早めるため、GPUでの(IBLとエリアライトの)ブルートフォースサンプリングによるライティングインテグレーションを行うゲーム内リファレンスモードを追加した([@fig:3])。このレンダリング時間は決して短いわけではないが、そのイテレーション間隔はエクスポータを使うよりも格段に短くなる。

#### 注 <!-- paragraph -->

正しいリファレンスを使うことが重要である。これは明らかなように思えるが、リファレンスが良くない場合、近似もまた良くならない。髪シェーディングモデルを近似するときはリファレンスとして現実世界に一番近いものを使おう。式を近似するときは誤差を計算できるように、Fresnelの式に対するOren-NayrやSchlickの近似式のような、すでにある近似式ではなくオリジナルの式を必ず使おう。唯一完全に信用できるリファレンスは現実世界のみである。

# マテリアル(Material) {id="sec:3"}

## マテリアルモデル(Material models) {id="sec:3.1"}

### 外観(Appearance)

![光と物質の相互作用の多様性を示す様々な表面の外観。](assets/Figure4.png){#fig:4}

表面の外観は入射光と表面の性質との相互作用によって生み出され、[@fig:4]に見られるように、現実世界では一面一辺倒であるような単純なものから層を成したり不均質であったりするような複雑なものまで数多くの外観を観察することができる。これらの様々な外観は伝導性(conductivity)、平均自由行程(mean-free-path)、吸収(absorption)といった固有の物理的性質により分類できる。これらのマテリアルの性質に基づき、全波長域の中からある範囲の外観を表現できるとする様々なマテリアルモデルが発表されている。マテリアルモデルの分野は広大であり、用いるトレードオフや求める正確さはモデルにより様々である。BSDFと呼ばれるマテリアルモデルは反射率を示すBRDFと透過率を示すBTDFに分けることができる。この文章では"標準的な"外観を表現できるとするマテリアルモデルにおける反射の部分に焦点をあてようと思う。ここで言う"標準的"とは、とりわけ日常生活の中で遭遇しやすいもののことを指す。故に、これから扱うものは平均自由行程の短い反射的で等方性を持つ誘電体または導体の表面に限定することとする。

### マテリアルモデル(Material models)

!["標準的な"物質の厚板(slab)との光の相互作用。(a)左:光の相互作用。(b)右:ディフューズ項$f_d$とスペキュラ項$f_r$との相互作用のBSDFモデル。](assets/Figure5.png){#fig:5}

![$D$項でモデル化される様々なラフネスの表面。上:マイクロファセットとの光の相互作用。中:BRDF$f_r$のローブ。下:球における外観。](assets/Figure6.png){#fig:6}

この標準的なマテリアルモデルの文脈では、[@fig:5]が示すように、表面の応答$f$は"ディフューズ"($f_d$)と呼ばれる低い角周波数の部分と"スペキュラ"($f_r$)と呼ばれる高い各周波数の部分の2つの項に分けられる。界面は空気と物質の2つの媒質で区切られる。平坦な界面からなる表面であれば、誘電体と導体の両方においてFresnelの法則によって簡単に表すことができる。[@fig:6]が示すような不規則な界面であれば、この種の表面との光の相互作用の特徴にうまく適合するマイクロファセットによるモデル[@Cook1982]により表すことができる。マイクロファセットモデルは以下の式により表される。より詳しい導出は[@Heitz2014]を参照のこと。

$$
f_{d/r}(\boldsymbol v) = \frac{1}{|\boldsymbol{n} \cdot \boldsymbol{v}| |\boldsymbol{n} \cdot \boldsymbol{l}|} \int_{\Omega} f_m(\boldsymbol{v}, \boldsymbol{l}, \boldsymbol{m}) G(\boldsymbol{v}, \boldsymbol{l}, \boldsymbol{m}) D(\boldsymbol{m}, \alpha) \langle \boldsymbol{v} \cdot \boldsymbol{m} \rangle \langle \boldsymbol{l} \cdot \boldsymbol{m} \rangle d\boldsymbol{m}
$$ {#eq:1}

$D$項はマイクロファセット分布(つまり、法線分布関数(NDF))をモデル化し、$G$項はマイクロファセットによる遮蔽(マスキング-シャドウイング)をモデル化する。この式はディフューズ項$f_d$とスペキュラ項$f_r$の両方に対して対応するマイクロファセットBRDF$f_m$を定めることで用いることができる。スペキュラ項の場合、$f_m$は完全鏡面(perfect mirror)であり、Fresnelの法則$F$によりモデル化される。したがって、以下のよく知られた式を導くことができる。

$$
f_r(\boldsymbol v) = \frac{F(\boldsymbol{v}, \boldsymbol{h}, f_0, f_{90}) G(\boldsymbol{v}, \boldsymbol{l}, \boldsymbol{h}) D(\boldsymbol{h}, \alpha)}{4 \langle \boldsymbol{n} \cdot \boldsymbol{v} \rangle \langle \boldsymbol{n} \cdot \boldsymbol{l} \rangle}
$$ {#eq:2}

$D$項は[@fig:6]が示すように、表面の外観において重要な役割を担っている。[@Walter2007; @Burley2012]はGGX分布のような"ロングテール"のNDFが現実世界の表面をキャプチャするときに良好な結果を示すことを指摘している。$G$項もまた高いラフネス値において重要な役割を担っている。@Heitz2014 はSmithの可視性関数が$G$項として正確かつ厳密であることを示している。それに加えて、以下に示すような、マスキングとシャドウイングの間にある相関をモデリングしたマスキング-シャドウイング関数の更に正確な形式があるにもかかわらず、Smithの可視性関数の近似バージョンが使われる傾向があることを指摘している。[@fig:7]はシンプルなSmithの関数と高さ相関(hight-correlated)のSmithの関数との差異を示している。

$$
G(\boldsymbol{v}, \boldsymbol{l}, \boldsymbol{h}, \alpha) = \frac{\chi^+(\boldsymbol{v} . \boldsymbol{h}) \chi^+(\boldsymbol{l} . \boldsymbol{h})}{1 + \Lambda(\boldsymbol{v}) + \Lambda(\boldsymbol{l})} \text{ with } \Lambda(\boldsymbol{m}) = \frac{-1 + \sqrt{1 + \alpha^2 \tan^2(\theta_m)}}{2} = \frac{-1 + \sqrt{1 + \frac{\alpha^2 (1 - \cos^2(\theta_m))}{\cos^2(\theta_m)}}}{2}
$$ {#eq:3}

![ラフネスを増加させたときの黒い誘電体の球(上)とクロムの金属的な球(下)におけるSmithの可視性関数の相関なしと相関ありの比較。ラフネス値が大きい場合に注目。](assets/Figure7.png){#fig:7}

ディフューズ項の場合、$f_m$はLambertモデルに従い、そのときの[@eq:1]を簡略化すると以下のようになる。

$$
f_d(\boldsymbol v) = \frac{\rho}{\pi} \frac{1}{|\boldsymbol{n} \cdot \boldsymbol{v}| |\boldsymbol{n} \cdot \boldsymbol{l}|} \int_{\Omega} G(\boldsymbol{v}, \boldsymbol{l}, \boldsymbol{m}) D(\boldsymbol{m}, \alpha) \langle \boldsymbol{v} \cdot \boldsymbol{m} \rangle \langle \boldsymbol{l} \cdot \boldsymbol{m} \rangle d\boldsymbol{m}
$$ {#eq:4}

これまではこのような単純なLambertモデルでも問題なかったが、[@fig:8]が示すように、スペキュラ項との一貫性を保つため、ディフューズ項の計算に表面のラフネスを取り入れる必要がある[@Burley2012](すなわち、スペキュラ項とディフューズ項で同じラフネスを用いるべきである)。[@eq:4]は解析的に解くことができないが、 @Oren1994 はガウス分布のNDFとV型空洞のG項を用いることでOren-Nayarモデルとして知られる[@eq:4]の経験的近似を発見した。モデルを正しくサポートするためには、[@Gotanda2014]で説明されているようにGGXのNDFにも[@eq:4]に対する同じような近似を作るべきである。[@sec:B]にその詳細を載せるが、さらなる研究が必要である。

![鏡のBRDF$f_m$を持つスペキュラ項$f_r$(左)とディフューズBRDF$f_m$を持つディフューズ項$f_d$(右)におけるマイクロスケールでの相互作用。](assets/Figure8.png){#fig:8}

@Burley2012 は現実世界の表面を観察した結果から以下の式で示されるディフューズモデルを発表した。このモデルは経験的だが、MERLデータベースのマテリアルの主な特徴を再現でき、加えて単純であるため、Frostbiteで用いることとした。このディフューズ項はマテリアルのラフネスを計算に取り入れており、グレージング角での自己反射をいくらか生み出すことができる。

$$
f_d = \frac{\rho}{\pi} (1 + F_{D90}(1 - \langle \boldsymbol{n} \cdot \boldsymbol{l} \rangle)^5) (1 + F_{D90}(1 - \langle \boldsymbol{n} \cdot \boldsymbol{v} \rangle)^5) \text{ where } F_{D90} = 0.5 + \cos(\theta_d)^2 \alpha
$$ {#eq:5}

### エネルギー保存則(Energy conservation)

エネルギー保存則は扱うエネルギー量が受け取ったエネルギー量より多くならないことを考慮する上で重要な概念である。加えて、グレージング角においてディフューズ項よりスペキュラ項のほうが光をより散乱させるという振る舞いを正しく扱うことが可能になる。Frostbiteでは、計算を簡単にするため、半球状の固定照明による与えられた方向に対する反射率の総計である方向性半球反射率(hemispherical-directional reflectance)がBRDF全体で1未満であることでエネルギーが保存されているとした。

$$
\rho_{hd}(\boldsymbol{v}) = \int_{\Omega} f(\boldsymbol{v}, \boldsymbol{l}) \langle \boldsymbol{n} \cdot \boldsymbol{l} \rangle d\boldsymbol{l} = \int_{\Omega} (f_r(\boldsymbol{v}, \boldsymbol{l}) + f_d(\boldsymbol{v}, \boldsymbol{l})) \langle \boldsymbol{n} \cdot \boldsymbol{l} \ge 1
$$ {#eq:6}

スペキュラモデルとディフューズモデルとの関連性が直接的ではないため、適切な導出を作ることは簡単ではない(スペキュラ項とディフューズ項がともにマイクロファセットモデルの基づく場合は[@sec:C]を参照)。Disneyのディフューズモデルはエネルギー保存則を満たしていないことに注意する必要がある(脚注4: @Burley2012 が説明するように、これはアーティストがあらゆるラフネス値を通して同じディフューズ色を得られるようにすることを目的とした仕様によるものである)。[@fig:9] (a)はDisneyのディフューズモデルの方向性半球反射率を表しているが、見るからに値は1を越えており、エネルギー保存則を満たしていないことが分かる。

![様々な視野角とラフネスにおけるDisneyのディフューズBRDFの方向性半球反射率のプロット。(a)左:オリジナルの反射率。(b)中:新しい再正規化を施した反射率。(c)右:スペキュラ項とディフューズ項の組み合わせ。](assets/Figure9.png){#fig:9}

そこで、我々は自己反射の特性を維持しつつエネルギーのゲインを補正するような修正を追加した。[@lst:1]は再正規化ファクタを導入したDisneyの評価関数を示している。[@fig:9] (c)ではスペキュラのマイクロファセットモデル$f_r$とDisneyのディフューズモデル$f_d$を合成した$f$の方向性半球反射率を表しており、完全に1にはなっていないが十分に近い値になっている。[@fig:10]ではオリジナルと再正規化バージョンとの比較を示している。

Listing: エネルギーの再正規化を含むDisneyのディフューズBRDFのコード。`linearRoughness`は知覚的に線形なラフネスである([@sec:3.2.1]を参照)。

~~~{.c .numberLines id="lst:1"}
float Fr_DisneyDiffuse(float NdotV, float NdotL, float LdotH, float linearRoughness) {
    float energyBias     = lerp(0, 0.5, linearRoughness);
    float energyFactor   = lerp (1.0, 1.0 / 1.51, linearRoughness);
    float fd90           = energyBias + 2.0 * LdotH*LdotH * linearRoughness;
    float3 f0            = float3(1.0f, 1.0f, 1.0f);
    float lightScatter   = F_Schlick(f0, fd90, NdotL).r;
    float viewScatter    = F_Schlick(f0, fd90, NdotV).r;

    return lightScatter * viewScatter * energyFactor;
}
~~~

![(a)上:Disneyのディフューズ項とLambertのディフューズ項との比較。(b)下:オリジナルのDisneyのディフューズ項と再正規化バージョンとの比較。](assets/Figure10.png){#fig:10}

### 形状の特性(Shape characteristics)

スペキュラのマイクロファセットによるBRDFはしばしば無視(bypass)されるが最終的な外観に強い影響を及ぼすある性質を持つ。特に以下の2つの現象が重要である。

- **半角パラメータ化(Half-angle parameterization)**: このパラメータ化は通常の入射角では等方的になるがグレージング角では異方的になるというBRDF形状の非線形な変換を暗に示している。さらなる洞察は[@sec:4.9]を参照のこと。
- **オフスペキュラ(Off-specular)**: BRDFローブは時折反射した視線方向(反射方向(mirror direction))を中心とすると仮定されるが、[@fig:11]に見られるように、ラフネスが大きくなると$\langle \boldsymbol{n} \cdot \boldsymbol{l} \rangle$とマスキング-シャドウイング項$G$によってBRDFローブは法線方向に引っ張られる(gets shifted)ようになる。これは"*オフスペキュラピーク(Off-specular peak)*"と呼ばれ、表面の粗い外観を表すのに重要な役割を担っている。

![様々な視野角でのBRDFローブ形状の例。グレージングな視野角では支配的なローブは反射方向$R$ではなく方向$M$の方を向いている。上段:$\alpha = 0.4$。下段:$\alpha = 0.8$。](assets/Figure11.png){#fig:11}

ラフネス値が大きいとオフスペキュラピークによる差異が大きくなる可能性がある。この重要な特徴を計算に入れるため、この"支配的な向き(dominant direction)"のモデル化を試みた。これはエリアライト([@sec:4.7])と画像ベースライト([@sec:4.9])を評価するときに用いた。

### Frostbiteの標準モデル(*Frostbite* standard model)

まとめると、Frostbiteの"標準"マテリアルモデルは他のゲームエンジンで用いられるものと近い[@Karis2013; @Neubelt2013; @Burley2012]。これは以下から構成されている。

- **スペキュラ項** $f_r$: Smithの相関あり可視性関数とGGXのNDFによるスペキュラのマイクロファセットモデル([@eq:2])。
- **ディフューズ項** $f_d$: エネルギーの再正規化を施したDisneyのディフューズ項。

両パートでは、ライティングを統合するときに支配的向きに対して補正をかける(オフスペキュラピークを制御する)ことができるようになっている(詳しくは次節)。Frostbiteはクリアコートのような異なるタイプのマテリアルや表面下散乱を含むマテリアルもサポートしているが、この文書では標準的マテリアルモデルのみに焦点を当ててゆく。

Listing: BSDFの評価コード。

~~~ {.c .numberLines id="lst:2"}
float3 F_Schlick(in float3 f0, in float f90 , in float u) {
    return f0 + (f90 - f0) * pow(1.f - u, 5.f);
}

float V_SmithGGXCorrelated(float NdotL , float NdotV , float alphaG) {
    // 相関あり版G_SmithGGXのオリジナル式
    // lambda_v             = (-1 + sqrt(alphaG2 * (1 - NdotL2) / NdotL2 + 1)) * 0.5f;
    // lambda_l             = (-1 + sqrt(alphaG2 * (1 - NdotV2) / NdotV2 + 1)) * 0.5f;
    // G_SmithGGXCorrelated = 1 / (1 + lambda_v + lambda_l);
    // V_SmithGGXCorrelated = G_SmithGGXCorrelated / (4.0f * NdotL * NdotV);

    // 最適化バージョン
    float alphaG2 = alphaG * alphaG;
    // 注意:"NdotL *"と"NdotV *"の関係が他と逆なのは間違いではなく意図したもの
    float Lambda_GGXV = NdotL * sqrt((-NdotV * alphaG2 + NdotV) * NdotV + alphaG2);
    float Lambda_GGXL = NdotV * sqrt((-NdotL * alphaG2 + NdotL) * NdotL + alphaG2);
    return 0.5f / (Lambda_GGXV + Lambda_GGXL);
}

float D_GGX(float NdotH, float m) {
    // 1/PIはあとでかける
    float  m2 = m * m;
    float f = (NdotH * m2 - NdotH) * NdotH + 1;
    return  m2 / (f * f);
}

// 以下は上記の関数を呼び出す例
float  NdotV = abs(dot(N, V)) + 1e-5f; // avoid  artifact
float3 H     = normalize(V + L);
float  LdotH = saturate(dot(L, H));
float  NdotH = saturate(dot(N, H));
float  NdotL = saturate(dot(N, L));

// スペキュラBRDF
float3 F     = F_Schlick(f0, f90, LdotH);
float  Vis   = V_SmithGGXCorrelated(NdotV, NdotL, roughness);
float  D     = D_GGX(NdotH, roughness);
float  Fr    = D * F * Vis / PI;

// ディフューズBRDF
float  Fd    = Fr_DisneyDiffuse(NdotV, NdotL, LdotH, linearRoughness) / PI;
~~~

## マテリアルシステム(Material system) {id="sec:3.2"}

### マテリアル(Material) {id="sec:3.2.1"}

Frostbiteを利用するゲームジャンルはスポーツ、レーシング、FPS、オープンワールドに至るまで多岐に渡る。これらのゲームが求めるさまざまな要件を満足するため、エンジンはライティングやマテリアル対応に関する柔軟な制御方法を提供する必要がある。それに加えて、移行を簡潔に行うために前の非PBRなライティングモデルとの互換性を維持する必要があった。ライティングパスは制御可能であり、ディファード、フォワード、またはそのハイブリッドをサポートしている(詳しくは[@sec:4.11])。

Frostbiteでは、"マテリアル"は以下により定義される。

- **ライティングパス**: ディファード、フォワード、またはその両方。
- **入力パラメータ**の集まり: ディフューズ、滑らかさ(smoothness)、厚さ(thickness)、など。
- **マテリアルモデル**: 粗い表面、半透明、肌、髪、など(非PBRな粗い表面も含む)。いわゆるシェーダコード。
- ディファードパスの場合の**Gバッファのレイアウト**。バッファ数は可変。

ゲーム開発班は指定のライティングパスで利用可能なものの中からマテリアル一式を選ぶことができる。各マテリアルは*materialID*属性によりそのゲームで識別される。最も一般的なケースをカバーする*ベース*マテリアル("標準"マテリアルと呼んでいるもの)は常に存在し、他のマテリアルと共有するパラメータ(例えば、ラフネス)を定義する。ディファードシェーディングでは、*ベース*マテリアルは一般に @Burley2012 が示す"Disney"モデル一式になる。それと、"二色(two-color)"と"旧(old)"の2つのベースマテリアルもサポートする。

**Disneyのベースマテリアル**: 我々のDisneyマテリアルでは以下のパラメータを用いる。

Normal
: 標準的な法線。

BaseColor
: @Burley2012 のように、非金属オブジェクトにおけるディフューズのアルベドと金属オブジェクトにおける垂直入射($f_0$)を定義する。この属性の下位部はマイクロスペキュラオクルージョン(micro-specular occlusion)を定義する[?]。

Smoothness
: オブジェクトのラフネスを定義する。値が1のときに滑らかであるとするとアーティストが直観的に扱えるため、また、Frostbiteの非PBRマテリアルモデルがすでにこれを使っているため、ラフネスではなくsmoothnessを選択した。@Burley2012 と同様に、視覚的に線形なsmoothnessにリマップされる($1 - \alpha_{lin}$)。

MetalMask
: @Burley2012 のように、"金属らしさ"または表面の導電性(すなわち、誘電体か導体か)を定義する。この変数の二値的な性質をアーティストに示すためこのような名前にした。

Reflectance
: 非金属マテリアル(つまり、MetalMask < 1)のためのアーティスト・フレンドリーな範囲で垂直入射におけるFresnel反射率($f_0$)を定義する。この属性の下位部は非金属マテリアルにおけるマイクロスペキュラオクルージョン(micro-specular occlusion)を定義する[?]。

**smoothness**では、[@fig:12]に示すような様々なリマッピング関数をアーティストに試してもらいながら分析を行い、@Burley2012 と同じように、アーティストが最も満足するように見える"二乗"のリマッピングを選択した。**reflectance**では、以下のリマッピング関数を選択した。

$$
f_0 = 0.16 \times \text{reflectance}^2
$$

このリマッピングは、4%誘電体の反射率をRGBの128にマッピングしつつ、宝石の高いFrenel値を含むことができる範囲に$f_0$をマップすることを目標としている。宝石の$f_0$はルビーで約8%、ダイヤモンドで約17%になる。我々は近似として関数が16%を上限とするよう制限することにした。[@fig:13]は一般的な値での比較を示している。実践ではリアルタイムの制限により、1%や2%における変化はほとんど目立たない(脚注5:誘電体における$f_0$の小さな変化はほとんどのリアルタイムエンジンがサポートしていない屈折光の方向に関係している)ため、この式は4%以上で急激に変化する値によく合っている。

非金属オブジェクト(**Reflectance**)と金属オブジェクト(**BaseColor**)の両方における垂直入射でのFresnel反射率($f_0$)では、マイクロスペキュラを制御するために2%(水の反射率)以下である下位部が使われる(詳細は[@sec:4.10]を参照)。これは特定のエンコーディングのために非金属オブジェクトで使われるマイクロスペキュラオクルージョンの値が異なる範囲であることを暗に示している。

TODO: マイクロスペキュラオクルージョンとは？

![Smoothnessのリマッピング比較。$(1 - \text{Smoothness})^2$の曲線は @Burley2012 のリマッピングに対応する。暗い青色の曲線はCrytekのRyseが採用したリマッピング$(1 - 0.7 \times \text{Smoothness})^6$[@Schulz2014]に対応する。これのレンダリング結果は示していないが、その曲線は4乗のものと非常に近い。](assets/Figure12.png){#fig:12}

![リマッピング関数の反射率プロットといくつかのリファレンス値。](assets/Figure13.png){#fig:13}

Disneyマテリアルは[@sec:3.1]で示した"標準"マテリアルモデルを用いる。関連するGバッファのレイアウトは[@tbl:2]に示す。このレイアウトは以下の制約からなる。

- 全ての基本属性(`Normal`、`BaseColor`、`Smoothness`、`MetalMask`、`Reflectance`)はディファードデカールをサポートためにブレンド可能である必要がある。ブレンド不可能な属性はアルファチャンネルに格納する。また、ブレンドの質に影響を与える圧縮やエンコードは避けることとした。
- MSAAをサポートすると`BaseColor`属性の彩度サブサンプリングが使えなくなる。[@Mavridis2012]
- 通常使うパラメータは同じバッファに集める必要がある。
- `MaterilId`は全てのマテリアルで同じ場所に格納する必要がある。
- ベース/標準マテリアルでは4つのバッファのみを使うことがパフォーマンス要件である。(深度バッファは含まない)

|＼|R|G|B|A|
|:-:|:-:|:-:|:-:|:-:|
|GB0|`Normal`(10)|左に同じ|`Smoothness`|`MaterialId`(2)|
|GB1|`BaseColor`|左に同じ|左に同じ|`MaterialData`(5)/`Normal`(3)|
|GB2|------|`MetalMask`|`Reflectance`|`AO`|
|GB3|`Radiosity`/`Emissive`|左に同じ|左に同じ|左に同じ|
: DisneyのディファードのベースマテリアルのGバッファレイアウト。 {#tbl:2}

`MaterialData`は`MateriralId`により様々な値に解釈される。例えば、ディファードな肌マテリアルではディフュージョンプロファイル指数(diffusion profile index)が格納され、異方的なマテリアルでは異方性の強さが格納される。`AO`は常に存在する(マテリアルタイプの非依存である)アンビエントオクルージョン項である(詳細は[@sec:4.10]を参照)。`Radiosity`はGバッファを生成するときに評価された間接ディフューズライティングを格納するライトバッファである。`Normal`は適用後でも通常のブレンドが可能である非可逆なエンコーディング手法を用いて2つに分割する(このアルゴリズムは後のトークで説明するかもしれない)。

**二色のベースマテリアル**: Disneyのパラメータ化(単一のベース色、メタルマスク、スカラの反射率)は、例えば金属酸化物のような金属と誘電体の界面では複合マテリアルのスペキュラハイライトを満足に説明できない(fall short when expressing)ため、$f_0$とディフューズ色を持つ"二色"のディファードのベースマテリアルをサポートする。関連するGバッファのレイアウトは[@tbl:3]に示す。$f_0$項は`Reflectance`と同じような下位範囲でマイクロオクルージョンをサポートする。パラメータ化間の変換は状況に応じて必要とされる。Disneyのパラメータ化から"二色"のパラメータ化への変換はその定義により自明であり、ライティング計算でGバッファをアンパックするときにすでに行われている。逆の変換は非線形な最適化が必要になりさらに混沌としてくるため、アセットが必要とするときのみ行うこととする。この変換の詳細は[@sec:D]で提供する。

|＼|R|G|B|A|
|:-:|:-:|:-:|:-:|:-:|
|GB0|`Normal`(10)|左に同じ|`Smoothness`|`MaterialId`(2)|
|GB1|`DiffuseColor`|左に同じ|左に同じ|`MaterialData`(5)/`Normal`(3)|
|GB2|`f0 Color`|左に同じ|左に同じ|`AO`|
|GB3|`Radiosity`/`Emissive`|左に同じ|左に同じ|左に同じ|
: 二色のディファードのベースマテリアルのGバッファレイアウト。 {#tbl:3}

**旧ベースマテリアル**: これは昔の非PBRエンジンでのベースマテリアルである。詳しくは[@Coffin2011]を参照のこと。レガシーなコンテンツをサポートするため、また、移行を簡単にするため、基本的なアート規則に基づいた非PBRマテリアルとPBRマテリアルとの自動変換機能を追加した。この変換は入力パラメータを格納する前にシェーダ内で行われる。ただし、適切に作られたアセットと比べると質は低くなる。質を維持したまま変換する方法は未だ見つかっていない。

[@fig:14]は異なるマテリアルパラメータの変換を強調して、マテリアルモデルとライティング機能との依存関係を示している。パフォーマンスに関して言えば、エリアライト([@sec:4.7])とIBL([@sec:4.9])はマテリアルモデルに依存した事前インテグレーションに頼っている。これは**ライティングとマテリアルがエンジン内で結合している**ことを暗示している。PBRのアプローチにより主張したライティングとマテリアルの分割はアセット制作段階でのみ妥当である。つまり、アーティストはシェーダ内のあらゆるライティング情報にアクセスできない。しかし、傘の下(under the hood)ではこの分割は保たれず、新しいマテリアルを追加するとしばしば暗黙的に新しいライティングコードが追加される。

![Frostbiteにおけるマテリアルの模式図。**入力パラメータ**(`MaterialRootData`)のリストは**マテリアルモデル**のライティング関数を評価するのに使われるライティング評価構造(`MaterialData`)に変換される。ライティング関数はすべてのライトタイプ(点(punctual)、範囲(area)、IBL)を含む。マテリアルがディファード**ライティングパス**を用いるなら、**Gバッファレイアウト** にパッキングされて、のちにアンパックして変換される。フォワード**ライティングパス**を用いるなら、直接変換される。同じライティング評価構造(`MaterialData`)にアンパックできるならば、様々なマテリアルが同じライティングコードを共有することができる。](assets/Figure14.png){#fig:14}

### レンダループ(Render loop)

前節ではマテリアルの定義を示した。この節ではマテリアルをどのようにレンダリングするかについて説明する。フォワードライティングパスを用いた表面では、マテリアルモデルのシェーダコードをセットアップし、パラメータリストを送り、表面をレンダリングする。ディファードシェーディングパスでは、より込み入ったことをやっている。効率的にマテリアル一式を管理できるようにするため、以下の注意事項(considerations)を取り決めた。

- マテリアルモデルは、細かな調整を動的分岐に頼るなどして、できる限り同じライティングコードを共有してみる。ライティングコードに違いが多すぎるならば、ステンシルバッファを用いて異なるライティングパスを用意しよう。
- マテリアルは、パラメータを格納するための細かな調整を動的分岐に頼るなどして、ベースマテリアルのレイアウトの一貫性を維持してみる。マテリアルに違いが多すぎるならば、ステンシルバッファを用いて異なるGバッファパスを用意しよう。
- "調整パス(fix-up pass)"を行うことで、異なるGバッファレイアウトを持つマテリアルによるライティングパスを共有してみる。

ジオメトリ・ヘビーなパスでは、Gバッファの生成コストを減らしながらライティングパスは同じものを使いたい。典型的なユースケースとしてはベジテーションが挙げられる。ベジテーションでは、ベースとなるGバッファレイアウトの上から2つのバッファのみを埋めて、[@lst:3]に示される"調整"パスにより失ったGバッファパラメータを後から付け足す、ということを行う。調整パスは同じバッファに読み書きできるハードウェア能力に依存した複数パスにより達成することが必要になるかもしれない。

Listing: Gバッファ調整サンプル。

~~~ {.c .numberLines id="lst:3"}
void　psMain(
    in float4 pos : SV_Position,
    in float2 texCoord: TEXCOORD0,
    out float4 outGBuffer2 : SV_Target0,
    out float4 outGBuffer3 : SV_Target1)
{
    float4 gbuffer0 = FETCH_TEXTURE(g_gbufferTexture0, texCoord.xy, 0);
    float4 gbuffer1 = FETCH_TEXTURE(g_gbufferTexture1, texCoord.xy, 0);
    float3 worldNormal = normalize(unpackNormal(gbuffer0.rg, gbuffer1.a));

    // グローバルディフューズプロブからラジオシティ用の値を読む。
    float indirectLight = calcShL2Lighting(worldNormal, ...);

    // グローバルな共通値でsmoothness、metalMask、reflectanceを調整する。
    // アンビエントオクルージョンを1にセットする。
    outGBuffer2 = float4(g_smoothness, 0, g_reflectance, 1);
    outGBuffer3 = packLightingRGBA(indirectLight);
}
~~~

レンダリングループの疑似コードは以下のようになる。

~~~ {.c }
// Gバッファ生成
ForEach 異なるステンシルビット do
    Render バッファ数nを持つディファードマテリアルについてGバッファパス
        必要ならば、シェーダ内で分岐する
    Render バッファ数(n + 1)を持つディファードマテリアルについてGバッファパス
        必要ならば、シェーダ内で分岐する

// 調整パス
ForEach ディファードシェーディングパスの共有を必要とするディファードマテリアル
    Render Gバッファの調整パス

// デカール
Render 共通パラメータに影響するディファードデカール

// ディファードシェーディング
ForEach 必要な個別のディファードシェーディングパス do
    Render ディファードシェーディング
        必要ならば、シェーダ内で分岐する

// フォワードなレンダリング及びライティング
ForEach フォワードマテリアルモデル do
    Render フォワードライティング
~~~

実践では、マテリアルパラメータとライティング関数が結びついているためにゲームチームが**すべて**をカスタマイズするのはむしろ困難である。`MaterialId`はマテリアルをカスタマイズする簡単な方法を提供する。ただし、ライティングコードの再利用と動的分岐を必須としなければならない制約がある。例としては、ライティングを2回サンプリングする必要があるクリアコートのモデルをゲームチームが追加したことがある。このシステムは非PBRエンジンからの移行とユーザの様々な要求にフィットすることを可能にした。

## PBRとデカール(PBR and decals) {id="sec:3.3"}

TODO

# (Lighting) {id="sec:4"}

## (General) {id="sec:4.1"}

## (Analytical light parameters) {id="sec:4.2"}

## (Light unit) {id="sec:4.3"}

## (Punctual lights) {id="sec:4.4"}

## (Photometric lights) {id="sec:4.5"}

## (Sun) {id="sec:4.6"}

## (Area lights) {id="sec:4.7"}

## (Emissive surfaces) {id="sec:4.8"}

## (Image based lights) {id="sec:4.9"}

## (Shadow and occlusion) {id="sec:4.10"}

## (Deferred / Forward rendering) {id="sec:4.11"}

# (Image) {id="sec:5"}

# (Transition to PBR) {id="sec:6"}

\appendix

# (Listing for reference mode) {id="sec:A"}

# (Oren-Nayar and GGX's diffuse term derivation) {id="sec:B"}

# (Energy conservation) {id="sec:C"}

# (Optimization algorithm for converting Disney's parametrization) {id="sec:D"}

# (Rectangular area lighting) {id="sec:E"}

# (Local light probe evaluation) {id="sec:F"}

# 参考文献(References)
