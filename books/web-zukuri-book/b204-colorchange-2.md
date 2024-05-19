---
title: "📄B204：カラーモード変更(2)"
---

## このページについて

ここでは、カラーモード変更機能について、2回に分けて説明します。  
2回目の今回は、Javascript を使いスタイル適用に必要なクラス付与のコードを作成していきます。

## 前回までの内容について

これまでの内容についておさらいします。

### 必要とする機能

前回、必要な機能について下記のように定義しました。

1. チェックボックスにチェックを入れると、カラーモードを手動変更できるようになる
2. チェック無しの場合は、OS やブラウザの設定を優先する
3. 手動モード時になるとチェックボックスがもう1つ出現し、ライト/ダークモードを切り替えられる
4. OSやブラウザの設定に関わらず、手動モード時は必ずそちらの設定を優先する

### そのためのスタイル

そして、上記の内容を実現するための手法として、「`html` タグにクラス `is_lightMode`/`is_darkMode` を付与することで、より詳細度を上げたスタイルに上書きする」方法を採用しました。

```css:devUtilities.css
/* 単純に詳細度を上げたスタイルで上書きする */
:root.is_lightMode {
  --sc_txt_body: var(--sumi-900);
  /* 以下、セマンティックカラーに関するカスタムプロパティが続く */
}

:root.is_darkMode {
   --sc_txt_body: var(--white);
   /* 以下、ダークモード時のカスタムプロパティ */
}


/* style.css 側を考慮し、適切な形に上書きする */
@media (prefers-color-scheme: dark) {
  .is_lightMode .bl_accordion_summary::after {
    filter: invert(0);
  }
}

.is_darkMode .bl_accordion_summary::after {
  filter: invert(1);
}

@media (prefers-color-scheme: dark) {
  .is_lightMode .bl_searchForm_inputSelect::after {
    filter: invert(0);
  }
}

.is_darkMode .bl_searchForm_inputSelect::after {
  filter: invert(1);
}
```

ここまでで、上書きするためのクラス `is_lightMode`, `is_darkMode` を用意できれば、
`devUtilities.css` に記述したスタイルに上書きされるようになりました。
これによりOS設定を反映しながらも、必要に応じて手動でカラーモードを切り替えることが可能になっています。

### 機能実装に使うコントロール要素

以前に作成した `html` 文から、JavaScript で使うコントロール群は下記の３つです。

- `js_colorMode_controller`: コントロール群格納要素、表示/非表示を管理
- `js_darkMode`: トグルボタン、カラーモード「ライト↔ダーク」の切り替えを管理
- `js_colorMode_chkbox`: トグルボタン、機能有効化を管理

## Javascriptで実装すべき機能について

ここからは上記の内容を利用するために必要となる機能について考えていきます。

- 手動モード時は必ずクラス `is_lightMode`/`is_darkMode` のどちらかが `html` タグに含まれている状態を作る
- 逆に、解除時には上記のクラスが `html` タグに残っていてはいけない（完全に除去）
- 手動モード切り替えのチェックボックスの内容に合わせ、コントロール群格納要素の `hidden` 属性を切り替え、表示/非表示を管理
- カラーモードのチェックボックスの内容に合わせ、`html` タグのクラス（`is_lightMode`, `is_darkMode`）を管理
- 2つのチェックボックスの値は、同セッション中で維持される（ページ移動時、前の設定値を引き継ぐ）

### クラス付与、除去について

`html` タグは、 `document.documentElement` で参照することができます。`要素.classList` によるクラス管理と組み合わせると、`html` へのクラスのつけ外しは下記のコードで済みます。

```javascript
document.documentElement.classList.remove("除去するクラス名");
document.documentElement.classList.add("追加するクラス名");
```

### `hidden` 属性の切り替え

要素の `hidden` 属性の管理は、`要素.hidden = {true/false};` で行えます。

```javascript
Element.hidden = true;
```

今回は、手動モード時のみカラーモード切替のチェックボックスを表示させるのに使っています。

