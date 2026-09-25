+++
title = "行列・クォータニオン"
date = 2026-09-25T00:00:00+09:00
weight = 450
+++

ZAX に行列型はない。行列は**スタック上に連続して並んだ N 個の `floatN`** を N x N 行列として読む
「ビュー」であり、クォータニオンは Unity と同じ `(x, y, z, w)` 配置の `float4` である。
どちらも新しい型を導入せず、`M` / `Q` を接頭辞に持つ関数群が既存の値を行列・クォータニオンとして解釈する。

## 行列の表現

### スタック上の配置

N x N 行列は N 個の `floatN` 値からなり、**第 0 行を先に push する**。
つまりスタックの深い側が第 0 行、先頭が第 N-1 行である。
行の型は N と一致していなければならず (`float3` 3 個で 3x3、`float4` 4 個で 4x4)、
`int` を含む行や次元の揃っていない並びは行列として認識されない。

```
1 0 0 vec3
0 1 0 vec3
0 0 1 vec3        ; float3 x 3 = 3x3 の単位行列 (Midentity3 と同じ)
```

診断メッセージでは行列の形は `float3 x 3` のように表示される。

### 列ベクトル規約

行列はベクトルの**左から**掛ける (column-vector convention)。

* `M v Mmulv` は M·v
* `A B M*` は A·B (A を先に push)
* 変換の合成は数式と同じ順で書ける。`T R M*` は T·R であり、「R を先に適用し、次に T」

4x4 のアフィン変換は平行移動を第 4 列 (各行の w 成分) に持つ。`Mtranslate` はこの配置で生成し、
`Mtfpoint` はこの配置を前提に点を変換する。

### 通常の演算との関係

行列はビューなので、行に対しては通常の演算がそのまま効く。

