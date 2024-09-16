+++
title = 'Custom Mipmap Combiner'
date = 2024-09-14T20:22:43+09:00
images = ["window.png"]
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools`

## 概要

最上位のテクスチャとは異なるテクスチャをもとにしたミップマップを持つ Texture2D アセットを生成するツールです。
通常、ミップマップは最上位テクスチャを繰り返し縮小することで生成されますが[^1]、このツールを使うことでミップレベルによって全く関係ない内容が得られるテクスチャアセットを生成することができます。

なお、(一部の Direct3D ネイティブな画像形式を除き)主要な画像ファイルフォーマットはミップマップをデータとして持てないため、**このツールで生成したテクスチャアセットを PNG ファイルなどで出力することはできません。**

## 使い方

Window > kb10uy > Custom Mipmap Combiner を選択すると以下のようなウィンドウが開きます。

![Window](./window.png?width=360px "Custom Mipmap Combiner ウィンドウ")

* **テクスチャサイズ:** 生成されるテクスチャの一辺のピクセル数を 256～4096 から選択します。なお、長方形や NPOT サイズのテクスチャは生成できません(コピー元として指定することはできます)。
* **フォーマット**: 生成されるテクスチャフォーマットを指定します。いずれも非圧縮フォーマットであることに注意してください。
    - RGBA32: 各チャンネルは 8bit 整数になります。 sRGB/Linear 変換の都合上、**コピー元が sRGB テクスチャの場合色がわずかに変化します[^srgb-lack]**。
    - RGBAHalf: 各チャンネルは 16bit 浮動小数点数になります。**テクスチャサイズは RGBA32 の倍になります。**
    - RGBAFloat: 各チャンネルは 32bit 浮動小数点数になります。**テクスチャサイズは RGBA32 の 4 倍になります。**
* **ミップレベル 0**: 最上位のテクスチャのコピー元を指定します。

**ミップマップコピー元定義**のリストに必要に応じて要素を追加します。**ミップレベル >=** の数値が同じエントリがある場合、最も先頭のものが採用されます。

* **ミップレベル >=**: ミップレベル何番以降でこのエントリのテクスチャを参照するようにするか指定します。
* **コピー元テクスチャ**: 当該ミップレベルでコピーするテクスチャのコピー元を指定します。
* **バイアス**: ミップバイアスを指定します。 0 が指定されている場合、サンプリングの際に**コピー元テクスチャ**の同じミップレベルを参照します。

![Example](./example.png?width=360px "設定例")

例として上記のように設定した場合、以下のように動作します。

* ミップレベル 0: M_B2_Boots のミップレベル 0 からコピー
* ミップレベル 1: M_B2_Boots のミップレベル 1 からコピー
* ミップレベル 2: M_B2_Metal_MatCap のミップレベル 2 からコピー
* ミップレベル 3: M_B2_Metal_MatCap のミップレベル 3 からコピー
* ミップレベル 4: M_B2_Boots_Smoothness のミップレベル 4 からコピー
* ミップレベル 5: M_B2_Boots_Smoothness のミップレベル 5 からコピー
* ...

最後に、最下部のテキストフィールドに保存先のパスを指定して Generate ボタンを押すと生成されます。

[^srgb-lack]: sRGB はそのカーブの特性上暗い色をより細かく表現できるようになっています。inear 空間に変換して再度整数化すると、暗い色のサンプリング感覚が変換前よりも荒くなってしまいます。

## その他

このツールで生成したテクスチャアセット(.asset ファイルのもの)は VRCSDK などの AssetPostProcessor の処理の対象外になります。