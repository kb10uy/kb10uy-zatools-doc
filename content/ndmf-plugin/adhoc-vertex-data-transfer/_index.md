+++
title = 'Ad-Hoc Vertex Data Transfer'
date = 2026-07-20T21:30:00+09:00
images = ["component-initial.png"]
weight = 201
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 5.0.0)

## 概要

アバターのビルド時に頂点データにテクスチャのデータを転送する(ベイクする)コンポーネントです。
頂点カラーや拡張 UV などに非破壊的にデータを設定できます。

主な用途としては、アウトラインマスクのデータを頂点カラーに焼き込んだりシェーダーで特定の UV インデックス向けにデータを設定するものを想定しています。

NDMF Preview に対応しています。


## 使い方

SkinnedMeshRenderer が付いている GameObject に `Zatools Transfer Vertex Data on Build` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

* **ソーステクスチャ**: 転送するデータの元となるテクスチャを指定します。
* **ソース UV**: どの UV で**ソーステクスチャ**をサンプリングするか指定します。通常は UV0 で問題ありません。
* **書き込み先**: データの転送先を指定します。
    - *Disabled*: 処理しない
    - *Vertex Color*: 頂点カラー
    - *UV 0～7*: 特定の UV チャンネル
* **転送モード**: **ソーステクスチャ**のデータをどのように転送するかを指定します。一部のモードでは追加で値を入力するフィールドが表示されます。
    - *Copy*: そのままコピー
    - *1 - x*: 0～1 の値が 1～0 になるようにコピー
    - *Constant, Luminance*: xyz には入力された定数値、w にはテクスチャの RGB から計算された輝度をコピー
    - *Constant (0 to 1), Luminance*: 基本的な動作は *Constant, Luminance* と同じで、定数部分について -1～+1 が 0～1 になるようにマッピングされる
    - *Constant * Luminance, Constant*: xyz には入力された定数値にテクスチャの RGB から計算された輝度をかけたもの、w には別に入力された定数値をコピー
    - *Constant * Luminance (0 to 1), Constant*: 基本的な動作は *Constant * Luminance, Constant* と同じで、定数部分について -1～+1 が 0～1 になるようにマッピングされる

### アウトラインマスク用のプリセット

適用したい SkinnedMeshRenderer がある GameObject に対し、右クリックメニューの `Zatools: kb10uy's Various Tools > Add Vertex Data Transfer (Outline Mask Preset)` でアウトラインマスク用にセットアップされたコンポーネントを生成できます。
これは各種設定が lilToon の `RGBA → Normal & Width` モードなどで使えるようになっており、**ソーステクスチャ**に既存のアウトラインマスクのテクスチャを指定するだけで適用できます。
