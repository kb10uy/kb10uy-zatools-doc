+++
title = 'Convex Depth Wrapper'
date = 2026-05-16T05:30:00+09:00
images = ["component-initial.png"]
weight = 105
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 3.7.0)

## 概要

アバターのビルド時に凸包の深度を書き込むメッシュを生成します。

ワールド側で AO が適用される際に白目の部分が遮蔽されているとみなされて意図しない影が加算されてしまう問題を軽減できます。

NDMF Preview に対応しています。


## 使い方

顔面の SkinnedMeshRenderer が付いている GameObject (通常 Body です) に `Zatools Convex Depth Wrapper` を追加します。

設定項目はありません。

NDMF Preview を有効にすると凸包メッシュ形状のクロスハッチが描画されるようになります。
![Preview](./preview.png?width=640px "NDMF Preview を有効にした状態")
