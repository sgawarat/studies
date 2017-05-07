# Job System 2.0

## 新job systemの要件

- 基本の実装をよりシンプルにする。
    - ジョブを直接いじるような低レベルも、parallel_forのような高レベルもできるようにする。
- 負荷分散を自動でやってくれる。
- 適用できる箇所から順次lock-free版に入れ替えて性能向上を図る。
- "dynamic parallelism"に対応する。
    - parallel_forのような高レベルなものが動的に作業を更に小さなジョブへ分割できるように、親子関係の変更と実行中の依存関係の追加をできるようにする。

## Work stealing

　グローバルなキューは競合による性能低下を引き起こすため、ワーカースレッド別にローカルなキューを設けることを考えたい。しかし、ローカルなキューはそのスレッドにより生成されるジョブのみをpush/popするため、初期ジョブの配分により処理負荷に偏りができてしまう。
　**Work stealing**は自分のスレッドのキューが空のとき、他のスレッドのキューからジョブを盗んで(steal)くるという考え方を導入する。すなわち：

- push()/pop()は同スレッドにより呼び出され、LIFO方式の終端(private end)を操作する。
    - キャッシュ効率を高めるほうに働く。
- steal()は他スレッドにより呼び出され、FIFO方式の終端(public end)を操作する。
    - 負荷分散の効率を高めるほうに働く。

とした deque により実現される。しばしばこれを**work-stealing queue**と呼ぶ。

## ジョブシステム

### ジョブ構造

　ジョブは関数ポインタと親ジョブのポインタを持ち、親子関係を制御するための未終了ジョブ数(unfinished jobs)[^unfinished_jobs]を持つ。またFalse sharing[^false_sharing] を避けるためにキャッシュライン[^cache_line]一杯までパディングする。このパディング領域はユーザデータの格納領域として利用される。

### 追加(add)

　dynamic parallelismを実現するため、ジョブの生成とシステムへの追加を分ける。
　親ジョブがある場合、親ジョブの未終了ジョブ数を+1する。

### 待機(wait)

　ジョブの完了を待つ。

### 終了(finish)

　処理を済ませたジョブは終了(finish)することで、自身の未終了ジョブ数を-1する。未終了ジョブ数が0になったとき、親の未終了ジョブ数を-1したのちにジョブが使用する領域を解放したいが、0になった直後に解放しようとするとwaitしたスレッドがジョブの完了を確認できないまま領域が解放され得る。そこで、解放は以後の適切なタイミングで行うとして、finishの段階では未終了ジョブ数をさらに-1して完了したことを示すのみとする。

## アロケータ

　ジョブシステムは毎フレーム同じようなことを繰り返すという特殊な仕様上、一般用途化されたnew/deleteでは無駄が多すぎるし、PoolAllocatorでは性能は改善するだろうが、解放を適切なタイミングで行うという要件にマッチしているとはいえない。
　そこで、ジョブシステムが一括して領域を確保するリングバッファを用いてジョブを生成することを考える。これを導入することで、終了したジョブはフレーム終わりに一括して解放すれば良いため、finish時にジョブをdeleteする必要がなくなるという利点がある。また、このリングバッファはthread_localにできる。

## Lock-free

　work-stealing queueを実現する方法として、circular bufferを持つdequeを用いる。すなわち：

- 次にpushする終端位置を示す**bottom**を持つ。
    - push()はbottomが示す位置にstoreして、bottomをincrementする。
    - pop()はbottomをdecrementして、bottomを示す位置からloadする。
- 次にstealする終端位置を示す**top**を持つ。
    - steal()はtopが示す位置からloadして、topをincrementする。

とする。添字はmodか、ビットマスク(キューサイズが2の累乗の場合)を適用した値を用いる。

### pushとstealが同時に発生するとき

　steal()はキューが空であることを確認するためにbottomを読み出すが、その後にpush()がbottomをincrementした場合、steal()はキューが空であると誤認するだけで何ら処理を行わないため、致命的な競合は発生しない。

### popとstealが同時に発生するとき

　steal()はtop->bottomの順に読み出し、`top < bottom`の場合にCAS[^CAS]命令を用いてtopをincrementする。ここで、CAS後は以前のtopがpush可能であることに注意する。
　pop()はまずすぐさまbottomをdecrementすることで、以後のsteal()にジョブを奪わせないようにする。その後は以下のパターンが考えられる：

- そもそもキューが空ならば、ジョブは取り出せない。
    - このときdecrementしたbottomを元の値に戻す必要がある。
    - bottomをdecrementしても引き続き`top >= bottom`であり、steal()はキューを空だと認識する。
- ジョブが2つ以上残っている場合、steal()とpop()は別のジョブを取り出すので競合しない。
- ジョブが残り1つのとき：
    - steal()が先行していない場合、steal()を呼び出してもキューが既に空だと判断するので、pop()は安全にジョブを取り出せる。
    - steal()が先行している場合、ジョブは既に取り出されている可能性があるので、pop()はsteal()と同じtop方向からジョブの取り出しを試みる。
        - steal()と同様に、CAS後は以前のtopがpush可能であることに注意する。
        - このとき、次のpush先を正しくするため、bottomに`t+1`与える必要がある。

### メモリバリア

　特定の箇所には実行順序を入れ替えられないようにメモリバリアを挟む必要がある。

- push()における"ジョブの取り出し"と"incrementしたbottomの代入"の間。
- steal()における"topの読み出し"と"bottomの読み出し"の間。
- pop()における"bottomの書き出し"と"topの読み出し"の間。
    - ここには読み書き両方のバリアが必要になる。
    - XCHGなどのatomicな読み書き命令で代用しても良い。

## 参考文献

https://blog.molecular-matters.com/2015/08/24/job-system-2-0-lock-free-work-stealing-part-1-basics/
https://blog.molecular-matters.com/2015/09/08/job-system-2-0-lock-free-work-stealing-part-2-a-specialized-allocator/
https://blog.molecular-matters.com/2015/09/25/job-system-2-0-lock-free-work-stealing-part-3-going-lock-free/

### false sharing

http://www.isus.jp/products/psxe/intelguide-3-4/
http://faithandbrave.hateblo.jp/entry/2016/07/08/174657

### キャッシュライン

http://stackoverflow.com/questions/14707803/line-size-of-l1-and-l2-caches

[^false_sharing]: あるコアでキャッシュライン上の値を更新するとき、同じキャッシュラインを持つ別のコアのキャッシュが強制的に更新されることで性能低下が起こる問題。

[^unfinished_jobs]: 自身と自分が生成した子ジョブのうち、まだ完了していないジョブの数。

[^cache_line]: 基本サイズは64B（未確認）

[^CAS]: compare-and-swap