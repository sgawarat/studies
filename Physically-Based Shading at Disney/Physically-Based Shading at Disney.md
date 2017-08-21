---
bibliography: bibliography.bib
link-citations: true
---
# Physically Based Shading at Disney

## まえがき(Introduction)

Tangled(邦題:塔の上のラプンツェル)における物理ベース髪シェーディングを含めた成功に続き、より広い範囲のマテリアルに対応した物理ベースシェーディングモデルを検討し始めた。物理ベース髪シェーディングでは、アーティストのための制御機能を整備しつつ、素晴らしいビジュアルのリッチさを達成することができた。しかし、髪のライティングとシーンとの統合は、伝統的なアドホックなシェーディングモデルと大きさのない(punctual)ライトを用いていたため、困難を伴った。その後の作品で我々は、マテリアルと環境(environments)に対するライトの応答にさらなる一貫性を持たせつつ、すべてのマテリアルにおいてリッチさを向上させたり、簡単化した制御機能を用いることで、アーティストの生産性を向上させたりしたかった。

調査を初めた段階では、どのモデルを使うべきかという以前に、どれほど物理ベースであればよいのかということさえ明確ではなかった。エネルギー保存則を完全に満たしたものであるべきなのだろうか。屈折率のような物理的パラメータを採用するべきなのだろうか。

ディフューズでは、Lambertが標準として受け入れられているように見える。一方のスペキュラは、界隈(the literature)で最も注目を集めているように見える。Ashikhmin-Shirley [-@Ashikhmin2000] のようなモデルは物理的にもっともらしくありつつも直観的で実践的であることを目的としている。一方で、@He1991 のようなものはより包括的な物理モデルを提供している。データフィッティングの改善を目的としたものもあるが、そのいくつかは手動の操作がふさわしいようにできている。我々はいくつかのモデルを実装してみて、アーティストに選んだり組み合わせたりしてもらったが、逃れようとしていたパラメータ爆発の所に結局のところ戻ってきてしまった。

多種多様な測定マテリアル(measured materials)を扱う研究として、5つの人気のモデルを比較した @Ngan2005 のものがある。いくつかのモデルは全体的にその他のモデルよりもうまくできているが、興味深いことに、そこにはモデルの性能との強い相関があった。つまり、いくつかのマテリアルはそのすべてのモデルによってうまく表現されるが、その他のマテリアルではまったくだめだった。追加のスペキュラローブを加えると、ごく少数のケースでのみ、これの助けとなった。これは、難しいマテリアルでは何が表現できないのか、という質問を投げかけている(begs the question)。

この質問の答えとして、BRDFモデルをより直観的に評価するため、我々は、測定マテリアルと解析的BRDFを一緒に表示と比較を行うことができる、新しいBRDFビュアーを開発した。測定マテリアルのデータを見る新しく直観的な方法を発見し、知られたモデルにはうまく表現されていなかった、測定モデルの興味深い特徴を見つけた。

このコースノートでは、測定データにフィットするモデルとそのモデルに足りないものについて収集した(glean)洞察に加えて、測定マテリアルを研究することで得られた結果を共有したい。そして、すべての現行のプロダクションで使われている我々の新しいモデルを提供したい(present)。また、プロダクションでこの新しいモデルを採用した経験を説明し、どのようにして単純さ(simplicity)と頑強さ(robustness)を維持しつつアーティストの制御機能を正しいレベルで追加することができたかを論じたい。

## マイクロファセットモデル(The microfacet model)

マイクロファセットモデルは、表面の反射が光のベクトル$\boldsymbol l$と視線のベクトル$\boldsymbol v$との間で起こる得るなら、$\boldsymbol l$と$\boldsymbol v$の中間に平行な(alined halfway)法線を持つ表面またはマイクロファセットの一部が存在することを仮定する(postulate)。ときおりマイクロサーフェスの法線として参照される、この"ハーフベクトル(half vector)"は$\boldsymbol{h} = \frac{\boldsymbol{l} + \boldsymbol{v}}{|\boldsymbol{l} + \boldsymbol{v}|}$として定義される。等方的な材質のためのマイクロファセットモデルの一般的な形式は以下で表される。

$$
f(\boldsymbol{l}, \boldsymbol{v}) = \text{diffuse} + \frac{D(\theta_h) F(\theta_d) G(\theta_l, \theta_v)}{4 \cos\theta_l \cos\theta_v}
$$