ここで注意なのが、`hidden` 属性がついた要素が必ずしも非表示になるとは限らないということです。`display` に関する指定が何も無い場合には、ブラウザはデフォルトで `display: none;` を適用します。一方、その要素が何らかの `display` プロパティを使用している場合、`hidden` 属性をつけただけでは 非表示にはなりません。

https://developer.mozilla.org/ja/docs/Web/HTML/Global_attributes/hidden

確実に要素を非表示にするためには、スタイルシート上で `hidden` 属性を混ぜたセレクタを使い `display: none;` を明確に指示してあげるのが良いでしょう。

```css:devUtilities.css
#js_colorMode_controller[hidden] {
  display: none;
}
```

### セッション中の設定維持について

ページ切り替えの度にカラーモードが元に戻ってしまうのは手間なので、設定を引き継ぐようにします。
維持する期間については永続でなく、セッション中だけで十分と考えます。

そのため Javascript の `sessionStorage` を使い、設定を維持するようにしていきます。

## 機能の実装

ここからは、上記の機能をコードに反映していきます。

### 要素の定義

まずは、下記の要素を変数に定義していきます。

- 機能有効化するチェックボックス `js_colorMode_chkbox`
- ダーク↔ライト間のカラーモード切替用のチェックボックス `js_darkMode`
- カラーモード切替のチェックボックスをラップする要素 `js_colorMode_controller`

```javascript
const colorModeChkbox = document.getElementById('js_colorMode_chkbox');
const darkModeChkbox = document.getElementById("js_darkMode");
const colorModeCtrl = document.getElementById('js_colorMode_controller');
```

### 初期設定

次に初期設定を考えていきます。
`sessionStorage` を使って設定の引き継ぎをしていますので、登録されていればそれを引き継ぎ、登録がなければデフォルト値を用意します。

```javascript
/**
 * セッション中に維持保存される、カラーモード設定情報
 *
 * @param {boolean} isShow - 機能が有効化されているか
 * @param {boolean} isDark - ダークモード設定か
 */
let currentColorState;
if (sessionStorage.getItem('currentColorState')){
  currentColorState = sessionStorage.getItem('currentColorState');
  currentColorState = JSON.parse(currentColorState);
} else {
  currentColorState = {
    isShow: false,
    isDark: false,
  };
}
```

### カラーモード切替機能

ここからは、カラーモードを管理する機能を作成していきます。  
必要な引数は、2つの `boolean` 型です。

1. `isShow` : カラーモードのチェックボックスを表示するか（≒有効化するか）
2. `isDark` : （手動時において）ダークモード設定か

大まかな処理の内容を、下記に記します。

- `isShow` が `true` の場合は下記を実行
  - `isDark` が `true` の場合は下記を実行
    - `html` タグから `is_lightMode` クラスを除去
    - `html` タグから `is_darkMode` クラスを付与
  - `isDark` が `false` の場合は下記を実行
    - `html` タグから `is_darkMode` クラスを除去
    - `html` タグから `is_lightMode` クラスを付与
- `isShow` が `false` の場合は下記を実行
  - コントロール要素に `hidden` 属性を追加
  - `html` タグから `is_lightMode`, `is_darkMode` クラスを除去
- 現在の表示状態と値をJSON形式にして、sessionStorage である `currentColorState` に保存

上記をコードに起こしたのが、下記になります。

```javascript
function changeColorMode(isShow, isDark) {
  if (isShow) {
    colorModeCtrl.hidden = false;
    if (isDark) {
      document.documentElement.classList.remove("is_lightMode");
      document.documentElement.classList.add("is_darkMode");
    } else {
      document.documentElement.classList.remove("is_darkMode");
      document.documentElement.classList.add("is_lightMode");
    }
  } else {
    colorModeCtrl.hidden = true;
    document.documentElement.classList.remove('is_lightMode', `is_darkMode`);
  }
  currentColorState.isShow = isShow;
  currentColorState.isDark = isDark;
  sessionStorage.setItem('currentColorState', JSON.stringify(currentColorState));
}
```

### 各種コントロールにイベント付与

次に、先程作成した機能をフォームのコントロールにひも付けしていきます。

