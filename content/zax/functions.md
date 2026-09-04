+++
title = "関数・定数"
date = 2026-09-04T23:00:00+09:00
weight = 500
+++

「引数」列は RPN での並び順 (`a b f` なら a が第 1 引数)。「クラス」は 3.9 節を参照。
記号を持つ演算子 (`+` `<` など) は[演算子](./operators.md)にまとめてある。

## 算術・丸め

| 名前 | 引数 | クラス | 意味 |
|---|---|---|---|
| `neg` | a | Ei | -a |
| `abs` | a | Ei | 絶対値 |
| `sign` | a | Ei | -1 / 0 / +1 |
| `min` | a b | Ei | 成分ごとの最小 |
| `max` | a b | Ei | 成分ごとの最大 |
| `clamp` | x lo hi | Ei | x を [lo, hi] に丸める |
| `saturate` | x | Ef | clamp(x, 0, 1) |
| `floor` `ceil` `round` | x | Ef | 床・天井・最近接丸め |
| `frac` | x | Ef | x - floor(x) |

## 指数・対数

| 名前 | 引数 | クラス | 意味 |
|---|---|---|---|
| `sqrt` | x | Ef | 平方根 |
| `rsqrt` | x | Ef | 1 / sqrt(x) |
| `pow` | a b | Ef | a の b 乗 |
| `exp` `exp2` | x | Ef | e^x / 2^x |
| `log` `log2` `log10` | x | Ef | 自然対数 / 底 2 / 底 10 |

## 三角関数

| 名前 | 引数 | クラス | 意味 |
|---|---|---|---|
| `sin` `cos` `tan` | x | Ef | 弧度法 |
| `asin` `acos` `atan` | x | Ef | 逆関数 |
| `atan2` | y x | Ef | atan2(y, x)。**第 1 引数が y** |
| `degrees` | x | Ef | 弧度から度 |
| `radians` | x | Ef | 度から弧度 |

## 補間

| 名前 | 引数 | クラス | 意味 |
|---|---|---|---|
| `lerp` | a b t | Ef | a + t (b - a)。t のクランプはしない |
| `step` | edge x | Ef | x >= edge なら 1、そうでなければ 0 |
| `smoothstep` | edge0 edge1 x | Ef | t = saturate((x - edge0) / (edge1 - edge0)) として 3t^2 - 2t^3 |

引数順を間違えやすい。`lerp` は始点・終点・係数、`step` は**閾値が先**、
`smoothstep` は下端・上端・入力である。

## ベクトル

| 名前 | 引数 | クラス | 意味 |
|---|---|---|---|
| `dot` | a b | Rs | 内積 (結果は float) |
| `cross` | a b | F3 | 外積 (float3 のみ) |
| `length` | v | Rs | ノルム |
| `lengthsq` | v | Rs | ノルムの二乗 |
| `distance` | a b | Rs | a と b の距離 |
| `normalize` | v | Ef | 正規化 (零ベクトルは NaN) |
| `reflect` | i n | Ef | i - 2n (i・n)。n は正規化済みを仮定 |
| `vec2` | x y | Cn | float2 構築 |
| `vec3` | x y z | Cn | float3 構築 |
| `vec4` | x y z w | Cn | float4 構築 |

`vec*` の引数はすべてスカラでなければならない (ベクトルの連結はできない)。
ベクトルから組み直す場合は `...` で展開する。
Rs クラスの関数はスカラにも適用でき、`3.0 length` は 3.0 を返す (意味は薄いがエラーにはならない)。

## 色空間

| 名前 | 引数 | クラス | 意味 |
|---|---|---|---|
| `rgb2yuv` | rgb | F3 | BT.709 系の YUV へ変換 (Y = 0.2126R + 0.7152G + 0.0722B) |
| `yuv2rgb` | yuv | F3 | 上記の逆変換 |
| `srgb2linear` | c | Ef | sRGB 伝達関数の逆 (成分ごと) |
| `linear2srgb` | c | Ef | sRGB 伝達関数 (成分ごと) |

`srgb2linear` / `linear2srgb` は Ef なので `float4` に適用すると**アルファ成分も変換される**。
アルファを保ちたい場合は `#rgb` で切り出してから変換し、`... @color #a vec4` で組み直す。

## 定数

| 名前 | 型 | 値 |
|---|---|---|
| `PI` | `float` | 3.14159265… |
| `HALF_PI` | `float` | PI / 2 |
| `TAU` | `float` | 2 PI |
| `E` | `float` | 2.71828… |
| `EPSILON` | `float` | 1.1920929e-7 (float の機械イプシロン) |
| `INF` | `float` | 正の無限大 |
| `TRUE` | `float` | 1.0 |
| `FALSE` | `float` | 0.0 |
| `X_AXIS` | `float3` | (1, 0, 0) |
| `Y_AXIS` | `float3` | (0, 1, 0) |
| `Z_AXIS` | `float3` | (0, 0, 1) |

負の無限大は `INF neg` と書く。NaN を直接書く手段はない。
