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

### 上級者向け: 任意のマテリアルを割り当てる (>= 3.7.2)

**上級者向け/割り当てるマテリアル** に何らかのマテリアルを設定した場合、デフォルトの深度書き込みマテリアルの代わりにそのマテリアルが設定されます。
UV は生成されていませんが何かに使えるかもしれません。


### MA Material Swap で切り替えられるようにする

任意の GameObject に対し、右クリックメニューの `Zatools: kb10uy's Various Tools > Add MA Material Swap for Depth Wrapper` で深度を書き込むマテリアルを無効化する Material Swap コンポーネントを追加できます。
主にMA Menu Item に追加することを想定しています。デフォルトでこの Material Swap が有効な状態だとこのコンポーネントが無効化されたままになってしまうため注意してください。


## 上級者向け: 通常時オフでセットアップする (>= 3.7.2)

適用したい SkinnedMeshRenderer がある GameObject に対し、右クリックメニューの `Zatools: kb10uy's Various Tools > (Advanced) Inverted Convex Depth Wrapper Setup with Toggle` で**デフォルト無効・Expressions Menu から有効化するセットアップ**を生成できます。
このとき先にコンポーネントを追加する必要はなく、この処理の中で専用の設定で追加されます。

![Component](./inverted.png?width=800px "通常時オフのコンポーネントをオンにする MA Menu Item")

適用対象の GameObject の子に上のような Menu Item が追加されますので、適宜 MA Menu Install Target で移動するなどしてお使いください。


## 上級者向け: 個別の SkinnedMeshRenderer に生成する (>= 3.8.0)

空の GameObject にこのコンポーネントを追加すると、以下のように空の SkinnedMeshRenderer とともに **生成対象** というフィールドが出現します。
この **生成対象** に凸包メッシュを生成したい SkinnedMeshRenderer (`Body` など)を設定すると、生成元 Mesh の submesh ではなく独立した Mesh として生成されるようになります。

![Component](./separate.png?width=800px "個別セットアップ")

通常の、適用対象に直接追加する方法で問題が発生する場合はこちらの方法をお試しください。

## 備考

ShadowCaster パスで深度を上書きするため、以下のような副作用が発生する可能性があります。問題があるシーンでは上記の手順で一時的に無効化するなどしてください。

* メッシュが落とす影の形が変化する
    - 顔のメッシュの場合は鼻先から目元・額にかけて凹んでいることが多いため、この部分が凸になった分だけ影が落ちるようになります。
* 被写界深度エフェクトでわずかにピントが合う深度が変化する
    - ポストプロセスの AO 以外に、VRChat 標準カメラや [VirtualLens2](https://booth.pm/ja/items/2280136) 等の被写界深度シミュレーションも ShadowCaster パスの描画結果を参照している。
    - このためピントが合う位置は凸包メッシュの表面になり、目の周辺よりはわずかに手前側にずれる(とはいえ前髪よりは奥になるのでそんなに問題はないかも)。

一方 ShadowCaster パス以外では何も描画しないため、半透明メッシュが RenderQueue の前後によって貫通したりするようなことは発生しません。
