+++
title = 'Missing BlendShape Inserter'
date = 2024-09-26T21:23:11+09:00
images = ["window.png"]
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools`

## 概要

複数の AnimationClip のあいだで操作する BlendShape の対象を統一するためのツールです。

Write Defaults オフで AnimatorController を運用する場合、ステート遷移した際に次のステートに設定されている AnimationClip にないプロパティーは前のステートの値が残り続けます。
このツールは、そのような複数の AnimationClip に対して操作対象の BlendShape の和集合をとり、それら全てがアニメーションで設定されるようにアニメーションを編集します。

**デフォルトでアセットを破壊的に編集するので注意してください！(オプションで別ファイル保存に変更可)**

## 使い方

Window > kb10uy > Missing BlendShape Inserter を選択すると以下のようなウィンドウが開きます。

![Window](./window.png?width=800px "Missing BlendShape Inserter ウィンドウ")

上部エリアに BlendShape の操作対象を統一したい AnimationClip アセットをドラッグ&ドロップで追加します。いくつか追加すると以下のような表示になります。

![Window](./added.png?width=800px "AnimationClip をいくつか追加した状態")

* **チェックボックス①**: 各 AnimationClip に対して変更を適用するかどうか。
    - チェックを外した場合、BlendShape の和集合の対象にはなりますがアニメーション編集の対象にはなりません。
* **ボタン②**: 各 AnimationClip をリストから削除します。
* **リストビュー③**: 各 AnimationClip に対して実際にどの BlendShape がどんな値で追加されるかのプレビュー。
* **収集対象の GameObject のパス**: AnimationClip に設定されているプロパティーのうち、ここで指定したパスの SkinnedMeshRenderer に対する BlendShape を和集合の対象にする。
    - デフォルトでは多くのアバターの頭部メッシュの名前になっている `Body` が設定されています。
* **追加で設定する値**: 不足している BlendShape 操作について、どのような値を設定するか。
    - `All Zero`: 全て 0 にする。
    - `Copy from SkinnedMeshRenderer`: **BlendShape 値のコピー元** で指定した SkinnedMeshRenderer に現在設定されてる BlendShape の値をコピーする。デフォルト表情を変更して非ゼロの値が含まれている場合などに使用します。
* **元アセットを上書きする**: リストにある AnimationClip アセットに直接不足している BlendShape 操作を追加するかどうか。
    - チェックを外した場合、**保存先** に指定したパスに `XXXXXX_filled.anim` という名前でコピーが保存されます。
