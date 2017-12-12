---
title: >
Decima Engine: Advances in Lighting and AA [@Carpentier2017]
numberSections: false
---
# デシマエンジン[Decima Engine]

コイツが何なのか知らない人たち用:
Decima EngineはKillzoneシリーズのためにGuerrilla Gamesによって作られたゲームエンジンである。
しかし、より大きくより美しい世界をサポートするため、それ以来拡張と最適化を行い続けた。そこには、Horizon Zero Dawnのそれも含まれる。
そして近年、コジマプロダクションとのDecima Engineの共有も始まり、彼らは現在それをDeath Strandingの開発に用いている。

# トピックス[Topics]

- GGXの球エリアライト
- 高さフォグ
- 1080pにおけるAA
- PS4Proでの2160pチェッカーボード

# GGX球エリアライト[GGX spherical area light]

- 形状上でGGXを積分するのは難しい。
- 様々な近似アプローチ
    - パフォーマンス vs クオリティ
    - 形状固有(例えば、多角形ライトの[@Hill2016])
- 球ライトでは: ポイントライトを'たわませる[warp]'安価なトリック[@Karis2013]
    - 歪みを引き起こし得るが、改善できる。

Decima Engineは恐らくエリアライトを用いるライティングをサポートする最初のゲームエンジンのひとつであった。
ただし、初期の実装はGGXをサポートしておらず、我々を制限する要因となっていた。
そこで、最近エリアライトシステムをアップグレードし始め、まずは我々の球エリアライトにGGXを追加しようとした。
しかしながら、GGXのようなマイクロファセットBRDFをエリアライトに統合することは難しく、ある種の近似的なテクニックを必要とする。
実際に、まさにそれを行うためにいくつかの異なるテクニックがここ数年で出現してきた。これらのテクニックのいくつかはパフォーマンスよりクオリティを優先しており、いくつかはその逆を優先している。そして、これらのテクニックの多くは形状タイプのただひとつに対して特に適するようになっている。我々の新しい球エリアライトでは、色々と調べた結果、Brian Karisによって述べられるライト曲げ[bending]トリックを選択することとした。
これは安価であり、多くの様々なゲームエンジンで上手く使われている。
そして、その結果はしばしばかなりまともである。
しかし、特にグレージング角で、予期せぬ形状が現れることがある。
左側の画像でそれをはっきりと確認することができる。しかし、我々はそのテクニックを改善する方法をひとつ見つけ、右側の画像に見えるものがそれである。では、これはどうやって動作するのだろう？

<!-- p.6 -->

- どうやってピクセルあたり1つのポイントライトを用いてエリアライトを近似するのだろう？

まずはBrian Karisのテクニックを調べることから始めよう。その発想は単一のポイントライトで右図のような結果を近似したいということである。
そして、反射ベクトルの方向でエリアライトの外周[perimeter]に向かってポイントライトを動かすことでそれを行うことができる。

<!-- p.7 -->

- ピクセルの反射ベクトルに向かってポイントライトを動かすことで[@Karis2013]
    - Phongモデルに対して素晴らしい[@Picott1992]
    - マイクロファセットモデルではあまり良くない。依然としてピーク応答を'取り逃がす'可能性がある。

これはPhongベースのライティングではとてもうまく動作し、1992年にはすでに述べられていた。
しかし、GGXのようなマイクロファセットベースのモデルではそのように常にうまく動作するわけではない。
これはポイントライトに対して選択される位置が最も支配的な影響を常に上手に代表するわけではないためである。

言い換えれば、ここでの一般的なアイデアはピクセルにピークを持ち込むことであるが、この方法だとピクセルは依然としてピークを取り損なう可能性がある。

<!-- p.8 -->

- アイデア: **その応答を最大化する** ために外周上でポイントライトを動かす。
    - 支配因子[dominant factor]: $N \cdot H$

代わりにしたいことは、エリアライトの外周における異なる位置にポイントライトを動かすことである。
そして、我々はポイントライトが最も強くピクセルを照らすであろう位置を選択する。
Fresnel項、ジオメトリ項、$N \cdot L$項を無視すると、これは法線$N$とハーフベクトル$H$が最も近いときに起こる。

言い換えれば、我々は$N \cdot H$を最大化するライトベクトルを探したい。

<!-- p.9 -->

- 反射ベクトルに向かって光を曲げる[@Karis2013]。

この問題を解決するため、ちょっとした数学を必要になる。なので、必要な変数を定義するところから始めよう。

法線$N$、視線ベクトル$V$、反射ベクトル$R$があるとする。
そして、球エリアライトがあるとする。

このライトの中心が単位方向$L_c$にあると仮定しよう。
そして、その距離上の外周の半径に等しい正弦$s$で球の形状を定義しよう。

