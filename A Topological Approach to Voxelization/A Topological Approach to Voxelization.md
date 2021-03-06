# まえがき[Introduction]

TODO

## 基本的定義[Basic definitions]

我々の表記法は主にCohen-Or and Kaufman[@COK95]に従う。整数の座標を持つ3次元の点の集合を$\mathbb{Z}^3$する。我々は点$p(x, y, z) \in \mathbb{R}^3$の集合とボクセル$V = (x, y, z) \in \mathbb{Z}^3$を関連付ける。ここで、$V_x \le p_x \le V_x + 1$であり、$y$と$z$に対して同様である。$\mathbb{R}^3$に組み込まれた連続で単純で閉じた2次元多様体の表面$S$を考えるとすると、$\mathbb{R}^3 - S$は過不足なく[exactly]2つの連結成分[connected components]$I$および$O$を持つ。$I$と$O$の全体に含まれるボクセルの空でない集合をそれぞれ$I^d$と$O^d$とする
表面$S$のボクセル化とは$I^d$や$O^d$と共通の要素を持たないボクセルの離散集合$S^d$である。ボクセルの$k$連結経路[k-connected path]$\Pi_k = (V_0, \dots, V_n)$が存在しないならば、$S^d$は$k$分離[k-separating]であると言われる。ここで、$V_{i + 1} \in N_k(V_i)$、すなわち、$V_0 \in I^d$であり、$V_n \in O^d$であり、$\Pi_k \cap S^d \ne \emptyset$である。別の言い方をすれば、$I^d$と$O^d$の間のすべての$\Pi_k$が$S^d$にあるボクセルを含むならば、$S^d$は$S$の$k$分離ボクセル化である。$N_k(V)$はボクセル$V$の$k$近傍の集合であり、$\mathbb{Z}^3$において、我々にとって関心のある近傍集合は以下である。

$$
N_6(x, y, z) = (x \pm 1, y, z) \cup (x, y \pm 1, z) \cup (x, y, z \pm 1)
$$

$$
N_{26}(x, y, z) = {x - 1, x, x + 1} \times {y - 1, y, y + 1} \times {z - 1, z, z + 1} - (x, y, z)
$$

ここで、$\times$はデカルト積[cartesian product]を表す。ゆえに、任意の$V$に対して、$N_6(V)$は6個の要素を持ち、$N_{26}(V)$は26個の要素を持つ。したがって、6連結経路$\Pi_6$はボクセルの面を通って隣接するボクセルをたどる[walk into]ことができるのみであり、26連結経路$\Pi_{26}$ではそれに加えて[as well]対角線上にたどること[diagonal walks]が可能である。紙面の都合により[due to space constraints]、本論では18分離および18連結のボクセル化および経路を考えない[we shall not consider]。
2Dボクセル化においても状況は変わらず、この場合においては、我々はそれらに対応する意味で$S \subset \mathbb{R}^2$、$S^d \subset \mathbb{Z}^2$、$V$、$I$、$O$、$\Pi_k$を用いる。その次元は文脈により明らかとなるだろう。2Dにおける近傍関係[neighborship relations]は以下である。

$$
\begin{aligned}
N_4(x, y) &= (x \pm 1, y) \cup (x, y \pm 1) \\
N_8(x, y) &= {x - 1, x, x + 1} \times {y - 1, y, y + 1} - (x, y)
\end{aligned}
$$

ここで、$N_4$は3Dにおける$N_6$と類似し、$N_8$は$N_{26}$と類似している。近傍集合の図は、例えば、[@HYFK98]に見ることができる。
我々は、その表面が依然として$\mathbb{R}^3$において対応するボリュームを2つの連結成分に分けるようなボクセル空間$\mathbb{Z}^3$の部分集合を考慮することによって、閉じていない表面に分離可能性[separability]の概念を拡張できる。これについては[@fig:1]を参照のこと。閉じている表面と閉じていない表面のどちらでも、対応する$I^d$および$O^d$が空でない場合に制限していることに注意したい。そうでない場合、$\mathbb{Z}^3$で分離する手段が存在しないので、分離可能性を論じる意味がなくなってしまう[it makes no sense to speak of separability]。

