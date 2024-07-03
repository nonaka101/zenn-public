---
title: "作成②：Calculatorクラスと入力処理"
---

## 本項について

前項で状態管理できる所々の定数を定義できました。本項では、コア部となる `Calculator` クラスを作成していきます。

また、式へ入力するメソッド `pushExpression()` も定義していきます。

## `Calculator` クラスの生成

まずは、クラスを生成します。インスタンス化される際の初期設定として、[電卓が保持する情報](./b2-methods-and-properties#電卓が持つべき情報)として**式**と**状態**を準備するようします。

```javascript:Calculatorクラスの生成
class Calculator {
  constructor() {
    this._expression = '';
    this._state = CALC_STATE.Start; // -> 0
  }
}
```

:::message

`_expression`, `_state` と 最初にアンバーバーをつけているのは、直接アクセスするのは非推奨であることを示すためです。

:::

### クラス内メソッドの準備

ここではクラス内にメソッド等を準備していきます。

まず式を外部に引き出すための ゲッター と、[電卓が持つべき機能](./b2-methods-and-properties#電卓が持つべき機能)で説明した処理を、クラス内に用意します。

```javascript:Calculatorクラスの生成
class Calculator {
  constructor() {
    this._expression = '';
    this._state = CALC_STATE.Start; // -> 0
  }

  // 式を外部から参照できるためのゲッター
  get expression(){
    return this._expression;
  }

  // 式の入力
  pushExpression(input) {
    // 入力値を調べ、問題なければ式に追記し状態遷移を施す
  }

  // 計算実行
  calculate() {
    // 式を右辺、演算子、左辺に分割し、計算処理を行う
  }

  // リセット
  reset(){
    // 式と状態を、初期に戻す
  }

  // バックスペース
  back(){
    // 一度リセットし、式の末尾を消したものを使って、状態を再計算
  }
}
```

これで `const calculator = new Calculator();` でインスタンス生成し、メソッドも呼び出せるようになりました。ここからは、用意したメソッドの中身を埋めていきます。

## メソッド：`pushExpression()`

このメソッドは、**外部から1文字の入力を受け取り**、下記の処理を行います。

1. 入力不可な入力値の場合は、`false` を返して処理を中断
2. 入力値を、**各状態下での処理を施す**（例：式に追加、など）
3. 処理が完了した場合に、`true` を返す（処理中に問題があれば `false` を返す）

このように、入力が上手く行ったか失敗したかは `true`, `false` で返すことになります。入力の際に返ってきた真偽値に応じて、ボタン等のUIを操作できるようするためです。

### 入力不可な入力値の判定

まずは、受け入れるべきでない入力値を弾く処理を書いていきます。これは、前項で用意した状態関係の情報を使います。

```javascript:入力可能情報と、各種状態に関する情報
// 電卓全体で入力可能な値集
const VALID_INPUTS = [
  "0", "1", "2", "3", "4", "5", "6", "7", 
  "8", "9", ".", "+", "-", "*", "/", "="
];

// 状態管理用の定数
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

// 各状態に対応する、無効とされる入力集
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

上記を使い、「電卓全体で入力可能な値で**ない**場合」または「状態下で無効とされる入力集」を、無効とみなして `false` を返して処理を中断させます。

```javascript:ガード節的に、無効な入力を最初に弾いておく
pushExpression(input) {
  if ((!VALID_INPUTS.includes(input)) || (INVALID_INPUTS_BY_STATE[this._state].includes(input))) {
    return false;
  }

  // 諸々の処理が完了できたら true を返す
  return true;
}
```

### 各状態での処理分岐

受入可能な入力値を判別できるようになりました。ここからは、それを各状態に合わせた処理を施していくことになります。

状態数は多く、`if` による分岐は不適切です。ここでは、`switch` 文による分岐処理を行います。

```javascript:状態に応じたswitch文
pushExpression(input) {
  // 許容されているデータのみ処理に進める（入力としてありえるもの、かつ現ステートで無効な値）
  if ((!VALID_INPUTS.includes(input)) || (INVALID_INPUTS_BY_STATE[this._state].includes(input))) {
    return false;
  }
  switch (this._state){
    case CALC_STATE.Start:
      break;
    case CALC_STATE.NegativeNum1:
      break;
    case CALC_STATE.OperandZero1:
      break;
    case CALC_STATE.OperandInteger1:
      break;
    case CALC_STATE.OperandDecimal1:
      break;
    case CALC_STATE.Operator:
      break;
    case CALC_STATE.NegativeNum2:
      break;
    case CALC_STATE.OperandZero2:
      break;
    case CALC_STATE.OperandInteger2:
      break;
    case CALC_STATE.OperandDecimal2:
      break;
    case CALC_STATE.Result:
      break;
    default:
      console.error(`無効なステートが検出されました ${this._state}`);
      return false;
  }
  // 諸々の処理が完了できたら true を返す
  return true;
}

```

ここからは、各状態の処理を埋めていきます。そのために[以前の状態遷移図](./b1-architecture#電卓の挙動を再現した状態遷移図)であった表を使っていきます。

下表は、各状態の式の例を表したものです。

| `CALC_STATE`    | 状態名       | 式の例                     |
| :-------------- | :----------- | :------------------------- |
| Start           | 初期状態     | （なし）                   |
| NegativeNum1    | 負数状態１   | `-`                        |
| OperandZero1    | 左辺ゼロ入力 | `0`, `-0`                  |
| OperandInteger1 | 左辺整数入力 | `1`, `123`, `-1`           |
| OperandDecimal1 | 左辺小数入力 | `0.`, `0.1`, `-1.23`       |
| Operator        | 演算子       | `1+`, `-1-`, `0.12*`       |
| NegativeNum2    | 負数入力２   | `1+-`, `10--`, `-0.12--`   |
| OperandZero2    | 右辺ゼロ入力 | `1+0`, `1+-0`              |
| OperandInteger2 | 右辺整数入力 | `1+1`, `1+123`, `1+-12`    |
| OperandDecimal2 | 右辺小数入力 | `1+0.`, `1*0.1`, `1--1.23` |
| Result          | 計算結果     | `2`                        |

下表は、列見出しで「状態名」、行見出しで「入力値」を表し、その交点にあるセルは「その状態で入力が為された場合、次に遷移する状態名」を表します。

| 状態名＼取りうる値 | `0`             | `1-9`           | `.`             | `-`          | `+`, `*`, `/` | `=`    |
| ------------------ | --------------- | --------------- | --------------- | ------------ | ------------- | ------ |
| Start              | OperandZero1    | OperandInteger1 | OperandDecimal1 | NegativeNum1 | -             | -      |
| NegativeNum1       | OperandZero1    | OperandInteger1 | OperandDecimal1 | -            | -             | -      |
| OperandZero1       | -               | OperandInteger1 | OperandDecimal1 | Operator     | Operator      | -      |
| OperandInteger1    | OperandInteger1 | OperandInteger1 | OperandDecimal1 | Operator     | Operator      | -      |
| OperandDecimal1    | OperandDecimal1 | OperandDecimal1 | -               | Operator     | Operator      | -      |
| Operator           | OperandZero2    | OperandInteger2 | OperandDecimal2 | NegativeNum2 | -             | -      |
| NegativeNum2       | OperandZero2    | OperandInteger2 | OperandDecimal2 | -            | -             | -      |
| OperandZero2       | -               | OperandInteger2 | OperandDecimal2 | Operator(*)  | Operator(*)   | Result |
| OperandInteger2    | OperandInteger2 | OperandInteger2 | OperandDecimal2 | Operator(*)  | Operator(*)   | Result |
| OperandDecimal2    | OperandDecimal2 | OperandDecimal2 | -               | Operator(*)  | Operator(*)   | Result |
| Result             | OperandZero1    | OperandInteger1 | OperandDecimal1 | Operator     | Operator      | -      |

(*): 一度 Result を経由して Operator に遷移する

:::message alert

各状態の中には `this.calculate()` で、自身の式に対し計算実行する箇所があります。詳細については次項で解説するので、ここでは「式を評価して計算結果に置き換える」処理と考えてください。

:::

#### `CALC_STATE.Start`

式には何も入力されていない、初期状態です。

| 入力値 | 入力操作                                 |
| :----: | :--------------------------------------- |
|  `0`   | OperandZero1 に遷移                      |
|  `-`   | NegativeNum1 に遷移                      |
|  `.`   | "0" を先に追加し、OperandDecimal1 に遷移 |
| `1-9`  | OperandIntger1 に遷移                    |

```javascript
case CALC_STATE.Start:
  switch (input){
    case "0":
      this._expression += input;
      this._state = CALC_STATE.OperandZero1;
      break;
    case "-":
      this._expression += input;
      this._state = CALC_STATE.NegativeNum1;
      break;
    case ".":
      this._expression += "0";
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal1;
      break;
    default: // -> case [1-9]
      this._expression += input;
      this._state = CALC_STATE.OperandInteger1;
  }
  break;
```

#### `CALC_STATE.NegativeNum1`

式は `-` で、左辺にマイナス符号が入力された状態です。

| 入力値 | 入力操作                                 |
| :----: | :--------------------------------------- |
|  `0`   | OperandZero1 に遷移                      |
|  `.`   | "0" を先に追加し、OperandDecimal1 に遷移 |
| `1-9`  | OperandIntger1 に遷移                    |

```javascript
case CALC_STATE.NegativeNum1:
  switch (input){
    case "0":
      this._expression += input;
      this._state = CALC_STATE.OperandZero1;
      break;
    case ".":
      this._expression += "0";
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal1;
      break;
    default: // -> case [1-9]
      this._expression += input;
      this._state = CALC_STATE.OperandInteger1;
  }
  break;
```

#### `CALC_STATE.OperandZero1`

式は `0` のように、ゼロが入力された状態です。

|  入力値  | 入力操作                                   |
| :------: | :----------------------------------------- |
|   `.`    | OperandDecimal1 に遷移                     |
| `演算子` | Operator に遷移                            |
|  `1-9`   | 式にある "0" を消し、OperandIntger1 に遷移 |

```javascript
case CALC_STATE.OperandZero1:
  switch (input){
    case ".":
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal1;
      break;
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression += input;
      this._state = CALC_STATE.Operator;
      break;
    default: // -> case [1-9]
      this._expression = this._expression.slice(0, -1);
      this._expression += input;
      this._state = CALC_STATE.OperandInteger1;
  }
  break;
```

#### `CALC_STATE.OperandInteger1`

式は `1`, `123` のように、整数が入力されている状態です。

|  入力値  | 入力操作               |
| :------: | :--------------------- |
|   `.`    | OperandDecimal1 に遷移 |
| `演算子` | Operator に遷移        |
|  `0-9`   | 変化なし               |

```javascript
case CALC_STATE.OperandInteger1:
  switch (input){
    case ".":
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal1;
      break;
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression += input;
      this._state = CALC_STATE.Operator;
      break;
    default: // -> case [0-9]
      this._expression += input;
  }
  break;
```

#### `CALC_STATE.OperandDecimal1`

式は `0.`, `0.12` のように、小数が入力されている状態です。

|  入力値  | 入力操作        |
| :------: | :-------------- |
| `演算子` | Operator に遷移 |
|  `0-9`   | 変化なし        |

```javascript
case CALC_STATE.OperandDecimal1:
  switch (input){
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression += input;
      this._state = CALC_STATE.Operator;
      break;
    default: // -> case [0-9]
      this._expression += input;
  }
  break;
```

#### `CALC_STATE.Operator`

式は `1+`, `0.123/` のように、演算子が入力された状態です。

| 入力値 | 入力操作                                 |
| :----: | :--------------------------------------- |
|  `0`   | OperandZero2 に遷移                      |
|  `.`   | "0" を先に追加し、OperandDecimal2 に遷移 |
|  `-`   | NegativeNum2 に遷移                      |
| `1-9`  | OperandIntger2 に遷移                    |

```javascript
case CALC_STATE.Operator:
  switch (input){
    case "0":
      this._expression += input;
      this._state = CALC_STATE.OperandZero2;
      break;
    case ".":
      this._expression += "0";
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal2;
      break;
    case "-":
      this._expression += input;
      this._state = CALC_STATE.NegativeNum2;
      break;
    default: // -> case [1-9]
      this._expression += input;
      this._state = CALC_STATE.OperandInteger2;
  }
  break;
```

#### `CALC_STATE.NegativeNum2`

式は `1+-`, `0.1*-` のように、（演算子以降の）右辺に負符号が入力された状態です。

| 入力値 | 入力操作                                 |
| :----: | :--------------------------------------- |
|  `0`   | OperandZero2 に遷移                      |
|  `.`   | "0" を先に追加し、OperandDecimal2 に遷移 |
| `1-9`  | OperandIntger2 に遷移                    |

```javascript
case CALC_STATE.NegativeNum2:
  switch (input){
    case "0":
      this._expression += input;
      this._state = CALC_STATE.OperandZero2;
      break;
    case ".":
      this._expression += "0";
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal2;
      break;
    default: // -> case [1-9]
      this._expression += input;
      this._state = CALC_STATE.OperandInteger2;
  }
  break;
```

#### `CALC_STATE.OperandZero2`

式は `1+0` のように、右辺にゼロが入力された状態です。

| 入力値   | 入力操作                                     |
| :------- | :------------------------------------------- |
| `.`      | OperandDecimal2 に移行                       |
| `=`      | 計算処理し、Result に移行                    |
| `演算子` | 計算処理し、演算子を引き継ぎ Operator に移行 |
| `1-9`    | 式にある "0" を消し、OperandIntger2 に移行   |

```javascript
case CALC_STATE.OperandZero2:
  switch (input){
    case ".":
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal2;
      break;
    case "=":
      this._expression = this.calculate();
      this._state = CALC_STATE.Result;
      break;
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression = this.calculate();
      this._expression += input;
      this._state = CALC_STATE.Operator;
      break;
    default: // -> case [1-9]
      this._expression = this._expression.slice(0, -1);
      this._expression += input;
      this._state = CALC_STATE.OperandInteger2;
  }
break;
```

#### `CALC_STATE.OperandInteger2`

式は `1+1`, `1+123` のように、右辺に整数が入力された状態です。

|  入力値  | 入力操作                                     |
| :------: | :------------------------------------------- |
|   `.`    | OperandDecimal2 に移行                       |
|   `=`    | 計算処理し、Result に移行                    |
| `演算子` | 計算処理し、演算子を引き継ぎ Operator に移行 |
|  `0-9`   | 変化なし                                     |

```javascript
case CALC_STATE.OperandInteger2:
  switch (input){
    case ".":
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal2;
      break;
    case "=":
      this._expression = this.calculate();
      this._state = CALC_STATE.Result;
      break;
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression = this.calculate();
      this._expression += input;
      this._state = CALC_STATE.Operator;
      break;
    default: // -> case [0-9]
      this._expression += input;
  }
  break;
```

#### `CALC_STATE.OperandDecimal2`

式は `1+1.`, `1+0.12` のように、右辺に小数を入力している状態です。

|  入力値  | 入力操作                                     |
| :------: | :------------------------------------------- |
|   `=`    | 計算処理し、Result に移行                    |
| `演算子` | 計算処理し、演算子を引き継ぎ Operator に移行 |
|  `0-9`   | 変化なし                                     |

```javascript
case CALC_STATE.OperandDecimal2:
  switch (input){
    case "=":
      this._expression = this.calculate();
      this._state = CALC_STATE.Result;
      break;
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression = this.calculate();
      this._expression += input;
      this._state = CALC_STATE.Operator;
      break;
    default: // -> case [0-9]
      this._expression += input;
  }
  break;
```

#### `CALC_STATE.Result`

式に対し `=` を入力し、計算結果を表示している状態です。

|  入力値  | 入力操作                                                 |
| :------: | :------------------------------------------------------- |
| `演算子` | Operator に移行                                          |
|   `.`    | 計算結果を消し、"0" を先に追加し、OperandDecimal1 に移行 |
|   `0`    | 計算結果を消し、OperandZero1 に移行                      |
|  `1-9`   | 計算結果を消し、OperandIntger1 に移行                    |

```javascript
case CALC_STATE.Result:
  switch (input){
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression += input;
      this._state = CALC_STATE.Operator;
      break;
    case ".":
      this._expression = "0";
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal1;
      break;
    case "0":
      this._expression = input;
      this._state = CALC_STATE.OperandZero1;
      break;
    default: // -> case [1-9]
      this._expression = input;
      this._state = CALC_STATE.OperandInteger1;
  }
  break;
```

#### その他 `case default`

上記のどれにも当てはまらないケースは、想定外のためエラーとして扱います。

```javascript
default:
  console.error(`無効なステートが検出されました ${this._state}`);
  return false;
```
