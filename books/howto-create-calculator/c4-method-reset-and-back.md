---
title: "作成④：リセット＆バックスペース処理"
---

## 本項について

本項では機能ボタンとなる「リセット」「バックスペース」を扱っていきます。

## リセット処理

リセット処理は非常に単純です。`Calculator` が保持している式と状態を、初期状態へと変更するだけです。

```javascript
class Calculator {
  constructor() {
    this._expression = '';
    this._state = CALC_STATE.Start; // -> 0
  }

  pushExpression(input){
    // 省略
  }

  reset(){
    this._state = CALC_STATE.Start;
    this._expression = '';
  }

  back(){
    // 1文字消す処理
  }
}
```

## バックスペース処理

バックスペース処理については、[以前の項](./b3-implement-ideas#状態リセットからの再計算)にて**リセット処理後に、末尾文字を抜いた式で再計算**する手法を採用しました。このことから、処理の流れとしては下記のようになります。

1. 状態が計算結果（`CALC_STATE.Result`）でない、かつ 式に何かしらの情報がある場合に処理を行う
2. 現在の式から、末尾文字を抜いた部分文字列 `expr` を取得
3. リセット処理（≒式や状態を初期状態にする）
4. `expr` を1文字ずつ `pushExpression()` を使って再入力
5. 正常に完了したことを知らせるため `true` を返す（逆に、問題があれば `false` を返す）

```javascript
class Calculator {
  constructor() {
    this._expression = '';
    this._state = CALC_STATE.Start; // -> 0
  }

  pushExpression(input){
    // 省略
  }

  reset(){
    this._state = CALC_STATE.Start;
    this._expression = '';
  }

  back(){
    //リザルト状態でない、かつ 1文字以上の入力がある場合が処理対象
    if(
      (this._state == CALC_STATE.Result) ||
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

これで、バックスペース処理の実装ができました。

### バックスペース処理のハック的な操作

一応、`CALC_STATE.Result` 状態で `Calculator.back()` 処理はできないよう条件分岐を書いています。これは「計算結果に対し1字消す」という処理は適切ではないためです。ですが、これは `CALC_STATE.Result` 状態を動かすことで回避できることを意味します。

例えば、式 `100+100` に `=` を送って `200`（`CALC_STATE.Result`）状態にあるとします。この状態では `back()` 処理は動きません。

しかし `+` を送り `200+`（`CALC_STATE.Operator`）に遷移した後でなら、`back()` を実施して `200` という数値を操作することができます。つまりここから `back()` 処理で `20` と削ることができてしまうということです。

これは履歴管理すれば解決する問題ではありますが、このためだけに（複雑な）履歴情報を用意すべきかと言われると微妙なところだと考えます。
