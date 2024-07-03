---
title: "作成⑤：有限数外に関する状態と処理"
---

## 本項について

前項までの内容で、コアの基本部分が完了しました。ここまでのコードをまとめてブラウザに読み込むだけで、下記のことができるでしょう。

- 通常の計算処理をする（小数の丸め誤差を回避しながら）
- 計算結果を使って、計算を続ける
- リセット、バックスペース処理を行う

しかしゼロ除算などで算出された、有限数を超えた計算結果に対し、現在のコードは対応しきれていないのが現状です。

そこで本項は、新たな状態として「エラー」状態を定義し、これまでのコードに修正を加えて行きます。本項の内容を持って、作成段階で携わった `Calculator` クラスは完成となります。

## 有限数外の計算について

算出する方法の例としては、`1/0`, `0/0` があります。この2つを現段階の `Calculator` で計算処理させると、それぞれ `Infinity`, `Nan`(Not a Number) を返します。

### 他の電卓アプリでの計算結果について

ここで、他の電卓アプリで上記計算を行ったらどうなるか、見ていきたいと思います。

#### iPhone の場合

![計算結果として「エラー」と表示されている](/images/books/howto-create-calculator/ui-calc-iphone-01-02.png =320x)
*iPhoneでの 1÷0 および 0÷0 の結果*

#### Windows の場合

![計算結果として「0で割ることはできません」と表示されている](/images/books/howto-create-calculator/ui-calc-windows-01.png =320x)
*Windowsでの 1÷0 の結果、一部ボタンが非活性化*

![計算結果として「結果が定義されていません」と表示されている](/images/books/howto-create-calculator/ui-calc-windows-02.png =320x)
*Windowsでの 0÷0 の結果、一部ボタンが非活性化*

#### Android の場合

![計算結果として「ゼロでは除算できません」と表示されている](/images/books/howto-create-calculator/ui-calc-android-01.png =320x)
*Androidでの 1÷0 の結果*

![計算結果として「ゼロでは除算できません」と表示されている](/images/books/howto-create-calculator/ui-calc-android-02.png =320x)
*Androidでの 0÷0 の結果*

#### Chromebook の場合

![計算結果として「∞」と表示されている](/images/books/howto-create-calculator/ui-calc-chromebook-01.png =320x)
*Chromebookでの 1÷0 の結果*

![計算結果として「Not a number」と表示されている](/images/books/howto-create-calculator/ui-calc-chromebook-02.png =320x)
*Chromebookでの 0÷0 の結果*

### 本書での有限数を超えた計算結果の扱いについて

本書では、`Infinity`, `NaN` が表示されるところまでは許容するものとします。ただし、それを使って引き続き計算しようとするのは禁止します。誤作動の元となりかねないためです。

そうした事情から、これまでのコードに対し新たに「エラー」状態を実装します。

## Error状態の追加

### 状態の定義を更新

まずは `CALC_STATE.Error` と状態を追加し、関連する部分にも更新をかけます。

```javascript:各定数の末尾にError状態を新設
// 状態管理用の定数
const CALC_STATE = Object.freeze({
  Start: 0,            // 初期状態
  NegativeNum1: 1,     // 左辺：マイナス符号を入力
  OperandZero1: 2,     // 左辺：ゼロ入力状態
  OperandInteger1: 3,  // 左辺：整数モード
  OperandDecimal1: 4,  // 左辺：少数モード
  Operator: 5,         // 演算子入力
  NegativeNum2: 6,     // 右辺：マイナス符号を入力
  OperandZero2: 7,     // 右辺：ゼロ入力状態
  OperandInteger2: 8,  // 右辺：整数モード
  OperandDecimal2: 9,  // 右辺：少数モード
  Result: 10,          // 計算結果
  Error: 11,           // エラー（Infinity, NaN など）
});

// 各状態下で制限される入力集
const INVALID_INPUTS_BY_STATE = Object.freeze({
  0:  ["+", "*", "/", "="],       // Start
  1:  ["+", "-", "*", "/", "="],  // NegativeNum1
  2:  ["0", "="],  // OperandZero1
  3:  ["="],       // OperandInteger1
  4:  [".", "="],  // OperandDecimal1
  5:  ["+", "*", "/", "="],       // Operator
  6:  ["+", "-", "*", "/", "="],  // NegativeNum2
  7:  ["0"],  // OperandZero2
  8:  [],     // OperandInteger2
  9:  ["."],  // OperandDecimal2
  10: ["="],  // Result
  11: ["+", "*", "/", "="]  // Error（計算処理の継続を禁止）
})
```

