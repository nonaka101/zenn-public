---
title: "📄B316：js_pageTop(2)"
---

## このページについて

本項は 前項で行った「ページトップに戻る」ボタンを使い、JavaScript による機能を実装していきます。

## 機能の定義

このボタンに付与したい機能は、次のようになります。

1. ボタンを押すと、ページ最上段までスクロールとフォーカスを移す
2. スクロール位置に応じて、`hidden` 属性をつけ外しする

2番に関しては、ボタンの表示/非表示の話になります。フェードイン/アウトのアニメーションについては、前項のスタイリングで定義済みです。なので、ここではその切り替えのための `hidden` 属性のみを考えていきます。

:::message

前項のフェード効果含め、**jQuery** を使えば簡略化することができます。が、WordPress で jQuery を使用するのは少し難しいため、これくらいならと自作したコードにとどめています。

:::

## 機能の実装

ここからは、上記の機能をコードに反映していきます。

### 要素の定義

最初に、要素の定義から行っていきます。

ボタン本体は `id="js_pageTop"` を使っているので、これを使います。

```javascript
const pageTopBtn = document.getElementById("js_pageTop");
```

次にフォーカス要素になります。まず前提として、フォーカス先要素（≒ページの初めにあるヘッダー要素）は このようになっているものとします。

```html:ヘッダーへのアンカー（ページ上、最初にフォーカスされるもの）
<body>
  <div class="ly_mainArea">
    <header class="ly_mainArea_header">
      <div class="bl_header">
        <a class="bl_header_siteTitle" href="./index.html" id="js_anchor_header">
          <!-- サイトのロゴとタイトル -->
        </a>
        <!-- /.bl_header_siteTitle -->
<!-- 〜〜〜 以下省略 〜〜〜 -->
```

それを踏まえ、フォーカス先要素（ヘッダー要素 `js_anchor_header`）を定義します。ここでは無名関数を使ってフォーカスする機能をまとめて定義しています。

```javascript
const pageTopFocusHeader = () => document.getElementById("js_anchor_header").focus();
```

:::message

アクセシビリティ上、スクロールだけでは不十分で**フォーカスの移動も必須**です。

マウスやタップ操作で検証してみると、フォーカス機能を実装しなくても問題ないように見えます。ただしフォーカスはドキュメント後方に残ったままです。なので例えばキーボードで Tab キーを押してみると、そちらに制御が戻ってしまいます。

:::

### ボタン押下時の処理

次に、ボタンを押した際の「ページ最上段までスクロール」と「ヘッダー要素にフォーカス」する処理を実装します。

```javascript
pageTopBtn.addEventListener("click", () =>{
  window.scroll({ top: 0, behavior: "smooth"});
  setTimeout(pageTopFocusHeader, 1000);
})
```

スクロール機能は、`window.scroll()` で制御しています。`behavior: "smooth"` を指定し、スクロールが働いていることをユーザーに伝わるようにしています。

フォーカス機能については、既に定義した `pageTopFocusHeader` で行っています。即時に実行してしまうと上で実装したスムーズスクロールの意味がなくなってしまうので、`setTimeout()` で 1秒の遅延を施しています。

### スクロール判定と処理

次にスクロール量に応じて表示/非表示を切り替える処理です。ここでは関数を作成し、イベント登録する方針でいきます。  
また、ページ読み込み時の初期状態ではボタンは非表示にしておきたいので、スクロールイベントに関わらず関数は１度実行するようにします。

大まかな流れは、下記のようになります。

```javascript
function changeOpacity() {
  // スクロール量が100を越えているか？
  if (window.scrollY > 100) {
    // 越えている場合、ボタンが非表示なら表示に切り替える（既に表示状態なら何もしない）
  } else {
    // 越えてない場合、ボタンが表示状態なら非表示に切り替える（既に非表示なら何もしない）
  }
}

// 上記の処理を初期状態で１度実行し、イベント登録しておく
changeOpacity();
window.addEventListener("scroll",() => changeOpacity());
```

前項で、`button` 要素の `hidden` 属性でフェードイン/アウトをするようスタイリングを施しました。なのでここで行うべき処理とは、`pageTopBtn` への `hidden` プロパティの切り替えです。

```javascript:ボタン要素のhidden属性を切り替え（制御はまだ不完全）
function changeOpacity() {
  if (window.scrollY > 100) {
    // 非表示状態の場合は、表示に変更
    pageTopBtn.hidden = false;
  } else {
    // 表示状態の場合は、非表示に変更
    pageTopBtn.hidden = true;
  }
}
```

属性処理は上記のコードでいいのですが、この状態はまだきちんと制御できていない状態です。表示/非表示に関する「現在の状態」を考慮していないので、このままですとスクロールが発生するたびに `hidden` 属性のつけ外し処理が発生してしまいます。

