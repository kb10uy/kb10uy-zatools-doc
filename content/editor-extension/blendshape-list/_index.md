+++
title = 'BlendShape List'
date = 2024-09-08T11:55:57+09:00
images = ["window.png"]
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools`

## 概要

SkinnedMeshRenderer の全ての BlendShape の名前をテキストとしてコピーできます。
基本的に BlendShape の選択はドロップダウンなどで既に表示されている内容から選択する形式のため文字列として必要になることは少ないですが、[declavatar](https://declavatar.kb10uy.dev) などで必要になることがあります。


## 使い方

Window > kb10uy > BlendShape List を選択すると以下のようなウィンドウが開きます。

![Window](./window.png?width=360px "BlendShape List ウィンドウ")

**メッシュ** に任意の SkinnedMeshRenderer コンポーネントが付いている GameObject かアセットを指定すると、下のテキストボックスに BlendShape の名前のリストが各行 1 つずつで表示されます。

**テキストファイルに保存** ボタンで、テキストボックスに表示されている内容をファイルに保存することができます。