ディフューズ項は形式不明の関数である。しばしば想定されるLambertでは定数値で表現される。スペキュラ項において、$D$はマイクロファセットの分布関数であり、スペキュラのピーク形状の要因となる。$F$はFresnel反射係数である。$G$は幾何減衰(geometric attenuation)、もしくは、シャドウイングのファクタである。

$\theta_l$と$\theta_v$は法線に対する$\boldsymbol l$と$\boldsymbol v$の入射角である。$\theta_h$は法線とハーフベクトルとのなす角である。 $\theta_d$は$\boldsymbol l$とハーフベクトル(もしくは、対称的に考えて、$\boldsymbol v$と$\boldsymbol h$)との"差"の角度である。

ほとんどの物理的にもっともらしいモデルは、マイクロファセットの形式で具体的に記述されていなくても、それらが分布関数、Fresnelファクタ、そして、幾何学的シャドウイングファクタと見なせる追加のファクタを持つという点で言えば、マイクロファセットモデルであると解釈することができる。マイクロファセットモデルとその他のモデルとの間で唯一異なるのは、マイクロファセットの導出に由来する明示的ファクタ$\frac{1}{4 \cos\theta_l \cos\theta_v}$が含まれているかどうかである。このファクタが含まれていないモデルでは、暗黙の(implied)シャドウイングファクタを、$D$と$F$を因数分解したあとで、モデルへ$4 \cos\theta_l \cos\theta_v$をかけることにより定めることができる。

## 測定BRDFの可視化(Visualizing measured BRDFs)

### The "MERL 100"

