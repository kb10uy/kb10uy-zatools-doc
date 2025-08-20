+++
title = 'Ad-Hoc Normal Bending'
date = 2025-08-04T11:30:00+09:00
images = ["component-initial.png"]
weight = 202
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 3.1.0)

## 概要

アバターのビルド時にメッシュ(の一部)の法線を指定した方向に向かって「曲げる」コンポーネントです。
法線の編集のためだけに Blender などで編集して再インポートする手間を省き、非破壊的に適用できるようになります。

主な用途としては、口腔内の舌や歯などの法線を正面方向に向けることなどを想定しています。

NDMF Preview に対応しています。


## 使い方

SkinnedMeshRenderer が付いている GameObject に `Zatools Bend Normal on Build` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

* **法線方向**: 法線を曲げる方向の Transform を指定します。 None の場合はアバタールートの Transform (通常はアバター正面方向)を採用します。この Transform の Z 軸正の方向(forward)に向かって曲げられます。
* **ウェイト**: 法線方向で指定した Transform にどれくらい曲げるかを指定します。0 のときは元の法線が維持され、1 のときは法線方向と完全に一致します。中間の値はクォータニオンで補間されます。
* **マスクテクスチャ**: 法線を曲げる部分を指定するマスクテクスチャ。None の場合は全て白いテクスチャが仮想的に割り当てられます。 [AAO Remove Mesh by Mask に含まれるエディタ](https://vpm.anatawa12.com/avatar-optimizer/ja/docs/reference/remove-mesh-by-mask/#mask-texture-editor)などを利用すると便利です。
* **マスクのサンプリング方法**: マスクテクスチャのうち白と黒どちらを曲げるポリゴンの領域として採用するかを指定します。初期値は White で白い部分のポリゴンが対象になりますが、上記エディタで作成する場合白地に黒塗りとなるので Black を指定してください。


## その他

法線を曲げた結果は輪郭線の出る方向などに影響するので注意してください。
