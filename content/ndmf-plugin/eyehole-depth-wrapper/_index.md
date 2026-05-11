+++
title = 'Eyehole Depth Wrapper'
date = 2026-05-11T13:30:00+09:00
images = ["component-initial.png"]
weight = 105
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 3.6.0)

## 概要

アバターのビルド時に眼の周りに深度のみを書き込むメッシュを生成します。

ワールド側で AO が適用される際に白目の部分が遮蔽されているとみなされて意図しない影が加算されてしまう問題を軽減できます。

NDMF Preview に対応しています。


## 使い方

顔面の SkinnedMeshRenderer が付いている GameObject (通常 Body です) に `Zatools Eyehole Depth Wrapper` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

* **まばたき BlendShape 名**: メッシュを生成したい領域の、眼の頂点が動く BlendShape の名前。初期値は `vrc.blink` で、多くのアバターはこの BlendShape でまばたきするようになっています。
* **凸包の深さ**: メッシュを生成する頂点を選択する際の「深さ」。値を小さくすると顔面に近い頂点のみが選ばれ、大きくすると奥の方の頂点まで選ばれるようになります。初期値は 25mmです。
* **基底**: 右目と左目を分割する際の空間的な基底となる Transform を指定します。この Transform から見て YZ 平面で空間を分割した際、+X 側にある頂点が「右」、 -X 側にある頂点が左となります。None の場合はアバタールートの Transform (通常はアバター正面方向)を採用します。
* **割り当てるマテリアル**: 採用された領域に割り当てるマテリアルを指定します。

NDMF Preview を有効にするとメッシュが生成される位置にクロスハッチが描画されるようになります。このクロスハッチが眼窩(へこんでいる白目の部分)を適切に覆うようにしてください。
![Preview](./preview.png?width=640px "NDMF Preview を有効にした状態")

## メッシュ 生成方法 (発展的なトピック)

1. **まばたき BlendShape 名**で動く頂点のうち、**基底**の座標系で最も手前 (+Z) の頂点から**凸包の深さ**以内のものだけに絞り込む
2. 残った頂点を**基底**の YZ 平面 (X=0) で左右に分割し、XY 平面に射影して二次元の点群にする
3. 左右それぞれの点群について Andrew's Algorithm で凸包を求める
4. 各凸包の重心を中心とした Triangle Fan でポリゴンを生成し、1 つのサブメッシュとして元のメッシュに追加する
