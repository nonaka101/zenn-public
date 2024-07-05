---
title: "作成③：四則演算処理"
---

## 本項について

前項で `Calculator` クラスを定義し、`pushExpression()` で式にを入力できるようしました。本項では、式を評価し 計算値に置き換えるための `calculate()` メソッドを定義していきます。

:::message

本項は、下記の制限を回避する意味合いが強い項となっています。これらが当てはまらない場合は、もっと簡素な手法で同じ結果を導き出せるはずです。

- 通常の四則演算処理だと、小数計算に丸め誤差が生じてしまう
- 上記に対し、`Decimal.js` などに代表される外部ライブラリで解決ができない

:::

## 四則演算処理

### これまでのおさらい

まずは、ここまでで わかっていることを おさらいしてみます。

- `Calculator` クラスは、`_expression` プロパティで**式を文字列の形で保持**しています
- 状態管理されているので、計算処理は適切なタイミング（両辺と演算子が揃った状態）で行われます
- 計算するタイミングで、文字列である式を「左辺」「演算子」「右辺」に分割します
- 分割は、**2文字目以降で最初にヒットする演算子記号**を目印として行います

### 式の分割

計算処理本体に移る前に、現時点で わかっている「式の分割」について見ていきましょう。

式（`1+1`, `0.123*456`, `-123/-0.456`）などに対し、下記の手順で分割していきます。

1. 文字列形式の式 `expression` を受け取る
2. `expression` を（文字列の）配列とみなし、2文字目以降（≒ 開始位置 1）から演算子記号を探索する
3. ヒットした位置 `opetatorIndex` に対し、下記の処理を行う
   - 左辺 `operandLeft` は、文字列の 0 番目〜 `operatorIndex` までの部分文字列
   - 演算子 `operator` は、文字列の `operatorIndex` 番目にある部分文字列
   - 右辺 `operandRight` は、文字列の `operatorIndex + 1` 番目以降の部分文字列
4. 上記 `operandLeft`, `operator`, `operandRight` を配列形式で返す

まず、演算子であるかを判定する関数を用意します。

```javascript:文字が演算子記号であるかを判定し、真偽値で返す関数
function isOperator(char) {
  return ['+', '-', '*', '/'].includes(char);
}
```

次に上記関数を使い、先程の手順を再現します。

```javascript
function splitExpression(expression) {
  let operatorIndex = 1; 

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
```

::::message

ループ処理しない方法として `indexOf()` を使った手法があります。ですが本項においては４種類ある演算子の関係上、利便性があまりありません。

右辺が負の数であるパターンを考えなければならないので、もし使うのであれば下記のようにしなければなりません。

```javascript
function splitExpression(expression) {
  const addIndex = expression.indexOf('+', 1);
  const subtractIndex = expression.indexOf('-', 1);
  const multiplyIndex = expression.indexOf('*', 1);
  const divisionIndex = expression.indexOf('/', 1);
  
  // 配列に格納し、ヒットしない結果（-1）を除外
  let arrayIndex = [addIndex, subtractIndex, multiplyIndex, divisionIndex].filter(i => i !== -1);
  
  // 最小値を算出
  let operatorIndex = Math.min(...arrayIndex);
  
  const operandLeft = expression.substring(0, operatorIndex);
  const operator = expression[operatorIndex];
  const operandRight = expression.substring(operatorIndex + 1);
  
  return [operandLeft, operator, operandRight];
}
```

これなら、式 `100+-200` の場合、`+` 位置は `4`、`-` は `5`、`*` と `/` は `-1` となり、`-1` を除いた最小値 `4` が演算子位置として算出できます。

計算量的には両者ともに `O(n)` になるようですが、`indexOf()` 手法は必ずヒットしない演算子が2つ以上発生し、固定のオーバーヘッドが生じます。それに対しループ手法は、途中で抜けることを考えると平均的な実行時間は短くなるのではと考えられます。

:::details 追記：正規表現を利用した場合

```javascript
function splitExpression(expression) {
  const operatorIndex = expression.match(/^.{1,}?([\+|\-|\*|\/])/gui)[0].length -1;

  const operandLeft = expression.substring(0, operatorIndex);
  const operator = expression[operatorIndex];
  const operandRight = expression.substring(operatorIndex + 1);
  
  return [operandLeft, operator, operandRight];
}
```

正規表現を使って 1行で位置を取得する方法です。

`operatorIndex` は、式 `expression` に対し、「行頭1文字」〜「以降にヒットする演算子記号」を、非貪欲的な形で**ヒットした文字列を配列形式**に取得しています。例えば式が `'-102*-1'` なら、`match()` まで返すのは配列 `["-102*"]` です。  
（※ `g` フラグを外すと　`["-102*", "*"]`）

上記の配列の `[0]` にあるのは、「1文字目から欲しい演算子記号」までの文字列なので、これ自体の長さにインデックス調整用に 1 を引けば、目的とする数値を取得できます。

`'-102*-1'.match(/^.{1,}?([\+|\-|\*|\/])/gui)[0].length -1` なら、`4` になるというわけです。

置き換えることはできるのですが、後半の単元で `isOperator()` を流用したりしているので、このままにしたいと思います。

:::

::::

### 計算処理

演算子位置は割り出せたので、次は本体となる計算処理です。ただし、ここでは**小数計算での丸め誤差**について考えなければならない関係から、**文字列形式を使った手法**を取る必要があります。

#### 丸め誤差について

