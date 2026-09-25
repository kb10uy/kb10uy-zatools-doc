+++
title = "処理系実装"
date = 2026-09-04T23:00:00+09:00
weight = 200
+++

## パイプライン

```
source (string)
  |  ZaxTokenizer.TryTokenize        字句解析 (空白分割 + 分類)
  v
List<ZaxToken>
  |  ZaxCompiler.TryCompile          型スタックによる抽象解釈 + コード生成
  v
ZaxProgram { Instructions, Constants, Variables, StackSize, ResultTypes }
  |
  +- ZaxEvaluator.Evaluate           マネージド 1 回評価 (エディタ UI 用)
  +- ZaxNativeProgram + ZaxEvaluateJob   Burst / IJobParallelFor で N 要素一括評価
```

## 字句解析器

`ZaxTokenizer` はスキャナではない。空白と `;` でソースを切り刻み、断片ごとに種別を判定するだけの
実装であり、トークンの内部構造 (`+` と `12` の連結など) は一切見ない。

不正な断片を見つけても診断を積んで走査を続けるため、字句エラーは複数まとめて報告されうる。

## コンパイラ

`ZaxCompiler.TryCompile` は 1 パスである。トークンを先頭から走査しながら、

* **型スタック** (`List<ZaxValueType>`) を実行時スタックの抽象解釈として更新し、
* 同時に命令を出力する。

主な性質は次の通り。

* 型スタックの最大長がそのまま `StackSize` になる。
* 定数は値 (型込み) で重複排除され、同じ値は同じ索引を共有する。
* 変数は**参照されたものだけ**が `program.Variables` に初出順で記録される。
* 期待結果型はリストで与えられ、残った値の個数がその長さと一致しなければならない。各スロットについて
  生成型が `int`、期待型が `float` の場合のみ、そのスロットを指す `Convert` 命令を末尾に挿入する。
  リストが `null` なら個数も型も検査せず、残った値の型列がそのまま `ResultTypes` になる。
* 関数表 (`ZaxFunctionInfo`) は種別 (`Kind`) を持つ。通常の関数はアリティ固定で `Call` 命令になり、
  固定シグネチャの関数 (クォータニオン系) は `Quaternion` 命令になる。行列関数は `ZaxMatrixFunctions.Infer`
  が型スタック全体を見て消費数・生成数・次元を決め、`Matrix` 命令になる。
* 意味エラーは 1 個目で走査を打ち切る (字句エラーと違って積み上がらない)。

## 値表現

`ZaxValue` は `int4` のビット列 + 型タグ (`ZaxValueType`) の構造体である。

* スカラ・ベクトルを問わず常に 4 レーン分の領域を持ち、未使用レーンは 0。
* `int` はビットパターンではなく数値として `float` へ変換される (`Broadcast` / `ToFloat4`)。
* スカラをベクトルへ変換すると**全成分へブロードキャスト**され、ベクトルを短い型へ変換すると
  先頭から切り詰められる。

## 命令セット

| OpCode | オペランド | スタック効果 | 意味 |
|---|---|---|---|
| `Constant` | 定数索引 | → v | 定数プールから push |
| `Variable` | 変数索引 | → v | 変数フレームから push |
| `Call` | アリティ (+ 関数 ID / 引数型 / 結果型) | a1…an → r | 組み込み関数適用 |
| `Swizzle` | パック済み成分列 | v → v' | 成分の並べ替え・抽出 |
| `Quaternion` | アリティ (+ 関数 ID / 結果型) | a1…an → r | クォータニオン関数適用 |
| `Matrix` | N・K (+ 関数 ID / 行型 / 結果行型) | 関数ごと | 行列関数適用。N は入力行列の次元、K は `Mto*` / `Mswap` の第 2 次元 |
| `Convert` | スロット索引 (+ 結果型) | s[i] → s[i]' | 結果型強制のための変換。先頭ではなく指定スロットを書き換える |
| `Unpack` | (引数型) | v → c1…cn | ベクトルを成分へ展開 |
| `Dup` | — | a → a a | |
| `Drop` | — | a → | |
| `Swap` | — | a b → b a | |
| `Over` | — | a b → a b a | |
| `Rot` | — | a b c → b c a | |

`ZaxInstruction` は OpCode・関数 ID・結果型・引数型 (各 1 バイト) と 16bit オペランドからなる
固定長構造体で、`NativeArray` にそのまま載る。

