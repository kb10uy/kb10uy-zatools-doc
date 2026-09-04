+++
title = 'Ad-Hoc Advanced Mesh Duplication'
date = 2026-09-05T00:00:00+09:00
images = ["component-initial.png"]
weight = 206
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 6.0.0)

**上級者向けのコンポーネントです！**

## 概要

アバターのビルド時に既存の SkinnedMeshRenderer から部分的にメッシュを複製しつつ、それらの頂点属性を自由に変更できるコンポーネントです。

NDMF Preview に対応していますが、パフォーマンス上の問題によりデフォルトで無効になっています。

## 使い方

アバター内に作成した空の GameObject に `Zatools Advanced Duplicate Mesh on Build` を追加します。SkinnedMeshRenderer が同時に追加されます。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

- **複製対象**: メッシュの複製元になる SkinnedMeshRenderer。指定しない場合実行されません。

### 頂点の選択

- **選択マスクテクスチャ**: 複製対象の頂点を選択するためのマスクテクスチャ。元テクスチャが sRGB でインポートされている場合 sRGB to Linear 変換が適用されます。
- **サンプリング UV チャンネル**: **選択マスクテクスチャ**をどの UV チャンネルでサンプリングするか指定します。
- **閾値**: 下の **ZAX 式**の返り値のうち、この値以上の頂点によって構成されるポリゴンが複製されます。
- **ZAX 式**: [ZAX](../../zax/_index.md) で頂点選択の式を指定します。

### 頂点属性の改変

要素を追加すると上から順に適用されます

- **有効**: この改変要素を適用するかどうか。
- **書き込み先**: 書き込み先の頂点属性。
- **追加テクスチャ**: 下の **ZAX 式**で追加で参照するテクスチャ。元テクスチャが sRGB でインポートされている場合 sRGB to Linear 変換が適用されます。
- **サンプリング UV チャンネル**: **追加テクスチャ**を*この改変要素が適用されるタイミングで*どの UV チャンネルでサンプリングするか指定します。
- **ZAX 式**: ZAX で書き込む頂点属性の計算式を指定します。

### 後処理

- **法線を再計算する**: `Mesh.RecalculateNormals()` を実行するかどうか。
- **接線を再計算する**: `Mesh.RecalculateTangents()` を実行するかどうか。

## ZAX 式で利用可能な変数について

- `@position` (float3) 頂点座標
- `@normal` (float3) 頂点法線
- `@tangent` (float4) 頂点接線
- `@color` (float4) 頂点カラー
- `@uv0~7` (float4) UV 値
- `@scratch0~3` (float4) スクラッチ領域。改変要素間でデータを受渡しするのに利用可能で、頂点には書き込まれません

これ以外に、頂点の選択では `@mask` (float4) が、それぞれの改変要素では `@texture` (float4) が利用可能です。
