---
title: "応用⚓：コード全文"
---

## 本項について

最後に、ここまでのコードをまとめて掲載しておきます。

## HTML

```html
<div>
  <section aria-label="ディスプレイ">
    <span id="b01js_label"></span>
    <output id="b01js_output"></output>
  </section>
  <section aria-label="機能">
    <button type="button">閉じる</button>
    <button type="button" id="b01js_AllClear">全消去</button>
    <button type="button" id="b01js_BackSpace">1字消去</button>
    <button type="button" id="b01js_Copy">コピー</button>
  </section>
  <section aria-label="テンキー・実行">
    <button type="button" id="b01js_9" data-calc="9">9</button>
    <button type="button" id="b01js_8" data-calc="8">8</button>
    <button type="button" id="b01js_7" data-calc="7">7</button>
    <button type="button" id="b01js_6" data-calc="6">6</button>
    <button type="button" id="b01js_5" data-calc="5">5</button>
    <button type="button" id="b01js_4" data-calc="4">4</button>
    <button type="button" id="b01js_3" data-calc="3">3</button>
    <button type="button" id="b01js_2" data-calc="2">2</button>
    <button type="button" id="b01js_1" data-calc="1">1</button>
    <button type="button" id="b01js_0" data-calc="0">0</button>
    <button type="button" id="b01js_DecimalPoint" data-calc="." aria-label="小数点">.</button>
    <button type="button" id="b01js_Equal" data-calc="=" aria-label="計算実行">=</button>
  </section>
  <section aria-label="四則演算">
    <button type="button" id="b01js_Add" data-calc="+" aria-label="足す">＋</button>
    <button type="button" id="b01js_Subtract" data-calc="-" aria-label="引く">−</button>
    <button type="button" id="b01js_Multiply" data-calc="*" aria-label="掛ける">×</button>
    <button type="button" id="b01js_Divide" data-calc="/" aria-label="割る">÷</button>
  </section>
</div>
```

:::message

スタイルを含めたものについては、[応用②：今回使用するHTML文](./d2-prepare-html)を参照してください。

:::

## JavaScript

