---
title: "📄B315：ly_dialog(2)"
---

## このページについて

前項では `ly_dialog` の基本構造となる、`dialog` 要素を使ったHTMLと JavaScript を作成しました。

2回目となる本項では、スタイルを中心に、`ly_dialog` のラッパーと本体要素を作成していきます。

## 構造の定義

前項のおさらいになりますが、ダイアログに必要な要件は下記のようになっていました。

- ボタン押下により、メニューを展開する
  - 逆に言えば、ボタンを押さない限りメニューは格納状態（≒スクリーンリーダーによる読み上げ対象からは外れる）
  - 展開/格納状態がスクリーンリーダー操作者に伝わる
- ダイアログ展開時は**モーダル状態**となる
  - ダイアログメニューを画面最前面に表示
  - 中の要素が多ければ、縦方向へのスクロールで移動できる
  - 一方で、ダイアログ外の要素による縦スクロールは制限する
  - フォーカスは制御され、ダイアログ外にフォーカスが移らない
  - ダイアログは、下記により閉じることができる
    - ダイアログ内に用意した「閉じる」ボタン
    - Escキー による操作
    - ダイアログ要素の範囲外をクリック

これらの殆どは、`dialog` 要素を使うことで既に解決できています。`dialog` 要素と JavaScript を使えば、下記の要件は達成することができます。

- 展開/格納状態を管理
- 最前面に表示
- フォーカスの制限
- （複数の）閉じる機能

一方で、スクロール管理などのスタイルに関わる要件は、CSS側で解決していく必要があります。本項ではこのスタイル部を中心に勧めていくことになります。

## 作成

作成に当たり、「展開ボタン」と「ダイアログ本体」に分けて解説していきます。

### 展開ボタン

まずは展開ボタンからです。前項では下記の所まで作成していました。

```html
<button type="button" aria-controls="menu" id="menu-button">メニュー</button>
```

この状態は何らスタイリングを行っていないので、単純なボタンとなります。ここを、SVGを使いラベル付きアイコンの形にしていきたいと思います。

![ボタンアイコン、メニューというテキストの上にハンバーガーメニューのアイコンが挿入されている](/images/books/web-zukuri/bl_header_btnIcon-01.png)
*目標とする形*

#### SVGの挿入

まずはSVGを付与します。カラーモードで色を変えるため、`≡` マークのSVGをインラインで挿入します。また、スタイリングするためのクラス（`bl_header_btnIcon`）を付与しておきます。  
（※ `bl_header` に関しては、**B401：フロントページ(3)ヘッダー**の節で細かく取り扱っていきます）

```html
<button type="button" aria-controls="menu" id="menu-button" class="bl_header_btnIcon">
  <svg 
    role="graphics-symbol img"
    aria-hidden="true" 
    xmlns="http://www.w3.org/2000/svg"
    viewBox="0 0 24 24" width="24" height="24"
  >
    <path fill-rule="evenodd" clip-rule="evenodd"
      d="M21 5.5H3V7H21V5.5ZM21 11.2998H3V12.7998H21V11.2998ZM3 17H21V18.5H3V17Z"></path>
  </svg>
  メニュー
</button>
<!-- /.bl_header_btnIcon -->
```

SVGタグの `role="graphical-symbol img"` でアイコン（`graphical-symbol` が認識できない場合は `img`）であることを伝えていますが、`aria-hidden="true"` でAOM上からは消しています。これは、`button` のテキストノードに「メニュー」があるので、スクリーンリーダーはSVGなしで「メニュー ボタン」と読み上げてくれるからです。

:::message

このあたりの考えについては、**Z000：附則**の章で取り扱いたいと思っています。

:::

#### アイコンボタンのスタイリング

次にアイコンボタンのスタイリングを行います。主な要件は下記のとおりです。