### `pushExpression()` の更新

ここからは、式の入力処理を担っている `pushExpression()` を改修していきます。

#### 新規 `case` 文の追加

`pushExpression()` では、各状態事に `switch` 文で処理を書いていました。そのため、新規に設けたエラー状態にも専用の処理が必要になります。

式としては `NaN` や `Infinity` と表示されている状態です。ここから継続した計算はできないよう、演算子などに入力制限をかけています。

エラー状態では基本的に、それまでの計算内容を初期化し、初期状態と同じような動きをさせます。

```javascript:状態毎の処理にエラー状態のコードを追加
pushExpression(input) {
  // 許容されているデータのみ処理に進める（入力としてありえるもの、かつ現ステートで無効な値）
  if ((!VALID_INPUTS.includes(input)) || (INVALID_INPUTS_BY_STATE[this._state].includes(input))) {
    return false;
  }
  switch (this._state){
    case CALC_STATE.Error:
      /* [Nan ] : 計算数値外（Infinity や Nan）の状態
       * ------------------------------------------------
       *   0    : 初期化後 OperandZero1 に移行
       *   -    : 初期化後 NegativeNum1 に移行
       *   .    : 初期化後 "0" を先に追加し、OperandDecimal1 に移行
       *  1-9   : 初期化後 OperandIntger1 に移行
       */
      switch (input){
        case "0":
          this._expression = input;
          this._state = CALC_STATE.OperandZero1;
          break;
        case "-":
          this._expression = input;
          this._state = CALC_STATE.NegativeNum1;
          break;
        case ".":
          this._expression = "0";
          this._expression += input;
          this._state = CALC_STATE.OperandDecimal1;
          break;
        default: // -> case [1-9]
          this._expression = input;
          this._state = CALC_STATE.OperandInteger1;
      }
      break;
    case CALC_STATE.Start:
    // 以下省略
}}
```

#### `calculator()` 処理後に、有限数かの判定処理を追加

次に、状態内で `calculator()` 処理をさせている所を対象に、エラー状態に該当するかの判定処理を追記していきます。

下記は、[以前の項](./c2-core-push-expression#calc_stateoperandzero2)で作成した「右辺ゼロ入力」状態の処理です。

```javascript:作成②で作った、右辺ゼロ入力時の状態
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

これが、下記のように変わります。`this._expression = this.calculate()` の後ろに注目してください。

```javascript
case CALC_STATE.OperandZero2:
  switch (input){
    case ".":
      this._expression += input;
      this._state = CALC_STATE.OperandDecimal2;
      break;
    case "=":
      this._expression = this.calculate();
      if(isFinite(this._expression)){
        this._state = CALC_STATE.Result;
      } else {
        console.warn(`計算不能値 ${this._expression} のため、Calculator はエラー状態に移行しました`);
        this._state = CALC_STATE.Error;
      }
      break;
    case "+":
    case "-":
    case "*":
    case "/":
      this._expression = this.calculate();
      if(isFinite(this._expression)){
        this._expression += input;
        this._state = CALC_STATE.Operator;
      } else {
        console.warn(`計算不能値 ${this._expression} のため、Calculator はエラー状態に移行しました`);
        this._state = CALC_STATE.Error;
      }
      break;
    default: // -> case [1-9]
      this._expression = this._expression.slice(0, -1);
      this._expression += input;
      this._state = CALC_STATE.OperandInteger2;
  }
  break;
```

このように計算後、`isFinite(this._expression)` で有限数かを判定し、有限数（≒問題なし）であれば以前の処理を行い、有限数でない（≒`NaN` など）場合は、エラー状態に遷移させています。

これで、新規に設けたエラー状態に関する改修は完了です。
