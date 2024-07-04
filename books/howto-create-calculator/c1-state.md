---
title: "作成①：状態関係の定義"
---

## 本項について

これまでの設計内容に基づき、ここからは JavaScript を使って電卓機能を実装していきます。

## 状態を管理できる形へ

まずは、状態を管理するための準備を整えていきます。

### 定数として状態を定義

下表は、状態名と、そこに 0 から 10 の数値を割り振ったものです。

|State名|数値|説明|
|:---|:---:|:---|
|Start|0|初期状態|
|NegativeNum1|1|左辺：マイナス符号を入力|
|OperandZero1|2|左辺：ゼロ入力状態|
|OperandInteger1|3|左辺：整数モード|
|OperandDecimal1|4|左辺：小数モード|
|Operator|5|演算子入力|
|NegativeNum2|6|右辺：マイナス符号を入力|
|OperandZero2|7|右辺：ゼロ入力状態|
|OperandInteger2|8|右辺：整数モード|
|OperandDecimal2|9|右辺：小数モード|
|Result|10|計算結果|

この表に従い、コード内で定数を定義します。内部処理としては数値で条件分岐などを管理していきます。その一方で、名前から数値を引き出せるようにもしています。人にとって数値と状態を紐づけできるようするためです。

コード化する際には、JavaScript には `enum`（列挙型）は無いため、オブジェクトとして作成します。  
（※ 変更不可にするため、`Object.freeze()` を使用しています）

```javascript:数値管理された状態をenum風に管理
const CALC_STATE = Object.freeze({
  Start: 0,            // 初期状態
  NegativeNum1: 1,     // 左辺：マイナス符号を入力
  OperandZero1: 2,     // 左辺：ゼロ入力状態
  OperandInteger1: 3,  // 左辺：整数モード
  OperandDecimal1: 4,  // 左辺：小数モード
  Operator: 5,         // 演算子入力
  NegativeNum2: 6,     // 右辺：マイナス符号を入力
  OperandZero2: 7,     // 右辺：ゼロ入力状態
  OperandInteger2: 8,  // 右辺：整数モード
  OperandDecimal2: 9,  // 右辺：小数モード
  Result: 10,          // 計算結果
});
```

これで状態管理を数値化しつつ、`CALC_STATE.OperandZero1` で `2` を呼び出せる、といったように 状態名でも管理できるようになりました。

### 入力可能な情報を定義

次に、電卓が入力を受け付ける値一覧を定義します。

```javascript:Calculatorが受け取れる文字列一覧
const VALID_INPUTS = [
  "0", "1", "2", "3", "4", "5", "6", "7", "8", "9", ".", "+", "-", "*", "/", "="
];
```

#### 状態下での入力制限情報を定義

上記は、**電卓全体で**受け入れる入力値でした。しかし実際は**各状態下で入力値に制限が生じます**。下表は 各状態に対応する、入力できない文字列一覧です。

|状態|制限される入力値|
|:---|:---|
|初期状態|`["+", "*", "/", "="]`|
|左辺：マイナス符号を入力|`["+", "-", "*", "/", "="]`|
|左辺：ゼロ入力状態|`["0", "="]`|
|左辺：整数モード|`["="]`|
|左辺：小数モード|`[".", "="]`|
|演算子入力|`["+", "*", "/", "="]`|
|右辺：マイナス符号を入力|`["+", "-", "*", "/", "="]`|
|右辺：ゼロ入力状態|`["0"]`|
|右辺：整数モード|`[]`|
|右辺：小数モード|`["."]`|
|計算結果|`["="]`|

上表を定数として定義します。その際、入力集は数値管理できるようにし、[状態定義](#定数として状態を定義)で割り振った数値と対応関係を持たせます。こうすることで、状態名から必要となる入力集を引き出せるようにしておきます。

```javascript:各状態に対応する、無効とされる入力集
const INVALID_INPUTS_BY_STATE = Object.freeze({
  0:  ["+", "*", "/", "="],       // 初期状態
  1:  ["+", "-", "*", "/", "="],  // 左辺：マイナス符号を入力
  2:  ["0", "="],  // 左辺：ゼロ入力状態
  3:  ["="],       // 左辺：整数モード
  4:  [".", "="],  // 左辺：小数モード
  5:  ["+", "*", "/", "="],       // 演算子入力
  6:  ["+", "-", "*", "/", "="],  // 右辺：マイナス符号を入力
  7:  ["0"],  // 右辺：ゼロ入力状態
  8:  [],     // 右辺：整数モード
  9:  ["."],  // 右辺：小数モード
  10: ["="],  // 計算結果
})
```