- SVGとラベルが縦方向に中央揃い
- ボタンであることが分かるよう、カーソルはポインターに
- 色についてはSVGとテキスト両者で、セマンティックカラーを使用
- 全体のサイズが、縦横 `48px`
  - SVGは `24px`（タグの属性で定義済み）
  - ラベルの `font-size` は `10px` に
  - 両者間の `gap` は `2px`
  - 余白関係は `0`

```css:style.css
.bl_header_btnIcon {
  display: flex;
  flex-direction: column;
  gap: 2px;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  padding: 0;
  margin: 0;
  font-size: 10px;
  color: var(--sc_txt_body);
  text-decoration: none;
  cursor: pointer;
  background: none;
  border: none;
}
```

:::message

ここでは他と違い、`rem` ではなく `px` による絶対単位による指定を行っています。これは、このボタンを格納するヘッダー `bl_header` の要件によるものです。

ヘッダー要素は、フォントサイズが大きくなっても画面占有率を上げないよう心がけています。ユーザーにとって見やすくするための操作なのに、ヘッダーが大きくなってコンテンツの閲覧に邪魔になるようではいけないからです。なのでヘッダー部に関しては、フォントサイズ変更に影響を受けないよう、絶対単位の `px` を使用しています。

:::

### ダイアログ本体

次はダイアログ本体です。こちらは全体を構成する「ダイアログ」と、ヘッダー部の「メニューヘッダー」に分けて説明していきます。

![ダイアログのスクリーンショット、本体とメニューヘッダーの それぞれに赤枠がついている](/images/books/web-zukuri/ly_dialog-05.png)

:::message

画像からわかるように、ダイアログ内のコンテンツは**B306：bl_tagMenu**を使用しています。つまりここで行うスタイリングとは、このコンテンツを格納するためのレイアウトを指しています。

:::

#### ダイアログ

##### `ly_dialog_wrapper`

最初にラッパー要素となる `ly_dialog_wrapper` を作成します。

