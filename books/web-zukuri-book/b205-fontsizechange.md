---
title: "📄B205：フォントサイズ変更"
---

## このページについて

ここでは、「フォントサイズ変更」用のチェックボックスとスライダーにつける機能とそのコードについて説明します。  

機能実装に使うコントロール要素については、以前に作成した `html` 文から下記の４つになります。

- `js_fontSize_controller`: コントロール群格納要素、表示/非表示を管理
- `js_fontSize_range`: スライダー、フォントサイズ値を管理
- `js_fontSize_result`: アウトプット、フォントサイズ値の表示
- `js_fontSize_chkbox`: トグルボタン、機能有効化を管理

## 機能の定義

フォントサイズ変更に付与したい機能は、次のようになります。

1. チェックボックスにチェックを入れると、フォントサイズ（ルート）を調整できるようになる
2. チェック無しの場合は OS やブラウザの設定を優先する
3. スライダーでデフォルト値 16px から、最小値 10px、最大値 32px まで調整が可能
4. 同セッション中は、設定を維持する

これをどう達成するかについて、まずは考えていきます。

### ルートフォントサイズについて

`rem` は `html` 要素に対して相対的な値となり、基本的には `1rem = 16px` となります。
これは `html` 要素に `font-size` に関する特段のスタイリングがされていない場合、ブラウザやOSの設定値が適用されます。
言い換えると、`html` 要素に `font-size: **px;` というスタイルを上書きすれば、ページ内の `rem` の大きさを調整することができます。

そのため、ここでは Javascript を使い、`html` 要素に対しスタイルを上書きする処理を使って機能を実現していきます。

![フォントサイズを32に手動設定した状態](/images/books/web-zukuri/about-font-size-02.png)

![開発ツールでマークアップを確認すると、htmlタグにスタイルが上書きされていることが確認できる](/images/books/web-zukuri/about-font-size-03.png)

### セッション中の設定維持について

フォントサイズを大きく設定している人と同じ状況を作るため、ページ切り替えの際にフォントサイズが元に戻らないよう設定を引き継ぐようにします。
維持する期間については永続でなく、セッション中だけで十分と考えます。

そのため Javascript の `sessionStorage` を使い、設定を維持するようにしていきます。

## 機能の実装

ここからは、上記の機能をコードに反映していきます。

### 要素の定義

まずは、下記の要素を変数に定義していきます。

- 機能有効化するチェックボックス `js_fontSize_chkbox`
- 有効時に表示されるフォーム `js_fontSize_controller`
- フォントサイズ調整のスライダー `js_fontSize_range`
- 現在の値を出力するアウトプット `js_fontSize_result`

```javascript
const fontSizeChkbox = document.getElementById('js_fontSize_chkbox');
const fontSizeCtrl = document.getElementById('js_fontSize_controller');
const fontSizeRange = document.getElementById('js_fontSize_range');
const fontSizeResult = document.getElementById('js_fontSize_result');
```

### 設定の引継ぎ、もしくは初期化

次に初期設定を考えていきます。
`sessionStorage` を使って設定の引き継ぎをしていますので、登録されていればそれを引き継ぎ、登録がなければデフォルト値を用意します。

```javascript
/**
 * セッション中に維持保存される、フォントサイズ設定情報
 *
 * @param {boolean} isShow - 機能が有効化されているか
 * @param {int} size - px単位のフォントサイズ
 */
let currentFontState;

if (sessionStorage.getItem('currentFontState')){
  currentFontState = sessionStorage.getItem('currentFontState');
  currentFontState = JSON.parse(currentFontState);
} else {
  currentFontState = {
    isShow: false,
    size: 16,
  };
}
```

### コントロール値の設定

`currentFontSize` から、現在の設定値を反映します

```javascript
fontSizeRange.value = currentFontState.size;
fontSizeResult.textContent = currentFontState.size;
```

### フォントサイズ有効化の機能

ここからは、フォントサイズ変更を有効化した際の機能を実装します。  
必要な引数は、`isShow` （boolean型で有効化するかどうか）を使います。

大まかな処理の内容は、下記のようになります。

- `isShow` が `true` の場合は下記を実行
  - コントロール要素の `hidden` 属性を除去
  - `<html>` 要素に `font-size` のスタイルを追加し、値を `fontSizeRange.value`(px) に設定
- `isShow` が `false` の場合は下記を実行
  - コントロール要素に `hidden` 属性を追加
  - `<html>` 要素の `font-size` スタイルを除去
- 現在の表示状態と値をJSON形式にして、sessionStorage である `currentFontState` に保存

