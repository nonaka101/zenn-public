---
title: "📄B401：フロントページ(3) ヘッダー"
---

## このページについて

本項では、ヘッダー要素を扱っていきます。その中で、ヘッダーに特化したブロックモジュール `bl_header` を構成していきます。

## 作成

### `ly_mainArea_header` の構成とその役割

**B314：ly_mainArea**(2) で扱った `ly_mainArea_header` は、下記のような要件をしておりました。ここで大切なのは、モバイル画面とデスクトップ画面でレイアウトが大きく変わるということです。

- モバイル画面では、グリッドから外れ `position: fixed;` で固定表示
  - 画面上部に固定表示（≒下にスクロールしても動かない）
  - 他要素より前面に表示（メニューダイアログなど一部の例外を除く）
    - `box-shadow` で影を表現し少し浮いたような見た目に（他要素との区切りが目立つように）
- デスクトップ画面では、グリッド構成に組み込む
  - グリッド構成は、1行目 1列目から 3 列分を使用
  - （モバイル画面の）影の表現は不要なので消す

![投稿ページのスクリーンショット１](/images/books/web-zukuri/page-single-01.png)
*デスクトップ画面*

![投稿ページのスクリーンショット２](/images/books/web-zukuri/ly_mainArea-05.png)
*モバイル画面*

本テーマでは、ヘッダー要素は その画面によって役割が変わります。デスクトップ画面では**フロントページへのリンクが仕込まれたサイトロゴ＋サイト名**のみの構成となります。

一方でモバイル画面時は、**左右サイドバーは非表示となります**。左右の余裕がないためです。なくなった要素は代わりに**メニューダイアログが引き継ぎます**。そのためモバイル画面時のヘッダーは**メニューダイアログの展開/格納ボタン**を役割として持つようになります。  
（※プラスして本テーマでは デジタル庁の旧サイトに合わせ、「検索」ページへのリンクボタンも入れています）

### `bl_header` について

画面幅に応じて役割が変わるため、ここでは新たにブロックモジュール `bl_header` を作成します。

:::message

`ly_mainArea_header` と `bl_header` とでクラスを分けています。これはそれぞれの責任範囲によるものです。

`ly_mainArea_header` はヘッダー要素（自体）のレイアウトに対しその責任を負うものとなります。一方の `bl_header` は、ヘッダー要素内の構成（モバイル画面時はボタンを表示する、など）に責任を負います。両者を混ぜて１つに統一することもできますが、そうするとどのスタイルがどちらに影響を与えているかが分かりづらくなり、修正や新規にバリエーションを作ることが困難になります。なので、責任範囲（役割？）に応じてクラスを分けています。

:::

`bl_header` に持たせる役割は、下記のようになります。

- モバイル画面時（ページ上部に固定表示）
  - サイトロゴ＋サイト名
  - メニューボタン欄
    - 検索ボタン（検索ページへのリンク）
    - メニューボタン（ダイアログメニュー開閉のコントロール）
- デスクトップ画面時（ページ左上にグリッド構成）
  - サイトロゴ＋サイト名 のみ表示

おおまかに言えば、`@media (min-width: 960px) {...}` を使って、メニューボタン欄の `display: none;` を調整することで実現できそうです。

### ベース部の構成

先程の内容から、クラス付けをしていきます。

- `ly_mainArea_header` との責任範囲から、`div` 要素を挟んで `bl_header` クラスを作成する
- `bl_header` 直下は、「サイトロゴ＋サイト名」と「メニューボタン欄」の要素を設ける
  - 「サイトロゴ＋サイト名」要素は、トップページへのリンク要素となる（＝ `a` タグ使用）
  - 「メニューボタン欄」要素は、ナビゲーションの役割を持つ（＝ `nav` 要素）

```html:header要素に絞って表示
<header class="ly_mainArea_header">
  <div class="bl_header">
    <a class="bl_header_siteTitle" href="./index.html" id="js_anchor_header">
      <!-- ロゴ画像 -->
      <!-- サイト名 -->
    </a>
    <!-- /.bl_header_siteTitle -->
    <nav class="bl_header_mobileMenu" id="anchor_mobileMenu" aria-label="メニュー" tabindex="-1">
      <!-- 検索ボタン -->
      <!-- メニューボタン -->
    </nav>
    <!-- /.bl_header_mobileMenu -->
  </div>
  <!-- /.bl_header -->
</header>
<!-- /.ly_mainArea_header -->
```

