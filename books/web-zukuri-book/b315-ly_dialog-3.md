---
title: "📄B315：ly_dialog(3)"
---

## このページについて

前項までで `ly_dialog` の HTML と JavaScript、そして一部のスタイルを作成しました。

本項では残るスタイルを進め、`ly_dialog` を完成させていきます。

## 作成

前項までで、`ly_dialog` が完成しています。

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

本項では、`ly_dialog` 内のヘッダーとなる、`bl_menuHeader` を完成させていきます。

今回扱う `bl_menuHeader` は、`ly_dialog` と接頭辞や部品名が異なります。これは、`ly_dialog` はレイアウトを扱うものだったのに対し、`bl_menuHeader` は他の場所でも再利用可能なモジュールであり、その設計の考え方（役割や責任範囲）が異なるからです。

### メニューヘッダー

ここからは、メニューヘッダーを作成していきます。

![ダイアログ画面のメニューヘッダー部を拡大表示](/images/books/web-zukuri/bl_menuHeader-01.png)

#### 本体

まずはベースとなる `bl_menuHeader` を設けます。この中に「タイトル要素」と「閉じるボタン」を格納する想定です。

```html:親となるly_dialogは省略
<div class="bl_menuHeader">
  <!-- メニュータイトル -->
  <!-- 閉じるボタン -->
</div>
<!-- /.bl_menuHeader -->
```

スタイルですが、格納要素は「タイトル」と「ボタン」の2つだけの想定です。なので、要件としては下記になります。

- 横方向のフレックスボックス
- タイトルは画面左、ボタンは画面右（間はコンテンツのサイズに応じて自動調整）

```css
.bl_menuHeader {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}
```

#### メニュータイトル

次にタイトル要素です。これは `h2` 要素を使います。

```html
<div class="bl_menuHeader">
  <h2 class="bl_menuHeader_title">メニュー</h2>
  <!-- 閉じるボタン -->
</div>
<!-- /.bl_menuHeader -->
```

スタイルに関しては、テキスト関係のものに留まります。

```css
.bl_menuHeader_title {
  padding: 0;
  margin: 0;
  font-size: 1.75rem;
  line-height: 1.5;
}
```

:::message

ここのヘッダーは**B401：フロントページ(3)ヘッダー**で扱う `header` タグと違い、`rem` を用いて可変にしています。これは、要素が画面に固定されるかどうかが関わってきます。

メイン画面のヘッダー（モバイル画面時）は、画面上部に固定するようになっています。そのため、フォントサイズによる可変にしてしまうと画面占有率が高くなり、かえって邪魔になってしまうことが考えられます。そのため、あちらでは絶対単位 `px` によるスタイリングを施しています。

一方のこちらは、ヘッダーは固定ではありません。下へスクロールすれば画面からは消えるので、邪魔になることもありません。なので、フォントサイズによる可変に対応させています。

※なお後述しますが、ヘッダーが固定でないことは「閉じるボタン」にも影響しています。

:::

#### 閉じるボタン

最後に、閉じるボタンです。既にダイアログ本体下部に１つありますが、ここでも同じ機能のボタンを設けます。

:::message

「ダイアログ内に閉じるボタンが複数ある理由」は、２つあります。

1つ目は、スクリーンリーダーやキーボード操作者が扱いやすいようにするためです。上部のボタンは「すぐにダイアログを開閉できるように」、下部のボタンは「上まで戻らなくてもダイアログを閉じられるように」という意図です。

2つ目の理由は、フォントサイズを大きくしているユーザーがボタンを見失うことを防ぐ意図があります。デフォルトのフォントサイズを使用している場合は、閉じるボタンが見えなくなることは稀です。ですが大きくしている場合、ボタンが画面内に入ることが難しくなります。こうなるとダイアログを閉じることができなくなり、ユーザーにとっては不便に感じることでしょう。一番上と一番下に設けておくことは、迷ってもボタンまでたどり着きやすいようする意図があります。

:::

要件としては、下記のようになります。

- 正方形の形を取り、サイズはタイトル要素と近しい値とする
  - `font-size: 1.75rem;`, `line-height: 1.5;` なので、`2.625rem`
- マシンリーダブルとなるよう `button` タグを使用
  - カーソルはポインターを使用
