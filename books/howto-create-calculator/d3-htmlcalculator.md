---
title: "応用③：htmlCalculatorクラス"
---

## 本項について

ここでは、これまでに作成した[Calculatorクラス](./c9-js-code)や[HTML文](./d2-prepare-html)を用いて、新規クラス `htmlCalculator()` を作成していきます。

## `htmlCalculator`クラスの定義

最初にクラス `htmlCalculator` を定義していきます。

### `extends` キーワードによるサブクラス生成

これから作成する `htmlCalculator` は、既存の `Calculator` クラスを拡張する形で実装します。既存の機能や情報を使いつつ、HTML要素と結び付けて操作できることを目的としているためです。

このため `extends`, `super` キーワードを使って、サブクラスという形で定義します。

```javascript
class htmlCalculator extends Calculator{
  constructor(){
    // super キーワードで親となる Calculator を呼び出す（this を使う前に配置する必要）
    super();
  }
}
```

### 機能や情報等を準備

[応用の１回目](./d1-extend-class)にて、`htmlCalculator` がどのような動きをするかについて考えました。下図はそこで使ったイメージ図になります。

![htmlCalculatorクラスのイメージ図。既存クラスを拡張し、各種ボタン（HTML要素）と結びつけている](/images/books/howto-create-calculator/html-calculator-01.png)

ここでは、既存の情報や機能を拡張している他に、下記２点を新規に盛り込んでいることがわかります。

|名前|区分|役割|
|:---:|:---:|:---|
|`_label`|情報|ディスプレイの補助欄（ラベル）に掲載する情報。例えば、`=` で計算実行した際の計算式 など|
|`refreshButtons()`|機能|現在の状態を基に、入力できないボタンを非活性状態にする|

ここから、`htmlCalculator` クラスに下記を盛り込んでいきます。

- インスタンス生成時、内部に `_label` を定義（初期値は空文字列）
- `Calculator` の `_expression` 同様、ゲッターで `_label` を取得できるようにしておく
- 新規メソッド `refreshButtons()` を定義

```javascript
class htmlCalculator extends Calculator{
  constructor(){
    super();
    this._label = '';
  }

  get label(){
    return this._label;
  }

  refreshButtons(){
    // 現在の状態を基に、入力できないボタンを非活性状態にする処理
  }
}
```

これで、基本的な部分が定義できました。

## `refreshButtons()`に関連した処理

ここからは、新たに定義した機能 `refreshButtons` を作り込んでいきます。

### 入力不可な情報一覧の取得

`htmlCalculator` は `Calculator` のサブクラスなので、`Calculator` が持っていた `_state` 等の**情報や機能を継承しています**。