<!-- p.10 -->

$$
T = \frac{R - (L \cdot R) L}{\|R - (L \cdot R) L\|} \\
L = \sqrt{1 - s^2} L_c + sT
$$

$$
N \cdot H = H \cdot \frac{L + V}{\|L + V\|}
$$

$L_c$と正規直交でありつつ$R$を指す単位ベクトル$T$も定義しよう。
ようは、$T$は$L_c$に垂直な平面へ投影した反射ベクトル$R$であり、再正規化されている。

この単位ベクトル$T$を使って、ライトの外周を指し、反射ベクトル$R$に最も近い単位ベクトル$L$を簡単に見つけることができる。
すでに得ている正弦$s$と$s$から計算できる余弦を用いて$L_c$を$L$に回転することでこれを行う。

そして、この$L$はハーフベクトル$H$を計算するのに用いることができる。

これがまさにKarisのアプローチで用いられるハーフベクトルである。

<!-- p.11 -->

- 代わりに、$L \cdot H$を最大化するために$L$を曲げる。

$$
B = T \times L_c \\
L = \sqrt{1 - s^2} L_c + s(\cos(\varphi) T + \sin(\varphi) B)
$$

- $N \cdot H$を最大化する$\varphi$は何だろう？

我々は$B$と呼ぶ別のベクトルを導入することでこのアイデアを容易に拡張できる。

$B$はbi-tangent^[従接線？]であり、$L_c$、$T$、$B$と共に正規直交基底を作る。
エリアライトの外周にある任意の点を指す正規化されたライト方向$L$を定義するためにこれを用いることができる。

…角度$\varphi$により外周上を回転するものを含む。

元の問題を振り返ってみると、このトリックは最も大きい$N \cdot H$へと導く$\varphi$を見つけることである。
この方法で、陰影付けしたいピクセルに対するより支配的なスペキュラ応答を見つけるだろう。
そして今度は、これがエリアライトにより良い近似をもたらす。

<!-- p.12 -->

- $N \cdot H$を最大化する$\varphi$を解くのは難しい。
    - sqrt、sin、cos…
- 代わりに、等価な問題を解く。
    - $\varphi$の代わりに$x = \tan(\varphi / 2)$を解く。
    - $N \cdot H$の代わりに$f(x) = (N \cdot H)^2$を最大化する。
- $f(x)$は有理式[rational polynomial]として書き直すことができる。
    - $f(x) = \frac{ax^4 + bx^3 + cx^2 + dx^2 + e}{gx^4 + hx^3 + ix^2 + jx^2 + k}$
- 繰り返し$f(x)$を最大化する。

$\varphi$を直接解くことはむしろ難しい。
これは関係する大量の平方根、正弦、余弦が存在するためである。

しかし、より解きやすい等価な問題にこの問題を再定式化できる。
例えば、角度$\varphi$を解く代わりに、半分の$\varphi$の正接である$x$を解くことができる。
この導出はここで示すには少々長ったらしい[tedious]であるが、一般的な半角の公式を用いるすべての正弦と余弦を取り除くことができる。
更に推し進めて、$N \cdot H$の代わりに$(N \cdot H)^2$を最大化しようとすることができる。これは残りの平方根から我々を解放する。

そして、最後に残るものは$f(x)$と呼ばれる比較的単純な有理式である。

そして、今すべきことはこの$f(x)$を最大化する$x$を見つけることであり、それは$(N \cdot H)^2$を最大化する。
残念ながら、これは未だに解析的に解くには非自明な問題である。
しかし、未だに反復的なアプローチを用いて数値的にこれを解くことができる。

<!-- p.13 -->

- $x$の初期推定値から始める。
    - $x_0 = 0$
    - $f(x_0)$はKarisのアプローチの$(N \cdot H)^2$に等しい。
