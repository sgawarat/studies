# D3D12 and Vulkan: Lessons learned

## バリア制御(Barrier control)

- バリアはD3D12/Vulkan世代の新しいコンセプト。
- 残念ながら、**みんな** 間違っている。
- 2つの失敗ケース
    - 多すぎ(too many) or 広すぎ(too broad): **パフォーマンスの低下**
    - バリア欠落: **データ破壊(corruptions)**
- D3D11ドライバはその傘の下でうまくこれをこなしている。

## つまるところのバリアとは？(What's a barrier, anyway?)

- レンダターゲットをテクスチャにするとき。
    - decompressionが必要になるかもしれない。

- UAVをリソースにするとき。
    - 誤って行えば、コストがかかる。 -- flushやアイドル待ち
    - 正しく行えば、トランジションが実質タダに。

## バリアが欠落していると(Missing barriers)

- フォーマット問題。 -- GPUやドライバ特有のcorruption。
- 同期問題。 -- 時間依存のcorruption。

## サブリソース(Subresources)

- 個別に追跡する必要がある。
    - ダウンサンプリング。
    - シャドウマップアトラス。
- すべてのサブリソースをトランジションするなら、１つ１つやらずに`D3D12_RECOURCE_BARRIER_ALL_SUBRESOURCES`を使う。

## Placedリソースと初期ステート(Placed resoruces & initial states)

- placedリソースなどとして作成したレンダターゲットは使う前に **必ず** クリアする。
- Go into **clear state directly**, ランダムなステートやトランジションで始めない。

## 不必要なトランジション(Unnecessary transitions)

- 間違った型へのトランジション。
    - 普通はないが、時折起こる。
    - バリデーションレイヤで必ず確認する。

- 読み込みから読み込みへのトランジション。
    - 例えば、インデックスバッファからシェーダリソースへ。
    - 今後に必要なすべてのステートにまとめる。
        - UAV -> IB -> SRVなら、UAV -> IB|SRVにまとめる。

## 高価なトランジション(Costly transitions)

- `COMMON`はコピー/表示のためであり、一般的な"全部入り"ステートではない。
- 通常にシェーダアクセスがしたいなら、
    - D3D12: `PS_RESOURCE | NON_PS_RESOURCE`
    - Vulkan: `VK_ACCESS_SHADER_READ_BIT`