```javascript
/* ≡≡≡ ▀▄ HTML要素郡 ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    各種入力ボタンや、出力先のラベルなどを管理する
---------------------------------------------------------------------------- */

// 出力要素
const b01_calcLabel = document.querySelector('#b01js_label');
const b01_calcOutput = document.querySelector('#b01js_output');

// 計算ボタン（「式への入力」で使用する 0〜9, 小数点, 四則演算子）
const b01_calcBtn1 = document.querySelector('#b01js_1');
const b01_calcBtn2 = document.querySelector('#b01js_2');
const b01_calcBtn3 = document.querySelector('#b01js_3');
const b01_calcBtn4 = document.querySelector('#b01js_4');
const b01_calcBtn5 = document.querySelector('#b01js_5');
const b01_calcBtn6 = document.querySelector('#b01js_6');
const b01_calcBtn7 = document.querySelector('#b01js_7');
const b01_calcBtn8 = document.querySelector('#b01js_8');
const b01_calcBtn9 = document.querySelector('#b01js_9');
const b01_calcBtn0 = document.querySelector('#b01js_0');
const b01_calcBtnDecimalPoint = document.querySelector('#b01js_DecimalPoint');
const b01_calcBtnAdd = document.querySelector('#b01js_Add');
const b01_calcBtnSubtract = document.querySelector('#b01js_Subtract');
const b01_calcBtnMultiply = document.querySelector('#b01js_Multiply');
const b01_calcBtnDevide = document.querySelector('#b01js_Divide');
const b01_calcBtnEqual = document.querySelector('#b01js_Equal');

// 機能ボタン（クリア, バックスペース）
const b01_calcBtnAllClear = document.querySelector('#b01js_AllClear');
const b01_calcBtnBackSpace = document.querySelector('#b01js_BackSpace');















/* ≡≡≡ ▀▄ Functions ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    主に、丸め誤差を回避するための四則演算関係のもの。
---------------------------------------------------------------------------- */

/**
 * 入力された数値の小数点位置を返す関数。
 * @param {string} value - 入力文字列。例: '1.01'
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
 * @param {string} input - 入力文字列。例: '1+1'
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
  const z = 10 ** (getDecimalPosition(x) + getDecimalPosition(y));
  x = (x + '').replace('.', '');
  y = (y + '').replace('.', '');
  return (x * y) / z;
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
  const z = 10 ** Math.max(getDecimalPosition(x), getDecimalPosition(y));
  return multiplication(x, z) / multiplication(y, z);
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
  '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
  '.', '+', '-', '*', '/', '='
];

/**
 * 各STATEに対応する、無効とされる入力集
 */
const INVALID_INPUTS_BY_STATE = Object.freeze({
  0:  ['+', '*', '/', '='],       // 初期状態
  1:  ['+', '-', '*', '/', '='],  // 左辺：マイナス符号を入力
  2:  ['0', '='],  // 左辺：ゼロ入力状態
  3:  ['='],       // 左辺：整数モード
  4:  ['.', '='],  // 左辺：小数モード
  5:  ['+', '*', '/', '='],       // 演算子入力
  6:  ['+', '-', '*', '/', '='],  // 右辺：マイナス符号を入力
  7:  ['0'],  // 右辺：ゼロ入力状態
  8:  [],     // 右辺：整数モード
  9:  ['.'],  // 右辺：小数モード
  10: ['='],  // 計算結果
  11: ['+', '*', '/', '=']  // エラー（これ以上の計算処理を遮断）
})


/** 簡易的な電卓機能を持つ、ベースとなるクラス */
class Calculator {
  constructor() {
    this._expression = '';
    this._state = CALC_STATE.Start; // -> 0
  }

  get expression(){
    return this._expression;
  }

  pushExpression(input) {
    // 許容されているデータのみ処理に進める（入力としてありえるもの、かつ現ステートで無効な値でない）
    if ((!VALID_INPUTS.includes(input)) || (INVALID_INPUTS_BY_STATE[this._state].includes(input))) {
      return false;
    }
    switch (this._state){
      case CALC_STATE.Error:
        /* [Nan ] : 計算数値外（Infinity や Nan）の状態
         * ------------------------------------------------
         *   0    : 初期化後 OperandZero1 に移行
         *   -    : 初期化後 NegativeNum1 に移行
         *   .    : 初期化後 '0' を先に追加し、OperandDecimal1 に移行
         *  1-9   : 初期化後 OperandIntger1 に移行
         */
        switch (input){
          case '0':
            this._expression = input;
            this._state = CALC_STATE.OperandZero1;
            break;
          case '-':
            this._expression = input;
            this._state = CALC_STATE.NegativeNum1;
            break;
          case '.':
            this._expression = '0';
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
         *   .    : '0' を先に追加し、OperandDecimal1 に移行
         *  1-9   : OperandIntger1 に移行
         */
        switch (input){
          case '0':
            this._expression += input;
            this._state = CALC_STATE.OperandZero1;
            break;
          case '-':
            this._expression += input;
            this._state = CALC_STATE.NegativeNum1;
            break;
          case '.':
            this._expression += '0';
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
         *   .    : '0' を先に追加し、OperandDecimal1 に移行
         *  1-9   : OperandIntger1 に移行
         */
        switch (input){
          case '0':
            this._expression += input;
            this._state = CALC_STATE.OperandZero1;
            break;
          case '.':
            this._expression += '0';
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
         *  1-9   : 式にある '0' を消し、OperandIntger1 に移行
         */
        switch (input){
          case '.':
            this._expression += input;
            this._state = CALC_STATE.OperandDecimal1;
            break;
          case '+':
          case '-':
          case '*':
          case '/':
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
          case '.':
            this._expression += input;
            this._state = CALC_STATE.OperandDecimal1;
            break;
          case '+':
          case '-':
          case '*':
          case '/':
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
          case '+':
          case '-':
          case '*':
          case '/':
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
         *   .    : '0' を先に追加し、OperandDecimal2 に移行
         *   -    : NegativeNum2 に移行
         *  1-9   : OperandIntger2 に移行
         */
        switch (input){
          case '0':
            this._expression += input;
            this._state = CALC_STATE.OperandZero2;
            break;
          case '.':
            this._expression += '0';
            this._expression += input;
            this._state = CALC_STATE.OperandDecimal2;
            break;
          case '-':
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
         *   .    : '0' を先に追加し、OperandDecimal2 に移行
         *  1-9   : OperandIntger2 に移行
         */
        switch (input){
          case '0':
            this._expression += input;
            this._state = CALC_STATE.OperandZero2;
            break;
          case '.':
            this._expression += '0';
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
         *  1-9   : 式にある '0' を消し、OperandIntger2 に移行
         */
        switch (input){
          case '.':
            this._expression += input;
            this._state = CALC_STATE.OperandDecimal2;
            break;
          case '=':
            this._expression = this.calculate();
            if(isFinite(this._expression)){
              this._state = CALC_STATE.Result;
            } else {
              console.warn(`計算不能値 ${this._expression} のため、Calculator はエラー状態に移行しました`);
              this._state = CALC_STATE.Error;
            }
            break;
          case '+':
          case '-':
          case '*':
          case '/':
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
          case '.':
            this._expression += input;
            this._state = CALC_STATE.OperandDecimal2;
            break;
          case '=':
            this._expression = this.calculate();
            if(isFinite(this._expression)){
              this._state = CALC_STATE.Result;
            } else {
              console.warn(`計算不能値 ${this._expression} のため、Calculator はエラー状態に移行しました`);
              this._state = CALC_STATE.Error;
            }
            break;
          case '+':
          case '-':
          case '*':
          case '/':
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
          case '=':
            this._expression = this.calculate();
            if(isFinite(this._expression)){
              this._state = CALC_STATE.Result;
            } else {
              console.warn(`計算不能値 ${this._expression} のため、Calculator はエラー状態に移行しました`);
              this._state = CALC_STATE.Error;
            }
            break;
          case '+':
          case '-':
          case '*':
          case '/':
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
         *   .    : 計算結果を消し、'0' を先に追加し、OperandDecimal1 に移行
         *   0    : 計算結果を消し、OperandZero1 に移行
         *  1-9   : 計算結果を消し、OperandIntger1 に移行
         */
        switch (input){
          case '+':
          case '-':
          case '*':
          case '/':
            this._expression += input;
            this._state = CALC_STATE.Operator;
            break;
          case '.':
            this._expression = '0';
            this._expression += input;
            this._state = CALC_STATE.OperandDecimal1;
            break;
          case '0':
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

/**
 * ベースとなる Calculator を、ラベルやボタン等のUIに適用させるために拡張させたもの。
 */
class htmlCalculator extends Calculator{
  constructor(btns){
    super();
    this._label = '';

    // 操作用ボタン登録
    this._btn = {};
    this._btn['0'] = btns['0'] || null;
    this._btn['1'] = btns['1'] || null;
    this._btn['2'] = btns['2'] || null;
    this._btn['3'] = btns['3'] || null;
    this._btn['4'] = btns['4'] || null;
    this._btn['5'] = btns['5'] || null;
    this._btn['6'] = btns['6'] || null;
    this._btn['7'] = btns['7'] || null;
    this._btn['8'] = btns['8'] || null;
    this._btn['9'] = btns['9'] || null;
    this._btn['.'] = btns['.'] || null;
    this._btn['+'] = btns['+'] || null;
    this._btn['-'] = btns['-'] || null;
    this._btn['*'] = btns['*'] || null;
    this._btn['/'] = btns['/'] || null;
    this._btn['='] = btns['='] || null;
  }

  get label(){
    return this._label;
  }

  pushExpression(input){
    switch (this._state){
      case CALC_STATE.OperandZero2:
      case CALC_STATE.OperandInteger2:
      case CALC_STATE.OperandDecimal2:
        // 計算処理が実行される場合にラベル取得
        if((isOperator(input)) || (input == '=')){
          this._label = this.expression;
        }
        break;
      case CALC_STATE.Result:
      case CALC_STATE.Error:
        // 計算処理を破棄する場合にラベル除去
        if((!isOperator(input)) && (input != '=')){
          this._label = '';
        }
        break;
    }
    const result = super.pushExpression(input);
    this.refreshButtons();
    return result;
  }

  reset(){
    this._label = '';
    super.reset();
    this.refreshButtons();
  }

  back(){
    const buf = this._label;
    const result = super.back();
    this._label = buf;
    return result;
  }

  refreshButtons(){
    const invalidNames = INVALID_INPUTS_BY_STATE[this._state];
    for(const name of VALID_INPUTS){
      // 現状態で制限されるボタンは disable 付与、それ以外は解除
      this._btn[name].disabled = invalidNames.includes(name);
    }
  }
}

const buttons = {
  '0': b01_calcBtn0,
  '1': b01_calcBtn1,
  '2': b01_calcBtn2,
  '3': b01_calcBtn3,
  '4': b01_calcBtn4,
  '5': b01_calcBtn5,
  '6': b01_calcBtn6,
  '7': b01_calcBtn7,
  '8': b01_calcBtn8,
  '9': b01_calcBtn9,
  '.': b01_calcBtnDecimalPoint,
  '+': b01_calcBtnAdd,
  '-': b01_calcBtnSubtract,
  '*': b01_calcBtnMultiply,
  '/': b01_calcBtnDevide,
  '=': b01_calcBtnEqual,
}

const calculator = new htmlCalculator(buttons);
calculator.reset();














/* ≡≡≡ ▀▄ 各種HTML要素への機能割当 ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    押下時 calculator を動かすためのイベント関係と処理を登録
---------------------------------------------------------------------------- */

// 式へ入力するためのボタン郡
const b01_calcBtns = [
  b01_calcBtn1, b01_calcBtn2, b01_calcBtn3, b01_calcBtn4, b01_calcBtn5,
  b01_calcBtn6, b01_calcBtn7, b01_calcBtn8, b01_calcBtn9, b01_calcBtn0,
  b01_calcBtnDecimalPoint, b01_calcBtnAdd, b01_calcBtnSubtract,
  b01_calcBtnMultiply, b01_calcBtnDevide, b01_calcBtnEqual,
];

for (const btn of b01_calcBtns){
  btn.addEventListener('click', (e) => {
    if(calculator.pushExpression(e.target.getAttribute('data-calc')) === true) {
      b01_calcOutput.textContent = calculator.expression;
      b01_calcLabel.textContent = calculator.label;
    } else {
      b01_calcOutput.textContent = calculator.expression;
      b01_calcLabel.textContent = '不正な入力です';
    }
  })
}

// 「全消去」
b01_calcBtnAllClear.addEventListener('click', () => {
  calculator.reset();
  b01_calcOutput.textContent = calculator.expression;
  b01_calcLabel.textContent = calculator.label;
})

// 「1字消去」
b01_calcBtnBackSpace.addEventListener('click', () => {
  if(calculator.back() == true){
    b01_calcOutput.textContent = calculator.expression;
    b01_calcLabel.textContent = calculator.label;
  } else {
    // 処理失敗時については、特に処理を設けない
  }
});

// 「コピー」（※本書では特に説明を設けていないものになります）
const b01_calcBtnCopy = document.querySelector('#b01js_Copy');
b01_calcBtnCopy.addEventListener('click', () => {
  navigator.clipboard
  .writeText(calculator.expression)
  .then(
    // コピー処理成功時
  )
  .catch(e => {
    // コピー処理失敗時
    console.error(e);
  });
});
```