そして **状態**に関しては[以前に定数として定義](./c1-state#定数として状態を定義)し、それと対応させた形で[入力不可な情報一覧も定義](./c1-state#状態下での入力制限情報を定義)しました。

```javascript:入力可能情報と、各種状態に関する情報（Error状態含む）
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
  Error: 11,           // エラー（Infinity, NaN など）
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
  11: ["+", "*", "/", "="]  // エラー（これ以上の計算処理を遮断）
})
```

`htmlCalculator` は、自身の状態を `this._state` で取得できます。そのため、現状態での入力制限一覧は `INVALID_INPUTS_BY_STATE[this._state]` で取得可能です。

そして式への入力可能な情報一覧は別途 `VALID_INPUTS` で定義されているので、下記のような流れで活性/非活性状態を操作できそうです。

1. 現在の状態から、入力不可な情報一覧を取得（→ `invalidNames`）
2. 式への入力可能な情報一覧から、順繰りにループ処理
   - 文字が `invalidNames` に含まれる場合は、そのボタン要素の `disabled` を有効化する
   - 逆に 含まれていない場合は、`disabled` を無効化する

```javascript:入力可能な一覧から、現状態で入力不可なものを選別できるように
class htmlCalculator extends Calculator{
  constructor(){
    // super キーワードで、親である Calculator の `_state` 等が利用可能に
    super();
  }

  refreshButtons(){
    const invalidNames = INVALID_INPUTS_BY_STATE[this._state];
    for(const name of VALID_INPUTS){
      // invalidNames.includes(name) で判定し、ボタンの活性・非活性操作を行う。
    }
  }
}
```

### 入力不可情報を使ったボタン操作

先程のコードで、入力可能な情報一覧から現状態で入力不可な文字を選別できるようできました。あとは、これにボタン要素を対応させれば完成です。

先程のループは、`VALID_INPUTS` を順繰りに回しました。なので `name` に入るのは、`'0'` や `'9'`、`'+'` や `'='` といった１文字になります。つまり先程のコードを言い換えると、**これを使って、ボタン操作をしなければならない**ということです。

#### 引数として、オブジェクトの形で入力ボタンを受け取る

まず、各種入力ボタンを引数として受け取るようにします。クラス内部で操作を行うためです。

入力ボタンは `"0", "1", "2", "3", "4", "5", "6", "7", "8", "9", ".", "+", "-", "*", "/", "="` の計16個ですが、どれを指しているか わかるようにしておく必要があります。

ここでは やや強引な手法ではありますが、**プロパティ名に入力値を取るオブジェクト** を作成します。こうすることで、先程作ったコード（ループ処理で入力不可な文字を選別）に組み込みやすくなります。

```javascript
// 事前に定義したボタン要素を、入力値をプロパティ名としたオブジェクト内に格納
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
```

:::message

プロパティ名に `*` や `=` を使っていますが、一応これでも動作はします。ただ気持ち悪く感じる場合は、別途置き換える関数等を用意しても良いかもしれません。

```javascript
// マッピング用のオブジェクトを作成
const translationMap = {
    '0': 'zero',
    // 〜 中略 〜
    '=': 'equal',
};

// 関数を定義して翻訳を実行
function translate(symbol) {
    return translationMap[symbol] || 'unknown';  // 未定義のシンボルには 'unknown' を返す
}
```

:::

#### 内部でボタン管理し、ループ処理で操作

`'0'` や `'9'`、`'+'` や `'='` の１文字で対応するボタンを呼び出すため、ここではオブジェクトの形で `_btn` を定義します。先程のオブジェクトと同じく、プロパティ名に各種文字を、値にボタン要素を割り振ることで、必要な処理を実現させています。

```javascript:式への入力ボタンを登録
class htmlCalculator extends Calculator{
  constructor(btns){
    super();

    // 操作用ボタン登録（引数となる btns にもプロパティ名を割り振っておく必要あり）
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

  refreshButtons(){
    const invalidNames = INVALID_INPUTS_BY_STATE[this._state];
    for(const name of VALID_INPUTS){
      // ボタン要素の disabled 管理に、真偽値判定をそのまま利用
      this._btn[name].disabled = invalidNames.includes(name);
    }
  }
}
```

これで、`this._btn['=']` でイコールボタン要素を呼び出せるようになりました。`HTMLButtonElement.disabled = true` で ボタン要素の `disabled` 属性を操作できることと組み合わせ、`refreshButtons()` を作成することができました。

### 各種メソッドへの適用

ここまでで、現状態に合わせてボタンの活性状態を切り替える `refreshButtons()` 関数ができました。これを呼び出せば自動的にその処理を行ってくれます。

では、**この処理が必要になるタイミング**とは どこでしょうか？それは、**式**（**状態**）**が変化**する下記の時です。

- `pushExpression()` で式を入力
- `reset()` で初期化
- `back()` で式の末尾を消去

そのため、これらの処理に `refreshButtons()` を行うよう**オーバーライド**していきます。サブクラスで親のメソッドを呼び出すためには、`super` キーワードを使います。

```javascript:refreshButtonsを各種タイミングに適用
class htmlCalculator extends Calculator{
  constructor(btns){
    super();
    // ボタン登録処理は、ここでは省略
  }

  pushExpression(input){
    // Calculator.pushExpression() は真偽値を返す。
    // そのため結果を一時的に保持し、refreshButtons() 終了後に返す。
    const result = super.pushExpression(input);
    this.refreshButtons();
    return result;
  }

  reset(){
    super.reset();
    this.refreshButtons();
  }

  back(){
    // Calculator.pushExpression() と同様、真偽値の一時的な保管を行っている。
    // ただし back() 処理の中身は、初期化からの pushExpression() による再計算である。
    // そのため、refreshButtons() はここでは不要。
    const result = super.back();
    return result;
  }

  refreshButtons(){ /* 省略 */ }
}
```

## `_label`に関連した処理

ここからは、新たに定義した情報 `_label` を `Calculator` での処理に埋め込んでいきます。

### `reset`処理

まずは初期化する機能 `reset()` です。これは `_label` も初期化するだけになります。

```javascript
class htmlCalculator extends Calculator{
  constructor(btns){
    super();
    this._label = '';
    // ボタン登録処理は、長いので省略
  }

  get label(){
    return this._label;
  }

  pushExpression(input){
    const result = super.pushExpression(input);
    this.refreshButtons();
    return result;
  }

  reset(){
    // ラベルも初期化
    this._label = '';
    super.reset();
    this.refreshButtons();
  }

  back(){
    const result = super.back();
    return result;
  }

  refreshButtons(){ /* 省略 */ }
}
```

### `back`処理

次にバックスペース処理ですが、こちらは少し厄介です。

なぜならこの処理は、「**一度リセット**して（末尾を除去した）式を再計算」するものだからです。リセット処理が入る関係上、`_label` 情報は空になってしまいます。

```javascript:Calculatorにあるback処理
back(){
  if(
    (this._state == CALC_STATE.Result) ||
    (this._state == CALC_STATE.Error) ||
    (this._expression.length === 0)
  ){
    return false;
  }
  const expr = this._expression.slice(0, -1);

  // ↓ ここでリセット処理が行われている
  this.reset();

  if(expr.length >= 1){
    for (var i = 0; i < expr.length; i++) {
      this.pushExpression(expr[i]);
    }
  }
  return true;
}
```

（計算式などの）**ラベル情報が、バックスペース処理で消えて良いケースというのはありません**。そのため、ここでは（状態等に関係なく一律に）ラベル情報を退避させておき、（リセット含む諸々の）処理後に再設定する形にしています。

```javascript
class htmlCalculator extends Calculator{
  constructor(btns){
    super();
    this._label = '';
    // ボタン登録処理は、長いので省略
  }

  get label(){
    return this._label;
  }

  pushExpression(input){
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
    // リセットされる前に、ラベル情報を保管
    const buf = this._label;
    const result = super.back();

    // 保管していたラベル情報を再設定
    this._label = buf;
    return result;
  }

  refreshButtons(){ /* 省略 */ }
}
```

### `pushExpression`処理

最後に、式への入力処理です。ここに関するラベル操作は、**状態によって変わります**。

#### ラベルを設定するケース

ラベルを設定するケースとしては、**計算式に Calculate 処理が行われ、式が計算結果へと更新された時**が挙げられます。

- 右辺への入力状態（ゼロ、整数、小数）から、演算子またはイコールが入力された時

#### ラベルを消去するケース

ラベルを消去するケースとしては、**これまでの計算過程を棄て、新規に計算式を作る**状況が挙げられます。

- `NaN` などのエラー状態（≒これ以上の続行が不可）から、初期化する時
- `=` で計算結果状態になり、そこから数値を入力した時（演算子の場合は、計算結果を使った継続処理となる）

#### 上記ケースを基にコード化

上記２つのケースをコード化します。

見るべき状態は、設定時は `CALC_STATE.OperandZero2`, `CALC_STATE.OperandInteger2`, `CALC_STATE.OperandDecimal2` の３種、消去時は `CALC_STATE.Result`, `CALC_STATE.Error` の２種になります。

そのため、`switch` 文を使いケース分けを行います。

```javascript
class htmlCalculator extends Calculator{
  constructor(btns){
    super();
    this._label = '';
    // ボタン登録処理は、長いので省略
  }

  get label(){
    return this._label;
  }

  pushExpression(input){
    switch (this._state){
      case CALC_STATE.OperandZero2:
      case CALC_STATE.OperandInteger2:
      case CALC_STATE.OperandDecimal2:
        // 計算処理が実行される場合（演算子、イコール入力時）にラベル取得
        if((isOperator(input)) || (input == '=')){
          this._label = this.expression;
        }
        break;
      case CALC_STATE.Result:
      case CALC_STATE.Error:
        // 計算処理を破棄する場合（数値、小数点入力時）にラベル除去
        if((!isOperator(input)) && (input != '=')){
          this._label = '';
        }
        break;
      // 他の状態に関しては処理不要、なので default文も必要ない
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

  refreshButtons(){ /* 省略 */ }
}
```

## まとめ

今回は、HTML要素と結びつけるためのサブクラス `htmlCalculator` を作成しました。そこでは ラベル情報やボタンの非活性化処理等の、新規となるものを加え入れながら、既存の機能にも `super` を使ったオーバーライドを施していきました。

![htmlCalculatorクラスのイメージ図。既存クラスを拡張し、各種ボタン（HTML要素）と結びつけている](/images/books/howto-create-calculator/html-calculator-01.png)

:::details 今回作成したコードまとめ（長いので格納しています）

```javascript
// Calculator を拡張したサブクラスの定義
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

// ボタン要素（事前に定義されているものとする）を、入力値をプロパティ名としたオブジェクト内に格納
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

// サブクラスをインスタンス化（引数として、ボタン要素郡を渡す）
const calculator = new htmlCalculator(buttons);
```

:::

次項は応用の最終段階となります。用意した[HTML文](./d2-prepare-html)と今回の `htmlCalculator` を結びつけることで、ブラウザにレンダリングされた要素で電卓を操作できるようにしていきます。
