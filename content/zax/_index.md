+++
title = "ZAX (Zatools Arithmetic Expression)"
date = 2026-09-04T23:00:00+09:00
weight = 25
+++

**ZAX (Zatools Arithmetic Expression)** は [RPN (逆ポーランド記法)](https://ja.wikipedia.org/wiki/%E9%80%86%E3%83%9D%E3%83%BC%E3%83%A9%E3%83%B3%E3%83%89%E8%A8%98%E6%B3%95) によるスタックベースの数式・ベクトル処理エンジンです。

~~思いついちゃったから作っちゃいました。~~

## 概要

ZAX は、Zatools 内の一部の数値処理をより細かくカスタマイズするために導入されています。
基本的には上級者向けの機能であり、多くのユースケースは既定のプリセットでカバーされるはずです。
またパフォーマンス上の理由から、既定のプリセットの自体を ZAX 上に構築することはありません。

## コード例

### 基本

ZAX Code:
```
; @uv0: float4
@uv0 #xy 2.0 * 0.1 0.2 vec2 + .. 0.0 0.0 vec4
```

HLSL Equivalent:
```hlsl
// float4 uv0;
return vec4(uv0.xy * 2.0 + float2(0.1, 0.2), 0.0, 0.0);
```

### 応用

ZAX Code:
```
0 0 1 vec3
2 / 0.5 +
...

@color #xyz
0.299 0.587 0.114 vec3
dot

vec4
```

HLSL Equivalent:
```hlsl
// float4 color;

float3 block1 = float3(0.0, 0.0, 1.0) / 2 + 0.5;
float block2 = dot(color.xyz, float3(0.299, 0.587, 0.114));

return float4(block1.xyz, block2);
```