- ニュートン法を用いて$x_1$を計算する。
    - $x_{n+1} = x_{n} - \frac{f'(x_n)}{f''(x_n)}$
- 我々の最終$(N \cdot H)^2$として$f(x_1)$を用いる。

反復的なアプローチは推測から始まり、それを改善しようとする。
Karisのアプローチから始める。これは$x = 0$で始めることに等しい。

そして、$x_1$を得るためにニュートン法のイテレーションを一回だけ適用する。
これはすでに最大値にかなり近くなるので、我々は単純に最終的な推定値として用いることとする。

$x_1$を$f(x)$に接続することは我々が探していた$(N \cdot H)^2$をもたらし、GGX関数への入力として直接これを用いる。

<!-- p.14 -->

$N \cdot H$最大化なし[@Karis2013]

```hlsl
float GetNoHSquared(float radiusTan, float NoL, float NoV, float VoL) {
    // radiusTanがディレクショナルライトである場合、radiusCosは事前計算できる。
    float radiusCos = rsqrt(1.0 + radiusTan * radiusTan);

    // Rがディスク内に収まる場合、早期離脱する。
    float RoL = 2.0 * NoL * NoV - VoL;
    if (RoL >= radiusCos)
        return 1.0;

    float rOverLengthT = radiusCos * radiusTan * rsqrt(1.0 - RoL * RoL);
    float NoTr = rOverLengthT * (NoV - RoL * NoL);
    float VoTr = rOverLengthT * (2.0 * NoV * NoV - 1.0 - RoL * RoL);

    // 曲がったライトベクトルに基づいて(N.H)^2を計算する。
    float newNoL = NoL * radiusCos + NoTr;
    float newVoL = VoL * radiusCos + VoTr;
    float NoH = HoV + newHoL;
    float HoH = 2.0 * newVoL + 2.0;
    return max(0.0, NoH * NoH / HoH);
}
```

実装がどれだけ複雑かを感じてもらうため、これがHLSLでどうなるかをここに示す。
これはKarisのアプローチの実装であり、すべてスカラの計算である。

<!-- p.15 -->

$N \cdot H$最大化あり

```hlsl
float GetNoHSquared(float radiusTan, float NoL, float NoV, float VoL) {
    // radiusTanがディレクショナルライトである場合、radiusCosは事前計算できる。
    float radiusCos = rsqrt(1.0 + radiusTan * radiusTan);

    // Rがディスク内に収まる場合、早期離脱する。
    float RoL = 2.0 * NoL * NoV - VoL;
    if (RoL >= radiusCos)
        return 1.0;

    float rOverLengthT = radiusCos * radiusTan * rsqrt(1.0 - RoL * RoL);
    float NoTr = rOverLengthT * (NoV - RoL * NoL);
    float VoTr = rOverLengthT * (2.0 * NoV * NoV - 1.0 - RoL * RoL);

    // dot(cross(N, L), V)を計算する。これはすでに計算され、利用可能である。
    float triple = sqrt(saturate(1.0 - NoL * NoL - NoV * NoV - VoL * VoL + 2.0 * NoL * NoV * VoL));

    // 曲がったライトベクトルを改善するためにニュートン法のイテレーションを一回行う。
    float NoBr = rOverLengthT * triple, VoBr = rOverLengthT * (2.0 * triple * NoV);
    float NoLVTr = NoL * radiusCos + NoV + NoTr, VoLVTr = VoL * radiusCos + 1.0 + VoTr;
    float p = NoBr * VoLVTr, q = NoLVTr * VoLVTr, s = VoBr * NoLVTr;
    float xNum = q * (-0.5 * p + 0.25 * VoBr * NoLVTr);
    float xDenom = p * p + s * ((s - 2.0 * p)) + NoLVTr * ((NoL * radiusCos + NoV) * VoLVTr * VoLVTr + q * (-0.5 * (VoLVTr + VoL * radiusCos) - 0.5));
    float twoX1 = 2.0 * xNum / (xDenom * xDenom + xNum * xNum);
    float sinTheta = twoX1 * xDenom;
    float cosTheta = 1.0 - twoX1 * xNum;
    NoTr = cosTheta * NoTr + sinTheta * NoBr;  // NoTrを更新するために新しいTを使う
    VoTr = cosTheta * VoTr + sinTheta * VoBr;  // VoTrを更新するために新しいTを使う

    // 曲がったライトベクトルに基づいて(N.H)^2を計算する。
    float newNoL = NoL * radiusCos + NoTr;
    float newVoL = VoL * radiusCos + VoTr;
    float NoH = HoV + newHoL;
    float HoH = 2.0 * newVoL + 2.0;
    return max(0.0, NoH * NoH / HoH);
}
```

そして、これがニュートン法のイテレーションを一回分追加したときにどうなるかを示す。
ご覧の通り、これは割と多くの命令の追加を行うが、ライトシェーダ全体を比較すれば、これは比較的小さい変更であるのかもしれない。
これがどれだけ使われるかによって、パフォーマンスの違いは目立つかもしれないし、事実上タダになるかもしれない。

<!-- p.16 -->

ここでは$N \cdot H$最大化ありとなしでの太陽の反射を確認する。
ご覧の通り、小さな角度でより自然に見える反射を得るので、かなりうまく動作している。

しかし、このスクリーンショットがHorizon Zero Dawnで作られているとしても、このテクニックはHorizonを出荷した後に出来上がったことは言及しておくべきだろう。
しかし、内部使用のために増設[retro-fit]され、他のタイトルのために使う用意が整っている。

# 高さフォグ[Height fog]

TODO

# 参考文献[References]