上記のマークアップでは更に、ブロックスキップとページトップに戻るボタンの関係から、下記の内容を盛り込んでいます。

- 「サイトロゴ＋サイト名」要素は、「ページトップに戻る」ボタン押下時にフォーカスを移す関係で `id` でフラグメントをつける
- 「メニューボタン欄」要素は、（ヘッダー直下の）ブロックスキップでのリンク対象となるので、`id` でフラグメントをつける
  - ここで設定するアンカーは、モバイル画面用のものとなる（詳細は**B309：bl_blockSkip**参照）
  - `tabindex="-1"` は、フォーカス可能状態にするため（Tabキーによる手動フォーカスは不可）

次にスタイルを規定します。ベース部の `bl_header` は、中に入る「サイトロゴ＋サイト名」「メニューボタン欄」を横方向のフレックスボックスで管理します。

```css
.bl_header {
  display: flex;
  flex-flow: row nowrap;
  gap: 8px;
  align-items: center;
}
```

:::message

`.bl_header` を `justify-content: space-between;` にして同じことができそうです。

ただその場合、デスクトップ画面時ではメニューボタン欄は `display: none;` になるので、単独の要素に対して適用されることになります。  
デスクトップ画面時の「サイトロゴ＋サイト名」要素は、下に入るタグメニューと開始位置と合わせた方が適切で、そのためボックスの始点に配置したいです。その際の指示が、`speace-between` だと違和感を感じたので、現状はデフォルト（`normal`）にしています。

:::

### `bl_header` 子要素

次に子要素となる、`bl_header_siteTitle` と `bl_header_mobileMenu` です。まずは `bl_header_siteTitle` から見ていきます。

#### `bl_header_siteTitle`

現状のHTMLは下記のようになっています。

```html:bl_headerに絞って表示
<div class="bl_header">
  <a class="bl_header_siteTitle" href="./index.html" id="js_anchor_header">
    <!-- ロゴ画像 -->
    <!-- サイト名 -->
  </a>
  <!-- /.bl_header_siteTitle -->
  <nav class="bl_header_mobileMenu" id="anchor_mobileMenu" aria-label="メニュー" tabindex="-1">
    <!-- 検索ボタン -->
    <!-- メニューボタン -->
  </nav>
  <!-- /.bl_header_mobileMenu -->
</div>
<!-- /.bl_header -->
```

ここから、`bl_header_siteTitle` 内に、画像要素とテキスト要素を入れていくことになります。大枠となる `bl_header_siteTitle` のスタイルは下記になります。

- フレックスボックスとして内部を管理（行方向）
- 自身で `a` タグを使っているが、下線などの装飾は不要
- 中のテキスト色等にセマンティックカラーを使用

```css
.bl_header_siteTitle {
  display: flex;
  flex: none;
  gap: 8px;
  align-items: center;
  color: var(--sc_txt_body);
  text-decoration: none;
}
```

次に内部要素を見ていきます。サイトロゴとサイト名は、WordPress側で設定した内容を持ってくることになります。  
（現状は `wp_get_attachment_image_src()` や `get_bloginfo('name')` を使っているのでクラスをつけることもできますが、変更可能性も考慮しここでは単純な形にするためクラス等は設けない想定でいっています）

```html:bl_header_siteTitleに絞って表示
<a class="bl_header_siteTitle" href="./index.html" id="js_anchor_header">
  <img src="./assets/images/Dummy_logo.PNG" alt="Logo">
  <h1>Web-Zukuri</h1>
</a>
<!-- /.bl_header_siteTitle -->
```

:::message

上記では、ヘッダーにあるサイト名に `h1` 要素を使っていますが、これはフロントページのみの挙動です。他のページでは `span` 要素になります。

これは `h1` 要素が、「ページ内に１つしか設けられない、そのページの内容を説明するもの」という役割に関係しています。フロントページではサイト名がそれにあたりますが、その他のページではサイト名ではなく、そのページ種類に沿った内容にすべきだと考えています（例えば アーカイブページならアーカイブした内容になりますし、検索ページなら「サイト内検索」が適切でしょう）

（ただ、`h1` 要素をページの頭から離れた `main` タグ内で使うことになるので「（それまでにある内容に対して）見出し構成的にどうなのか？」という不明瞭な状態になっています）

:::

中に入る要素も決まりましたので、次はそれらに対するスタイリングを施していきます。クラス付け等を施していない関係上、ここでは子結合子と要素セレクタを用いています。

