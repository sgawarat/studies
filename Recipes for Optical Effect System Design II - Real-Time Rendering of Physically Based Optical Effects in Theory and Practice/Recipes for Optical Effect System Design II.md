---
title: Recipes for Optical Effect System Design II [Gotanda2015c]
numberSections: false
---
# 現実のレンズ構造[Real Lens Structure]

- フォトリアリスティックな(物理ベース)光学効果を達成するのに役立つ。
    - 以下のための実装
        - 光学の口径食
        - フォーカス呼吸
        - 収差と歪曲[distortion]を伴うボケ
        - レンズゴースト

# 物理的な制限[Physical Limitations]

- レンズ構造はレンズパラメータに対する制限を与えもする。
    - フォーカス距離
    - F値
    - ズーム

# レンズデータベース[Lens Database]

- レンズデータベースは実装とパラメータ制限の両方で役立つ。
    - この制限は不正確なカメラパラメータを設定できなくする。
        - 小さすぎるF値(例えば、F0.x)
        - 大判[big format]のセンサーで近すぎるフォーカス距離

<!-- p.5 -->

- 我々のレンズデータベースは大量のパラメータを近似する。

```cpp
struct LensParameter
{
    string szName[64];
    u8 nAppertureAngleNumber;

    f32 fDesignedFilmSize;

    f32 fMinFStop;
    f32 fMaxFStop;
    f32 fFStopZoom;

    f32 fMinFocusDepth;

    f32 fMinProjectionDistance;
    f32 fMaxProjectionDistance;
//
//                                        
//       -----------------               Vignetting Distance
//         ||              ---------------  <----------->
//         ||                             ----------------               |
//         ||                  ||    |                 ||                |
//       Entrance              ||  Open Ap ||Vignetting||                |
//       Size                  ||   Size   ||Size      ||                |
//         ||                  ||    |                 ||                |
//         ||                             ----------------               |
//         ||              ---------------               <-Frange Back ->
//       -----------------           ^Iris
//                                    <- Iris Distance ->
//          <------------ Entrance Distance ------------>
//
    f32 fEntranceDistance;
    f32 fEntranceSize;
    f32 fApertureDistance;
    f32 fOpenApertureSize;
    f32 fVignettingDistance;
    f32 fVignettingSize;
    f32 fFrangeBack;
    f32 fNaturalVignettingPower;
    f32 fFocusingZoomAjustiveRateWIDE;
    f32 fFocusingZoomAjustiveRateTELE;

    u8 nFocalLengths;
    u8 nApertures;
    f32 afFocalLength[5];
    f32 afAperture[5];
    f32 afVignettingEV[5][5];
};
```

<!-- p.6 -->

- 我々のデータベースには大量のレンズがある。
    - このテーブルはデータベースに含まれるいくつかのレンズを示す。

|レンズデータベースの例|
|-|
|Kanon EF24mm F1.4L USM|
|Kanon EF300mm F2.8L IS USM|
|Kanon EF24-70mm F2.8L USM|
|Kanon EF100mm F2.8 macro|
|Kanon EF28-300mm F3.5-5.6L IS USM|
|AskaNP EF28-300mm F2.0|
|AskaNP EF14-1200mm F1.4|
|AskaNP EF8-2400mm F1.0|

# 現実のレンズ？[Real Lens?]

- これらのレンズは非現実的か？
    - もし存在するなら、
        - とても高価
        - とても重く大きい

# 非現実的なレンズ[Unrealistic Lenses]

- (35mmフォーマットの)現実のレンズはときに扱いやすくないことがある。
    - アーティストはシーンに対して適切なレンズを選び取ることは手間がかかりすぎる[too costly]と感じるかもしれない。
        - レンズが交換可能なカメラと似た状況である！

<!-- p.9 -->

- レンズデータベースは物理ベース光学効果を実装するのに必要である。
    - 仮想的なレンズ
    - 有用性[usability] vs. 物理的妥当性[physical plausibility]

# 仮想的なレンズ[Virtual Lens]

- まだ現実的 -> AskaNP EF28-300mm F2.0
- あまり現実的ではないが、より便利 -> AskaNP EF14-1200mm F1.4
- 素晴らしい！これが欲しい！ -> AskaNP EF8-2400mm F1.0
- レンズ構成(パラメータ制限)もこの現実性のルールに則る。

# おわりに[Conclusion]

- レンズデータベースは実装とパラメータ制限の両方で役立つ。
    - 光学効果の物理的にもっともらしい実装
    - おかしなパラメータを設定できなくする。
    - 現実のレンズは不便すぎるときがある。

# 参考文献[References]
