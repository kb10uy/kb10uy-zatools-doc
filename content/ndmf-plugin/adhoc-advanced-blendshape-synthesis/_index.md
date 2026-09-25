+++
title = 'Ad-Hoc Advanced BlendShape Synthesis'
date = 2026-09-25T16:00:00+09:00
images = ["component-initial.png"]
weight = 107
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 6.2.0)

**上級者向けのコンポーネントです！**

## 概要

アバターのビルド時に BlendShape (シェイプキー) を既存のそれらから合成して新規追加できるコンポーネントです。
BlendShape の合成のためだけに Blender などで編集して再インポートする手間を省き、非破壊的に適用できるようになります。

フェイシャルトラッキング用の BlendShape を合成によって作成することなどを想定しています。

NDMF Preview に対応していますが、パフォーマンス上の問題によりデフォルトで無効になっています。

## 使い方

SkinnedMeshRenderer が付いている GameObject に `Zatools Advanced BlendShape Synthesis on Build` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

- **ソース BlendShape**: 合成時に参照する BlendShape のリスト。

### 合成する BlendShape

- **名前**: 作成する BlendShape 名。既存の BlendShape と重複する場合は定義自体が無視されます。
- **初期ウェイト**: SkinnedMeshRenderer に設定するウェイト。NDMF Preview を有効にしても SkinnedMeshRenderer のインスペクター側には追加された BlendShape は表示されないためここで指定します。
- **ZAX 式**: [ZAX](../../zax/_index.md) で頂点・法線・接線の delta 値の式を指定します。**全て float3 でスタックに 3 つの値を残す必要がある**ので注意してください(デフォルトの式は index 0 の BlendShape をそのままコピーします)。

## ZAX 式で利用可能な変数について

- `@position` (float3) 元の頂点座標
- `@normal` (float3) 元の頂点法線
- `@tangent` (float4) 元の頂点接線
- `@v0, @v1, ... @vN` **ソースBlendShape**で指定した BlendShape の #N の頂点位置差分。
- `@n0, @n1, ... @nN` **ソースBlendShape**で指定した BlendShape の #N の法線差分。
- `@t0, @t1, ... @tN` **ソースBlendShape**で指定した BlendShape の #N の接線差分。
