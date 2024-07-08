---
title: "応用④：HTMLとの連携"
---

## 本項について

応用の最後となる本項では、`htmlCalculator` とHTMLを繋げ、ボタン要素等で操作できるようにしていきます。

本項で使うHTML文は[応用②：今回使用するHTML文](./d2-prepare-html)を、`htmlCalculator` の中身については[応用③：htmlCalculatorクラス](./d3-htmlcalculator)を参照してください。

## `htmlCalculator`インスタンスの生成

### 各種要素の取得

ここでは、HTMLに割り振った `id` を基に JavaScript 上で要素を定義し、扱えるようにします。

```javascript:ディスプレイ部要素以外は全てbutton要素
// 出力要素（ディスプレイ部）
const b01_calcLabel = document.querySelector('#b01js_label');  // span 要素
const b01_calcOutput = document.querySelector('#b01js_output');  // output 要素

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
```

### インスタンス生成

扱う要素が出揃ったので、`htmlCalculator` を生成します。その際、引数として各種入力ボタンを（オブジェクトの形式で）渡します。

```javascript
// 引数用にオブジェクトを準備（プロパティ名は自身の入力値）
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

// 生成し、初期化（≒ラベルと式を消し、refreshButtons() を実行）
const calculator = new htmlCalculator(buttons);
calculator.reset();
```

これで、`htmlCalculator` が生成されました。

## ボタン処理を記述

ここからは、生成した `htmlCalculator` を用いた操作を、ボタンのクリックイベントに割り振っていきます。

### 機能ボタン

まずは、「全消去」「１字消去」ボタンです。これらはそれぞれ `calculator.reset()` と `calculator.back()` に対応します。

```javascript
// 「全消去」ボタン
b01_calcBtnAllClear.addEventListener('click', () => {
  // リセット処理を施し、ディスプレイの式とラベル情報（≒空欄）を取得
  calculator.reset();
  b01_calcOutput.textContent = calculator.expression;
  b01_calcLabel.textContent = calculator.label;
})

// 「1字消去」ボタン
b01_calcBtnBackSpace.addEventListener('click', () => {
  if(calculator.back() == true){
    // 処理成功したので、ディスプレイに出力する情報を calculator から引き出す
    b01_calcOutput.textContent = calculator.expression;
    b01_calcLabel.textContent = calculator.label;
  } else {
    // false が返ってくる、処理に失敗した際の処理（ここでは特に設けないものとする）
  }
});
```

### 式への入力ボタン

16種類ある入力ボタンですが、クリック時に行う内容は共通しています。

1. `data-calc` 属性にある内容を、`calculator.pushExpression` で送る
2. 返り値が `true` の場合（≒処理に成功）、ディスプレイ（式とラベル）を更新（再取得）
3. 返り値が `false` の場合（≒処理に失敗）、ディスプレイの式を更新（再取得）し、ラベルに「不正な入力です」と表示させる

そのため 各ボタン個別にコードを作らず、ループ処理でまとめてイベント登録処理を行います。

```javascript
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
```

これでHTMLとの連携が完了し、ボタン要素で電卓を操作できるようになりました。
