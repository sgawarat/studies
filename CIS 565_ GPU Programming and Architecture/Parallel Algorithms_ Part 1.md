---
title: >
Parallel Algorithms: Part 1 [@Cozzi2017b]
numberSections: false
---
# 並列リダクション[Parallel Reduction]

- 数値の配列が与えられたとき、総和を見つける並列アルゴリズムを設計する
- 以下を考慮する
    - *演算強度[arithmetic intensity]*: 計算とメモリアクセスの比

<!-- p.4 -->

- 数値の配列が与えられたとき、以下を見つける並列アルゴリズムを設計する
    - 総和
    - 最大値
    - 値の積[product of values]
    - 平均値
- これらのアルゴリズムはどれだけ異なるか？

<!-- p.5 -->

- *リダクション[reduction]*: データのセットから単一の結果を計算する処理
- *並列リダクション*: それを並列にやる

<!-- p.6 -->

- 例: 総和を見つける

<!-- p.10 -->

- 勝ち抜き戦に似ている[Similar to brackets for a basketball tournament]
- $n$個の要素に対して$\log n$個のパス

<!-- p.11 -->

- +=に注目
- 配列はその場で修正する

# All-Prefix-Sums

- *All-Prefix-Sums*
    - 入力
        - $n$個の要素の配列: $[a_0, a_1, \dots, a_{n-1}]$
        - 二項の結合オペレータ[binary associate operator]: $\oplus$
        - 単位元[identity]: $I$
    - 配列を出力する: $[I, a_0, (a_0 \oplus a_1), \dots, (a_0 \oplus a_1 \oplus \dots \oplus a_{n-1})]$

<!-- p.16 -->

- 例
    - $\oplus$が加算なら、配列`[3 1 7 0 4 1 6 3]`は
    - `[0 3 4 11 11 15 16 22]`に変換される
- 順次処理のように見えるが、効率的な並列ソリューションがある

# 走査[Scan]

- *排他的走査[Exclusive Scan]*: 結果のj番目の要素は入力のj番目の要素を含まない
    - 入力: `[3 1 7 0 4 1 6 3]`
    - 出力: `[0 3 4 11 11 15 16 22]`
- *包含的走査[Inclusive Scan]*(*プリスキャン[Prescan]*): jを含むすべての要素を総和する
    - 入力: `[3 1 7 0 4 1 6 3]`
    - 出力: `[3 4 11 11 15 16 22 25]`

<!-- p.18 -->

- どうやって*包含的走査*から*排他的走査*を生成するか？
    - 入力: `[3 1 7 0 4 1 6 3]`
    - 包含的: `[3 4 11 11 15 16 22 25]`
    - 排他的: `[0 3 4 11 11 15 16 22]`
        - 右にずらして、単位元を挿入する
- 反対方向ならどうする？

<!-- p.19 -->

- *包含的走査* の並列アルゴリズムを設計する
    - 入力: `[3 1 7 0 4 1 6 3]`
    - 出力: `[3 4 11 11 15 16 22 25]`
- 考慮:
    - 加算の合計数

<!-- p.20 -->

- シングルスレッドでは素直

```c
out[0] = in[0];
for (int k = 1; k < n; ++k)
    out[k] = out[k - 1] + in[k];
```

- 長さ$n$の配列に対する$n-1$個の加算
    - (配列のインデックスを無視)
- 我々の並列バージョンにはいくつの加算があるだろう？

<!-- p.21 -->

- *ナイーブな並列走査*
- これは排他的か包含的か？
- スレッドごと
    - ひとつの総和を書き出す
    - 2つの値を読み込む

<!-- p.31 -->

- もう一度言うけど、これは並列動作である！

<!-- p.37 -->

- `k = 7`のみ考慮する

# 作業効率の高い並列走査[Work-Efficient Parallel Scan]

- 加算の数
    - *順次走査*: $O(n)$
    - *ナイーブ並列走査*: $O(n\log_2(n))$
- どうしたら速くできるか？

<!-- p.40 -->

- *平衡二分木[Balanced binary tree]*
    - $n$個のリーフ = $\log_2 n$個のレベル
    - レベル$d$ごとに$2^d$個のノードを持つ

<!-- p.42 -->

- (概念として)平衡二分木を用いて、2段階で走査を行う
    - アップスウィープ[Up-Sweep] (並列リダクション)
    - ダウンスウィープ[Down-Sweep]

<!-- p.43 -->

- アップスウィープ

```c
// 並列リダクションと同じコード
for d = 0 to log_2(n) - 1
    for all k = 0 to n - 1 by 2^(d + 1) in Parallel
        x[k + 2^(d + 1) - 1] += x[k + 2^d - 1];
```

<!-- p.45 -->

- ダウンスウィープ
    - その場で走査を構築するために部分的な総和を用いるツリーを逆に"横断[traverse]"する
        - ルートを0にセットする
        - 各パスで、ノードは値をその左の子ノードに渡して、右の子ノードを前の左の子ノードの値と自身の値の総和にセットする

<!-- p.46 -->

```c
x[n - 1] = 0
for d = log_2(n) - 1 to 0
    for all k = 0 to n - 1 by 2^(d + 1) in parallel
        t = x[k + 2^d - 1];  // 左の子ノードを保存する
        x[k + 2^d - 1] = x[k + 2^(d + 1) - 1];  // 左の子ノードをこのノードの値にセットする
        x[k + 2^(d + 1) - 1] += t;  // 右の子ノードを前の左の子ノード＋このノードの値にセットする
```