- 「ダークモード」チェックボックス変更時は、`isDark` に自身の状態を渡す（`isShow` は `sessionStorage` を使用）
- 「カラーモード手動」チェックボックス変更時は、`isShow` に自身の状態を渡す（`isDark` は `sessionStorage` を使用）

```javascript
// 「ダークモード」チェックボックス
darkModeChkbox.addEventListener("change", (e) =>{
  changeColorMode(currentColorState.isShow, e.target.checked);
})

// 「カラーモード手動」チェックボックス
colorModeChkbox.addEventListener("change", (e) => {
  changeColorMode(e.target.checked, currentColorState.isDark);
})
```

### 初期状態の反映

最後に、初期状態を反映させる処理をさせます。

```javascript
darkModeChkbox.checked = currentColorState.isDark;
colorModeChkbox.checked = currentColorState.isShow;
changeColorMode(currentColorState.isShow, currentColorState.isDark);
```

## コード全文

```javascript:devUtils.js
/* ≡≡≡ ▀▄ DarkMode ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    htmlタグに `is_darkMode`/`is_lightMode` クラスをつけ外しすることで、カラーモードを制御する
    カラーモードの制御には、CSSのカスタムプロパティを使用しているので詳細はそちらを参照(devUtilities.css)
  ■ 備考
    完全にOSやブラウザの設定とは切り離し、強制的な上書き処理を行う（検証用）
    変更した情報は `sessionStorage` を使って保存され引き継ぐことができる
---------------------------------------------------------------------------- */

// 各種要素の取得
const colorModeChkbox = document.getElementById('js_colorMode_chkbox');
const darkModeChkbox = document.getElementById("js_darkMode");
const colorModeCtrl = document.getElementById('js_colorMode_controller');

/**
 * セッション中に維持保存される、カラーモード設定情報
 *
 * @param {boolean} isShow - 機能が有効化されているか
 * @param {boolean} isDark - ダークモード設定か
 */
let currentColorState;
if (sessionStorage.getItem('currentColorState')){
  currentColorState = sessionStorage.getItem('currentColorState');
  currentColorState = JSON.parse(currentColorState);
} else {
  currentColorState = {
    isShow: false,
    isDark: false,
  };
}

/**
 * カラーモード機能の有効無効を管理
 *
 * 有効の場合は isDark に応じてクラスを付与、向こうの場合はクラス除去
 *
 * @param {boolean} isShow - 機能が有効化されているか
 * @param {boolean} isDark - ダークモード設定か
 */
function changeColorMode(isShow, isDark) {
  if (isShow) {
    colorModeCtrl.hidden = false;
    if (isDark) {
      document.documentElement.classList.remove("is_lightMode");
      document.documentElement.classList.add("is_darkMode");
    } else {
      document.documentElement.classList.remove("is_darkMode");
      document.documentElement.classList.add("is_lightMode");
    }
  } else {
    colorModeCtrl.hidden = true;
    document.documentElement.classList.remove('is_lightMode', `is_darkMode`);
  }
  // 設定情報の更新
  currentColorState.isShow = isShow;
  currentColorState.isDark = isDark;
  sessionStorage.setItem('currentColorState', JSON.stringify(currentColorState));
}

// 「ダークモード」チェックボックス
darkModeChkbox.addEventListener("change", (e) =>{
  changeColorMode(currentColorState.isShow, e.target.checked);
})

// 「カラーモード手動」チェックボックス
colorModeChkbox.addEventListener("change", (e) => {
  changeColorMode(e.target.checked, currentColorState.isDark);
})

// 初期状態の反映
darkModeChkbox.checked = currentColorState.isDark;
colorModeChkbox.checked = currentColorState.isShow;
changeColorMode(currentColorState.isShow, currentColorState.isDark);
```

## まとめ

このページでは、カラーモード変更機能に関する Javascript コードについて説明しました。  
これで、1回目のスタイルと組み合わせることで、「基本は OS（ブラウザ）の状態を使用したカラーモードで表示し、検証時には強制的に切り替える」ことができるようになりました。
