+++
title = 'Face Tracking Data Installer'
date = 2026-11-04T00:00:00+09:00
images = ["component-initial.png"]
weight = 304
+++

収録パッケージ: [kb10uy's Various Tools](https://github.com/kb10uy/kb10uy-zatools) / `org.kb10uy.zatools` (>= 6.3.0)

## 概要

VRCSDK 3.10.6 以降 VRCAvatarDescriptor で設定できるようになった Face Tracking のデータを JSON で記述し、ビルド時に設定できるコンポーネントです。

厳密にはこのコンポーネントによって実現される機能は VRCAvatarDescriptor 標準の機能でも実現可能ですが、
[はっきり](https://mstdn.maud.io/@kb10uy/117548600054890022)[言って](https://mstdn.maud.io/@kb10uy/117548601359839263)[話に](https://mstdn.maud.io/@kb10uy/117548611930313452)[ならない](https://mstdn.maud.io/@kb10uy/117548694964592167)
のでより厳格かつ軽量に設定できるコンポーネントを実装しています。

## 使い方

アバター内の適当な GameObject に `Zatools Face Tracking Data Installer` を追加します。
なお、アバター内にこのコンポーネントを複数追加することが可能です。

![Component View](./component-initial.png?width=480px "コンポーネント追加直後の状態")

- **Face Tracking メッシュ**: Face Tracking で動作させたい SkinnedMeshRenderer のリスト。VRCAvatarDescriptor と同様に複数設定可能です。
- **定義 JSON**: 所定の形式(後述)で記述された JSON ファイルの TextAsset。

コンパイル結果の概略がコンポーネント下部に表示されます。

## 定義 JSON ファイルの形式

[定義 JSON のデータ形式を宣言している JSON Schema](/schemas/face-tracking-data-installer.schema.json) をこのサイトで公開しているのでこれを参照していただくのが最も確実です[^1]。
以下のサンプルデータも参考にしてください。

```json
{
  "$schema": "https://zatools.kb10uy.dev/schemas/face-tracking-data-installer.schema.json",
  "Version": 1,
  "UseAllTrackingKeys": true,
  "VisemeDampen": { "Mode": "DampenTracking", "Amount": 0.5 },
  "BlendShapeSets": [
    {
      "Name": "Eyes",
      "DefaultWeight": 1.0,
      "TrackingKeys": {
        "EyeClosedLeft": "blink_L",
        "EyeClosedRight": "blink_R",
        "EyeWideLeft": { "Shape": "surprised_L", "Multiplier": 0.5 },
        "EyeWideRight": { "Shape": "surprised_R", "Multiplier": 0.5 }
      }
    },
    {
      "Name": "Mouth",
      "DefaultWeight": 1.0,
      "TrackingKeys": {
        "JawOpen": "mouth_open",
        "TongueOut": "tongue_out"
      },
      "Visemes": {
        "aa": "vrc.v_aa",
        "oh": "vrc.v_oh"
      }
    }
  ],
  "ParameterBehaviors": [
    {
      "Operation": "ApplyBlendSetWeight",
      "Target": "Mouth",
      "Expression": {
        "Type": "Max",
        "TrackingKeys": [{ "Key": "Voice", "Invert": true }]
      }
    },
    {
      "Operation": "ForwardToAnimator",
      "Target": "FT/EyesClosed",
      "Expression": {
        "Type": "Min",
        "TrackingKeys": ["EyeClosedLeft", "EyeClosedRight"]
      }
    },
    {
      "Operation": "ApplyToBlendshape",
      "Target": "blush",
      "Weight": "Eyes",
      "Expression": {
        "Type": "Custom",
        "Expression": "clamp((CheekSquintLeft + CheekSquintRight) * 0.5, 0, 1)"
      }
    },
    {
      "Operation": "LimitBlendshapeMax",
      "Target": "surprised_L",
      "Expression": {
        "Type": "Multiply",
        "TrackingKeys": [{ "Key": "EyeClosedLeft", "Invert": true }]
      }
    },
    {
      "Operation": "LimitBlendshapeMin",
      "Target": "mouth_open",
      "Expression": {
        "Type": "Add",
        "TrackingKeys": ["Viseme_aa", "Viseme_oh"]
      }
    }
  ]
}
```

[^1]: LLM に投げたらやってくれるでしょう 多分