![MERL100に含まれるBRDFの画像スライス](assets/Figure1.png){#fig:1}

等方的なBRDFマテリアルのサンプル100種が @Matusik2003 によりキャプチャされた。そこには、塗料、木材、金属、織物、岩石、ゴム、プラスチック、その他の合成材質を含む広範囲のマテリアルが網羅されている。このデータセットは[Mitsubishi Electric Research Laboratories](http://www.merl.com/brdf/)から無料で手に入れることができ、一般的には新しいBRDFモデルの評価に使われる。これらのBRDFのスライスを[@fig:1]に示す。

MERL100の各BRDFは、$\theta_h$軸、$\theta_d$軸、$\phi_d$軸にそってそれぞれが$90$、$90$、$180$の三次元へ密にサンプルされる。これらは、スペキュラのピーク近くにデータサンプルが集中するように歪ませてある(warped)$\theta_h$軸を除いて、$1$度刻みになっている。これは、データが扱いやすいという点では良いことだが、特に水平近くでデータがどれだけ正確なのかについて明確でない。このため、一部の研究者はフィッティングを行うときに水平近くのデータを破棄しているが、このデータはマテリアルの外観(appearance)における重大な(profound)効果を持つ可能性を考えるためにはいまだ有効である。

### BRDFエクスプローラー(BRDF Explorer)

![ディズニーのBRDFエクスプローラー](assets/Figure2.png){#fig:2}

MERLの測定マテリアルを調べて(examine)、解析的モデルと比較するため、我々は[@fig:2]で示される、新しいツールであるBRDFエクスプローラーを開発した。ソースコードは[GitHub](https://github.com/wdas/brdf)から手に入れることができる。BRDFエクスプローラーは以下の機能を有する。

- GLSLで描かれた解析的BRDFの読み込み。
- @Ngan2005 がキャプチャした異方的マテリアルのサンプルを含む測定BRDFの読み込み。
- 複数データのプロット。(3D半球表示、極座標、様々なデカルト座標系)
- 計算済みアルベドのプロット。(半球での方向における反射率)
- 露出制御付き画像スライス表示。
- 重点サンプリングされたIBLによって照らされたオブジェクトの表示。
- 照らされた球の表示。
- パラメトリックモデルのためのUIによる動的制御。

このツールは、新しいモデルの開発と同様に、測定マテリアルと既存の解析的モデルとの比較において非常に貴重(invaluable)であった。驚くことに、モデルのパラメータとBRDF空間に対するより深い理解をもたらすことから、アーティストのためのインタラクティブなBRDFエディタとしても非常に有用であることが判明した。

### 画像スライス(Image slice)

![*赤いプラスチック(red-plastic)*と*光沢のある赤いプラスチック(specular-red-plastic)*のBRDF画像スライスと"スライス空間"の概略図(schematic view)](assets/Figure3.png){#fig:3}

測定データを可視化する最も単純で最も直観的な方法のひとつは、単に画像の束として見ることであり、我々は、これがデータに関する直観を得るための非常に強力なツールであることを発見した。結論として、MERL100のマテリアルの興味深い特徴はすべて$\phi_d = 90$のスライスに見ることができる。この空間の概略図は、2つのマテリアルのサンプルと合わせて、[@fig:3]で示される。その他のスライスは、[@fig:4]で示される通り、$phi_d = 90$のスライスに対して大まかに(roughly)歪んでいるだけである。この観察結果は、$f(\theta_h, \theta_d)$の形式に簡単化した等方的なBRDFモデルの基礎として、@Romeiro2008 や @Pacanowski2012 のような近年の研究で利用されていた。

![$\phi_d$が異なるときの*光沢のある赤いプラスチック(specular-red-plastic)*のスライス。右上の黒の範囲は$\boldsymbol l$か$\boldsymbol v$のいずれかが水平線の下に潜り込むBRDF領域の部分を表す。](assets/Figure4.png){#fig:4}

画像スライスでは、左端がスペキュラのピークを表し、上端がFresnelのピークを表す。下端では光のベクトルと視線のベクトルが完全に一致する(coincident)。すなわち、下端は再帰反射(retroreflection)を表す。特に右下はグレージング(grazing)再帰反射を表す。ディフューズ反射率はBRDF空間全体で表されるが、一般的には画像の中央をディフューズ応答として特定する(isolate)。

[@fig:3]の概略図には$\theta_l$または$\theta_v$の等値線(isoline)も含まれている。多くのディフューズ効果はこの等高線(contour)に従う傾向にある。これらの等値線は$\phi_d$をゼロに近づけると直線になり、$\phi_d$のスライスを比較することで、マテリアル応答のどの部分がディフューズ反射に由来しているかと、どの部分がスペキュラ反射に由来しているかについての洞察を得られる。もうひとつのヒントはもちろん色についてである。つまり、ディフューズ反射率はその表面に起因し、(表面が金属のように)色付けされているわけではない(is not tinted)。


## MERLマテリアルの観察結果(Observations from MERL materials)

### ディフューズの観察結果(Diffuse observations)

![ディフューズ色の種類を表すマテリアル群。上段:レンダリングされた球における点光源の応答。下段:BRDF画像スライス。](assets/Figure5.png){#fig:5}

ディフューズ反射率は、表面に対して屈折(refract)したり、散乱(scatter)したり、一部が吸収(absorb)したり、再放出(re-emit)したりする光を表す。一部の光が吸収されると、ディフューズ応答は表面の色で色付けされる。すると、色付けが行われた非金属マテリアルの部分はディフューズであるとみなすことができる。

![MERL100のマテリアルにおける自己反射の応答。左:50の滑らかなマテリアル($f(0) > 0.5$)。右:50の粗いマテリアル($f(0) < 0.5$)。$\theta_h = 0$近くのピークはスペキュラのピークであり、$\theta_h = 90$近くのピーク(または、落ち込み(drop))はグレージング自己反射を表す。](assets/Figure6.png){#fig:6}

![*赤いプラスチック(red-plastic)*、*光沢のある赤いプラスチック(specular-red-plastic)*、ランバートディフューズの点光源に対する応答。](assets/Figure7.png){#fig:7}

Lambertディフューズモデルは、屈折光が、指向性(directionality)が完全に失われてしまうくらい十分に散乱していると仮定する。したがって、ディフューズ反射率は定数[^lambert_diffuse_reflectance_is_constant]になる。しかし、Lambert的な応答を表すマテリアルは非常に少ないことを[@fig:1]と[@fig:5]における様々な画像スライスに見ることができる。

[^lambert_diffuse_reflectance_is_constant]: _ Lambertシェーダに含まれる$\boldsymbol{n} \cdot \boldsymbol{l}$はライティングの積分の一部であり、BRDFの部分ではないことに注意。

[@fig:6]に示されるように、多くのマテリアルにはグレージング自己反射での落ち込みが見られ、その他大勢にはピークが見られる。これは、画像スライスにおける見かけ上の(apparent)色付けに起因するディフューズ現象であるように見える。これはラフネスと強く相関している。つまり、より高いスペキュラピークをもつ滑らかな表面は輪郭が暗くなる(have a shadowed edge)傾向があり、粗い表面は逆に明るくなる(a peak instead of a shadow)傾向がある。この相関は自己反射の応答曲線や[@fig:7]のレンダリングされた球に見ることができる。

滑らかな表面において輪郭が暗くなる現象(grazing shadow)はFresnelの式により予測される。つまり、グレージング角では、表面で反射するエネルギーが多くなり、屈折してディフューズ的に再放出するエネルギーは少なくなる。しかし、ディフューズモデルは、Fresnelの屈折における表面のラフネスによる効果を考慮せずに、表面が滑らかであると仮定するか、Fresnel効果を無視するかのいずれかであることが多い。

Oren-Nayarモデル [-@Oren1994] は、ディフューズ形状を平坦化するような、粗いディフューズ表面における再帰反射の増加を予測する。

![Lambert、Oren-Nayar、Hanrahan-KruegerのディフューズモデルにおけるBRDFスライスと点光源の応答](assets/Figure8.png){#fig:8}

しかし、この再帰反射のピークは測定データほど強くはなく、粗い測定マテリアルは一般的にディフューズの平坦化を示さない。表面下散乱の理論から導き出されたHanrahan-Kruegerモデル [-@Hanrahan1993] はディフューズ形状の平坦化を予測するが、その輪郭においてあまり強いピークを持たない。Oren-Nayarとは対照的に、このモデルは完全に滑らかな表面を仮定する。Oren-NayarモデルとHanrahan-Kruegerモデルは[@fig:8]でその比較が示される。

再帰反射のピークの他にも、[@fig:5]の画像スライスにはさらなる(additional)ディフューズの変化(variation)が見られる。強度(intensity)と色の両方の変化は$\theta_l / \theta_v$の等値線に沿って見られる。これは、場合によっては、層状の(layered)表面下散乱によるものである可能性がある。しかし層状の表面下散乱モデルであっても一般的にその表面は滑らかであると考え、強い再帰反射のピークを生み出さない。

### スペキュラにおけるDの観察結果(Specular D observations)

マイクロファセット分布関数$D(\theta_h)$は、[@fig:6]が示すように測定マテリアルにおける再帰反射の応答から観察することができる。そのマテリアルは、表面のラフネスの指標(indication)とすることができる、ピークの高さに基づいて2つのグループに分けられる。最も高いピークは *鋼(steel)* で400以上あった。ピークが平坦になっていれば、曲線の残りの部分はディフューズ反射率由来である可能性が高い。MERLマテリアルの大多数は伝統的なスペキュラモデルより長いテールを有するスペキュラローブを持つ。例として、*クロム(chrome)* のサンプルを[@fig:9]に示す。このマテリアルのスペキュラ応答は、滑らかで高度に研磨された(highly polished)表面においては典型的であり、2、3度の幅(a couple of degrees wide)しかないスペキュラピークと何倍も広いスペキュラテールを持つ。奇妙なことに(curiously)、伝統的なBeckmann、Blinn Phong、Gaussianといった分布はこの幅においてはほぼ同じ(nearly identical)であり、そのピークとテールにおいてはいずれもうまく表現できていない。

![MERLのクロムにフィットしたスペキュラ分布。左:$\theta_h$に対するスペキュラピークの対数スケールのプロットで、黒はクロム、赤はGGX($\alpha = 0.006$)、緑はBeckmann($m = 0.013$)、青はBlinn Phong($n = 12000$)。右:クロム、GGX、Beckmannにおける点光源の応答。](assets/Figure9.png){#fig:9}

より広いテールが求められていたことは @Walter2007 がGGXを導入した動機になった。GGXは他の分布より長いテールを持つが、いまだクロムのサンプルの輝く(glowy)ハイライトを捉えるまでには至っていない。測定マテリアルをフィットさせるようにテールの応答をモデリングすることの重要性は @Loew2012 や @Bagher2012 といった最近のモデルの基礎ともなった。これらのモデルは両方とも、ピークとは別にテールを制御する追加のパラメータが加えられている。テールをモデリングするもうひとつの選択肢は、@Ngan2005 が提案したような、1つ目に加える形でより広い2つ目のスペキュラピークを使うことである。

GGXは他の分布関数より長いtailを持つが，上図のクロムのデータと比べるとtailは短い．他に，Löw et al.やBagher et al.は，ピークとは別にtailを調節することができるパラメータを追加したモデルを提案している．また，Nganははじめのピークに追加の広めのピークを足し合わせる方法を提案している．

### スペキュラにおけるFの観察結果(Specular F observations)

TODO

$F(\theta_d)$は$\theta_d$が大きくなるとspecular反射が強くなる現象を表現する．これは，なだらかな面ではgrazingな入射光を100%近く反射し，粗い面では100%にはならないが，よりspecularっぽい反射になる．

上図を見ると，計測値はgrazing angle付近でFresnel効果による予測より傾斜がきついのが分かる．これは，Torrance-Sparrow microfacetモデル[^Torrance-Sparrow]が高入射角で見られるoff-specular peak[^off-specular_peak]を説明する動機付けになった．

Microfacetモデルは，係数$\frac{1}{4 \cos\theta_l \cos\theta_v}$はgrazing angleで無限大に近づくが，microfacetのshadowing-masking[^shadowing-masking]効果によりgrazingな反射が減少するため，shadowing-maskingを表現する$G$項は$\frac{1}{4 \cos\theta_l \cos\theta_v}$との組み合わせにより事実上(effectively)Fresnel効果を増幅する．[^Amplified_Fresnel]

### Specular G(とalbedo[^albedo])の観察結果
diffuseからspecularを分離するがごとく，$D$や$F$の正確な推定が必要になるので，測定データから$G$項を分離するのは難しい．しかし，$G$の効果はdirectional albedo[^directional_albedo]上に間接的に見ることができる．

![][fig-albedo]

上図では，ほとんどの材質のDirectional albedoは70°までは比較的平らになっていて，grazing angleでは面の粗さにより異なる．なだらかな面では75°までわずかに上昇し，90°まで減少する．粗い面では全体にわたって増加する．注目すべき点として，極小数の材質が0.3を超える程度で全体的に極めて低い値になっている．

![][fig-specularG]

上図は$G$項のモデルに応じたalbedo応答を表す．"No G"は$G$と$\frac{1}{4 \cos\theta_l \cos\theta_v}$を完全に除去したモデルで，grazing angleで大きく減少するのが分かる．

前提を単純化することにより，Smithの方法で分布関数$D$からshadowing関数を導出することができる．WalterやSchlickはこの方法を用いている．図にあるように，WalterのSmithモデルには，測定データでは見られない，なだらかな面におけるgrazing angle下での著しい増加が確認できる．粗い面になれば，よりもっともらしく(plausible)なる．Smithの$G$はわずかな関数のみからなる解析的な形式を持ち，積分表による積分か他のある種の近似がよく用いられる．

Kurt et al.はfree parameter付きのdata-fittingモデルを提案している．図では$\alpha = 0.25$のときを表している．懸念事項として，Kurtのモデルはgrazing angle近くで発散する．特に粗い分布関数で顕著に現れる．別の選択肢としては，Smith$G$のWalterによる派生型か，より単純なSchlickのものを使い，free parameterとして$G$のroughnessを切り離す(decouple)方法がある．

## Disneyの"原則に基づいた"BRDF
### 原則
新しい物理ベースの反射モデルを開発するさいに，アーティストからart-directable[^art-directable]で必ずしも物理的に正確でないシェーディングモデルが必要だと警告されたため，厳密に物理的なものよりも"原則に基づいた"モデルを開発することを方針とした．モデルを実装するときに準拠した原則は以下の通り．

1. 物理的なパラメータよりも直感的なパラメータを使うべし．
1. 出来る限りパラメータを少なくすべし．
1. パラメータはそのもっともらしい範囲を0から1にすべし．
1. 道理にかなうときには，パラメータがそのもっともらしい範囲を超えることを許可するべし．
1. パラメータの組み合わせのすべてが可能な限り堅牢(robust)でもっともらしくあるべし．

各種パラメータの取捨選択を議論した結果，下記の1カラーと10スカラーに決まった．

### パラメータ
![][fig-brdf_parameters]

- baseColor
  : 表面色．通常はテクスチャマップにより与えられる．
- subsurface
  : 表面下(subsurface)を近似してdiffuse形状(shape)を制御する．
- metallic
  : 金属っぽさ．0に近づくにつれて誘電体(dielectric)っぽく，1に近づくにつれて金属っぽくなる．金属モデルはdiffuse要素を持たず，ベースの色でspecularに色が付く．
- specular
  : 入射する(incident)specularの量．このパラメータは屈折率を陽に表す(?)．[^This_is_in_lieu_of_an_explicit_index-of-refraction]
- specularTint
  : 入射するspecularをベースの色に近づけるためにアーティストによる制御用に許容されたもの．grazingなspecularは無色のまま．
- roughness
  : 表面の粗さ．diffuseとspecularの両方を制御する．
- anisotropic
  : 異方性の程度．specularハイライトのアスペクト比を制御する．
- sheen  :
追加のgrazingの要素．主に布用．
- sheenTint
  : sheenをベースの色に近づける量．
- clearcoat
  : もうひとつの特殊用途のspecular lobe．
- clearcoatGloss
  : clearcoatの光沢を制御する．0に近づくにつれてサテンっぽく，1に近づくにつれてグロスっぽくなる．

### Diffuseモデルの詳細
いくつかのモデルは以下のようなFresnel項を含んでいる．

$$
(1 - F(\theta_l))(1 - F(\theta_d))
$$[^Fresnel_factor]

ここで，$F(\theta)$はFresnel反射の項を表す．

Lambert diffuseモデルはエッジが暗くなりすぎるため，物理的なもっともらしさのためにFresnel項を追加するだけではより暗くなってしまう．

そこで，なだらかな面では影が付き，粗い面ではハイライトが付くような再帰性反射モデルを開発した．このモデルによる効果は，粗い面ではgrazing angleでの屈折が増加する原因であるmicrofacet(?)に入って出るためだと言えるかもしれないが，いずれにしろ，このモデルはアーティストが好み，以前に使っていたad-hocなモデルの機能に類似している．

このモデルは，Fresnel項の屈折率を無視し，入射するdiffuseの損失がないことを仮定する．これは，入射するdiffuse色を直接特定することが可能になる．ベースとなるdiffuseモデルの式を以下に示す．

$$
f_d = \frac{baseColor}{\pi}(1 + (F_{D90} - 1)(1 - \cos\theta_l)^5)(1 + (F_{D90} - 1)(1 - \cos\theta_v)^5)\\
where\\
F_{D90} = 0.5 + 2 \cos\theta_d^2 roughness
$$

この式では，入射するdiffuseの反射率がなだらかな面では0.5倍になり，粗い面では2.5倍になるようなFresnelを生成する．これは，MERLのデータとほどよく一致させ，アーティスト的にも満足するものになったように思われる．

![][fig-detail_diffuse]

subsurfaceパラメータはベースとなるdiffuse形状とHanrahan-Kruegerのsubsurface BRDFに触発されたものとをブレンドする．これは遠方の物体や，scatteringにおけるパスの平均の長さが短い物体にsubsurfaceな見た目を与えるには有用であるが，影または表面ににじむことがないように，完全なsubsurface transportの代替にはならない．[^full_subsurface_transport]

### Specular Dの詳細
よく使われるモデルの中でGGXが最も長いtailを持つが，多くのマテリアルに対してtailの長さが十分ではない．このモデルは実験データとの一致性がBlinnにより支持されたTrowbridge-Reitzの分布と等価である．ここでは，**Generalized-Trowbridge-Reitz**; **GTR** として一般化した式を導入する．

$$
D_{GTR} = c / (\alpha^2 \cos^2\theta_h + \sin^2\theta_h)^\gamma
$$

ここで，$\gamma = 1$のとき，Berryの式と一致し，$\gamma = 2$のとき，Trowbridge-Reitzの式と一致する．$c$はスケーリング用定数を表す．$\alpha$はroughnessパラメータを表し，0で完全になだらかな分布を，1で完全に粗い分布を生成する．

![][fig-GTR]

また，$\gamma = \frac{3}{2}$のときのGTRは$\theta = 2 \theta_h$としたときのHenyey-Greenstein phase関数と等価である．

plausibleなmicrofacet分布は正規化されていなければならず，効率的なレンダリングを行うにはimportance samplingに対応しなければならない．分布が半球上で積分可能であるには，この両方を満たさなければならない．幸いにも，この関数は単純なclosed-form integral[^closed-form_integral]である．効率的な異方性の形式と同じように正規化とimportance sampling関数はAppendix Bにしたがって導出される．

DisneyのBRDFでは，GTRを用いた2つのspecular lobeを持つ．1つ目(primary lobe)は$\gamma = 2$で2つ目(secondary lobe)は$\gamma = 1$になる．Primary lobeはベースとなる材質を表し，異方・金属的になり，secondary lobeはベースの材質にまたがるクリアコート層を表し，常に等方・非金属的になる．

roughnessについては，$\alpha = {\rm roughness}^2$とすることでより線形な変化として認識されることを発見した．これがないと，光沢のあるマテリアルに合わせるときに，とても小さくて非直感的な値になってしまっていた．

明示的な屈折率(IOR)を用いる代わりとして，specularパラメータは入射するspecularの量を決定する．このパラメータの正規化範囲は入射specularにおける$[0.0, 0.08]$へ線形にマップされる．この範囲は屈折率での$[1.0, 1.8]$の範囲に対応し，この範囲にはほとんどの基本マテリアルが含まれている．特に，範囲の中央はIORの$1.5$に対応し，とても典型的な値なので，Disneyのモデルではこの値をデフォルトとしている．*specular* パラメータはより高いIOR値になるように1を超えて指定することができるが，これは注意深く利用すべし．現実世界の入射の反射率が直感に反してとても低くなると考えると，アーティストにもっともらしいマテリアルを作ってもらうときに，このマッピングは大いに役に立った．

clearcoat層では，ポリウレタンの代表値であるIOR$1.5$の固定値を利用し，代わりに，アーティストが*clearcoat*パラメータを使用している層の全体強度(strength)をスケールできるようにする．正規化したパラメータの範囲は$[0, 0.25]$のoverall scale[^overall_scale]に対応する．この層は，ビジュアルに大きな影響があるにも関わらず，相対的に小さなエネルギー量を示す．そこで，ベース層からいずれのエネルギーも差し引かないことにした．また，0に設定すると，clearcoat層は事実上無効化され，無駄が生じない(incurs no cost)．

### Specular Fの詳細
Disneyの目的では，SchlickのFresnel近似は十分で，完全なFresnel式よりおおむね簡単であり，加えて，近似ににより引き起こされるエラーはその他の要因によるエラーよりはるかに小さい．

$$
F_{\rm Schlick} = F_0 + (1 - F_0)(1 - \cos\theta_d)^5
$$

ここで，定数$F_0$は直角入射におけるspecular反射率を表し，誘電体では無色になり，金属では有色になる．実際の値は屈折率に依存する．Specular反射はmicrofacetで行われることに注意されたし．すなわち，$F$は，入射方向と法線のなす角ではなく，光源方向とhalf-vectorのなす角である$\theta_d$に依存している．

Fresnel関数は入射specular反射とgrazing angleでのunity[^unity]間の(非線形)補間とみることができる．すべての光が反射されるように，応答がgrazing incidenceで無色になることに注意されたし．

### Specular Gの詳細
Smithのshadowing要素がprimary specularで利用可能なことを考えて，GGXのためにWalterのより導出された$G$を使い，光沢のある表面における極端なゲインを減らすため元のroughness$[0, 1]$を$[0.5, 1]$にリマップする．最終版では$\alpha_g = (0.5 + {\rm roughness} / 2)^2$とした．

このマッピングは，specularが小さなroughness値でいささか強すぎるというアーティストのフィードバックとともに，計測データとの比較に基いている．これはroughnessで変化し，少なくとも一部は物理ベースで，もっともらしく見える$G$関数をもたらす．Clearcoat specularでは，Smithの$G$を使わず，単に固定のsoughness値$0.25$をとしたGGXの$G$を使う．

## 参考文献
[Physically-Based Shading at Disney (notes), SIGGRAPH 2012](http://blog.selfshadow.com/publications/s2012-shading-course/burley/s2012_pbs_disney_brdf_notes_v3.pdf)

[Simon's Tech Blog: Microfacet BRDF](http://simonstechblog.blogspot.jp/2011/12/microfacet-brdf.html)

[CS6630: Realistic Image Synthesis Spring 2012 -- Cornell University Department of Computer Science](http://www.cs.cornell.edu/Courses/cs6630/2012sp/slides/05ufacet.pdf)

[再帰性反射 -- Tari Lari Run](http://bygzam.seesaa.net/article/134477830.html)

[表面が粗い物体のディフューズライティングで起こる現象 : Retro-Reflection(再帰性反射) -- hanecci's Blog](http://d.hatena.ne.jp/hanecci/20130519/p1)

[Torrance Sparrow モデルに対する解釈](http://www21.ocn.ne.jp/~glass-cg/cg/ts_explanation.pdf)

[Ken Torrance’s accomplishments -- Real-Time Rendering](http://www.realtimerendering.com/blog/ken-torrances-accomplishments/)

[Deriving the Smith shadowing function for the GTR BRDF -- CHAOSGROUP](http://docs.chaosgroup.com/display/RESEARCH/Deriving+the+Smith+shadowing+function+for+the+GTR+BRDF)

[Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs (1) -- graphics.hatenablog.com](http://graphics.hatenablog.com/entry/2014/02/12/060548)

[ゲームグラフィックス特論 第9回 高度な陰影付け](http://www.slideshare.net/tokoik/ss-13348390)

[閉形式(closed-form)とは -- minus9d's diary](http://minus9d.hatenablog.com/entry/20130624/1372084229)

<!-- 参考画像 -->
[fig-microfacet]: microfacet.png

[fig-grazing_retro-reflection]: grazing_retro-reflection.png

[fig-specularD]: specularD.png

[fig-specularF]: specularF.png

[fig-albedo]: albedo.png

[fig-specularG]: specularG.png

[fig-brdf_parameters]: brdf_parameters.png

[fig-detail_diffuse]: detail_diffuse.png

[fig-GTR]: GTR.png

<!-- 脚注 -->
[^microfacet]: micro- + facet(小さな面)

[^NDF]: Disneyの資料中では *Microfacet Distribution Function* だが，このテキストではその他の資料に合わせて *Normal Distribution Function; NDF* とする．

[^MERL]: [http://www.merl.com/brdf/](http://www.merl.com/brdf/)

[^directionality]: 有向性．ここではdiffuseのことなので，方向に関する要素に対して拡散の程度に何らかの相関があるということ．

[^Lambertian_response]: Lambert反射の性質．responseは反射や屈折をなどの光に対する応答(反応)のこと．

[^grazing_angle]: 面スレスレの角度，法線とのなす角が90°に近い

[^tail]: 単調減少するグラフの後半部分．long tailは後半部分の減少量が少なく，より漸近的であること．

[^specular_lobe]: retro-reflectionのspecular強度を半球にプロファイルしたもの．

[^Torrance-Sparrow]: Microfacetのshadowing-masking効果を導入することでoff-specular peakを理論的に説明するモデル．

[^off-specular_peak]: 粗い面でspecularのピークが鏡面反射方向からずれて観測される現象．

[^shadowing-masking]: shadowingはmicrofacetが入射光を遮る現象のこと．maskingはmicrofacetが反射光を遮る現象のこと．

[^Amplified_Fresnel]: 式的には，$G$項は$\frac{1}{4{\rm cos}\theta_l {\rm cos}\theta_v}$が$+\infty$に発散するよりも早く$+0$に収束するため，grazing angleにおいて$\frac{G}{4{\rm cos}\theta_l {\rm cos}\theta_v} < 1$となるためだと考えられる．

[^albedo]: 入射光に対する反射率の比．アルベド．

[^directional_albedo]: ある一方向に対するアルベド．

[^art-directable]: アーティストが思い通りの調整ができる．

[^This_is_in_lieu_of_an_explicit_index-of-refraction]: パラメータ*specular*で屈折率を表現できる，という意味か．原文だと，"This is in lieu of an explicit index-of-refraction."

[^Fresnel_factor]: 屈折に対するFresnelの法則，および，Helmholtzの相反性(reciprocity)により，表面下に入るときと出るときの2回分を説明する必要がある．

[^full_subsurface_transport]: あくまでシェーディングの話なのでGI的なことはできない，ということか．

[^closed-form_integral]: 閉形式の積分．閉形式は有限個の良く知られた(well-known)関数によって解析的に表される式のこと．ようは積分が上手に解けるということか．

[^overall_scale]: 要調査

[^unity]: 要調査