`Swizzle` のオペランドは 16bit にパックされている。下位 8bit に 2bit x 4 の成分索引、
bit 8-10 に長さが入る。`Matrix` のオペランドは下位 8bit に N、上位 8bit に K を持つ。

## 評価器

`ZaxEvaluator.Execute` は命令列を上から 1 回走査するだけのスタックマシンである。
結果は返さず、実行後のスタック先頭 `ResultCount` 個がそのまま結果になる。
`Call` の処理は 2 経路に分かれ、`Matrix` / `Quaternion` はインライン化されない別関数へ抜ける。

* **整数経路**: 命令の引数型が `int` のとき。`int` のまま加減乗除・剰余・`min` / `max` / `clamp` 等を行う。
  0 除算・0 剰余は例外ではなく **0 を返す**。
* **浮動小数経路**: それ以外。全引数を引数型へ変換 (スカラはブロードキャスト) してから `float4` に詰め、
  `Unity.Mathematics` の 4 成分演算を 1 回行い、結果を結果型の次元へ切り詰める。

* **行列経路** (`Matrix`): N 個の行を `float4x4` に読み込み (足りない行・列は単位行列で埋める)、
  `Unity.Mathematics` の行列演算を 1 回行って行に書き戻す。行を列として読み込むので内部では転置された状態で
  計算しているが、各関数がその差を吸収しており、利用者から見える意味論は列ベクトル規約である。
* **クォータニオン経路** (`Quaternion`): 各引数を `float4` に変換し、`Unity.Mathematics.quaternion` の演算を 1 回行う。

したがってベクトル演算は次元によらず常に 4 レーンぶん計算されている。未使用レーンは 0 が入るため、
`length` や `dot` の結果には影響しない。

## 逆アセンブル

`ZaxProgram.Disassemble()` は次の形式の文字列を返す。

式:

```
@normal 0.5 * 0.5 + ... @color #rgb rgb2yuv #x vec4
```

出力:

```
; stack = 4, result = float4
0000  Variable 0 -> float3
0001  Constant 0 -> float
0002  Call Mul/2 (float3) -> float3
0003  Constant 0 -> float
0004  Call Add/2 (float3) -> float3
0005  Unpack float3 -> 3 x float
0006  Variable 1 -> float4
0007  Swizzle 0324 -> float3
0008  Call RgbToYuv/1 (float3) -> float3
0009  Swizzle 0100 -> float
0010  Call Vec4/4 (float) -> float4
```

`0.5` が 2 回現れても定数索引は 0 で共有されている点、`Call` の `/2` がアリティである点、
`Swizzle` のオペランドが 2.5 節のパック形式で表示される点に注意。

行列・クォータニオン・変換命令は次の形式で表示される。

```
0003  Matrix Mmul N=3 K=3 (float3) -> float3
0004  Quaternion Qrotate/2 -> float3
0005  Convert [0] -> float
```

複数値結果のプログラムでは先頭行が `result = float3 x 3` のように表示される。

## ホスト API

名前空間は `KusakaFactory.Zatools.Foundation.Arithmetic`。いずれも `[PublicAPI]` ではないため、
マイナー・パッチリリースで変更されうる (README 参照)。

### コンパイル

```csharp
var variables = new[] { new ZaxVariable("pos", ZaxValueType.Float3) };
var diagnostics = new List<ZaxDiagnostic>();
if (!ZaxCompiler.TryCompile(source, variables, ZaxValueType.Float4, diagnostics, out var program))
{
    // diagnostics[0] を表示
}
```

`expectedResultType` に `null` を渡すと結果型を検査しない。複数の結果を受け取るには
`IReadOnlyList<ZaxValueType>` を取るオーバーロードに期待型の列を渡す。

`ZaxProgram` の主なメンバー:

| メンバー | 意味 |
|---|---|
| `Instructions` | 命令列 |
| `Constants` | 定数プール (重複排除済み) |
| `Variables` | **実際に参照された**変数のみ、初出順 |
| `StackSize` | 必要スタック段数 |
| `ResultTypes` | 結果型の列 (結果の個数ぶん) |
| `ResultCount` | 結果の個数 |
| `ResultType` | 結果が 1 個のときの結果型。2 個以上なら `InvalidOperationException` |
| `ResultDisplayName` | 結果型の表示名 (`float3 x 3` など) |
| `IndexOfVariable(name)` | 変数索引の逆引き |
| `Disassemble()` | 2.7 節の文字列 |