* `dup` `swap` などのスタック操作語は **1 行単位**で動く。行列単位で扱うには `Mdup` `Mdrop` `Mswap` を使う。
* swizzle `#…` はスタック先頭の行 (第 N-1 行) だけを対象にする。
* `*` は成分ごとの積であり行列積ではない。行列積は `M*` (`Mmul`)。
* 行列を式の結果として残すと[複数値結果](#複数値結果)になる。

## 行列関数

引数列は RPN での push 順 (「A B」は A を先に push)。M は N x N 行列 (N = 2, 3, 4)。
名前は大文字小文字を区別するので `mmul` は `UnknownName` になる。

### 演算

| 名前 | 記号 | 引数 | 結果 | 意味 |
|---|---|---|---|---|
| `Mmul` | `M*` | A B | N x N | 行列積 A·B |
| `Madd` | `M+` | A B | N x N | 成分ごとの和 |
| `Msub` | `M-` | A B | N x N | 成分ごとの差 A - B |
| `Mmulv` | | M v | `floatN` | M·v。v は行と同じ `floatN` |
| `Mscale` | | M s | N x N | 全成分を s 倍。s はスカラ |
| `Mtfpoint` | | M p | `float3` | 4x4 の M で点 p (`float3`) を変換。(p, 1) を掛けて xyz を取る |
| `Mtfdir` | | M d | `float3` | 4x4 の M で方向 d (`float3`) を変換。(d, 0) を掛けて xyz を取る。平行移動を受けない |
| `Mtranspose` | | M | N x N | 転置 |
| `Minverse` | | M | N x N | 逆行列。特異行列は NaN / Inf になる (エラーにはならない) |
| `Mdet` | | M | `float` | 行列式 |
| `Mtrace` | | M | `float` | 跡 (対角成分の和) |
| `Mdiagv` | | M | `floatN` | 対角成分をベクトルとして取り出す |
| `Mouter` | | a b | N x N | 外積 a bᵀ (M_ij = a_i b_j)。a b は同じ `floatN` |

`Mmul` `Madd` `Msub` は両辺の次元が一致していなければならない。
`Mtfpoint` は射影除算をしない (第 4 行は結果に影響しない)。
`Mmulv` に 4x4 と `float3` の組は渡せない。3 次元の点・方向を 4x4 で変換するには `Mtfpoint` / `Mtfdir` を使う。

### 構築

| 名前 | 引数 | 結果 | 意味 |
|---|---|---|---|
| `Midentity2` `Midentity3` `Midentity4` | | N x N | 単位行列 |
| `Mdiag` | v | N x N | v (`floatN`) を対角に持つ行列 |
| `Mtranslate` | t | 4 x 4 | 平行移動 t (`float3`) |
| `Mrotx` `Mroty` `Mrotz` | θ | 3 x 3 | 各軸まわりの回転。θ はスカラ、弧度法 |
| `Mrotaxis` | axis θ | 3 x 3 | 任意軸回転。axis は正規化済みの `float3`、θ は弧度法 |
| `Mroteuler` | e | 3 x 3 | オイラー角 e (`float3`、弧度法)。回転順序は Unity 既定の ZXY |
| `Mlookrot` | forward up | 3 x 3 | forward を +Z に向ける回転 (Unity の LookRotation)。両引数とも正規化済みで平行でないことを仮定 |
| `Mfromq` | q | 3 x 3 | クォータニオン q (`float4`) から回転行列 |

回転を生成する関数はすべて **3x3** を返す。`Mtranslate` は 4x4 を返すので、
平行移動と合成するには回転側を `Mto4` で 4x4 に拡張する。

### 次元変更・スタック操作

| 名前 | 引数 | 結果 | 意味 |
|---|---|---|---|
| `Mto2` `Mto3` `Mto4` | M | K x K | N x N を K x K へ。縮小は左上を切り出し、拡大は単位行列で埋める |
| `Mdup` | M | M M | 行列を複製 |
| `Mdrop` | M | | 行列を破棄 |
| `Mswap` | A B | B A | 2 つの行列を入れ替える。A と B の次元は異なってよい |

3x3 の回転を `Mto4` すると同次座標の回転行列 (第 4 行・第 4 列が (0, 0, 0, 1)) になる。
4x4 のアフィン変換を `Mto3` すると平行移動を捨てて回転・スケール部分だけが残る。

### 形状の推論とエラー

行列関数はアリティが固定ではなく、**スタック先頭の型から次元を推論する**。
先頭が `floatN` なら N x N 行列 (N 個の `floatN`) を期待し、`Mmulv` のようにベクトルを取るものは
そのベクトルの次元から行列の次元を決める。

* 期待する並びに対して値が足りなければ `NotEnoughOperands`
* 値はあるが型の並びが行列になっていなければ `MatrixShapeMismatch` (期待形状と実際の型列を報告する)

## クォータニオン

クォータニオンは `float4` で、Unity と同じ `(x, y, z, w)` 配置 (w が実部) である。
専用の型はないので `vec4` や swizzle で自由に読み書きでき、`normalize` で正規化できる。
恒等クォータニオンは定数 `Q_IDENTITY` = (0, 0, 0, 1)。

| 名前 | 引数 | 結果 | 意味 |
|---|---|---|---|
| `Qmul` | a b | `float4` | クォータニオン積 a·b。合成した回転は b を先に、a を後に適用する |
| `Qrotate` | q v | `float3` | v を q で回転 |
| `Qinverse` | q | `float4` | 逆元。非正規化でも正しい |
| `Qconjugate` | q | `float4` | 共役 (x, y, z の符号反転) |
| `Qaxisangle` | axis θ | `float4` | 正規化済みの axis まわりに θ (弧度法) 回転 |
| `Qeuler` | e | `float4` | オイラー角 e (`float3`、弧度法)。回転順序は ZXY |
| `Qlook` | forward up | `float4` | LookRotation。両引数とも正規化済みで平行でないことを仮定 |
| `Qslerp` | a b t | `float4` | 球面線形補間。最短経路を取る |
| `Qnlerp` | a b t | `float4` | 正規化線形補間。最短経路を取る |
| `Qfromm` | M | `float4` | 3x3 の回転行列 M からクォータニオン。M は正規直交を仮定 |

引数型は表の通りに固定されており、型昇格は行われない (`float3` の位置に `float4` は渡せない)。
ただしスカラ引数 (θ, t) には `int` も渡せる。

`Qmul` の順序は Unity の `q1 * q2` と同じで、`a b Qmul` で v を回転すると a(b(v)) になる。

## 複数値結果

行列を式の**結果**として返すには、ホストが結果として N 個の `floatN` を期待している必要がある
([結果型の強制](./syntax.md#結果型の強制))。現状の Zatools のコンポーネントはすべて単一の値を期待するので、
行列は式の途中で使い、最後は `Mmulv` `Mtfpoint` `Qfromm` などでベクトル・スカラに戻す。

ホスト側から行列を渡す手段もない (変数は単一の値)。4x4 を渡したい場合は `float4` の変数を 4 個宣言し、
式の側で `@m0 @m1 @m2 @m3` と行の順に並べる。

## 例

```
; 位置を Y 軸まわりに 90 度回転
HALF_PI Mroty @position Mmulv

; 回転してから平行移動 (T·R)
0 1 0 vec3 Mtranslate  PI Mrotz Mto4  M*  @position Mtfpoint

; ホストが行ごとに宣言した 4x4 で法線を変換
@m0 @m1 @m2 @m3 @normal Mtfdir normalize

; @rotation を適用した後に Y 軸 90 度回転を重ねる
Y_AXIS HALF_PI Qaxisangle @rotation Qmul @position Qrotate

; 2 つの回転の中間で回転
@qa @qb 0.5 Qslerp @position Qrotate

; 度で与えられたオイラー角を回転行列経由でクォータニオンへ
@euler radians Mroteuler Qfromm
```

## 落とし穴

* **`*` と `M*` は別物**: `*` は行 (ベクトル) 同士の成分ごとの積。行列積は `M*`。
* **第 0 行を最初に push する**: リテラルで行列を書くとき、上の行から順に書けばよい。
* **回転は 3x3、平行移動は 4x4**: 合成する前に回転側を `Mto4` する。
* **角度は弧度法**: 度で持っているなら `radians` を通す。
* **軸・forward は正規化済みを仮定**: `Mrotaxis` `Qaxisangle` `Mlookrot` `Qlook` は入力を正規化しない。
* **`Minverse` は特異行列を検出しない**: NaN / Inf が出るだけである。
* **スタック操作語は 1 行単位**: 行列を複製・破棄・入れ替えるなら `Mdup` `Mdrop` `Mswap`。
* **名前は大文字小文字を区別する**: `Mmul` `Qmul` の先頭は大文字。
