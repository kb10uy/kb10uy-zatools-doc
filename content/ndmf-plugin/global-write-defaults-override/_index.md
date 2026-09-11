+++
title = "Global WD Override"
date = 2024-09-11T19:57:00+09:00
images = ["component-initial.png"]
weight = 303
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 6.1.0)

## 概要

Playable Layers に指定される AnimatorController 内のステートの Write Defaults 設定を上書きします。
概ね [VRCFury の Fix Write Defaults](https://vrcfury.com/components/other#fix-write-defaults) と同等の機能です。

[MA の Merge Animator](https://modular-avatar.nadena.dev/ja/docs/reference/merge-animator) にも Write Defaults を揃える機能がありますが、あちらはあくまでアバターの AnimatorController で採用されている値に揃える挙動なのに対し、**このコンポーネントは MA の処理よりも後に全てのステートをさらに上書きします**。
Merge Animator の挙動では不足する場合の利用を主に想定しています。

## 使い方

アバタールートの GameObject に `Zatools Global WD Override` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

- **モード**: Write Defaults の値をどちらに倒すかを設定します。デフォルトは `Force ON` です。
