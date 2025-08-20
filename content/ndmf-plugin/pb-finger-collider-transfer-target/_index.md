+++
title = 'PB Finger Collider Transfer Target'
date = 2025-05-18T03:23:04+09:00
images = ["component-initial.png"]
weight = 103
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 2.1.0-rc.1)

## 概要

VRC Avatar Descriptor で設定される [Standard Colliders](https://creators.vrchat.com/avatars/avatar-dynamics/physbones/#standard-colliders) のうち、
指に割り当てられるものの一部を任意の Transform に割り当てるように変更します。
これにより、「見かけが指ではないオブジェクトで他人のアバターの PB に干渉する」といったことが可能になります。

[VRCFury Global Collider](https://vrcfury.com/components/global-collider/) や [d4rkAvatarOptimizer: Use Ring Finger as Foot Collider](https://github.com/d4rkc0d3r/d4rkAvatarOptimizer) と似たような機能を NDMF の範囲で実現することができます。
また NDMF のアバタービルド処理のタイミングで実行されるため、手動での作業時に必要な「Inspector を Debug ビューにしたままアバターをアップロードする」という手順が不要になります。

## 使い方

適当な GameObject を作成し、`Zatools PB Finger Collider Transfer Target` を追加します。
この GameObject を以後 Target と表記します。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

同時に Target にカプセル状のギズモが表示されます。このギズモがビルド後にコライダーに設定される範囲となります。
なお、中心座標・回転の指定は実装されていません。中心にしたい位置に新規 GameObject を適宜作成してください。

* **半径**: Target を中心としたカプセルの半径。
* **長さ**: Target を中心としたカプセルの長さ。球面部分の頂点から頂点までの長さであることに注意してください。

指以外の Humanoid ボーンに追従させる場合は [MA Bone Proxy](https://modular-avatar.nadena.dev/ja/docs/reference/bone-proxy) と併用するのが便利です。

![Example](./setting-example.png?width=640px "設定例")

## その他

**AAO と併用する場合、そちらのバージョンは 1.8.11 以降を使用してください。**

アバター内で 6 個を超えた分については、警告メッセージが表示され**処理されず単に無視されます。**
人差し指はこのコンポーネントによる*剥ぎ取り*の対象外であり、Disabled に設定されていてもこれを使用することはありません(この仕様は将来変更される可能性があります)。

*剥ぎ取り*の順序は次の通りです。VRCFury Global Collider とは戦略が異なるので注意してください。

1. 左手小指
2. 右手小指
3. 左手薬指
4. 右手薬指
5. 左手中指
6. 右手中指

各 Target ごとに、その子孫として GameObject が追加で 2 個生成されます(カプセルの中心、カプセルの下端)。
これは、VRChat の Finger Collider に対する特殊な挙動[^1]を回避するためのものです。

[^1]: [【VRChat】PB干渉リクちゃん（PhysBone干渉コライダー移動）の作り方｜LABO405](https://note.com/labo405/n/nac5615af9b0e)