<!-- p.51 -->

- アップスウィープ
    - 加算: $O(n)$
- ダウンスウィープ
    - 加算: $O(n)$
    - スワップ: $O(n)$
- この*排他的走査*は*包含的走査*に変換できる

# ストリームコンパクション[Stream Compaction]

- ストリームコンパクション
    - 要素の配列があるとする
        - 例えば非NULLのような、ある基準を満たす要素を持つ新しい配列を生成する
        - 順序を維持する

<!-- p.54 -->

- パストレーシング、衝突検出、疎行列の圧縮、などで使われる
- GPUからCPUへの帯域幅が削減できる

<!-- p.55 -->

- *ステップ1*: 含めるかどうかを示す一時的な配列を計算する
    - 対応する要素が基準を満たすならば1
    - 対応する要素が基準を満たさないならば0

<!-- p.60 -->

- 並列に動作する！

<!-- p.62 -->

- *ステップ2*: 一時的な配列で排他的走査を行う

<!-- p.63 -->

- 走査は並列に動作する
- この結果で何ができるか？

<!-- p.64 -->

- *ステップ3*: 散乱[scatter]
    - 走査結果は最終配列へのインデックスである
    - 一時的配列が1である場合のみ要素を書き込む

<!-- p.70 -->

- 散乱は並列に動作する！

# Summed Area Table

- *Summed Area Table* (*SAT*): 左下の角とエントリ位置との間の入力画像中のすべての要素の総和を格納する2Dテーブル

<!-- p.73 -->

- 例:

|||||
|-|-|-|-|
|2|1|0|0|
|0|1|2|0|
|1|2|1|0|
|1|1|2|2|
: 入力画像

|||||
|-|-|-|-|
|4|9|12|14|
|2|6|9|11|
|2|5|6|8|
|1|2|2|4|
: SAT

<!-- p.74 -->

- 利点
    - ピクセルあたり一定時間で、画像中のすべてのピクセルで様々な幅のフィルタを処理するのに使われる
    - SATで4ピクセルをサンプルするだけ:

$$
s_{filter} = \frac{s_{ur} - s_{ul} - s_{lr} + s_{ll}}{w \times h}
$$

<!-- p.75 -->

- 使い方
    - おおよその被写界深度
    - 光沢のある[grossy]環境の反射や屈折

<!-- p.87 -->

**どうやってGPUでこれを実装するのだろうか？**

<!-- p.88 -->

**どうやって包含的走査を用いてGPUでSATを計算するのだろうか？**

<!-- p.89 -->

- ステップ1:

行ごとに包含的走査を1回

<!-- p.90 -->

- ステップ2:

下から上へ、列ごとに包含的走査を1回

# 基数ソート[Radix Sort]

- 小さなソートキーで効率的
    - $k$ビットのキーで$k$回のパスが必要になる

<!-- p.92 -->

- 基数ソートの各パスは1ビットに基づいてその入力を分割する
- 第1パスは最下位ビット[Least Significant Bit; LSB]から始まり、後続のパスは最上位ビット[Most Significant Bit; MSB]に向かって移動する

MSB `010` LSB

<!-- p.93 -->

- 入力例:

<!-- p.94 -->

- 第1パス: LSBに基づいて分割する

<!-- p.95 -->

- 第2パス: 中間ビットに基づいて分割する

<!-- p.96 -->

- 最終パス: MSBに基づいて分割する

<!-- p.97 -->

- 完了:

# 並列基数ソート[Parallel Radix Sort]

- 並列性はどこ？

<!-- p.100 -->

1. 入力の配列をタイルに砕く
    - 各タイルはシェーダモデルの共有メモリに合わせる
2. *基数ソート* で *並列* にタイルをソートする
3. すべてのタイルがマージされるまで、*並列バイトニックマージ* を用いてタイルのペアをマージする

我々の焦点はステップ2にある

<!-- p.102 -->

- 並列性はどこ？
    - 各タイルは並列にソートされる
    - タイル内の並列性は何処？
        - 各パスは前のパスの後に順次に行われるので、並列性はない
        - 個々のパスを並列化できる？どうやって？
    - マージも並列性を持つ

<!-- p.103 -->

- *split* を実装する。以下があるとする
    - `n`パス目の配列`i`:
    - `n`ビット目の真偽を持つ配列`b`:
- 真のキーの前に偽のキーがある配列を出力する

<!-- p.104 -->

- ステップ1: 配列`e`を計算する
    - `e[k] = !b[k]`

<!-- p.105 -->

- ステップ2: `e`を排他的走査する
    - `f = ExclusiveScan(e)`

<!-- p.106 -->

- ステップ3: `totalFalses`を計算する
    - `totalFalses = e.back() + f.back()`

<!-- p.107 -->

- ステップ4: 配列`t`を計算する
    - `t[k] = k - f[k] + totalFalses`

<!-- p.112 -->

- ステップ5: アドレス`d`に基づいて散乱する
    - `d[k] = b[k] ? t[k] : f[k]`

<!-- p.118 -->

- $k$ビットのキーが与えられた場合、どうやって新しい*split*関数を用いてソートするか？
- いったん各タイルがソートされれば、どうやってタイルをマージして最終的なソートされた配列を提供するか？

# まとめ[Summary]

- 並列リダクション、走査、ソートは多くのアルゴリズムの基礎的要素である
- 並列プログラミング**と**GPUアーキテクチャの理解は効率的なGPU実装を生み出す
