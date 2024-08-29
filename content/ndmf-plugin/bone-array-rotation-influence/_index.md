+++
title = "Bone Array Rotation Influence"
date = 2024-08-27T22:18:35+09:00
images = ["component-initial.png"]
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools`

## 概要

複数のボーンの配列について、それらの親に仮のボーンを挿入し、隣のボーンから Rotation Constraint によって角度の影響を受けるようにします。

主にスカートの PhysBone の設定などで特定のボーン鎖だけが移動して貫通・破綻してしまうのを防ぐ目的での使用を想定していますが、それ以外の用途にも使用可能です。

## 使い方

アバター内の任意の GameObject に `KusakaFactory/Apply Rotation Influence for Bone Array` を追加します。スカートに対して適用する場合はスカートボーンの共通の親に設置するのがおすすめです。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

それぞれの項目について、以下のように設定します。

-   **Bone array is close loop**: 設定するボーン配列が輪をなしている(最初の要素と最後の要素が隣あわせになる)場合はチェックを入れてください。
-   **Offset Distance**: 元のボーンから親になる仮のボーンをどれだけ離すかの設定です。デフォルトは 1cm ですが、荒ぶりが気になる場合は値を小さくしてください。
-   **Sources**: ボーン配列と、それぞれのボーンが隣の要素から影響を受ける度合いです。リストの順番が意味をもつので、上から下に向かって順番に輪になるように設定してください。

これでアバターのビルド時に親ボーンの挿入と Rotation Constraint の挿入が非破壊的に実行されるようになります。必要に応じて Influence の値を適宜調整してください。

### Sources Manipulation (自動設定)

![Sources Manipulation](./sources-manipulation.png?width=480px "Sources Manipulation を開いた状態")

上述したようなスカートボーンの共通の親に設置した場合などに便利な操作が含まれています。

-   **Replace with direct children**: このコンポーネントが付いている GameObject の直接の子をリストに設定します。
    -   Influence は下のスライダーの値を採用します。デフォルトは 1.0 です。
    -   リストに設定される順番は Hierarchy に上から並んでいる順ではなく、このコンポーネントが付いている GameObject の Y 軸正側 から見た XZ 軸で Z 軸に近いものから左周りになります。**要するに、いい感じにぐるっと輪になるような順番になります。**
-   **Update All Influence**: 左側のスライダーの値で全ての Influence を更新します。

## その他

Rotation Constraint に設定される Rotation Offset の値は正しくない(手動で追加して Activate した場合の値と異なる)可能性があります。ご了承ください。
