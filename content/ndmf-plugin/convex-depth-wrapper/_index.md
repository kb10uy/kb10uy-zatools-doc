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

顔面などの SkinnedMeshRenderer が付いている GameObject (通常 Body です) に `Zatools Convex Depth Wrapper` を追加します。

設定項目はありません。

NDMF Preview を有効にすると凸包メッシュ形状のクロスハッチが描画されるようになります。
![Preview](./preview.png?width=640px "NDMF Preview を有効にした状態")


### MA Material Swap で切り替えられるようにする

任意の GameObject に対し、右クリックメニューの `Zatools: kb10uy's Various Tools > Add MA Material Swap for Depth Wrapper` で深度を書き込むマテリアルを無効化する Material Swap コンポーネントを追加できます。
主にMA Menu Item に追加することを想定しています。デフォルトでこの Material Swap が有効な状態だとこのコンポーネントが無効化されたままになってしまうため注意してください。

## 備考

ShadowCaster パスで深度を上書きするため、以下のような副作用が発生する可能性があります。問題があるシーンでは上記の手順で一時的に無効化するなどしてください。

* メッシュが落とす影の形が変化する
    - 顔のメッシュの場合は鼻先から目元・額にかけて凹んでいることが多いため、この部分が凸になった分だけ影が落ちるようになります。
* 被写界深度エフェクトでわずかにピントが合う深度が変化する
    - ポストプロセスの AO 以外に、VRChat 標準カメラや [VirtualLens2](https://booth.pm/ja/items/2280136) 等の被写界深度シミュレーションも ShadowCaster パスの描画結果を参照している。
    - このためピントが合う位置は凸包メッシュの表面になり、目の周辺よりはわずかに手前側にずれる(とはいえ前髪よりは奥になるのでそんなに問題はないかも)。