`Variables` は宣言した変数の部分集合である。束縛を作る側は「宣言順」ではなく
`program.Variables` の順に並べる必要がある。

### 単発評価

```csharp
var values = new[] { ZaxValue.FromFloat3(new float3(1, 2, 3)) };  // program.Variables と同順
var result = ZaxEvaluator.Evaluate(program, values);              // ZaxValue が返る
```

結果が 2 個以上のプログラムに対してこのオーバーロードを呼ぶと `ArgumentException` になる。
その場合は結果を受け取る `Span<ZaxValue>` を渡す。

```csharp
var results = new ZaxValue[program.ResultCount];
ZaxEvaluator.Evaluate(program, values, results);                  // results[i] が i 番目の結果
```

`ZaxValue` は `FromInt` / `FromFloat` / `FromFloat2..4` で構築し、`AsInt` / `AsFloat4` などで取り出す。
`ToString()` は不変カルチャの丸め往復可能表記を返す。

### ジョブ評価

```csharp
using var native = ZaxNativeProgram.Allocate(program, Allocator.TempJob);
var bindings = new NativeArray<ZaxVariableBinding>(program.Variables.Length, Allocator.TempJob);
bindings[0] = ZaxVariableBinding.FromArray(positions, ZaxValueType.Float3);   // 要素ごと
// bindings[1] = ZaxVariableBinding.Uniform(ZaxValue.FromFloat(0.5f));        // 全要素共通
var results = new NativeArray<ZaxValue>(count * program.ResultCount, Allocator.TempJob);
ZaxEvaluateJob.Schedule(native, bindings, results, innerloopBatchCount: 64).Complete();
```

制約 (`ZaxNativeProgram.Allocate` および `ZaxEvaluateJob.Create` が例外で弾く):

* `StackSize` は **64** 以下 (`MaxStackSize`)
* 参照変数は **16** 個以下 (`MaxVariableCount`)
* 束縛の個数が `program.VariableCount` と一致し、型が**厳密に**一致すること
* 結果配列長が `program.ResultCount` の倍数であること。要素数は結果配列長 / `ResultCount` で、
  要素 i の k 番目の結果は `results[i * ResultCount + k]` に書かれる
* 要素束縛の `Length` が要素数以上であること

`ZaxVariableBinding.FromArray<T>` は要素サイズ (stride) が対象型のバイト数以上であればよいので、
`NativeArray<float4>` を `Float3` として束縛し先頭 12 バイトだけ読む、といった使い方ができる。
`Data == null` の束縛 (`Uniform`) は全要素で同じ値を返す。

参照変数が 0 個のプログラムはジョブを組む必要がなく、1 回評価して全要素へ複製すればよい。

## 診断

コンパイル失敗時は `List<ZaxDiagnostic>` に診断が積まれる。各診断はコード・ソース内オフセット・
長さ・置換引数を持ち、`LocalizationKey` (`zax.diagnostic.*`) 経由で日本語 / 英語に翻訳される。

| コード | 発生条件 |
|---|---|
| `InvalidToken` | どの分類にも当てはまらない文字列、または `@` の後が識別子でない |
| `InvalidNumberLiteral` | 数値の形をしているが解析できない |
| `InvalidSwizzle` | `#` の後が 1 から 4 文字の単一セットでない |
| `EmptyExpression` | トークンが 0 個、または値を残さない |
| `UnknownName` | 未知の関数・定数・記号 |
| `UnknownVariable` | ホストが宣言していない `@name` |
| `NotEnoughOperands` | スタック不足 (必要数と実際の数を報告) |
| `ResultCountMismatch` | 実行後に残った値の個数が期待結果型の個数と一致しない (期待個数と実際の個数を報告) |
| `TypeMismatch` | シグネチャに引数型が適合しない |
| `SwizzleOnScalar` | スカラに `#…` を適用した |
| `SwizzleOutOfRange` | 対象の次元にない成分を指した |
| `UnpackOnScalar` | スカラに `...` を適用した |
| `ResultTypeMismatch` | 期待結果型に暗黙変換できない型を生成した |
| `MatrixShapeMismatch` | 行列関数が期待する型の並び (`float3 x 3, float3` など) にスタック先頭が適合しない |

UI は通常 `diagnostics[0]` のみを表示する。
