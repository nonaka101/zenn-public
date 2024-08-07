---
title: "概要②：前提となる機能や条件について"
---

## 本項について

本項では、これから作成する電卓が どのようなものにするかを定義していきます。

## 電卓の機能について

本項で扱う電卓とは、下図のようなボタン配列を持ちます。

![電卓のボタン配列](/images/books/howto-create-calculator/button-pattern1.png)

下記では、それぞれのボタンの内容について、簡単に説明していきます。

:::message

上記ボタンの内「コピー」と「閉じる」ボタンに関しては、説明を除外します。本書で扱う範囲からは外れてしまうためです。

:::

### 数値ボタン

![0から9までのボタン集](/images/books/howto-create-calculator/button-0-9.png)

数値を入力するもので、右辺または左辺に使用されます。

### 小数点とイコール ボタン

![小数点ボタンとイコールボタン](/images/books/howto-create-calculator/button-decimal-equal.png)

小数点ボタンは、数値を小数に変えるために使用します。イコールボタンは計算結果を表示します。

### 各種演算子ボタン（負符号ボタン）

![＋, -, *, /](/images/books/howto-create-calculator/button-operators.png)

右辺と左辺を結びつける演算子として扱います。ただし `-` だけは、負数を表すマイナス記号ともなりうるとします。

### Functionボタン

![全消去ボタンと、1字消去ボタン](/images/books/howto-create-calculator/button-reset-backspace.png)

「全消去ボタン」は、入力内容にかかわらず初期状態へと戻す処理を担います。「1字消去ボタン」は、入力式の後ろ1文字を消去します。

## 作成の際の条件について

本書では、電卓の作成に関して下記の条件を設けます。

- `-` は演算子にも、負符号にも使われるものとする
- [上記](#電卓の機能について)で説明したボタンのみを使用する（メモリ機能等は設けない）
- [上記](#電卓の機能について)で説明した、バックスペース機能やリセット機能が動作できるようにする
- 計算結果の扱いについては、下記のような動作をする
  - 式が `1+1` の状態で `=` が入力された場合、計算結果 `2` が表示される
  - 式が `1+1` の状態で `+` などの演算子が入力された場合、`=` を押した後に入力されたものとみなし、`2+` となる
- 1つの計算式に演算子は1つまでとする
  - 例えば `1+1` に `+` を入力した場合、`2+` の状態となる
  - ただし `-1--1` のように、式には `-` が 3つまで入り得る（これは `(-1) - (-1)` になる）

また、JavaScript の問題として「丸め誤差」があります。これは浮動小数点の計算が2進数で行われている関係上、表せない近似値になった際に生じるものです。

処置としては `.toFixed()` を使って丸めたり、`Decimal.js` のような外部ライブラリを使用することが多いです。ですが、本書では丸めず、そして組み込みのものだけで対処することを目指していきます。
