---
title: "作成⚓：コード全文"
---

## 本項について

本項は、作成段階で作ったコードをまとめてあります。

## コード全文

```javascript
/**
 * 入力された数値の小数点位置を返す関数。
 * @param {string} value - 入力文字列。例: "1.01"
 * @returns {int} - 小数位置。例： 2
 */
function getDecimalPosition(value){
  const strVal = String(value);
  if(strVal.indexOf('.') !== -1){
    return ((strVal.length - 1) - strVal.indexOf('.'));
  }
  return 0;
}


/**
 * 入力文字列をOperandLeft、Operator、OperandRightに分割する関数。
 * @param {string} input - 入力文字列。例: "1+1"
 * @returns {Array} - [OperandLeft, Operator, OperandRight]の配列。
 */
function splitExpression(expression) {
  let operatorIndex = 1; // デフォルトでは2文字目からOperatorを探す

  // Operatorが2文字目以降にあるかチェック
  // Memo: indexOf() で検索をかける方法は、右辺が負の数であるパターンで複雑になるだけなので不採用
  for (let i = 1; i < expression.length; i++) {
      if (isOperator(expression[i])) {
          operatorIndex = i;
          break;
      }
  }

  const operandLeft = expression.substring(0, operatorIndex);
  const operator = expression[operatorIndex];
  const operandRight = expression.substring(operatorIndex + 1);

  return [operandLeft, operator, operandRight];
}

/**
* 文字が演算子かどうかを判定するヘルパー関数。
* @param {string} char - 判定する文字。
* @returns {boolean} - 文字が演算子であればtrue、それ以外はfalse。
*/
function isOperator(char) {
  return ['+', '-', '*', '/'].includes(char);
}


const multiplication = (x, y) => {
  const n = 10 ** (getDecimalPosition(x) + getDecimalPosition(y));
  x = +(x + '').replace('.', '');
  y = +(y + '').replace('.', '');
  return (x * y) / n;
};


const addition = (x, y) => {
  const z = 10 ** Math.max(getDecimalPosition(x), getDecimalPosition(y));
  return (multiplication(x, z) + multiplication(y, z)) / z;
};

const subtract = (x, y) => {
  const z = 10 ** Math.max(getDecimalPosition(x), getDecimalPosition(y));
  return (multiplication(x, z) - multiplication(y, z)) / z;
};

const division = (x, y) => {
  const decimalLengthX = getDecimalPosition(x);
  const decimalLengthY = getDecimalPosition(y);
  const n = 10 ** (decimalLengthY - decimalLengthX);

  x = +(x + '').replace('.', '');
  y = +(y + '').replace('.', '');

  return (x / y) * n;
}


/* ≡≡≡ ▀▄ Calculator ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    電卓としての機能を持たせている。
    1. 状態遷移による入力管理
    2. コントロールからある程度独立したコア機能
---------------------------------------------------------------------------- */

/**
 * 数値のステージを enum 風に管理
 */
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
  Error: 11,           // エラー（Infinity, NaN など）
});

/**
 * Calculator が受け取れる文字列一覧
 */
const VALID_INPUTS = [
  "0", "1", "2", "3", "4", "5", "6", "7", "8", "9",
  ".", "+", "-", "*", "/", "="
];

/**
 * 各STATEに対応する、無効とされる入力集
 */
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
  11: ["+", "*", "/", "="]  // エラー（これ以上の計算処理を遮断）
})

class Calculator {
  constructor() {
    this._expression = '';
    this._state = CALC_STATE.Start; // -> 0
  }

  get expression(){
    return this._expression;
  }

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
        /* [    ] : 何も入力されていない状態
         * ------------------------------------------------
         *   0    : OperandZero1 に移行
         *   -    : NegativeNum1 に移行
         *   .    : "0" を先に追加し、OperandDecimal1 に移行
         *  1-9   : OperandIntger1 に移行
         */
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
      case CALC_STATE.NegativeNum1:
        /* [-   ] : 第一オペラントにマイナス入力
         * ------------------------------------------------
         *   0    : OperandZero1 に移行
         *   .    : "0" を先に追加し、OperandDecimal1 に移行
         *  1-9   : OperandIntger1 に移行
         */
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
      case CALC_STATE.OperandZero1:
        /* [0   ] : ゼロ入力モード
         * -------------------------------------------------
         *   .    : OperandDecimal1 に移行
         * 演算子  : Operator に移行
         *  1-9   : 式にある "0" を消し、OperandIntger1 に移行
         */
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
      case CALC_STATE.OperandInteger1:
        /* [3   ] : 整数入力モード
         * -------------------------------------------------
         *   .    : OperandDecimal1 に移行
         * 演算子  : Operator に移行
         *  0-9   : 変化なし
         */
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
      case CALC_STATE.OperandDecimal1:
        /* [3.  ] : 小数入力モード
         * -------------------------------------------------
         * 演算子  : Operator に移行
         *  0-9   : 変化なし
         */
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
      case CALC_STATE.Operator:
        /* [2+  ] : オペレータが入力された状態
         * ------------------------------------------------
         *   0    : OperandZero2 に移行
         *   .    : "0" を先に追加し、OperandDecimal2 に移行
         *   -    : NegativeNum2 に移行
         *  1-9   : OperandIntger2 に移行
         */
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
      case CALC_STATE.NegativeNum2:
        /* [2*- ] : 第二オペラントにマイナス入力
         * ------------------------------------------------
         *   0    : OperandZero2 に移行
         *   .    : "0" を先に追加し、OperandDecimal2 に移行
         *  1-9   : OperandIntger2 に移行
         */
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
      case CALC_STATE.OperandZero2:
        /* [2+0 ] : ゼロ入力モード
         * -------------------------------------------------
         *   .    : OperandDecimal2 に移行
         *   =    : 計算処理し、Result に移行（エラー値の場合は Error に移行）
         * 演算子  : 計算処理し、演算子を引き継ぎ Operator に移行
         *           （エラー値の場合は Error に移行）
         *  1-9   : 式にある "0" を消し、OperandIntger2 に移行
         */
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
      case CALC_STATE.OperandInteger2:
        /* [3+2 ] : 整数入力モード
         * -------------------------------------------------
         *   .    : OperandDecimal2 に移行
         *   =    : 計算処理し、Result に移行（エラー値の場合は Error に移行）
         * 演算子  : 計算処理し、演算子を引き継ぎ Operator に移行
         *         　（エラー値の場合は Error に移行）
         *  0-9   : 変化なし
         */
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
          default: // -> case [0-9]
            this._expression += input;
        }
        break;
      case CALC_STATE.OperandDecimal2:
        /* [3+2.] : 小数入力モード
         * -------------------------------------------------
         *   =    : 計算処理し、Result に移行（エラー値の場合は Error に移行）
         * 演算子  : 計算処理し、演算子を引き継ぎ Operator に移行
         *         　（エラー値の場合は Error に移行）
         *  0-9   : 変化なし
         */
        switch (input){
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
          default: // -> case [0-9]
            this._expression += input;
        }
        break;
      case CALC_STATE.Result:
        /* [5   ] : 計算結果の表示
         * -------------------------------------------------
         * 演算子  : Operator に移行
         *   .    : 計算結果を消し、"0" を先に追加し、OperandDecimal1 に移行
         *   0    : 計算結果を消し、OperandZero1 に移行
         *  1-9   : 計算結果を消し、OperandIntger1 に移行
         */
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
      default:
        console.error(`無効なステートが検出されました ${this._state}`);
        return false;
    }
    return true;
  }

  calculate() {
    let [strOpeLeft, operator, strOpeRight] = splitExpression(this._expression);
    let calcResult = 0;
    switch(operator){
      case '+':
        try {
          calcResult = addition(strOpeLeft, strOpeRight)
        } catch (error) {
          console.error(`Error: ${error.message}`);
          calcResult = NaN;
        }
        break;
      case '-':
        try {
          calcResult = subtract(strOpeLeft, strOpeRight);
        } catch (error) {
          console.error(`Error: ${error.message}`);
          calcResult = NaN;
        }
        break;
      case '*':
        try {
          calcResult = multiplication(strOpeLeft, strOpeRight);
        } catch (error) {
          console.error(`Error: ${error.message}`);
          calcResult = NaN;
        }
        break;
      case '/':
        try {
          calcResult = division(strOpeLeft, strOpeRight);
        } catch (error) {
          console.error(`Error: ${error.message}`);
          calcResult = NaN;
        }
        break;
      default:
        console.Error(`不明な演算子が検出されました。 operator: ${operator}`);
        calcResult = NaN;
    }
    return calcResult;
  }

  reset(){
    this._state = CALC_STATE.Start;
    this._expression = '';
  }

  back(){
    //リザルト状態やエラー状態でない、かつ 1文字以上の入力がある場合が処理対象
    if(
      (this._state == CALC_STATE.Result) ||
      (this._state == CALC_STATE.Error) ||
      (this._expression.length === 0)
    ){
      return false;
    }
    const expr = this._expression.slice(0, -1);
    this.reset();
    if(expr.length >= 1){
      for (var i = 0; i < expr.length; i++) {
        this.pushExpression(expr[i]);
      }
    }
    return true;
  }
}
```
