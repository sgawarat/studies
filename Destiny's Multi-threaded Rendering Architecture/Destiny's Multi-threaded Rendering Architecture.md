---
tile: Destiny's Multi-threaded Rendering Architecture [@Tatarchuk2015]
---
# Destiny's Multi-threaded Rendering Architecture

## このトークが扱わないこと

- Destinyのグラフィクスアルゴリズム
- Destinyのシェーダテクニック
- これらについてもっと学びたいなら、
    - 我々のテクニックのいくつかはSIGGRAPH 2009〜2014のトークやI3D 2015のキーノートを参照
        - **http://advances.realtimerendering.com**


<aside class="notes">
本日のトークは[TODO]
</asides>

## 前提

- タスクベースの並列化、ジョブマネージャの設計、同期プリミティブの基本に精通していること
- 役に立つDestinyエンジン設計は以下でその詳細を述べる
    - ***Multithreading the Entire Destiny Engine*** by Barry Genova (3月5日火曜日 午後4時〜午後5時)

<aside class="notes">
このトークでは、[TODO]
</asides>

## 本日のオススメ

- Destinyのコアレンダラアーキテクチャ
- GPUデータフローへのゲームシミュレーション
- ジョブ化と負荷分散の検討事項
- レイテンシリダクションテクニック
    - 入力およびレンダリングレイテンシリダクション
    - GPUを完全に飽和させ続ける
- 複雑さのカプセル化

## まえがき

- 粒度の粗い並列化
- Destinyのレンダラの目標
- シミュレーションとレンダリングの分離
- コアワークロードのジョブ化
- データ駆動レンダリングサブミッション
- 高度な最適化
- おわりに

<aside class="notes">
これは[TODO]
</asides>

# 粒度の粗い並列化

## A 10,000' View of a Frame in a Game

<aside class="notes">
ゲームの1フレームで起こっていることを高レベルで見ていこう。
</asides>

## A 10,000' View of a Frame in a Game

- ゲームオブジェクトをシミュレートする

<aside class="notes">
すべてのgame tickごとに、ゲームシミュレーションを走らせることから始める。これは物理エンジン、AI、アニメーション、ゲームプレイに影響を与えるいずれかのコードをを走らせる所である。
</asides>

## A 10,000' View of a Frame in a Game

- ゲームオブジェクトをシミュレートする
- レンダリングするものを決定する

<aside class="notes">
そして、最新のシミュレーションデータを使って、見えているであろう要素を決定する。
</asides>

## A 10,000' View of a Frame in a Game

- ゲームオブジェクトをシミュレートする
- レンダリングするものを決定する
- GPUコマンドを生成する

<aside class="notes">
そして、これらの要素をGPUコマンドのセットに変換する。我々はこの操作を"CPUサブミット"と呼んでいる。この結果はGPUに流し込む[flush]GPUコマンドバッファである。
</asides>

## A 10,000' View of a Frame in a Game

- ゲームオブジェクトをシミュレートする
- レンダリングするものを決定する
- GPUコマンドを生成する
- GPUワークを処理する
- 表示する

<aside class="notes">
[TODO]
</asides>

## A 10,000' View of a Frame in a Game

- ゲームオブジェクトをシミュレートする
- レンダリングするものを決定する
- GPUコマンドを生成する
- GPUワークを処理する
- 表示する

このパイプラインはsystem-on-a-thread設計にうまく対応する

## System-on-A-Thread設計

<aside class="notes">
Halo: Reachでは、スレッドのいくつかに主要なゲーム関数を対応させた。例えば、レンダリング、オーディオ、シミュレーションには単独のスレッドを対応させた。ここでは、各スレッドはハードウェアスレッドにも対応させた。
</asides>

##

<aside class="notes">
実行を見るための異なる方法として、frame tick diagramがある。
ここには、N番目のGame Tickでは、シミュレーションはフレームNのgame tickを計算しつつ、前のフレームをレンダリングしていることが示されている。
</asides>

##

シミュレーションが完了したら、次のgametick中にこのフレームをレンダリングし始めるようにgamestateをコピーする。
Halo: Reachでは、すべての出力システムの処理はシミュレーションスレッドが現在のgame tickのワークを終えて、完全なgamestateをコピーした後にのみ始めていた。

##

<aside class="notes">
完全なgamestateのコピーを用いて各レンダリングループの初めにシリアライズされた可視性の計算をを走らせた。可視性が行われるまで、ドローコールを生成し始めることはできない。
</asides>

## System-on-A-Threadの負荷分散

<aside class="notes">
静的なスレッドごとの負荷分散は準最適なワークロード分布を意味した。
シミュレーションやレンダリングループ(最大のワークロード)ではそのスレッドは高い使用率を示す傾向にあったが、他のスレッドはたくさんのアイドル時間が確認された。
</asides>

## System-on-A-Threadの観察結果: 悪い点

- 世代やプラットフォームをまたいでの採用は難しい
    - ヘテロジニアスなプラットフォームでスケールしない

<aside class="notes">
[TODO]
</asides>

## System-on-A-Threadの観察結果: 悪い点

- 世代やプラットフォームをまたいでの採用は難しい
    - ヘテロジニアスなプラットフォームでスケールしない
- 同期必須の完全なgamestateのダブルバッファ

<aside class="notes">
[TODO]
</asides>

## System-on-A-Threadの観察結果: 悪い点

- 世代やプラットフォームをまたいでの採用は難しい
    - ヘテロジニアスなプラットフォームでスケールしない
- 同期必須の完全なgamestateのダブルバッファ
- シリアライズされた先払いの重い可視性コスト
    - GPU idle bubblesの可能性

<aside class="notes">
[TODO]
</asides>

## System-on-A-Threadの観察結果: 良い点

- 便利なデータアクセス
- 拡張可能
- 簡単

<aside class="notes">
[TODO]
</asides>


## System-on-A-Threadの観察結果: 良い点

- 便利なデータアクセス
- 拡張可能
- 簡単
- 単純なスレッディングモデル

<aside class="notes">
[TODO]
</asides>


## System-on-A-Threadの観察結果: 良い点

- 便利なデータアクセス
- 拡張可能
- 簡単
- 単純なスレッディングモデル
- シミュレーションとレンダリングのパイプライン化された並行実行

<aside class="notes">
[TODO]
</asides>

## まえがき

TODO
