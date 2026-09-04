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
ZaxProgram { Instructions, Constants, Variables, StackSize, ResultType }
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
* 期待結果型が与えられ、生成型が `int`、期待型が `float` の場合のみ末尾に `Convert` 命令を挿入する。
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
| `Convert` | (結果型) | v → v' | 結果型強制のための変換 |
| `Unpack` | (引数型) | v → c1…cn | ベクトルを成分へ展開 |
| `Dup` | — | a → a a | |
| `Drop` | — | a → | |
| `Swap` | — | a b → b a | |
| `Over` | — | a b → a b a | |
| `Rot` | — | a b c → b c a | |

`ZaxInstruction` は OpCode・関数 ID・結果型・引数型 (各 1 バイト) と 16bit オペランドからなる
固定長構造体で、`NativeArray` にそのまま載る。

`Swizzle` のオペランドは 16bit にパックされている。下位 8bit に 2bit x 4 の成分索引、
bit 8-10 に長さが入る。

## 評価器

`ZaxEvaluator.Execute` は命令列を上から 1 回走査するだけのスタックマシンである。
`Call` の処理だけが 2 経路に分かれる。

* **整数経路**: 命令の引数型が `int` のとき。`int` のまま加減乗除・剰余・`min` / `max` / `clamp` 等を行う。
  0 除算・0 剰余は例外ではなく **0 を返す**。
* **浮動小数経路**: それ以外。全引数を引数型へ変換 (スカラはブロードキャスト) してから `float4` に詰め、
  `Unity.Mathematics` の 4 成分演算を 1 回行い、結果を結果型の次元へ切り詰める。

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

`expectedResultType` に `null` を渡すと結果型を検査しない。

`ZaxProgram` の主なメンバー:

| メンバー | 意味 |
|---|---|
| `Instructions` | 命令列 |
| `Constants` | 定数プール (重複排除済み) |
| `Variables` | **実際に参照された**変数のみ、初出順 |
| `StackSize` | 必要スタック段数 |
| `ResultType` | 結果型 |
| `IndexOfVariable(name)` | 変数索引の逆引き |
| `Disassemble()` | 2.7 節の文字列 |

`Variables` は宣言した変数の部分集合である。束縛を作る側は「宣言順」ではなく
`program.Variables` の順に並べる必要がある。

### 単発評価

```csharp
var values = new[] { ZaxValue.FromFloat3(new float3(1, 2, 3)) };  // program.Variables と同順
var result = ZaxEvaluator.Evaluate(program, values);              // ZaxValue が返る
```

`ZaxValue` は `FromInt` / `FromFloat` / `FromFloat2..4` で構築し、`AsInt` / `AsFloat4` などで取り出す。
`ToString()` は不変カルチャの丸め往復可能表記を返す。

### ジョブ評価

```csharp
using var native = ZaxNativeProgram.Allocate(program, Allocator.TempJob);
var bindings = new NativeArray<ZaxVariableBinding>(program.Variables.Length, Allocator.TempJob);
bindings[0] = ZaxVariableBinding.FromArray(positions, ZaxValueType.Float3);   // 要素ごと
// bindings[1] = ZaxVariableBinding.Uniform(ZaxValue.FromFloat(0.5f));        // 全要素共通
var results = new NativeArray<ZaxValue>(count, Allocator.TempJob);
ZaxEvaluateJob.Schedule(native, bindings, results, innerloopBatchCount: 64).Complete();
```

制約 (`ZaxNativeProgram.Allocate` および `ZaxEvaluateJob.Create` が例外で弾く):

* `StackSize` は **64** 以下 (`MaxStackSize`)
* 参照変数は **16** 個以下 (`MaxVariableCount`)
* 束縛の個数が `program.VariableCount` と一致し、型が**厳密に**一致すること
* 要素束縛の `Length` が結果配列長以上であること

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
| `ExtraOperands` | 実行後に値が 2 個以上残る |
| `TypeMismatch` | シグネチャに引数型が適合しない |
| `SwizzleOnScalar` | スカラに `#…` を適用した |
| `SwizzleOutOfRange` | 対象の次元にない成分を指した |
| `UnpackOnScalar` | スカラに `...` を適用した |
| `ResultTypeMismatch` | 期待結果型に暗黙変換できない型を生成した |

UI は通常 `diagnostics[0]` のみを表示する。
