+++
title = 'Ad-Hoc BlendShape Split'
date = 2025-08-20T23:24:17+09:00
images = ["component-initial.png"]
weight = 203
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 3.2.0)

## 概要

アバターのビルド時に既存の BlendShape (シェイプキー) をある平面で分割してそれぞれを新規 BlendShape として追加するコンポーネントです。

主に [Ad-Hoc BlendShape Mix](../adhoc-blendshape-mix/_index.md) と組み合わせて使うことを想定しています。

NDMF Preview に対応していますが、パフォーマンス上の問題によりデフォルトで無効になっています。


## 使い方

SkinnedMeshRenderer が付いている GameObject に `Zatools Bend Normal on Build` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

* **分割する BlendShape**: 分割したい BlendShape を名前で指定します。各 BlendShape の正確な名前の取得には [BlendShape List](../../editor-extension/blendshape-list/_index.md)などを利用してください。
* **基底**: 分割する際の空間的な基底となる Transform を指定します。この Transform から見て YZ 平面で空間を分割した際、+X 側にある頂点が「右」、 -X 側にある頂点が左となります。None の場合はアバタールートの Transform (通常はアバター正面方向)を採用します。
* **サフィックス(左側/右側)**: 生成される BlendShape の名前の末尾に追加される文字列を指定します。デフォルトはそれぞれ `_sL` と `_sR` です。例えば `Blink` を分割する場合、生成される BlendShape の名前は `Blink_sL` と `Blink_sR` となります。


## その他

各プレフィックスを空欄にすると、その側の BlendShape は生成されなくなります。また、既存の BlendShape と名前が重複する場合も生成されません。

このコンポーネントの処理は NDMF アバタービルドプロセスのうち Generating Phase で実行されます。
Transforming Phase 以降に動作する NDMF プラグインは生成された BlendShape を発見可能ですが、Edit Mode では(たとえ NDMF Preview が有効でも)表示されている **SkinnedMeshRenderer の BlendShapes のリストには表示されません。**
このため、生成される BlendShape を他プラグインで利用するためには存在しない BlendShape を名前で指定できる必要があります。

概要にもあるとおり、Ad-Hoc BlendShape Mix より前に動作するためそれと併用することで「両目同時にしか動かない BlendShape を別の片目ずつの BlendShape に適用させたい」というようなケースに利用可能です。
