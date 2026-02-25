+++
title = 'Ad-Hoc Mesh Split'
date = 2026-02-25T23:00:00+09:00
images = ["component-initial.png"]
weight = 204
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 3.4.0)

## 概要

アバターのビルド時にメッシュの一部を分割し、別のマテリアルを割り当てるコンポーネントです。

主な用途としては顔・髪の一部だけシェーダーごと入れ替える、Android/iOS 対応時に MatCap マスクの部分を別マテリアルにするといったものを想定しています。

NDMF Preview に対応しています。


## 使い方

SkinnedMeshRenderer が付いている GameObject に `Zatools Split Mesh on Build` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

* **マスクテクスチャ**: 別マテリアルを割り当てる領域を指定するマスクテクスチャ。None の場合は全て白いテクスチャが仮想的に割り当てられます。
* **マスクのサンプリング方法**: マスクテクスチャのうち白と黒どちらを対象領域として採用するかを指定します。初期値は White で白い部分のポリゴンが対象になりますが、白地に黒塗りとなるマスクの場合は Black を指定してください。
* **分割対象のマテリアル**: 指定されている SkinnedMeshRenderer のうち、どのマテリアルが割り当てられているポリゴンを対象とするかを指定します。None の場合は全てのマテリアルの領域を対象とします。
* **割り当てるマテリアル**: 採用された領域に割り当てるマテリアルを指定します。


## その他

UV や法線などは分割前のものが維持されます。