```css
.bl_header_siteTitle > img {
  width: auto;
  max-height: 36px;
}

.bl_header_siteTitle > h1,
.bl_header_siteTitle > span {
  margin: 0;
  font-size: 24px;
  font-weight: 700;
}
```

サイズに関して `px` を使っているのが特徴です。後に説明する `bl_header_mobileMenu` でもそうですが、`bl_header` 内では相対単位である `rem` や `em` は使いません。これはフォントサイズ設定の如何に関わらず、ヘッダーを一定に保つ意図があります。  
（フォントサイズを大きくして、それにつられてヘッダーも大きくしてしまうと画面上で邪魔になるためです）

#### `bl_header_mobileMenu`

次に、`bl_header_mobileMenu` を見ていきます。現状のHTML構成は下記になります。

```html:bl_headerに絞って表示
<div class="bl_header">
  <a class="bl_header_siteTitle" href="./index.html" id="js_anchor_header">
    <!-- ロゴ画像 -->
    <!-- サイト名 -->
  </a>
  <!-- /.bl_header_siteTitle -->
  <nav class="bl_header_mobileMenu" id="anchor_mobileMenu" aria-label="メニュー" tabindex="-1">
    <!-- 検索ボタン -->
    <!-- メニューボタン -->
  </nav>
  <!-- /.bl_header_mobileMenu -->
</div>
<!-- /.bl_header -->
```

中にに入るのは、２つのボタン形状の要素（「検索ページへのリンク」と「メニュー開閉ボタン」）になります。この２つのボタン形状は用途こそ違いますがスタイル上では同じものとして扱います。

まずは大枠となる `bl_header_mobileMenu` です。

- モバイル画面時
  - フレックスボックスとして内部を管理
  - 要素左側にマージンを取ることで、自身を右端に、サイトロゴ＋サイト名要素を左端に寄せる
  - 自身の内部に関しても、要素を右端へ寄せておく
- デスクトップ画面時
  - 表示しない（メニューボタン欄の役割は、表示されているサイドバーが受け持つ）

```css
.bl_header_mobileMenu {
  display: flex;
  flex: none;
  gap: 8px;
  align-items: center;
  justify-content: end;
  margin-left: auto;
}

@media (min-width: 960px) {
  .bl_header_mobileMenu {
    display: none;
  }
}
```

次に内部要素です。２つのボタン形状の要素ですが、これはそれぞれリンク要素（`a`）とボタン要素（`button`）を用います。両者のスタイルは同じにしたいので、クラス名を `bl_header_btnIcon` で統一します。

ボタンとしての役割を持つ「メニューボタン」については、下記の内容を付与しています。

- `type="button"` で、`reset`, `submit` でないことを明示
- `aria-controls` で、操作する対象を明示

```html:bl_header_mobileMenuに絞って表示
<nav class="bl_header_mobileMenu" id="anchor_mobileMenu" aria-label="メニュー" tabindex="-1">
  <a href="./search.html" class="bl_header_btnIcon">
    <!-- 「🔍」マーク -->
    検索
  </a>
  <!-- /.bl_header_btnIcon -->
  <button type="button" aria-controls="menu" id="menu-button" class="bl_header_btnIcon">
    <!-- 「≡」マーク -->
    メニュー
  </button>
  <!-- /.bl_header_btnIcon -->
</nav>
<!-- /.bl_header_mobileMenu -->
```

コメントにあるように、この２つの子要素は**SVGを用いたラベル付きアイコンボタン**にします。

![ヘッダーのアイコンボタン](/images/books/web-zukuri/bl_header_btnIcon-02.png)

:::message

ラベル付きにしているのは、アイコンだけで伝わるとは限らないからです。

例えば `≡` マークを「ハンバーガーメニュー」と捉えるかどうかは、想定する利用者を考えなければなりません。本テーマは、アクセシビリティに配慮し様々な人が利用できることをその根幹としているので、可能な限りアイコンとラベルは併用すべきと考えています。

:::

**B311：bl_searchForm** や **B315：ly_dialog(2)** でSVGを用いたラベル付きアイコンボタンは扱ってきているので、詳細については省略します。あちらとの大きな差異があるのは、こちらはサイズ指定に `px` を使用しているくらいです。