```html
<dialog class="ly_dialog_wrapper" id="menu">
  <!-- ダイアログ本体 -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

`id="menu"` をつけているのは、JavaScriptでダイアログの開閉（`Element.showModal()`）を行うためです。

これはダイアログ本体が入るラッパー要素として扱います。その要件は、下記のとおりです。

1. ダイアログ格納中は、表示させない
2. ダイアログ展開中は、後ろにあるページをスクロールさせない
3. 画面全体を覆う形
4. （ダイアログ本体を除く）画面全体にブラーをかけ、本体を目立たせる

まずは1番から見ていきましょう。これは、`dialog` 要素の `open` 属性を使って、`display` を調整します。

```css
.ly_dialog_wrapper:not([open]) {
  display: none;
}
```

ここに関しては、ブラウザがデフォルトで `display: none;` として扱ってくれるので、なくても大きな問題はありません。ただ古いブラウザで `dialog` が対応していないような場合に、展開/格納関係なく表示されたことがあったので、手動で設定をつけています。

次に2番の「展開中は後ろにあるページのスクロールを禁止する」です。これも `open` 属性を使って展開/格納状態を管理します。スクロールを禁止するのはメインページの全体なので、`html` をセレクタにします。これらを `has()` で組み合わせ、「`open` 状態の `dialog` 要素を持つ `html` 要素は、スクロールを禁ずる」といった書き方で制御します。

```css
html:has(dialog[open]) {
  overflow: hidden;
}
```

次に3番の「画面全体を覆う形」ですが、ここは詳細度の関係で少し複雑になってきます。まずは**ダメな例**として、普通に画面全体を覆う形のスタイルを書いてみます。

```css
.ly_dialog_wrapper {
  position: fixed;
  top: 0;
  left: 0;
  padding: 0;
  border: none;
  /* 下記のスタイルはうまく適用できない */
  width: 100dvw;
  height: 100dvh;
}
```

`width: 100dvw;`, `height: 100dvh;` として、`position: fixed;` で固定、開始位置を `top: 0;`, `left: 0;` としているので、一見するとこれでうまく行きそうに見えます。ですが、結果としては幅や高さは画面全体を使わず、余白が生じてしまいます。

![上記のスタイル適用時の、ダイアログを展開した図](/images/books/web-zukuri/dialog_style_probrem01.png)
*デフォルトスタイルによって四隅に余白が生じている（ブラーがかかっている箇所）*

これは、ブラウザに備わっているデフォルトのスタイル（例：`dialog:-internal-dialog-in-top-layer`）が影響しています。このデフォルトスタイルを見てみると、`max-width` や `max-height` が入っており、そこに `em` 単位が使われています。

上図を見る限り 余白は大した広さでもないので、デフォルトのスタイルを適用させたままでも良さそうに見えます。ですが `em` を使っている関係上、ルートフォントが大きいと余白も大きくなってしまいます。小画面の場合だと折り返しが生じやすくなってしまう可能性もあり、本テーマではデフォルトスタイルを上書きすることにしました。

![上図からフォントサイズやズーム機能を使った状態、余白が広がりダイアログが圧迫されている](/images/books/web-zukuri/dialog_style_probrem02.png)
*ルートフォントを 16px → 32px にした状態*

デフォルトスタイルを上書きするために、`max-width` や `max-height` の規程を追加します。

```css
.ly_dialog_wrapper {
  position: fixed;
  top: 0;
  left: 0;
  width: 100dvw;
  max-width: 100dvw;
  height: 100dvh;
  max-height: 100dvh;
  padding: 0;
  border: none;
}
```

最後に4番「中の本体を目立たせるため、画面全体にブラーをかける」です。これは疑似要素 `::backdrop` を使います。

```css
.ly_dialog_wrapper::backdrop {
  backdrop-filter: blur(4px);
}
```

:::message

現段階では、ダイアログは画面全体を使うようなっているため、ここの内容は意味を持たない状態です。

:::

これでラッパー要素は完成です。

##### `ly_dialog`

次に、ダイアログ本体の要素を見ていきます。`div` 要素を使い、ベース名となる `ly_dialog` を付与します。

```html
<dialog class="ly_dialog_wrapper" id="menu">
  <div class="ly_dialog">
    <!-- コンテンツ -->
  </div>
  <!-- /.ly_dialog -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

:::message

コンテンツとラッパーの間に `div` を挟んでいるのは、役割を分担することで設計変更をしやすくしておく意図があります。`ly_dialog_wrapper` 側でモーダルダイアログ全体の管理をし、`ly_dialog` 側は中に入るコンテンツの管理をさせることで、設変時に意図せぬ挙動を防ぎやすく、また問題の切り分けも容易になります。

:::

次にスタイルです。

- （画面全体を使っているので、中のコンテンツに対し）上下左右の余白を設ける
- ダイアログ側のスクロールは許可
- 背景と枠線はセマンティックカラーを使用
- 中の要素間で、上下に `1rem` の余白を設ける
  - ただし *最初の要素の上* と *最後の要素の下* に関しては不要

```css
.ly_dialog {
  padding: 16px;
  overflow-y: auto;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
}

.ly_dialog > * {
  margin-top: 1rem;
  margin-bottom: 1rem;
}

.ly_dialog > *:first-child {
  margin-top: 0;
}

.ly_dialog > *:last-child {
  margin-bottom: 0;
}
```

「画面全体を使っているのに枠線はいらないのでは？」と思われるかもしれません。これはスクロール可能かをユーザーに伝える意図があります。下図はダイアログの上端と下端をそれぞれ映した図です。枠線は赤色に変えて強調しています。

![スマホでダイアログの上端と下端をそれぞれ映したイメージ図、上端では下を覗く方向で枠線があり、下端では上を除く方向で枠線が存在している](/images/books/web-zukuri/ly_dialog-06.png)

このように四隅の一方向だけ枠線が存在しないことで、ユーザーに対し その方向へスクロールできることを示す意図があります。

##### 閉じるボタン

