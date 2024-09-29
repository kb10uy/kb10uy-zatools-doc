+++
title = "Enhanced EyePointer Installer"
date = 2024-08-27T22:18:25+09:00
images = ["component-initial.png"]
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools`

**2024-09-03 以降は[Automatic EyePointer Installer](https://github.com/kb10uy/eye-pointer-installer) ではなくこちらをご利用ください。**

## 概要

Siromori 氏の [Eye Pointer](https://booth.pm/ja/items/4742883) のセットアップを自動化するスクリプトです。
Constraint の挿入などをアバターのビルド時に自動で実行します。

## 使い方

Eye Pointer のプレハブをアバターに追加したあと、その GameObject に `KusakaFactory/Enhanced EyePointer Installer` を追加します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

* **VRC Constraint を使ってセットアップする**: Unity の Aim Constraint ではなく VRC Aim Constraint を目のボーンに設定します。Eye Pointer が v1.2 以降である必要があります。
* **ダミーボーンを挿入する**: 元の目のボーンの上に `DummyEye_L/R` を挿入します。元の目のボーンの角度が (0, 0, 0) から大きく離れている場合に有効です。
* ~~最適化された FX Layer を生成する~~: **現在この機能は開発中です。**

現状では両方のチェックボックスを有効にするのをおすすめします。

**v1.4.2 以降Eye Pointer のプレハブの配置場所はアバタールートの直下である必要はありません。** ビルド時に自動でアバタールートの直下に移動されます。


### Eye ボーンの判定方法

以下の順序で判定されます。

1. **Avatar Descriptor の Eye Look が有効な場合**、その設定の *Left Eye* と *Right Eye* を参照します。
2. **Avatar Descriptor の Eye Look が無効な場合**、アバタールートの Animator にセットされている Avatar アセット[^1]に記録されているボーンの対応付けのうち、 Left Eye と Right Eye の値を参照します。
3. いずれにもない場合、Enhanced EyePointer Installer は何もしません。

[^1]: 紛らわしいですが、Humanoid リグの FBX ファイルをインポートした後に Jaw を外したり Force T-pose したりするアレです。

### MA Merge Armature と併用する際の注意点

MA Merge Armature を利用して Unity 上で別アバターの頭部を合成するといった構成では、 **Eye Look の設定には身体側[^2]のボーンを設定する必要があります。**
頭部側の Eye ボーンを設定した場合、Merge Armature によってそのボーンが消滅し上記の参照先の検索に失敗してしまいます。

> Enhanced EyePointer Installer は処理中に Eye ボーンの Transform 値を操作しますが、
> MA の処理の一部でボーンの参照先を修正する[^ma-430]際に Transform 値が操作する前の値に戻ってしまいます[^ma-1036](Unity が悪い)。
> なお現在はボーンの参照先を修正する前の値に再設定する処理が追加されている[^ma-1062]ためこの問題は発生しません。

[^2]: 厳密には Merge Armature で統合先に指定されている、最終的に残る方の Armature に含まれる Eye ボーン

[^ma-430]: [キメラアバターでビルド後Validation時にエラーになることがある。](https://github.com/bdunderscore/modular-avatar/issues/430)
[^ma-1036]: [Avatar にバインドされる Humanoid ボーンの Transform に対する操作が Rebind humanoid avatar パスで破棄される](https://github.com/bdunderscore/modular-avatar/issues/1036)
[^ma-1062]: [Preserve local transform when rebinding humanoid avatar](https://github.com/bdunderscore/modular-avatar/pull/1062)


## 既知の問題

{{% expand title="v1.4.1 で修正済み" %}}
~~(v1.2.1) EyePointer v1.2 でダミーボーンを挿入するとフルトラキャリブレーション時に Constraint が意図しない挙動をすることが確認されています。また、他のプレイヤーからも同様に正面ではない(無効化されていない)状態で見えることがあります。~~

**上記の現象は Avatar Descriptor の Eye Look の設定に起因します。**
詳しくは[Issue #16 のコメント](https://github.com/kb10uy/kb10uy-zatools/issues/16#issuecomment-2336783558)を参照してください。
v1.4.1 以降はこの場合無意味な GameObject を追加し Eye Look をそれに対して有効化します。
{{% /expand %}}