上記をコードに起こしたのが、下記になります。

```javascript
function changeFontSize(isShow){
  let val = fontSizeRange.value;

  // 状態の反映
  if(isShow===true){
    fontSizeCtrl.hidden = false;
    document.documentElement.style.fontSize = val + 'px';
  } else {
    fontSizeCtrl.hidden = true;
    document.documentElement.style.fontSize = null;
  }

  // 設定情報の更新
  currentFontState.isShow = isShow;
  currentFontState.size = val;
  sessionStorage.setItem('currentFontState', JSON.stringify(currentFontState));
}
```

### 各種コントロールにイベント付与

次に、先程作成した機能をフォームのコントロールにひも付けしていきます。

- スライダー変更時は、`output`要素に反映させてから実行
- チェックボックス変更時は、その値を元に実行

```javascript
// スライダー
fontSizeRange.addEventListener('input', (e) => {
  fontSizeResult.textContent = e.target.value;
  changeFontSize(true);
})

// チェックボックス
fontSizeChkbox.addEventListener('change', (e) => {
  changeFontSize(e.target.checked);
})
```

### 初期状態の反映

最後に、初期状態を反映させる処理をさせます。

```javascript
fontSizeChkbox.checked = currentFontState.isShow;
changeFontSize(fontSizeChkbox.checked);
```

## コード全文

```javascript:devUtils.js
/* ≡≡≡ ▀▄ FontSizeChanger ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    スタイリングに `rem` と `px` を併用しているので、ルートのフォントサイズの変化による影響を見るためのもの
    内部的にはスライダーの値（10px〜32px）を、htmlタグのスタイル `font-size` に適用することで処理している
  ■ 備考
    OS（ブラウザ）の設定に反応しているかを確認できるように、チェックボックスで機能の有効無効を管理している
    未チェック状態では、OS（ブラウザ）側のフォントサイズが適用される
---------------------------------------------------------------------------- */

// 各種要素の取得
const fontSizeChkbox = document.getElementById('js_fontSize_chkbox');
const fontSizeCtrl = document.getElementById('js_fontSize_controller');
const fontSizeRange = document.getElementById('js_fontSize_range');
const fontSizeResult = document.getElementById('js_fontSize_result');

/**
 * セッション中に維持保存される、フォントサイズ設定情報
 *
 * @param {boolean} isShow - 機能が有効化されているか
 * @param {int} size - px単位のフォントサイズ
 */
let currentFontState;
if (sessionStorage.getItem('currentFontState')){
  currentFontState = sessionStorage.getItem('currentFontState');
  currentFontState = JSON.parse(currentFontState);
} else {
  currentFontState = {
    isShow: false,
    size: 16,
  };
}

// フォント設定情報からスライダー部のコントロール値を設定
fontSizeRange.value = currentFontState.size;
fontSizeResult.textContent = currentFontState.size;

/**
 * フォントサイズ変更機能の有効無効を管理
 *
 * 機能：スライドレンジの表示、ルートフォントサイズの制御（レンジの値、OS側の設定）、フォント設定情報の更新
 *
 * @param {boolean} isShow - 機能が有効化されているか
 */
function changeFontSize(isShow){
  let val = fontSizeRange.value;
  if(isShow===true){
    fontSizeCtrl.hidden = false;
    document.documentElement.style.fontSize = val + 'px';
  } else {
    fontSizeCtrl.hidden = true;
    document.documentElement.style.fontSize = null;
  }
  // 設定情報の更新
  currentFontState.isShow = isShow;
  currentFontState.size = val;
  sessionStorage.setItem('currentFontState', JSON.stringify(currentFontState));
}

// range を変更すると、result 値のサイズでルートフォントサイズが変更
fontSizeRange.addEventListener('input', (e) => {
  fontSizeResult.textContent = e.target.value;
  changeFontSize(true);
})

// chkbox を変更すると、機能の有効無効を切り替え
fontSizeChkbox.addEventListener('change', (e) => {
  changeFontSize(e.target.checked);
})

// chkbox を設定情報の内容に更新し、`changeFontSize()` を通す
fontSizeChkbox.checked = currentFontState.isShow;
changeFontSize(fontSizeChkbox.checked);
```

## まとめ

このページでは、「フォントサイズ変更」機能について説明しました。

これで開発ツールに関する機能は全て実装できました。全体に関するスタイルについても解説できたので、ここからは実際にサイトに表示していく要素を扱っていくことになります。

次節では、サイト上で扱うコンポーネント要素について１つずつ説明していきます。