最後に、前項で説明した「（ダイアログ内で）自身を閉じるボタン」を用意します。`class="el_btn el_btn__primary"` については**B305：el_btn**、`onclick="closeDialog()"` については前項を参照してください。

```html
<dialog class="ly_dialog_wrapper" id="menu">
  <div class="ly_dialog">
    <!-- コンテンツ -->
    <button class="el_btn el_btn__primary" type="button" onclick="closeDialog()">メニューを閉じる</button>
  </div>
  <!-- /.ly_dialog -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

## 完成形

最後に、ここまでの完成形をまとめておきます。

:::details ボタン

```html
<button type="button" aria-controls="menu" id="menu-button" class="bl_header_btnIcon">
  <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24"
    height="24">
    <path fill-rule="evenodd" clip-rule="evenodd"
      d="M21 5.5H3V7H21V5.5ZM21 11.2998H3V12.7998H21V11.2998ZM3 17H21V18.5H3V17Z"></path>
  </svg>
  メニュー
</button>
<!-- /.bl_header_btnIcon -->
```

```css:style.css
.bl_header_btnIcon {
  display: flex;
  flex-direction: column;
  gap: 2px;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  padding: 0;
  margin: 0;
  font-size: 10px;
  color: var(--sc_txt_body);
  text-decoration: none;
  cursor: pointer;
  background: none;
  border: none;
}
```

:::

:::details ダイアログ本体

```html
<dialog class="ly_dialog_wrapper" id="menu">
  <div class="ly_dialog">
    <!-- コンテンツ -->
    <button class="el_btn el_btn__primary" type="button" onclick="closeDialog()">メニューを閉じる</button>
  </div>
  <!-- /.ly_dialog -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

```css:style.css
/* ダイアログ表示中は、裏側にあるページをスクロールさせない */
html:has(dialog[open]) {
  overflow: hidden;
}

/* `max-*` は、デフォルトのスタイル（例：`dialog:-internal-dialog-in-top-layer`）を上書きするため */
.ly_dialog_wrapper {
  position: fixed;
  top: 0;
  left: 0;
  width: 100dvw;
  max-width: 100dvw;
  height: 100dvh;
  max-height: 100dvh;
  padding: 0;
  border: none;
}

.ly_dialog_wrapper::backdrop {
  backdrop-filter: blur(4px);
}

.ly_dialog_wrapper:not([open]) {
  display: none;
}

.ly_dialog {
  padding: 16px;
  overflow-y: auto;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
}

.ly_dialog > * {
  margin-top: 1rem;
  margin-bottom: 1rem;
}

.ly_dialog > *:first-child {
  margin-top: 0;
}

.ly_dialog > *:last-child {
  margin-bottom: 0;
}
```

```javascript:common.js
/* ≡≡≡ ▀▄ 「メニュー」ボタン ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    モバイル画面時、ヘッダーにある「メニュー」ボタンを押すと、モーダルダイアログを表示
  ■ 備考
    `showModal()` を使っているため、HTML側で autofocus 属性は不要
    （フォーカスは、自動で近くのフォーカス可能な要素に移るため「閉じる」ボタンが選択される）
---------------------------------------------------------------------------- */

// 各種要素の取得
const menuBtn = document.getElementById('menu-button');
const menuDialog = document.getElementById('menu');

// ボタン押下時、ダイアログをモーダル状態で表示
menuBtn.addEventListener('click', () => {
  menuDialog.showModal();
});

// ダイアログを閉じる（ダイアログ内 buttonタグの `onClick` イベントに使用）
function closeDialog(){
  menuDialog.close();
};

// モーダルダイアログ外をクリック時、ダイアログを閉じる
menuDialog.addEventListener('click', (e) => {
  if(e.target === menuDialog){
    menuDialog.close();
  }
});
```

:::

## まとめ

本項では、ダイアログ要素のスタイル部分を作成していきました。

次項では、残りのスタイルを完成させていきます。
