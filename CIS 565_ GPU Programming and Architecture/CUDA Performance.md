---
title: CUDA Performance [@Cozzi2017d]
numberSections: false
---
# 並列リダクション[Parallel Reduction]

<!-- p.9 -->

```cuda
__shared__ float partialSum[];
// ... 共有メモリに読み込む
unsigned int t = threadIdx.x;
for (unsigned int stride = 1; stride < blockDim.x; stride *= 2) {
    __syncthreads();
    if (t % (2 * stride) == 0)
        partialSum[t] += partialSum[t + stride];
}
```

<!-- p.10 -->

1-2行目: 共有メモリでの要素の総和の計算

<!-- p.11 -->

4行目: stride = 1, 2, 4, ...

<!-- p.12 -->

5行目: なぜ？

<!-- p.13 -->

- 6-7行目:
    - 同じ共有メモリで総和を計算する
    - `stride`が増えると、スレッドはどうなる？

<!-- p.15 -->

- 第1パス: スレッド1、3、5、7は何もしない
    - 本当は$n$個の要素に対して$n/2$個のスレッドのみが必要である

<!-- p.16 -->

- 第2パス: スレッド2、6は何もしない

<!-- p.17 -->

- 第3パス: スレッド4は何もしない

<!-- p.18 -->

- 一般に、必要なスレッド数は各パス後に半分になる

<!-- p.19 -->

- 実装を*調整*したらどうだろう？

<!-- p.24 -->

```cuda
__shared__ float partialSum[];
// ... load into shared memory
unsigned int t = threadIdx.x;
for (unsigned int stride = blockDim.x / 2; stride > 0; stride /= 2) {
    __syncthreads();
    if (t < stride)
        partialSum[t] += partialSum[t + stride];
}
```

<!-- p.27 -->

- 第1パス: スレッド4、5、6、7は何もしない
    - 本当は$n$個の要素に対して$n/2$個のスレッドのみが必要である

<!-- p.28 -->

- 第2パス: スレッド2、3は何もしない

<!-- p.29 -->

- 第3パス: スレッド1は何もしない

<!-- p.30 -->

- 何が違う？

<!-- p.31 -->

- `stride = 1, 2, 4, ...`
    - `if (t % (2 * stride) == 0)`
- `stride = ..., 4, 2, 1`
    - `if (t < stride)`

# Warp分割[Warp Partitioning]

- *Warp分割*: ひとつのブロックからのスレッドをWarpに分ける方法
- Warp分割の情報は以下で使われる
    - divergent branchesを最小化する
    - Warpを早期にリタイアさせる

<!-- p.33 -->

- *連続的に増加する*`threadIdx`に基づいて分割する

<!-- p.34 -->

- 1Dブロック
    - `threadIdx.x`は0から1023まで(Fermi以降)
        - `threadIdx.x`は0から511まで(G80/GT200)
    - $n$番目のWarp
        - $32n$番目のスレッドから始まる
        - $32(n+1)-1$番目で終わる
    - ブロックサイズが32の倍数でないならば、最後のWarpはパディングされる

<!-- p.35 -->

- 2Dブロック
    - `threadIdx`の増加は以下を意味する
        - `threadIdx.x`は増加
        - 行は`threadIdx.y == 0`から開始
- 例えば、64x8ブロックでは

<!-- p.37 -->

- 3Dブロック
    - `threadIdx.z == 0`から始まる
    - 2Dブロックと同様に分割する
    - `threadIdx.z`を増加し、繰り返す

<!-- p.38 -->

*divergent branchesはWarp内部にある*

<!-- p.39 -->

- `warpSize == 32`では、以下のコードではいずれのWarpもdivergent branchを持つ

```cuda
if (threadIdx.x > 15) {
    // ...
}
```

<!-- p.40 -->

- `warpSize > 1`では、以下のコードではいずれのWarpもdivergent branchを持つ

```cuda
if (threadIdx.x > warpSize - 1) {
    // ...
}
```

<!-- p.41 -->

- Warp分割の情報があるとすると、どちらの並列リダクションがより良いか？
    - `stride = 1, 2, 4, ...`
        - `if (t % (2 * stride) == 0)`
    - `stride = ..., 4, 2, 1`
        - `if (t < stride)`

<!-- p.42 -->

- `warpSize == 2`としてみる

<!-- p.43 -->

- 第1パス
    - `stride = 1, 2, 4, ...`
        - 4つのdivergent branches
    - `stride = ..., 4, 2, 1`
        - divergent branchesなし

<!-- p.44 -->

- 第2パス
    - `stride = 1, 2, 4, ...`
        - 2つのdivergent branches
    - `stride = ..., 4, 2, 1`
        - divergent branchesなし

<!-- p.45 -->

- 第3パス
    - `stride = 1, 2, 4, ...`
        - 1つのdivergent branch
    - `stride = ..., 4, 2, 1`
        - 1つのdivergent branch

<!-- p.47 -->

- 良い分割はWarpが早期にリタイアできるようにする
    - より良いハードウェアの利用

<!-- p.48 -->

- 並列リダクション

<!-- p.49 -->

- 第1パス
    - `stride = 1, 2, 4, ...`
        - Warpのリタイアなし
    - `stride = ..., 4, 2, 1`
        - 2つのWarpがリタイア

<!-- p.49 -->

- 第2パス
    - `stride = 1, 2, 4, ...`
        - 2つのWarpがリタイア
    - `stride = ..., 4, 2, 1`
        - 1つのWarpがリタイア

# Memory Coalescing

- *グローバルメモリ* に *row-major* で格納された行列があるとすると、*スレッド* の望ましいアクセスパターンとは？

<!-- p.55 -->

- a) column after column
    - *個別のスレッド* が増加する連続的なメモリアドレスを読む
- b) row after row
    - *隣接するスレッド* が増加する連続的なメモリアドレスを読む

<!-- p.58 -->

TODO