- 中の要素は `svg` とテキストノードで構成される ラベル付きアイコン
  - 両者は上下に並べられ、中央揃いとする
  - テキストは少し小さめでも良いため `0.7rem`
    - ただしセマンティックカラーはコントラスト比を高めのものを使用する
    - 下線などの装飾は不要
  - `svg` は残りのスペースを使用

```html
<div class="bl_menuHeader">
  <h2 class="bl_menuHeader_title">メニュー</h2>
  <button class="bl_menuHeader_btnIcon" type="button" onclick="closeDialog()">
    <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg"
      viewBox="0 0 24 24" width="24" height="24">
      <path stroke-width="1.4286" d="M 22,4.0142857 19.985715,2 12,9.9857149 4.0142857,2 2,4.0142857 9.9857149,12 2,19.985715 4.0142857,22 12,14.014286 19.985715,22 22,19.985715 14.014286,12 Z"/>
    </svg>
    閉じる
  </button>
</div>
<!-- /.bl_menuHeader -->
```

```css
.bl_menuHeader_btnIcon {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-evenly;
  width: 2.625rem;
  height: 2.625rem;
  padding: 0;
  margin: 0;
  font-size: 0.7rem;
  color: var(--sc_txt_body);
  text-decoration: none;
  cursor: pointer;
}

.bl_menuHeader_btnIcon > svg {
  width: 1.625rem;
  height: 1.625rem;
}
```

これで、`ly_dialog` 要素は全て完了しました。

## 完成形

最後に完成形をまとめておきます。

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
    <div class="bl_menuHeader">
      <h2 class="bl_menuHeader_title">メニュー</h2>
      <button class="bl_menuHeader_btnIcon" type="button" onclick="closeDialog()">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24" width="24" height="24">
          <path stroke-width="1.4286" d="M 22,4.0142857 19.985715,2 12,9.9857149 4.0142857,2 2,4.0142857 9.9857149,12 2,19.985715 4.0142857,22 12,14.014286 19.985715,22 22,19.985715 14.014286,12 Z"/>
        </svg>
        閉じる
      </button>
    </div>
    <!-- /.bl_menuHeader -->

    <!-- コンテンツ -->

    <button class="el_btn el_btn__primary" type="button" onclick="closeDialog()">メニューを閉じる</button>
  </div>
  <!-- /.ly_dialog -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

```css:style.css
/* ≡≡≡ ❒ ly_dialog ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    dialogタグを使ったモーダルダイアログ（モーダル機能に関しては、`common.js` を参照）
    モバイル画面の時にヘッダーに表示される「メニュー」ボタンから呼び出される
    画面全体をモーダルダイアログで覆い、デスクトップ画面でのサイドバーの内容を表示
  ■ 構成
    ly_dialog_wrapper : ラッパー
      ly_dialog : ダイアログ本体
        bl_menuHeader : メニューヘッダー
          bl_menuHeader_title : メニュー名
          bl_menuHeader_btnIcon : 「閉じる」ボタン
----------------------------------------------------------------------------- */

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

.bl_menuHeader {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.bl_menuHeader_title {
  padding: 0;
  margin: 0;
  font-size: 1.75rem;
  line-height: 1.5;
}

.bl_menuHeader_btnIcon {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-evenly;
  width: 2.625rem; /* bl_header と違い画面占有率を気にしない為 rem */
  height: 2.625rem;
  padding: 0;
  margin: 0;
  font-size: 0.7rem;
  color: var(--sc_txt_body);
  text-decoration: none;
  cursor: pointer;
  background: none;
  border: none;
}

.bl_menuHeader_btnIcon > svg {
  width: 1.625rem;
  height: 1.625rem;
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

本項では3回に渡り、モーダルダイアログを構成する `ly_dialog` を扱ってきました。これで `ly_dialog` は完了となり、レイアウトも全て解説できました。

ここからは、特殊なコンポーネントを扱っていきます。特殊というのは、エレメントモジュールやブロックモジュール、レイアウトなどの基本的なものから少し外れた、JavaScript や WordPress 用に設計されたモジュールになります。

次項では、JavaScript で制御する「ページトップに戻る」ボタン `js_pageTop` を扱っていきます。