```html
<nav class="bl_header_mobileMenu" id="anchor_mobileMenu" aria-label="メニュー" tabindex="-1">
  <a href="./search.html" class="bl_header_btnIcon">
    <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24"
      height="24">
      <path d="M21 20.5L15 14.5C16.2 13.2 16.9 11.4 16.9 9.5C17 5.4 13.6 2 9.5 2C5.4 2 2 5.4
          2 9.5C2 13.6 5.3 17 9.5 17C11.2 17 12.7 16.4 14 15.5L20 21.5L21 20.5ZM3.5
          9.5C3.5 6.2 6.2 3.5 9.5 3.5C12.8 3.5 15.5 6.2 15.5 9.5C15.5 12.8 12.8 15.5
          9.5 15.5C6.2 15.5 3.5 12.8 3.5 9.5Z">
      </path>
    </svg>
    検索
  </a>
  <!-- /.bl_header_btnIcon -->
  <button type="button" aria-controls="menu" id="menu-button" class="bl_header_btnIcon">
    <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24"
      height="24">
      <path fill-rule="evenodd" clip-rule="evenodd"
        d="M21 5.5H3V7H21V5.5ZM21 11.2998H3V12.7998H21V11.2998ZM3 17H21V18.5H3V17Z"></path>
    </svg>
    メニュー
  </button>
  <!-- /.bl_header_btnIcon -->
</nav>
<!-- /.bl_header_mobileMenu -->
```

```css
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

## 完成形

最後に、完成形を載せておきます。

:::details 全文（長いので格納しています）

```html
<header class="ly_mainArea_header">
  <div class="bl_header">
    <a class="bl_header_siteTitle" href="./index.html" id="js_anchor_header">
      <img src="./assets/images/Dummy_logo.PNG" alt="Logo">
      <!-- トップページのみ h1、それ以外は span -->
      <h1>Web-Zukuri</h1>
    </a>
    <!-- /.bl_header_siteTitle -->
    <nav class="bl_header_mobileMenu" id="anchor_mobileMenu" aria-label="メニュー" tabindex="-1">
      <a href="./search.html" class="bl_header_btnIcon">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24"
          height="24">
          <path d="M21 20.5L15 14.5C16.2 13.2 16.9 11.4 16.9 9.5C17 5.4 13.6 2 9.5 2C5.4 2 2 5.4
              2 9.5C2 13.6 5.3 17 9.5 17C11.2 17 12.7 16.4 14 15.5L20 21.5L21 20.5ZM3.5
              9.5C3.5 6.2 6.2 3.5 9.5 3.5C12.8 3.5 15.5 6.2 15.5 9.5C15.5 12.8 12.8 15.5
              9.5 15.5C6.2 15.5 3.5 12.8 3.5 9.5Z">
          </path>
        </svg>
        検索
      </a>
      <!-- /.bl_header_btnIcon -->
      <button type="button" aria-controls="menu" id="menu-button" class="bl_header_btnIcon">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24"
          height="24">
          <path fill-rule="evenodd" clip-rule="evenodd"
            d="M21 5.5H3V7H21V5.5ZM21 11.2998H3V12.7998H21V11.2998ZM3 17H21V18.5H3V17Z"></path>
        </svg>
        メニュー
      </button>
      <!-- /.bl_header_btnIcon -->
    </nav>
    <!-- /.bl_header_mobileMenu -->
  </div>
  <!-- /.bl_header -->
</header>
<!-- /.ly_mainArea_header -->
```

```css
.bl_header {
  display: flex;
  flex-flow: row nowrap;
  gap: 8px;
  align-items: center;
}

.bl_header_siteTitle {
  display: flex;
  flex: none;
  gap: 8px;
  align-items: center;
  color: var(--sc_txt_body);
  text-decoration: none;
}

.bl_header_siteTitle > img {
  width: auto;
  max-height: 36px;
}

.bl_header_siteTitle > h1,
.bl_header_siteTitle > span {
  margin: 0;
  font-size: 24px;
  font-weight: 700;
}

.bl_header_mobileMenu {
  display: flex;
  flex: none;
  gap: 8px;
  align-items: center;
  justify-content: end;
  margin-left: auto;
}

@media (min-width: 960px) {
  .bl_header_mobileMenu {
    display: none;
  }
}

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

## まとめ

本項では、レイアウトの中のヘッダーを扱いました。画面サイズに応じレイアウトが大きく変化することもあり、`bl_header` という専用のモジュールを作成しました。

次項ではサイドバー要素を作成します。またそれを流用しダイアログメニューを作成、今回作成したメニューボタンと連携させます。
