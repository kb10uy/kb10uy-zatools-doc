+++
title = 'Ad-Hoc BlendShape Mix'
date = 2024-09-04T20:45:21+09:00
images = ["component-initial.png"]
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools`

## 概要

アバターのビルド時に BlendShape (シェイプキー) の値を合成するコンポーネントです。
BlendShape の合成のためだけに Blender などで編集して再インポートする手間を省き、非破壊的に適用できるようになります。

主な用途としては、表情アニメーションで必要になることがある BlendShape 操作の打ち消しをアニメーション編集なしで達成することなどを想定しています。


## 使い方

SkinnedMeshRenderer が付いている GameObject に `KusakaFactory/Mix BlendShapes on Build` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

リストに合成したいシェイプキーの組み合わせを設定します。

* **コピー元**: 値をコピーする元になる BlendShape の名前を指定します。ここで指定した BlendShape は(他の定義でコピー先に指定されない限り)維持されます。
* **コピー先**: 値をコピーする先になる BlendShape の名前を指定します。ここで指定した BlendShape が改変されます。
* **係数**: コピー元の値をどれくらいの重みでコピーするかを指定します。-1～+1 で指定できます。

例として、以下のようなセットアップがあったとします。

![Component View](./example-1.png?width=480px "設定例")

このとき、 *eye_brink_1* には *eye_tare_1* の BlendShape の値が**逆向きに** 100% 合成されます。
つまり、アバターのビルド後に *eye_brink_1* と *eye_tare_1* を両方 100% にするとビルド前に *eye_brink_1* **だけ**を 100% にした状態と同じになります。

## その他

コピー元の BlendShape は常に合成前の値を参照します。 *A → B* と *B → C* を定義した場合、*C* に *A* の値は反映されません。
*A → B*, *A → C*, *B → C* というふうに定義すると *C* に *A, B* 両方の値が合成されます。

あるコピー元・コピー先のペアを係数を 2 以上にしたい場合、同じペアを複数定義することで代用できます。
同じペアに対して係数が 1, 1, -0.5 の定義があった場合、最終的な係数は 1.5 になります。

コピー元とコピー先は同じ BlendShape でも問題ありません。この場合、擬似的に限界突破ツールのようなことができます。
