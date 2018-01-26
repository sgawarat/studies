---
title: GPU Architecture Overview [@Cozzi2017]
numberSections: false
---
# CPUとGPUのトレンド[CPU and GPU Trends]

- FLOPS --- FLoating-point OPerrations per Second[秒あたり浮動小数点演算数]
- GFLOPS --- 10億(10^9)FLOPS
- TFLOPS --- 1000GFLOPS

# あなたの車に7.5TFLOPS！

- NVIDIA DRIVE PX 2
    - 最大で
        - 2つのTegra SoC(2.5TFLOPS)
        - 2つのPascal GPU(5TFLOPS)

# …あたりのFLOP[FLOP per...]

- 2003年以降のシングルコアのパフォーマンスの低下
    - 電力や熱の限界
- GPUは以下あたりのより高いFLOPを届ける
    - ワット
    - 大きさ[mm]
    - 費用[dollar]

# CPUレビュー[CPU Review]

- CPUダイの中の主要な構成要素は何か？

<!-- p.10 -->

- デスクトップアプリケーション
    - 軽めのスレッド化
    - 大量の分岐
    - 大量のメモリアクセス

# 分岐予測[Branch Prediction]

- 近年の予測器は90%以上の正確さ
    - パフォーマンス*と*エネルギー効率の向上
- 面積の増大
- フェッチステージのレイテンシーの増加の可能性

# メモリ階層[Memory Hierarchy]

- メモリ: 大きくなるほど、遅くなる
- 大まかな数値:

||レイテンシー|帯域幅|サイズ|
|-|-|-|-|
|SRAM(L1、L2、L3)|1-2ns|200GBps|1-20MB|
|DRAM(メモリ)|70ns|20GBps|1-20GB|
|フラッシュ/SSD(ディスク)|70-90us|200-500MBps|100-1000GB|
|HDD(ディスク)|10ms|1-150MBps|500-3000GB|

# キャッシング[Caching]

- 必要なデータを近くに置き続ける
- 活用する:
    - 時間的な局所性
        - たった今使われたチャンクはすぐに再び使われる可能性が高い
    - 空間的な局所性
        - 次に使うチャンクは前のものの近くにある可能性が高い

# キャッシュ階層[Cache Hierarchy]

- ハードウェア管理
    - L1 命令/データキャッシュ
    - L2 ユニファイド^[命令/データの区別を付けない]キャッシュ
    - L3 ユニファイドキャッシュ
- ソフトウェア管理
    - メインメモリ
    - ディスク

# IPCの改善[improvig IPC]

- IPC(Instructions Per Cycle[サイクルあたりの命令数])は1命令/クロックでボトルネックになる
- スーパースカラ --- パイプラインの幅を広げる

# スケジューリング[Scheduling]

- 以下の命令を考える:

```asm
xor r1, r2 -> r3
add r3, r4 -> r4
sub r5, r2 -> r3
addi r3, 1 -> r1
```

- `add`は`xor`に依存している(Read-After-Write、RAW)
- `sub`と`addi`は依存している(RAW)
- `xor`と`sub`は依存してい*ない*(Write-After-Write、WAW)

# レジスタリネーミング[Register Renaming]

- 代わりにこれならどうだろう:

```asm
xor p1, p2 -> p6
add p6, p4 -> p7
sub p5, p2 -> p8
addi p8, 1 -> p9
```

- `xor`と`sub`が並列に実行できる

# アウトオブオーダー実行[Out-of-Order Execution]

- スループットを最大化するための命令の入れ替え[reordering]
- フェッチ→デコード→リネーム→ディスパッチ→発行[issue]→レジスタ読み込み→実行→記憶[memory]→リードバック→コミット
- リオーダーバッファ(ROB)
    - 実行中の命令の状態を追跡し続ける
- 物理レジスタファイル(PRF)
- キュー/スケジューラを発行する
    - 実行する次の命令を選ぶ

# CPUでの並列化[Parallelism in the CPU]

- 命令レベル(ILP)の抽出をカバーする
    - スーパースカラ
    - アウトオブオーダー
- データレベル並列化(DLP)
    - ベクタ
- スレッドレベル並列化(TLP)
    - 同時マルチスレッディング(SMT)
    - マルチコア

# ベクタの動機づけ[Vectors Motivation]

```c
for (int i = 0; i < N; i++)
    A[i] = B[i] + C[i];
```

# CPUデータレベル並列化[CPU Data-Level Parallelism]

- Single Instruction Multiple Data (SIMD)
    - 実行単位(ALU)をかなり広くしよう
    - レジスタもかなり広くしよう

```c
for (int i = 0; i < N; i += 4) {
    // 並列化
    A[i] = B[i] + C[i];
    A[i+1] = B[i+1] + C[i+1];
    A[i+2] = B[i+2] + C[i+2];
    A[i+3] = B[i+3] + C[i+3];
}
```

# x86でのベクタ処理[Vector Operations in x86]

- SSE2
    - 4つ幅のパック済みfloat及びパック済みint命令
    - Intel Pentium 4以降
    - AMD Athlon 64以降
- AVX
    - 8つ幅のパック済みfloat及びパック済みint命令
    - Intel Sandy Bridge
    - AMD Bulldozer

# スレッドレベル並列化[Thread-Level Parallelism]

- スレッドの構成物
    - 命令ストリーム
    - プライベートなPC^[プログラムカウンタ]、レジスタ、スタック
    - 共有されたグローバル領域、ヒープ
- プログラマーによって生成、破棄される
- プログラマーかOSによってスケジュールされる

# 同時マルチスレッディング[Simultaneous Multithreading]

- 命令を複数スレッドから発行できる。
- ROBや他のバッファの分割が必要
- ✅ 最小のハードウェア重複
- ✅ アウトオブオーダーに対する更なるスケジューリング自由度
- ❌ キャッシュや実行リソース競合がシングルスレッドのパフォーマンスを低下させる可能性がある

# マルチコア[Multicore]

- 完全なパイプラインを複製する
- Sandy Bridge-E: 6コア
- ✅ 完全なコア、最終レベルのキャッシュ以外でリソース共有がない
- ✅ ムーアの法則の利点を得るより簡単な方法
- ❌ 利用率

# CPUの結論[CPU Conclusions]

- CPUはシーケンシャルプログラミングに最適化されている
    - パイプライン、分岐予測、スーパースカラ、アウトオブオーダー
    - 高クロック速度や高利用率で実行時間を減らす
- 低速なメモリは変わらない問題である
- 並列化
    - Sandy Bridge-Eは6-12のアクティブスレッドで素晴らしい
    - 32Kのスレッドだとどうだろう？

# グラフィクスの仕事量[Graphics Workloads]

TODO