なので「現在のボタンの状態」を管理し、必要な状況下でのみ `hidden` 属性のつけ外し処理を行うようにします。  
`pageTopBtn` から都度取得するのも手間なので、`boolean` 型の変数 `pageTopIsShow` を用意し、そこで管理するようにします。

```javascript
// 初期状態を true にしているのは、初回の処理で必ず関数の else 文を実行させるため
let pageTopIsShow = true;

function changeOpacity() {
  if (window.scrollY > 100) {
    // ボタンが非表示なら表示に切り替える（既に表示状態なら何もしない）
    if (pageTopIsShow == false) {
      pageTopBtn.hidden = false;
      pageTopIsShow = true;
    }
  } else {
    // ボタンが表示状態なら非表示に切り替える（既に非表示なら何もしない）
    if (pageTopIsShow == true) {
      pageTopBtn.hidden = true;
      pageTopIsShow = false;
    }
  }
}
```

これで必要な機能を全て実装できました。

## 完成形

最後に、前項でやった内容を含め、全文を載せておきます。

:::details JavaScript

```javascript:common.js
/* ≡≡≡ ▀▄「ページトップに戻る」ボタン ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    画面右下に固定表示される、ページ上部に戻るためのボタン機能
  ■ 機能
    - ボタンを押下したら、ページトップに移動し、フォーカスをヘッダー部に移す
    - 下に一定値スクロールした場合に表示状態にする
---------------------------------------------------------------------------- */

// 各種要素の読み込み
const pageTopFocusHeader = () => document.getElementById("js_anchor_header").focus();
const pageTopBtn = document.getElementById("js_pageTop");

// ボタン押下後、トップにスクロールし、フォーカスをヘッダー部に移す
pageTopBtn.addEventListener("click", () =>{
  window.scroll({ top: 0, behavior: "smooth"});
  setTimeout(pageTopFocusHeader, 1000);
})

/**
 * ページトップボタンを表すべきかの状態を示す
 * @type {boolean}
 */
let pageTopIsShow = true;

/** 下に一定値スクロールした場合に「ページトップ」ボタンを表示、そうでなければ非表示に */
function changeOpacity() {
  if (window.scrollY > 100) {
    if (pageTopIsShow == false) {
      pageTopBtn.hidden = false;
      pageTopIsShow = true;
    }
  } else {
    if (pageTopIsShow == true) {
      pageTopBtn.hidden = true;
      pageTopIsShow = false;
    }
  }
}

// 初期状態を、現在のスクロール位置と合わせる（＝非表示）
changeOpacity();
window.addEventListener("scroll",() => changeOpacity());
```

:::

:::details HTML&CSS

```html
<button type="button" id="js_pageTop" aria-label="ページトップに戻る">
  <svg
    role="graphics-symbol img"
    aria-hidden="true"
    xmlns="http://www.w3.org/2000/svg"
    width="24"
    height="24"
    viewBox="0 0 16 16"
  >
    <path fill-rule="evenodd" d="M8 15a.5.5 0 0 0 .5-.5V2.707l3.146 3.147a.5.5 0 0 0
      .708-.708l-4-4a.5.5 0 0 0-.708 0l-4 4a.5.5 0 1 0 .708.708L7.5 2.707V14.5a.5.5 0 0 0 .5.5z">
    </path>
  </svg>
</button>
<!-- /#js_pagetop -->
```

```css:style.css
#js_pageTop {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
  color: var(--sea-900);
  cursor: pointer;
  background-color: #fff;
  border: 1px solid #0028b5;
  border-radius: 50%;
  transition: background-color 0.5s cubic-bezier(0.4, 0.4, 0, 1);
  animation-name: fade_in;
  animation-duration: 0.5s;
}

#js_pageTop:hover {
  background-color: #e7eefd;
  border-color: #0f41af;
}

@keyframes fade_in {
  0% {
    visibility: hidden;
    opacity: 0;
  }

  1% {
    visibility: visible;
    opacity: 0.01;
  }

  100% {
    visibility: visible;
    opacity: 1;
  }
}

#js_pageTop[hidden] {
  visibility: hidden;
  animation-name: fade_out;
  animation-duration: 0.5s;
}

@keyframes fade_out {
  0% {
    visibility: visible;
    opacity: 1;
  }

  99% {
    visibility: visible;
    opacity: 0.01;
  }

  100% {
    visibility: hidden;
    opacity: 0;
  }
}

/* デスクトップ画面では余裕があるので少し内側に寄せる */
@media (min-width: 960px) {
  #js_pageTop {
    right: 40px;
    bottom: 40px;
  }
}
```

:::

## まとめ

本項では、2回に渡り「ページトップに戻る」ボタンである `js_pageTop` を解説しました。

次項は WordPress 用のモジュール `wp_blockContents` について解説していきます。
