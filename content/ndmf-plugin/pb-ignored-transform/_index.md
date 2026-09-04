+++
title = 'PB Ignored Transform'
date = 2026-08-14T20:00:00+09:00
images = ["component-initial.png"]
weight = 1001
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 5.1.0)

## 概要

アバターのビルド時に GameObject/Transform を PhysBone の Ignored Transform に登録します。

MA Bone Proxy で移動する予定のメッシュなどに付与することで、影響元になる PB のコンポーネントを探して手動で設定することなく影響外になるように設定できます。
また、ビルド中に生成される PB の影響内に移動する場合にも適用することができます。

## 使い方

PB から無視したい GameObject に `Zatools Ignore Self from PB` を追加します。
設定項目はありません。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")