下記は、ブラウザのコンソールを使って整数計算と小数計算を行った結果です。

```javascript:整数計算と小数計算
1+1;   // -> 2
0.1*3; // -> 0.30000000000000004
```

このように、JavaScript は整数での計算は問題ないのですが、小数計算などでは**非常に小さな誤差**が生じることがあります。これを**丸め誤差**といいます。

必要な小数桁数がはっきりしている場合は `toFixed()` で丸めることで解消できます。また 10進数で計算する `Decimal.js` を使うことでも、この問題を回避できるでしょう。ただし今回は、別の手法を使って この問題に対処していきます。

#### 文字列形式での計算

今回 対処する方法は、**小数含む計算式を強制的に整数に変換し、計算処理後に元に戻す**というものです。

例えば `0.1*0.1` は、答え自体は `0.01` になりますが小数計算は避けなければなりません。  
そこで両辺に `10` を掛けることで `1*1` という整数形式での計算式に変換します。そして、その結果 `1` に対し、変換処理を整合化（この場合 `10*10 = 100` で割る）することで、`0.01` という適切な計算結果にするというイメージです。

ただ、最初に行う変換（`0.1` に `10` をかけて整数化）を演算でやってしまうこと自体が、丸め誤差を生じかねません。そこで本項では、`0.1` を**文字列とみなし**、`.` の位置を文字列操作することで整数化を実現させます。

##### 小数点位置の算出

下記が、小数点位置を求めるコードです。

```javascript
function getDecimalPosition(value){
  const strVal = String(value);
  if(strVal.indexOf('.') !== -1){
    return ((strVal.length - 1) - strVal.indexOf('.'));
  }
  return 0;
}
```

`getDecimalPosition('0.123')` で `3`、`getDecimalPosition('123.4')` で `1` を返します。整数部である前方でなく、**小数部である後方を基準に、どこまでが範囲かを表している**わけです。

更にこれを利用して、整数化する際に必要な**10の累乗数**`n`（≒ `10**n`）も割り出せるようになりました。例えば `getDecimalPosition('0.123')` で `3` を導きだせることから、`0.123` を整数化するには `10**3`（＝`1000`）を掛ける必要がある、といった具合です。

##### 掛け算

上記の関数を使い、掛け算を実装します。関数 `multiplication()` に渡すのは、文字列形式の 左辺 `x` と 右辺 `y` です。

```javascript
const multiplication = (x, y) => {
  const z = 10 ** (getDecimalPosition(x) + getDecimalPosition(y));
  x = (x + '').replace('.', '');
  y = (y + '').replace('.', '');
  return (x * y) / z;
};
```

[先程の例](#文字列形式での計算)（`0.1*0.1` では、両辺に 10 をかけて計算し、最後に 100 で割る）を形にしたものになります。ただし実際には、整数化処理は `replace()` で `.` を除去する形に置き換わっています。

##### 足し算

足し算については、掛け算の関数も踏まえて作成することになります。

```javascript
const addition = (x, y) => {
  const z = 10 ** Math.max(getDecimalPosition(x), getDecimalPosition(y));
  return (multiplication(x, z) + multiplication(y, z)) / z;
};
```

小数位置の大きい方を使った10の累乗（`z`）で両辺を掛け算して整数化し、計算結果を `z` で割ることで元に戻しています。

##### 引き算

引き算に関しては、足し算とほとんど同じ処理になっています。

```javascript
const subtract = (x, y) => {
  const z = 10 ** Math.max(getDecimalPosition(x), getDecimalPosition(y));
  return (multiplication(x, z) - multiplication(y, z)) / z;
};

```

##### 割り算

割り算に関しては 両辺を整数化して割ることで、小数の場合と同じ結果を導き出せます。

```javascript
const division = (x, y) => {
  const z = 10 ** Math.max(getDecimalPosition(x), getDecimalPosition(y));
  return multiplication(x, z) / multiplication(y, z);
}
```

## `Calculator` クラスへの実装

四則演算処理用の関数が完成しました。これらクラス外に置いた関数を使って、`Calculator` クラスの `calculate()` メソッドを作成していきます。

### 演算方法による分岐

演算処理 `calculator()` は、`pushExpression()` 内の適切なタイミング（式に両辺と演算子が揃って計算可能な状態）で呼び出されます。計算処理を実行し、式を計算結果に置き換えるのが このメソッドの役割です。

まず最初にすることは、下記の処理です。

1. 式を「左辺」「演算子」「右辺」に分割
2. 計算結果 `calcResult` を用意
3. 演算子の内容に応じて計算処理を施し、`calcResult` に格納
4. `calcResult` を返す

```javascript
class Calculator {
  constructor() {
    this._expression = '';
    this._state = CALC_STATE.Start; // -> 0
  }

  pushExpression(input){
    // 省略
  }

  calculate() {
    let [strOpeLeft, operator, strOpeRight] = splitExpression(this._expression);
    let calcResult = 0;
    switch(operator){
      case '+':
        break;
      case '-':
        break;
      case '*':
        break;
      case '/':
        break;
      default:
    }
    return calcResult;
  }
}
```

### 各種演算処理

本項前半で用意した各種演算処理を用いて、`switch` 文を埋めていきます。その際、下記のようにしておきます。

- 計算処理に失敗した場合は、`calcResult` に `NaN` を格納
- 不明な演算子が検出された場合も、`calcResult` に `NaN` を格納

```javascript
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
```