![閉じていない表面への$k$分離可能性の概念の拡張。（a）$S$が依然として$\mathbb{R}^3$における対応するボリュームを2つの連結成分$I$および$O$に分離するような部分集合$\boldsymbol{z} \subset \mathbb{Z}^3$が存在する閉じていない表面$S$を考える。そして、対応する$I^d$および$O^d$は空ではない。（b）$\boldsymbol{z}$におけるボクセルに制限される$I^d$と$O^d$の間のすべての$\Pi_k$が$S^d$におけるボクセルを含むならば、$S^d$は$S$の$k$分離ボクセル化であると言える。そして、これは部分集合$\boldsymbol{z}$の任意の有効な選択に対して成り立つ[holds for]。この2Dの例では、$S^d$は8分離ではなく4分離である。](fig/1.png){#fig:1}

# 関連研究[Previous work]

TODO

# 交差対象を伴うボクセル化[Voxelizing with intersection targets]

我々の手法では、入力ジオメトリが$V$の中に含まれる*交差対象[intersection target]*と交差するかどうかのテストによって、ボクセル$V$がボクセル化$S^d$に含まれるかどうかを選択する。すべての場合において、$S$のsupercover[@COK95]に属するボクセルを考えるには、それとは別のボクセルにおける交差対象が明らかに入力ジオメトリと交差できないので、これで十分である。
本節の後半において、我々は、入力の実効次元がボクセル化戦略に与えるであろう影響度と退化した交差を扱う一般的な方法を考察する。[@sec:4]では、2Dにおける曲線に対する交差対象を導出し、[@sec:5]では、3Dにおける曲面および表面のボクセル化を検討する。最後に、[@sec:6]では、交差対象がボクセルごとに異なっても良い変種[variants]を調査する[examine]。

## 入力の実効次元[Effective dimension of input]

交差対象は様々な次元のプリミティブから構成しても良い。これらの例については[@fig:4]を参照のこと。交差対象の次元の適切な選択はその入力の*実効次元[effective dimension]*に依存する。これは入力プリミティブの次元と異なっても良い。例えば、ソリッドな円柱としてジオメトリ的に表現される1本の細い髪の毛[strand of hair]を考えよう。その物体がボリューメトリックな立体であるとしても、例えば、ボリューム内に点があるかどうかの単純なテスト[simple point-in-volume test]を用いてボクセル化するという発想はおそらくうまくいかない。これは我々の用語法においてボクセルの中心にある0次元の点である交差対象と対応するだろう。入力が実質的に1次元であり、交差対象が0次元であるので、交差は髪の毛の太さに依存する任意に小さな確率でのみ発生する。これは結果としてその入力を代表しないボクセル化となり、例えば、その連結性[connectivity]を維持することができない。

![色々な種類の入力に適する、様々な次元のプリミティブから構成される交差対象の例](fig/4.png){#fig:4}

この例の状況において、例えば、$S^d$に対する6または26連結性を立証したい[want to establish]場合、入力と交差対象との交差が確率で決まらないようにする[not rely on chance]ため、我々は少なくとも2次元の交差対象を用いなければならない。一方で、3次元の交差対象（例えば、ボクセル全体）は不必要に鈍く[unnecessarily blunt]、結果として冗長なボクセルが$S^d$に含まれてしまう可能性があるだろう。髪の毛が3Dの立体、2Dの表面、1Dのポリライン[polyline]または曲線のいずれかから構成されるかに関わらず同じ論拠[argument]が適用される。すなわち、実効次元は入力プリミティブの次元より関連性がある[relevant]。
2Dボクセル化において、2Dプリミティブの列が実質的に1次元である物体を形成するとき、似た状況が発生する。そのような例のひとつは、パスレンダリング[path rendering]で一般的な状況である、ゼロでない小さな太さを持つ線分から構成される曲線である。通常の0次元の交差対象（ピクセルの中心）によってパスをラスタライズすることはその結果の連結性を保証できないが、各ピクセル内に適切な1D交差対象を配置することによって、4または8連結性のいずれかを容易に保証することができる。また、完全な2D交差対象はこれらの目的に対して過剰であるだろう。
関連研究[previous literature]で示される手法は入力プリミティブの次元性[dimensionality]に対して厳密に特殊化されるため、実効次元より（不必要に）高いプリミティブの次元が発生する入力には適用できない。例えば、その入力が実質的に1次元であると分かったとしても、線のボクセル化アルゴリズムは2Dの表面から構成される細い髪の毛に対して使用できない。我々の交差対象手法はこの問題に悩まされず、同じ枠組みの中で任意の次元のプリミティブから構成される入力を扱うことができる。

## 退化した交差[Degenerate intersections]

入力プリミティブと交差対象の境界は、ボクセル化において本来不要な穴[spurious holes]を生成するのを避けたり、不必要な「厚い」結果を避けたりするため、正確に扱われなければならない。交差において境界（ボリュームの表面、三角形のような表面要素の辺、線の端点、または、曲線の線分）を常に含めることは穴を形成することを防ぐだろうが、入力が複数の交差対象の境界に触れるとき、対応するボクセルのすべてが$S^d$に含まれる。Cohen-Or and Kaufman [@COK95]は詳細に[at length]これらの問題を考察し、慎重に選択された2Dピクセルまたは3Dボクセルの境界部分が除外される交差対象を用いることを提案する。
このような手法はより複雑な交差対象に拡張することが難しく、それ故に、我々は退化した交差の問題に対するより汎用的でより単純な解法を提案する。我々はまず、任意のそのような退化がどちらかの入力ジオメトリを --- または、等価的に、交差対象を --- 適切な方向に若干摂動させることによって解決することができることを述べる[observe]。これは結果として、交差していないか、入力と交差対象が適切に交差しているかのどちらかとなる。ここでは境界が含まれているかどうかは問題とならない。
入力プリミティブと交差対象の間の任意の退化を取り除く微小摂動ベクトルを*演繹的に[a priori]*選択できるということが判明する。この摂動を入力全体に --- または、等価的に、すべての交差対象に --- 適用することは入力に対する微小の修正[modifications]が許可されると仮定して正しいボクセル化を生成する。そのような摂動ベクトルの例は$(\epsilon, \epsilon^2, \epsilon^3)$である。実践において、退化した交差は、退化していない結果が得られるまで、まずは$+x$へ、そして$+y$へ、などというように、微小に入力（または、交差対象）をずらすことで退化していない状態になるかどうかをテストすることによって解決される。
ボクセル対三角形やピクセル対辺のテストでは、このアプローチはCohen-Or and Kaufman [@COK95]の"reduced-voxels"と等価であるが、ボクセル境界の面、辺、頂点について、どれが含まれ、どれが外されるかを個別に選択する必要がない。標準の2Dラスタライゼーションで用いられる点対三角形のテストでは、一般的なグラフィックスAPIで使われるラスタライゼーション規則と等価である。これは、一貫した面の向き[consistent facing]を持つ三角形の扇が各ピクセル中心を、それが辺や頂点の上にあったとしても、たった一度だけ[exactly once]含める[cover]ことを保証する。実践において、我々は微小摂動ベクトルのアプローチが通常では交差計算中に行われる不等式テストの中へ注意深く配置された方程式に煮詰まる[boil down]ことを期待する。例えば、3Dでのポリゴン対ポリゴンのような、とあるテストでは、その実装は自明ではないかもしれない。
ほとんどの場合、退化した交差が発生するような稀なケースにおいて結果のボクセル化に少量の余分なボクセルを含むことはそこまで悲惨というわけではない[not disastrous]ことは記されるべきである。入力プリミティブと交差対象が互いに触れているだけでも交差を報告する保守的な[conservative]交差はボクセル化に加えることだけが可能であり、それ故に、その連結性や分離可能性の特性を一切破壊しない。薄さは妥協して解決される[compromised]かもしれず、これが関心事であるかどうかはアプリケーションに依存する。摂動手法の主な価値[principal value]は潜在的な退化を考える必要性を取り除くことによって我々の解析を単純化することである。

# 2Dにおける交差対象[Intersection targets in 2D]

まずは2Dにおけるボクセル化、すなわち、ラスタライゼーションを考えるとしよう。一般的な命名法[nomenclature]に従うため、我々はボクセルをピクセルと呼び。そして、入力ジオメトリはプリミティブ次元と実効次元の両方において0、1、2次元として良い。我々は入力の実効次元が適切なボクセル化戦略を選択するために既知であると仮定する。我々は単純で閉じた曲線の内側から外側に横切る[cross]任意の連続的なパスがこの曲線と少なくとも1点で交差しなければならないというジョルダン曲線定理[Jordan curve theorem]に強く依存する。また、上記のような退化した場合を扱うために、パスが複数の曲線と交差または接する[curves intersect or meet]点を横切る場合を個別に考慮することは、そのような状況がいずれも横切るパスの微小摂動によって解決されるので、必要ないことに注意する。

## 1次元の入力[One-dimensional input]

実質的に1次元である入力は実際に1次元の線分か曲線、または、薄い[slim]2次元プリミティブのいずれかから成るだろう。ピクセルグリッドにボクセル化するとき、我々はその結果を4連結または8連結のいずれかとする必要があるだろう。これらはそれぞれ8分離可能性および4分離可能性と等価である[@COK95]。それ故に、4分離可能性および8分離可能性を強制する[enforce]ための自明な方法は入力のcoverを生成することであるが、我々の入力は1次元であるので、1次元の交差対象で足りること、および、より少ないボクセルを生成することを期待するはずである。

### 4連結、8分離のボクセル化[4-connected, 8-separating voxelization]

すべてのピクセルに配置された[@fig:5]aに示される交差対象を考えるとする。この構造の重要な特徴は連続的な曲線がピクセルの隣接するカドのすべてのペアを接続することである。ピクセル内でその対象と交差する連続的な入力ジオメトリは他のピクセルへ、これらの一部が当該ピクセルの周りで単純閉曲線を形成するので、このピクセルの4近傍の交差対象を最初に通過することによってのみ拡張できる（[@fig:5]b）。それ故に、結果としての連続的な入力パスのボクセル化は4連結、すなわち、8分離である。代わりとなる方法は[@sec:5.2.1]で後に行われるように8分離可能性を直接証明することである。そこから4連結性が導かれる。

![2Dにおける1次元の入力の4連結、8分離のボクセル化に対する交差対象。（a）隣接するピクセルのカドが連続的な曲線によって接続される一般的な形状。（b）中心のボクセルにおいてその対象と交差する入力ジオメトリはその境界曲線を通る以外で灰色で塗られた[shaded]領域を脱出することができない。これは中心ピクセルの4近傍の交差対象の一部を成す。（c）カドで接続する[corner-connecting]曲線を中心で接するように伸長することで、実践的な斜めに横断する[cross-diagonal]交差対象を得る。（d）もうひとつの選択肢は曲線をピクセルの境界に押し出すことである。](fig/5.png){#fig:5}

[@fig:6]は[@fig:5]cの斜めに横断する対象を用いた1次元の入力の4連結ボクセル化の例を示す。ここでは、その結果に2次元のピクセル全体[full-pixel]の対象が生成するであろうより少ないボクセルを含む。[@fig:5]dの四角形の対象は、単一のピクセルの内側に全体が入り込む[fall in]任意の入力を含んだ、ゼロの実効次元を持つ点のような入力を無視する能力のためにのみ興味深い。そうでなければ、ピクセル全体の対象と同じ結果を生成する。

![斜めに横断する交差対象による1次元の入力のボクセル化の例。点は入力の曲線と交差対象との交差を示す。4連結性および8分離可能性はcoverの部分集合によって達成される。](fig/6.png){#fig:6}

### 8連結、4分離のボクセル化[8-connected, 4-separating voxelization]

以前の交差対象を修正することによって、1次元の入力の8連結、4分離のボクセル化を容易に生成できる。[@fig:7]aは隣接する辺の中間点が連続的な曲線によって接続される一般的な交差対象を図示する。[@fig:7]bに示される通り、その交差対象と入力が交差するピクセルの周りに単純閉曲線を再度得る。この曲線はピクセルの8近傍の交差対象の一部から成り、それ故に、連続的な入力ジオメトリがこれらの内の少なくとも1つと交差せずに脱出することを防ぐ。もう一度、我々は[@sec:5.2.1]における証明を用いて4分離可能性を直接示すことができるだろう。それから8連結性が導かれる。

![2Dにおける1次元の入力の8連結、4分離のボクセル化に対する交差対象。（a）隣接する辺の中心が連続的な曲線によって接続される一般的な形状。（b）中心のボクセルにおいてその対象と交差する入力ジオメトリは、灰色の領域の境界曲線がこれらの交差対象から成るので、ピクセルの8近傍にのみ拡張できる。（c）中心で曲線を接続すると、実践的な十字線[crosshair]の交差対象を生み出す。（d）真っ直ぐな線分で隣接する辺の中心を接続すると、ラインレンダリングに対して一般的なグラフィックスAPIで使われるダイアモンドパターンを生み出す。](fig/7.png){#fig:7}

実践的な交差対象はピクセルの辺の中間点[midpoints]をピクセル中心に接続することによって得られる（[@fig:7]c）。これは湾曲した[curved]入力プリミティブに対してすらも交差する非常に単純な対象である。OpenGLやDirectXといったグラフィックスAPIはラインレンダリングに対して"diamond exit"ルールを採用する。これは一般形状の特殊な場合として得られる[@fig:7]dにおける交差対象に対応する。ダイアモンド型の対象は十字線の対象よりその結果にピクセルを含みやすい[eager to include]。すなわち、入力が1つの象限[quadrant]に留まりつつも、カドからピクセルへ突き出ている[protruding]と考える。これは8連結性に対して不必要なピクセルを生成するコストでより審美的に満足する[aesthetically pleasing]結果を生み出すだろう。

#### 薄さ[Thinness]

8連結ボクセル化では、表面または曲面の法線ベクトルの主軸方向に1ボクセル分だけの厚さのレイヤを生成するとしておおまかに[loosely]定義される、いわゆる結果の*薄さ[thinness]*について懸念を抱くかもしれない。パラメータ化された連続的かつ微分可能な入力曲線$c = (f_x(t), f_y(t)), t \in [0, 1]$を考えるとする。ここで、すべての$t$に対して$|d f_y / d f_x| \ge 1$である。$c$は常に$x$軸ではなく$y$軸の方を向いているので、我々は$S^d$におけるすべての水平なレイヤに対して多くとも1つのボクセルを生成したい。
[@fig:7]cの十字線の対象がこの目標を満たす[fulfill]ことを確認することは容易である。点$(x_0, y_0)$でボクセル$V$にある水平な線分と交差する$c$を考える。ここで、$y_0 = Y + \frac{1}{2}$であり、$Y \in \mathbb{Z}$である。この水平なピクセルレイヤ内では、$y \in [y_0 - \frac{1}{2}, y_0 + \frac{1}{2}]$であり、絶対値の微分の比に関する制限のために境界$x \in [x_0 - \frac{1}{2}, x_0 + \frac{1}{2}]$を持つ。従って、$c$と潜在的に交差するかもしれない交差対象の垂直な線分は$c$がその水平な線分と交差する同じ十字線の対象に属さなければならない。これは$x,y$境界がそのような他の線分を含まないためである。明らかに、そのレイヤにおける他の水平な線分はいずれとも交差する可能性が存在しない。すなわち、同じレイヤにおける他のボクセルは$S^d$の中に存在しないだろう。似た推論を用いて、[@fig:7]dのダイアモンド型の対象が薄いボクセル化を生成することを確認できる。
[@fig:8]は[@fig:7]cの十字線の対象を用いた1次元の入力の8連結ボクセル化の例を示す。[@sec:5.2.2]に後に示される通り、そのボクセル化は4-tunnel-free[@COK95]である。

![十字線の交差対象を持つ1次元の入力の8連結および4分離ボクセル化の例。](fig/8.png){#fig:8}

# 3Dにおける交差対象[Intersection targets in 3D]

3Dでは、連結性と分離可能性の関係は2Dの場合よりも明瞭ではない。更に、1次元の入力を用いると、分離可能性の道理にかなう概念を与えることは一切できないが、それでも我々は結果のボクセル化に対する様々な連結性の特性を立証することに関心があるだろう。
[@sec:4]において用いられる種類の推論は依然として1次元の入力に対する連接性の特性を立証するために適用するが、2次元の入力に対する分離可能性は異なる戦略を用いて示される必要がある。我々は再びジョルダン曲線定理、または、より適切に言うならば、3Dの類似品であるジョルダン・ブラウワーの分離定理[Jordan-Brouwer separation theorem]を採用する。このとき、単純閉領域は2次元の入力ジオメトリによって形成され、内側領域と外側領域の間のパスは交差対象の部分を共に区分化すること[piecing]によって形成される。

## 1次元の入力[One-dimensional input]

3Dにおいて実質的に1次元の入力は1、2、または、3次元のプリミティブから成るだろう。
そのような入力が3D空間において分離を生成できないにもかかわらず、例えば、流体シミュレーション[flow simulation]やボクセル化された曲面の内部での塗りつぶし[a fill to proceed within the voxelized curve(s)]を可能にしたり、可視化されたボクセル化の見た目[appearance]が欠けていない[is not missing pieces]ことを保証したりするために、入力のボクセル化が6連結または26連結である必要があるだろう。我々は以降で両方の連結性に対する交差対象を導出するだろう。

### 6連結ボクセル化[6-connected voxelization]

[@sec:4.1.1]からの2Dの類似品に従って、[@fig:9]aに図示される3D交差対象を考えるとする。ボクセルの面ごとに、曲線がボクセルの外側からボクセルに侵入し、この表面と交差せずに他のある面を通って脱出するのを防ぐ連続的な表面を構築する。2Dの場合とまったく同じ推論に従って、連続的な1次元の入力オブジェクトが間にボクセルの6連結パスを形成せずに2つのボクセル間を拡張できないことを確認できる。

![3Dにおける1次元の入力の6連結ボクセル化に対する交差対象。（a）一般形状において、各面はボクセル内の表面によって閉じている。明確にするためにここでは下の面に対する部分[piece]のみが示される。（b）取り得る1つの実現方法[realization]は中心で接するピラミッドで各面を閉じることである。これは空間の八面体テッセレーションに対応する。ここでは、八面体の中心がボクセル面の中心に位置する。（c）もうひとつの選択肢はボクセルの境界に表面を押し出すことである。交差対象は依然として2次元であり、立方体の表面のみから成る。明らかに、これは空間の立方体テッセレーションに対応する。](fig/9.png){#fig:9}

この一般形状の2つの実践的な実現方法は[@fig:9]bおよびcに図示され、それぞれ空間の八面体および立方体テッセレーションに対応する。ボクセル化が3Dボクセル全体に対して入力を交差させることで生成されるcoverとほぼ同じだけのボクセルを含むので、これらが実践で適切かどうかは疑わしい[the practical relevance of these is dubious]。いくつかの状況では、3Dボクセルより2Dシートから構成される交差対象に対して入力を交差させる方が容易であるかもしれないが、一般には、これらの交差対象は結果のボクセル数をできるだけ少なくしておく必要があるときにのみおそらく有用である。

### 26連結ボクセル化[26-connected voxelization]

実質的に1次元の入力の26連結ボクセル化のケースはさらに興味深い。このはるかに緩和された連結性の要件に対してcoverを生成することは、多くの不必要なボクセルを含むので、あきらかにやり過ぎ[overkill]であろう。既存のアルゴリズムは、例えば、線分やパラメータ化された曲線のような1次元のプリミティブから形成される[is formed out of]入力に制限される。対して、我々の手法は実質的に1次元だが2または3次元のプリミティブのみで構成される入力から26連結ボクセル化を構築することができる。
[@fig:10]aはこの目的に対して適切な交差対象を示す。連結性は2Dにおける8連結ボクセル化に対して[@sec:4.1.2]で用いられたのと似た論拠[argument]から得られる[follow from]。入力がボクセル$V$内で対象と交差すると仮定しよう。$V$の26近傍の交差対象の部分は単一のボクセルの倍の長さの辺[side]を持つボックスを形成する（[@fig:10]b）。これは単純で閉じた表面であり、入力ジオメトリが$V$から目の前の近傍[immediate neighbors]の外側のボクセルへの連続的なパスを含むならば、この取り囲む[enclosing]ボックスと交差しなければならず、それ故に、$N_{26}(V)$における対象と交差する。

![3Dにおける1次元の入力の26連結ボクセル化に対する交差対象。（a）その対象は3つのすべての軸に沿ってボクセルを二分する[bisect]3つの四角形[quadrilaterals]から成る。これはボクセルのカドを中心とする立方体に空間を分割する。（b）中心のボクセル内で対象と交差する入力ジオメトリは26近傍の対象と交差せずに脱出できない。これは、これらの部分が倍の長さの辺を持つボクセルの周りに閉じた立方体を形成するためである。](fig/10.png){#fig:10}

その結果が十字線の対象による2Dの場合と同じ意味での薄い状態とならないことは注意すべきである。従って、支配的な方向に曲線に沿って歩を進め[steps]、レイヤあたり厳密に1つのボクセルを出力するアルゴリズムはより少ないボクセルを伴う26連結ボクセル化を生成するだろう。我々の手法の主な利点[benfit]は1次元プリミティブ以外の入力に対して機能することである。

## 2次元の入力[Two-dimensional input]

ほぼ間違いなく[arguably]3Dボクセル化の最も適した[relevant]使い方は2次元の入力の離散化である。この場合、結果のボクセル化の分離可能性は主な関心事である。例えば、ボクセル化した表面に対してレイをキャストすることを考えると、我々はできるだけ少ないボクセルをテストするためにレイから26連結パスを形成することを選択するだろう。この場合、表面のボクセル化は離散化したレイが元々分離している表面を貫通しない[not penetrate]ことを保証するために26分離である必要がある。あるいは、レイマーチがボクセルの6連結パスをテストするならば、より疎な入力の6分離ボクセル化で十分である。Cohen-Or and Kaufman [@COK97]は詳細な解析を提供する。

### 26分離ボクセル化[26-separating voxelization]

cover、すなわち、交差対象として3Dボクセル全体を用いることは、Cohen-Or and Kaufman [@COK95]によって示される通り、26分離である。しかし、我々の入力は実質的に2次元であるので、1次元の区分から構成される交差対象は関連するボクセルを求めることを保証するのに足るはずである。
[@fig:11]aにおける交差対象を考えるとしよう。26分離を保証するために必要な特性とは交差対象がボクセルのカドを互いに接続することのみである、と我々は主張する。これはひと目では分からない[not trivial to see]ので、ジョルダン曲線定理を用いて背理法[indirect proof]を定式化しようと思う。退化が[@sec:3.2]における微小摂動の手法によって扱われることを覚えておくことは重要であり、それ故に、表面が、例えば、ボクセルのカドを斜めに通過するような場合について心配する必要はない。別の言い方をすれば、摂動は、この場合でも、交差対象と表面の交差がボクセルの1つの内側で発生し、カドと辺と面では一切発生しないことを保証する。入力が3次元のボリューメトリックなプリミティブから構成されるが、依然として実質的に2次元の表面を形成するならば（例えば、ゼロでない厚さを持つ球殻[spherical shell]）、このボリューム内に含まれる適切な表面を考慮するだろう[shall consider]。

![3Dにおける2次元の入力の26分離ボクセル化に対する交差対象。（a）最も一般的な場合では、交差対象がなんとかして[in one way or another]ボクセルのすべてのカドを互いに接続するのに十分である。ここではボクセル内の共通の1点と接続しているが、これは必須ではない。（b）実践では、空間の対角線上の真っ直ぐな線分は単純で対象的な対象を生成する。（c）いくつかの状況では、個々の線分対表面の交差テストが近傍のボクセル間で共有され得るので、ボクセルの辺に沿ってカドを接続する対象を用いる方が便利であるかもしれない。](fig/11.png){#fig:11}

仮定を示し直すため[restating]、$\mathbb{R}^3$に組み込まれ、空でない2つの集合$I$および$O$に分割するような、単純で閉じた2次元の入力の表面$S$があるとしよう。$S^d$が上記の一般的な交差対象を用いて生成される$S$のボクセル化であり、加えて、それぞれ3Dボリューム全体が$I$および$O$に属するボクセルから成る空でない集合$I^d$および$O^d$が存在すると仮定する。
$S^d$が26分離であることを示すため、$V_{i+1} \in N_{26}(V_i)$、$V_0 \in I^d$、$V_n \in O^d$、かつ、$Pi \cap S^d = \emptyset$であるパス$\Pi = (V_0, \dots, V_n)$が存在しないことを示さなければならない。ジョルダン曲線定理に従い、$I$の点と$O$の点の間のすべての連続的なパスは$S$と交差しなければならない。連続的なパスが全体的に$\Pi$におけるボクセル内に含まれるならば、交差はボクセルのひとつ$V_i$の中で発生する。故に、これはそのような連続的なパスを構築するのに十分であり、$V_i$が$S^d$に含まれなければならないことを示す。
そのような連続的なパス$C(\Pi)$を構築するため、各$V_i$における交差対象の部分を接続する。交差対象はすべてのボクセルのカドを接続するので、$N_{26}$のいずれかに移動することが交差対象に沿って可能であることで、$\Pi$に存在するボクセル内に置くための$C(\Pi)$を含めることができる。$V_0 \in I^d$であり、それ故に、$V_0$内の任意の点が$I$の中にあることから、我々は$V_0$の交差対象における任意の点から$C(\Pi)$を開始する。同様に、$V_n$にある$C(\Pi)$の終点[endpoint]は自明なこととして$O$の中にある。
$C(\Pi)$は$I$の点と$O$の点の間の単純閉曲線であるので、ある点$p \in \mathbb{R}^3$で$S$と交差しなければならない。$C(\Pi)$が$\Pi$におけるボクセルに対応するボリューム内に完全に含まれるので、交差点$p$は$\Pi$の一部であるこれらのボクセルの1つ$V_i$にもなければならない。$p$が表面$S$と$V_i$の交差対象の両方の上にあるので、これは我々の$\Pi$の定義と矛盾する[contradict]。従って、$V_i \in S^d$であり、上で定義されるような26連結パス$\Pi$は存在できず、$S^d$が26分離であることを暗示する。
上記の推論は26連結の$\Pi$におけるボクセルによってカバーされるボリューム内に含まれる連続的なパス$C(\Pi)$を構築することを可能にする任意の交差対象に対して成り立つ。これは実践的な交差対象の定式化に多くの柔軟性を与える。[@fig:11]bとcは2つの選択肢を示すが、特に対称性を気にしないならば、他の多くの選択肢が存在する。

### 6分離ボクセル化[6-separating voxelization]

6分離ボクセル化は26分離より少ないボクセルを持つかもしれず、故に、26分離可能性が必要ないアプリケーションでは一般に好まれる。[@fig:12]aに図示される交差対象は互いにボクセル面の中心と接続し、6分離ボクセル化を生成することを確認できる。その論拠は、仮定の分離可能性に違反している[hypothetical separability-violating]ボクセルのパス$\Pi$が6連結であることを除いて、以前の場合と厳密に同じである。それ故に、連続的な$C(\Pi)$は6近傍の意味での近傍ボクセルの間を、すなわち、面を通って移動できる限り形成することができる。これは提案される交差対象で明らかに可能である。一般的な形状の最も実践的で対称的な実現方法はおそらく[@fig:12]bの十字線の対象である。
2Dで用いられるダイアモンドルールの任意の道理にかなう3D拡張は十字線の対象に含まれるものにより6分離ボクセル化を生成する、ということが即座に導かれる。例えば、Schwarz and Seidel [@SS10]の手法は入力の三角形がボクセルの面の中心の間にかかる八面体と交差する任意のボクセルを常に含み（[@fig:12]c）、それ故に、6分離である。しかし、不必要に大きな対象を用いることは6分離に対して必要のないボクセルを含み、彼らの手法を最小限にできないことを示す。
単純な解析は、Forest et al. [@FBP09]やCrassin et al. [CNS*11]によって用いられるGPUベースのボクセル化手法が6分離であることも明らかにする。これらは入力に関して3パスを行い、各パスでは3つの軸並行な投影に沿ってプリミティブをラスタライズする。結果のピクセルごとに、プリミティブの深度がその中心で計算され、その中心が奥行きで最も近いボクセルが$S^d$に含まれる。これが[@fig:12]bの十字線の交差対象を用いることと等価であることを理解するのは容易である。ここでは、各ラスタライゼーションパスが投影軸の方を向いている十字線の線分と入力を交差させることに対応する。拡張よって、このラスタライゼーションベースの手法は湾曲したプリミティブに対してさえ6分離の結果を生成するであろう一方、必要になるのは深度の点の計算だけである。この特性は上記のものを用いた技術なしに確認する[ascertain]ことが極めて困難であるかもしれないだろう。

#### 薄さ[Thinness]

十字線の交差対象によって生成されるボクセル化は以下の意味において薄くもある。おそらく曲がっていて[possibly curved]、連続的で、あらゆるところで微分可能な表面プリミティブを考える。ここでは、すべての点で、法線ベクトルの$z$成分は最も大きい絶対値を持つ。薄さのために、我々は1つより多いボクセルがボクセルの$z$を向いた[z-oriented]各列において生成されないことを必要とする。その表面が点$(x_0, y_0, z_0)$のボクセル$V$の$z$を向いた十字線の線分と交差するとしよう。ここで、$x_0 = X + \frac{1}{2}$、$y_0 = Y + \frac{1}{2}$、$X, Y \in \mathbb{Z}$である。この（$z$を向いた）ボクセルの列では、$x \in [x_0 - \frac{1}{2}, x_0 + \frac{1}{2}]$であり、$y$に対しても同様である。$y = y_0$の$xz$平面で表面をスライスすることを考える。すべての点で$z$方向に傾いている法線ベクトルにより、我々は$|dz / dx| \le 1$を得る。それ故に、$z$もまた$y = y_0$を維持するときにこの$x$の範囲において境界$z \in [z_0 - \frac{1}{2}, z_0 + \frac{1}{2}]$を持ち、従って、表面はこの列の中の$V$以外のボクセルにおける$x$を向いた十字線の線分に到達できない。$y$を向いた十字線の線分に対しても同様のことが成り立つ。一般に表面$z$はその列におけるもうひとつのボクセルの中間に達するだろうが、これは十字線の線分がある$xz$および$yz$平面では起こり得ないことに注意する。その表面は明らかに同じ列における他のいずれかの$z$を向いた十字線の線分と交差できない。それ故に、$V$を除いたこの列の他のボクセルは結果に含めることができない。それから薄さが導かれる。

#### 単一の2D投影を伴うボクセル化[Voxelization with a single 2D projection]

興味深い推論[corollary]が薄さの特性と十字線の対象を用いる3軸ラスタライゼーションの等価性[@FBP09; @CNS*11]から導かれる。平面の三角形がその幾何的な法線ベクトルの支配的な方向に沿って投影されるとき、投影された三角形の中にその中心があるすべてのピクセルは対応するボクセル列におけるユニークなボクセルを生成する。十字線の対象を伴うボクセル化は薄いので、他の投影方向はこれらの列においてそれ以上の[any more]ボクセルを生成しない。従って、そのようなすべてのピクセルに対して、支配的な軸に沿った投影を考えるだけで十分であり、辺を更に考慮する必要があるのみである。これは以下のアルゴリズムを提案する。
法線の支配的な軸に沿って2Dに投影される三角形をラスタライズする。投影された三角形がピクセルの中心を含むピクセルに対して、以前の手法にあるように[@FBP09' @CNS*11]、中心の深度がピクセル中心で計算される三角形の深度に最も近いボクセルを出力する。三角形の辺を含むが、その三角形が中心と重なっていない[not overlap]ピクセルに対して、ピクセルを二分する水平な線上の三角形の範囲[extents]を求め、これらを水平なピクセルの範囲[extents]にクランプし、こらら2つの点で三角形の深度を計算する。その値がボクセルの中心の深度の別々の側[different sides]にあるならば、対応するボクセルを出力する。そうでなければ、垂直方向に対してテストを繰り返す。最初のテスト（三角形対ピクセル中心）は投影軸に平行な十字線の線分に対して三角形を交差させることに対応し、後者のテストは投影軸に垂直な2つの十字線の線分に対して三角形をテストすることに対応する。こうして、6分離ボクセル化はたった1つの2D投影に基づいて生成できる。

#### Tunnel-freeness

Cohen-Or and Kaufman [@COK95]の定義に従って、6-tunnel-freeボクセル化は$S^d$の外側の2つの6近傍ボクセルの中心を接続する任意の線分が$S$と交差しないようなものである。[@fig:12]bにおける十字線の対象を用いることがこの要件を満たすと確認することは、$S$と交差するそのような任意の線分が$S^d$における近傍のボクセルの少なくとも１つで生成するだろうから、容易である。[@fig:7]cにおける十字線の対象を用いた2Dボクセル化でも同じことが成り立つ。ピクセル中心をそのカドや辺の中間点と接続する交差対象を用いることによって2Dでの8-tunnel-freeボクセル化を、ボクセル中心を面の中心や辺の中間点と接続する対象を用いて3Dでの18-tunnel-freeボクセル化を、ボクセル中心が面の中心や辺の中間点やカドと接続される対象を用いて26-tunnel-freeボクセル化を同様に得ることができるだろう。

# 拡張[Extensions]

TODO

# おわりに[Conclusions]

TODO

# 参考文献[References]

TODO
