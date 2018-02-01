---
title: CUDA Atomics [@Cozzi2017e]
numberSections: false
---
# Atomic Functions

- 8スレッドが`++count`を実行したら`count`の値はどうなる？

```cuda
__device__ unsigned int count = 0;
// ...
++count;
```

<!-- p.3 -->

- Read-Modify-Writeアトミック処理
    - 他のスレッドから干渉されないことが保証される
    - 順序は保証しない
- 共有メモリかグローバルメモリ
- コンピュート互換性1.1が必要 (> G80)

<!-- p.4 -->

- 8スレッドが以下の`atomicAdd`を実行したら`count`の値はどうなる？

```cuda
__device__ unsigned int count = 0;
// ...
// atomic ++count
atomicAdd(&count, 1);
```

<!-- p.5 -->

- どうやって`atomicAdd`を実装するか？

```cuda
__device__ int atomicAdd(int* address, int value);
```

<!-- p.6 -->

```cuda
__device__ int atomicAdd(int* address, int val) {
    // でっち上げキーワード: __lock
    int old;
    __lock (address) {
        old = *address;
        *address += val;
    }
    return old;
}
```

<!-- p.8 -->

- どうやってロック **せずに** `atomicAdd`を実装するか？
- アトミックなコンペア＆スワップだとしたらどうだろう？

```cuda
int atomicCAS(int* address, int compare, int value);
```

<!-- p.9 -->

- `atomicCAS`の疑似実装

```cuda
int atomicCAS(int* address, int compare, int val) {
    // でっち上げキーワード
    __lock(address) {
        int old = *address;
        if (old == compare) *address = val;
        return old;
    }
}
```

<!-- p.16 -->

- では、どうやって`atomicCAS`を仮定して`atomicAdd`を実装するか？

```cuda
__device__ int atomicAdd(int* address, int val) {
    int old = *address, assumed;
    do {
        assumed = old;
        old = atomicCAS(address, assumed, val + assumed);
    } while (assumed != old);
    return old;
}
```

<!-- p.22 -->

- どうやって異なるブロックからのスレッドは一緒に動作することができるか？
- アトミックは控えめに使う。なぜ？
