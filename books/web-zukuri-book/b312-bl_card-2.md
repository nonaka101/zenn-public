---
title: "📄B312：bl_card(2)"
---

## このページについて

ここでは、前項で作成した `bl_card` の様々なバリエーションについて解説します。基本となるクラスは前項のものを利用しつつ、モディファイアによる制御によって表現していきます。

## カード群の格納要素

まずは、カードを並べるための要素を作ります。

フロントページやアーカイブページなどでは、カードを複数並べて表示することになります。ここでは、カード群を格納する `bl_cardUnit` を用意し、どのように並べるかを規定していきます。

まずは構成です。複数のカード（`article.bl_card`）を格納するために、`div` 要素で `bl_cardUnit` というクラスを用意します。

```html
<div class="bl_cardUnit">
  <article class="bl_card">
    <!-- カード要素 -->
  </article>
</div>
<!-- /.bl_cardUnit -->
```

現段階ではカードは一定間隔で縦方向に並べるだけなので、直接このクラスに規定していくこともできます。ですが今後拡張する可能性があることを考え、新規にモディファイア `bl_cardUnit__1col` を作成し、こちらに縦１列のスタイルを施すようにしておきます。

最終的に、構成は下記のようになります。

```html
<div class="bl_cardUnit bl_cardUnit__1col">
  <article class="bl_card">
    <!-- カード要素 -->
  </article>
  <article class="bl_card">
    <!-- カード要素 -->
  </article>
  <article class="bl_card">
    <!-- カード要素 -->
  </article>
</div>
<!-- /.bl_cardUnit bl_cardUnit__1col -->
```

構成が完成したので、次はスタイリングを施します。

先程も申した通り、現段階では縦１列に一定間隔で並べるだけです。そして拡張性を考え、それはモディファイア `bl_cardUnit__1col` 側で規定することにしました。これらをまとめると、スタイリングは下記のようになります。

```css
.bl_cardUnit {
  display: flex;
  gap: 1rem;
}

.bl_cardUnit.bl_cardUnit__1col {
  flex-flow: column nowrap;
}
```

これでカード群を格納する要素が完成しました。

## カードのバリエーション

次は、カードのバリエーションについて見ていきます。本テーマでは下記の３種類のバリエーションを作成しています。

|バリエーション名|説明|
|:---|:---|
|スティッキー| WordPress で先頭固定表示に設定されたカード|
|クリッカブル|`a` タグを使い、クリックできるカード|
|小サイズ|基本形より小さいカード|

### スティッキー

![スティッキー設定のカード（フロントページのもの）](/images/books/web-zukuri/bl_card-04.png)
*タイトル前に 赤字で（！）が挿入されている*

スティッキーとは、WordPress で先頭固定表示に設定された特殊な記事に対して使われるバリエーションです。

![WordPress の記事管理画面、固定表示（スティッキー）設定の項目が存在している](/images/books/web-zukuri/bl_card-06.png)

上図のように、WordPress は特定の記事を固定表示させる機能があります。ユーザーが必ず目にすることが想定されており、基本的には**重要で緊急性の高いもの**に使われるものです。

スティッキー設定は WordPress側で行い、記事の出力もそちら側で制御されています。なのでここでは `html` の構成や `css` のスタイルをどうするかについて扱っていきます。

HTML側の構成は前項の基本からほとんど変化しません。モディファイアを `bl_card__sticky` とし、`bl_card` に付与するだけです。

```html:スティッキー
<article class="bl_card bl_card__sticky" aria-label="固定表示に設定された記事">
  <div class="bl_card_main">
    <div class="bl_card_body">
      <h2 class="bl_card_title">タイトル欄</h2>
      <p class="bl_card_content">記事の要旨欄</p>
    </div>
    <!-- /.bl_card_body -->
  </div>
  <!-- /.bl_card_main -->
</article>
<!-- /.bl_card bl_card__sticky -->
```

:::message

`aria-label` については WordPress側で出力するように設定するので、ここでは無視していただいて構いません。

:::

次にスタイルについてです。ここではタイトル要素に `(！)` マークをつけることで、重要であることを強調するようにします。なので、セレクタでは  `bl_card_title::before {...}` でタイトル要素の手前を指定し、そこに `(！)` マークを赤字のインラインSVGで出力するようにします。

- タイトル要素手前に `(!)` マークを出力
- マークは赤色のインラインSVGを使用
- 大きさには `1em` を使用し、タイトル要素と同じサイズにする
  - ただし同サイズのフォントとSVGで開始位置（ベースライン）のズレが生じるため、微調整を施す

```css:stickyモディファイア
.bl_card__sticky .bl_card_title::before {
  display: inline-block;
  width: 1em;
  height: 1em;
  margin-inline-end: 0.125em;
  vertical-align: -0.125em;
  content: "";
  background-image: url('data:image/svg+xml;utf-8,<svg viewBox="0 0 24 24" fill="red" xmlns="http://www.w3.org/2000/svg"><path d="M12 22C6.5 22 2 17.5 2 12C2 6.5 6.5 2 12 2C17.5 2 22 6.5 22 12C22 17.5 17.5 22 12 22ZM12 3.5C7.3 3.5 3.5 7.3 3.5 12C3.5 16.7 7.3 20.5 12 20.5C16.7 20.5 20.5 16.7 20.5 12C20.5 7.3 16.7 3.5 12 3.5ZM12 17.5C12.5523 17.5 13 17.0523 13 16.5C13 15.9477 12.5523 15.5 12 15.5C11.4477 15.5 11 15.9477 11 16.5C11 17.0523 11.4477 17.5 12 17.5ZM11.3008 6.5H12.8008V14H11.3008V6.5Z"/></svg>');
}
```

これでバリエーション `bl_card__sticky` は完成です。

### クリッカブル

![クリッカブルなカード（フロントページのもの）](/images/books/web-zukuri/bl_card-01.png)

![クリッカブルなカード（アーカイブと検索ページで使われているもの）](/images/books/web-zukuri/bl_card-02.png)

クリッカブルは、文字通りクリックすることができる要素につけられます。

構造としては、基本形での `bl_card_main` を、`div` タグから `a` タグに変えるだけです。

```html:フロントページのカード（クリッカブル）
<article class="bl_card bl_card__clickable">
  <a class="bl_card_main" href="投稿ページのURL">
    <div class="bl_card_thumbnail">
      <!-- サムネイル画像 -->
    </div>
    <!-- /.bl_card_thumbnail -->
    <div class="bl_card_body">
      <!-- テキスト諸要素 -->
    </div>
    <!-- /.bl_card_body -->
  </a>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <!-- メタ情報 -->
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card bl_card__clickable -->
```

```html:フロントページ外のカード（クリッカブル）
<article class="bl_card bl_card__clickable">
  <a class="bl_card_main" href="投稿ページのURL">
    <!-- サムネイル要素がなくなっただけ -->
    <div class="bl_card_body">
      <!-- テキスト諸要素 -->
    </div>
    <!-- /.bl_card_body -->
  </a>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <!-- メタ情報 -->
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card bl_card__clickable -->
```

スタイルで必要となるのは、**クリック可能要素であることを伝える**ことになります。ここでは、色を `body` 要素の背景色（`--sc_bg_primary`）と違うものにすることで表現しています。

```css:クリッカブル
.bl_card.bl_card__clickable > .bl_card_main > .bl_card_body {
  background-color: var(--sc_bg_body__tertiary);
}

.bl_card.bl_card__clickable > .bl_card_main > .bl_card_body:hover {
  background-color: var(--sc_bg_body__secondary);
}

.bl_card.bl_card__clickable > .bl_card_main > .bl_card_thumbnail:hover + .bl_card_body {
  background-color: var(--sc_bg_body__secondary);
}
```

クリック可能範囲は、サムネイル要素とテキスト諸要素を含む `bl_card_main` 全体です。なので `bl_card_body` をホバーした時に背景色が変化するように、`bl_card_thumbnail` のホバー時も変化しなければなりません。そのため、兄弟セレクタを使いスタイリングを施しています。

これでバリエーション `bl_card__clickable` は完成です。

### 小サイズ

![小サイズのカード（フッターにあるもの）](/images/books/web-zukuri/bl_card-03.png)

小サイズのカードは、フッターで使われるバリエーションです。最新の投稿記事を載せており、どのページからもアクセスすることができます。

他のカードに比べてテキストサイズは小さめであり、「記事タイトル」「日付データ」「メタデータ」の最小構成でできています。また、フッターは他と背景色が違うので、それに合わせる必要がでてきます。

構造は基本と変わりません。ただし「どのページからでも最新記事へのリンクを提供する」という性質上、必ずクリッカブルと組み合わせて使用することが想定されています。なので基本形の `bl_card_main` を、`div` タグから `a` タグへと変えています。

```html:フッターのカード（小サイズ＆クリッカブル）
<article class="bl_card bl_card__small bl_card__clickable">
  <a class="bl_card_main" href="投稿ページのURL">
    <div class="bl_card_body">
      <!-- テキスト諸要素 -->
    </div>
    <!-- /.bl_card_body -->
  </a>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <!-- メタ情報 -->
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card bl_card__small bl_card__clickable -->
```

スタイルの特徴は、大きく２点です。

まずは、全体のフォントサイズを `.bl_card{font-size: 1rem;}` で設定していたものを、`0.8rem` へと上書きしています。これにより一括で下位要素（`em` 使用）の調整を行うことができます。

もう一つは、クリッカブル同様、背景色の変更です。ただし `footer` 要素に関しては他と違い背景色が `--sc_bg_body__tertiary` なので、クリッカブルをそのまま使うと同色となってしまいます。なので流用せず配色を微調整してあげる必要があります。

```css
.bl_card.bl_card__small {
  font-size: 0.8rem;
  color: var(--sc_txt_body__small);
}

.bl_card.bl_card__small.bl_card__clickable > .bl_card_main > .bl_card_body {
  background-color: var(--sc_bg_body__primary);
}

.bl_card.bl_card__small.bl_card__clickable > .bl_card_main > .bl_card_body:hover {
  background-color: var(--sc_bg_body__secondary);
}

.bl_card.bl_card__small > .bl_card_meta {
  background-color: var(--sc_bg_body__primary);
}
```

## 完成形

最後に完成形をまとめておきます。

:::details スティッキー（HTML）

```html:スティッキー
<article class="bl_card bl_card__sticky" aria-label="固定表示に設定された記事">
  <div class="bl_card_main">
    <div class="bl_card_body">
      <h2 class="bl_card_title">タイトル欄</h2>
      <p class="bl_card_content">記事の要旨欄</p>
    </div>
    <!-- /.bl_card_body -->
  </div>
  <!-- /.bl_card_main -->
</article>
<!-- /.bl_card bl_card__sticky -->
```

:::

:::details クリッカブル１（HTML）

```html:フロントページのカード（クリッカブル）
<article class="bl_card bl_card__clickable">
  <a class="bl_card_main" href="./page.html">
    <div class="bl_card_thumbnail">
      <img src="./assets/images/m_02_white.png">
    </div>
    <!-- /.bl_card_thumbnail -->
    <div class="bl_card_body">
      <h2 class="bl_card_title">サンプル：固定ページ</h2>
      <p class="bl_card_content">
        このページは 固定ページ用のサンプルページとなります。
        ここでは、本テーマで使用しているコンポーネントの一部を、一覧として掲示しております。
      </p>
      <div class="bl_card_dates">
        <div class="bl_card_date">
          <span>投稿</span>
          <time datetime="2023-10-16">2023/10/16</time>
        </div>
        <!-- /.bl_card_date -->
        <div class="bl_card_date">
          <span>更新</span>
          <time datetime="2023-11-01">2023/10/21</time>
        </div>
        <!-- /.bl_card_date -->
      </div>
      <!-- /.bl_card_dates -->
    </div>
    <!-- /.bl_card_body -->
  </a>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <ul class="bl_card_categories" aria-label="カテゴリ">
      <li><a href="./archive.html">サンプルページ</a></li>
      <li><a href="./archive.html">コンポーネント</a></li>
    </ul>
    <!-- /.bl_card_categories -->
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card bl_card__clickable -->
```

:::

:::details クリッカブル２（HTML）

```html:フロントページ外のカード（クリッカブル）
<article class="bl_card bl_card__clickable">
  <a class="bl_card_main" href="./page.html">
    <div class="bl_card_body">
      <h2 class="bl_card_title">サンプル：固定ページ</h2>
      <p class="bl_card_content">
        このページは 固定ページ用のサンプルページとなります。
        ここでは、本テーマで使用しているコンポーネントの一部を、一覧として掲示しております。
      </p>
      <div class="bl_card_dates">
        <div class="bl_card_date">
          <span>投稿</span>
          <time datetime="2023-10-16">2023/10/16</time>
        </div>
        <!-- /.bl_card_date -->
        <div class="bl_card_date">
          <span>更新</span>
          <time datetime="2023-11-01">2023/10/21</time>
        </div>
        <!-- /.bl_card_date -->
      </div>
      <!-- /.bl_card_dates -->
    </div>
    <!-- /.bl_card_body -->
  </a>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <ul class="bl_card_categories" aria-label="カテゴリ">
      <li><a href="./archive.html">サンプルページ</a></li>
      <li><a href="./archive.html">コンポーネント</a></li>
    </ul>
    <!-- /.bl_card_categories -->
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card bl_card__clickable -->
```

:::

:::details 小サイズ（HTML）

```html:フッターのカード（小サイズ＆クリッカブル）
<article class="bl_card bl_card__small bl_card__clickable">
  <a class="bl_card_main" href="./page.html">
    <div class="bl_card_body">
      <h3 class="bl_card_title">サンプル：固定ページ</h3>
      <div class="bl_card_dates">
        <div class="bl_card_date">
          <span>投稿</span>
          <time datetime="2023-10-16">2023/10/16</time>
        </div>
        <!-- /.bl_card_date -->
        <div class="bl_card_date">
          <span>更新</span>
          <time datetime="2023-11-01">2023/10/21</time>
        </div>
        <!-- /.bl_card_date -->
      </div>
      <!-- /.bl_card_dates -->
    </div>
    <!-- /.bl_card_body -->
  </a>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <ul class="bl_card_categories" aria-label="カテゴリ">
      <li><a href="./archive.html">サンプルページ</a></li>
      <li><a href="./archive.html">コンポーネント</a></li>
    </ul>
    <!-- /.bl_card_categories -->
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card bl_card__small bl_card__clickable -->
```

:::

:::details スタイル（CSS）

```css:style.css
/* ≡≡≡ ❒ bl_card ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    カードのコンポーネント、下記の場所で使用される
    - トップページの記事一覧（サムネイル、タイトル、要旨、投稿/更新日、カテゴリ）
    - フッターの最新記事（タイトル、投稿/更新日、カテゴリ）
    - アーカイブページの記事一覧（タイトル、要旨、投稿/更新日、カテゴリ）
    - 検索ページでヒットした記事一覧（タイトル、要旨、投稿/更新日、カテゴリ）
  ■ 構成
    bl_cardUnit : カード群を並べるためのラッパー
    -> bl_cardUnit__1col : 縦1列に並べる
      bl_card : ベース
      -> bl_card__clickable : リンクが付与された記事用
      -> bl_card__sticky : 先頭に固定表示された記事用
      -> bl_card__small : フッターにある、最新記事用
        bl_card_main : コンテンツのメインとなる情報欄
          bl_card_thumbnail : 投稿のサムネイル画像
          bl_card_body : 各種テキスト情報欄
            bl_card_title : 記事のタイトル
            bl_card_content : 記事の要旨
            bl_card_dates : 記事の日付情報（投稿日、更新日）
              bl_card_date : 各日付情報
        bl_card_meta : コンテンツのメタ情報
          bl_card_categories : 記事が属するカテゴリ
---------------------------------------------------------------------------- */
.bl_cardUnit {
  display: flex;
  gap: 1rem;
}

.bl_cardUnit.bl_cardUnit__1col {
  flex-flow: column nowrap;
}

.bl_card {
  --card_space: 16px;

  /*
   * カスタムプロパティ `--card_r` は カード内のR値で、自身の幅に応じ変化
   * - ～40px      : R4
   * - 40px～120px : R8
   * - 120px～     : R12
   */
  --card_r: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );

  /* 最上位にある rem を基準に、残りは em 単位を使用（`__small` の場合とで調整 ） */
  display: block;
  font-size: 1rem;
  color: var(--sc_txt_body);
}

/* クリック可能なカードのスタイル */
.bl_card.bl_card__clickable > .bl_card_main > .bl_card_body {
  background-color: var(--sc_bg_body__tertiary);
}

.bl_card.bl_card__clickable > .bl_card_main > .bl_card_body:hover {
  background-color: var(--sc_bg_body__secondary);
}

.bl_card.bl_card__clickable > .bl_card_main > .bl_card_thumbnail:hover + .bl_card_body {
  background-color: var(--sc_bg_body__secondary);
}

/* フッターにある「最新記事」用のスタイル */
.bl_card.bl_card__small {
  font-size: 0.8rem;
  color: var(--sc_txt_body__small);
}

.bl_card.bl_card__small.bl_card__clickable > .bl_card_main > .bl_card_body {
  background-color: var(--sc_bg_body__primary);
}

.bl_card.bl_card__small.bl_card__clickable > .bl_card_main > .bl_card_body:hover {
  background-color: var(--sc_bg_body__secondary);
}

.bl_card.bl_card__small > .bl_card_meta {
  background-color: var(--sc_bg_body__primary);
}

/* スティッキー（WordPressで固定表示に設定したもの）用のスタイル */
.bl_card__sticky .bl_card_title::before {
  display: inline-block;
  width: 1em;
  height: 1em;
  margin-inline-end: 0.125em;
  vertical-align: -0.125em;
  content: "";
  background-image: url('data:image/svg+xml;utf-8,<svg viewBox="0 0 24 24" fill="red" xmlns="http://www.w3.org/2000/svg"><path d="M12 22C6.5 22 2 17.5 2 12C2 6.5 6.5 2 12 2C17.5 2 22 6.5 22 12C22 17.5 17.5 22 12 22ZM12 3.5C7.3 3.5 3.5 7.3 3.5 12C3.5 16.7 7.3 20.5 12 20.5C16.7 20.5 20.5 16.7 20.5 12C20.5 7.3 16.7 3.5 12 3.5ZM12 17.5C12.5523 17.5 13 17.0523 13 16.5C13 15.9477 12.5523 15.5 12 15.5C11.4477 15.5 11 15.9477 11 16.5C11 17.0523 11.4477 17.5 12 17.5ZM11.3008 6.5H12.8008V14H11.3008V6.5Z"/></svg>');
}

/* メイン部 */
.bl_card_main {
  display: block;
  text-decoration: none;
  border-top-left-radius: var(--card_r, 0);
  border-top-right-radius: var(--card_r, 0);
}

/*
 * ここからborderに関するスタイルが続く
 * カードを border で囲み、四隅を丸めるためのもの
 * カテゴリを出力する bl_card_meta の有無に対応できるようしている
 */
.bl_card_main > *:first-child {
  overflow: hidden;
  border: var(--sc_border_divider);
  border-bottom: none;
  border-top-left-radius: var(--card_r, 0);
  border-top-right-radius: var(--card_r, 0);
}

.bl_card_main > *:not(:first-child) {
  border-right: var(--sc_border_divider);
  border-left: var(--sc_border_divider);
}

.bl_card_main:not(:has(+ .bl_card_meta)) > *:last-child {
  overflow: hidden;
  border-bottom: var(--sc_border_divider);
  border-bottom-right-radius: var(--card_r, 0);
  border-bottom-left-radius: var(--card_r, 0);
}

/* サムネイル部 */
.bl_card_thumbnail > img {
  width: 100%;
  height: auto;
  aspect-ratio: 16/9;
  object-fit: cover;
  vertical-align: bottom;
}

/* ボディ部 */
.bl_card_body {
  padding: 2px var(--card_space);
}

.bl_card_title {
  margin: 0;
  margin-top: var(--space_1, 0.5rem);
  margin-bottom: var(--space_1, 0.5rem);
  font-size: 1.5em;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .bl_card_title {
    font-size: 1.75em;
    font-weight: 400;
  }
}

.bl_card_content {
  margin: 0;
  margin-top: var(--space_1, 0.5rem);
  margin-bottom: var(--space_1, 0.5rem);
  font-size: 0.875em;
  font-weight: 400;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

/* 投稿日および更新日の欄 */
.bl_card_dates {
  display: flex;
  flex-flow: row wrap;
  gap: 0.2rem 1rem;
  margin: 0;
  margin-top: var(--space_1, 0.5rem);
  margin-bottom: var(--space_1, 0.5rem);
}

.bl_card_date {
  flex: none;
}

.bl_card_date > span,
.bl_card_date > time {
  font-size: 0.75em;
  font-weight: 400;
  line-height: 1.7;
  color: var(--sc_txt_desc);
  letter-spacing: 0.02em;
}

.bl_card_date > span::after {
  content: ":";
}

/* 記事に関するメタ情報 */
.bl_card_meta {
  display: flex;
  flex-flow: column nowrap;
  gap: 8px;
  padding: 8px var(--card_space);
  background-color: var(--sc_bg_body__tertiary);
  border: var(--sc_border_divider);
  border-top-style: dashed;
  border-bottom-right-radius: var(--card_r, 0);
  border-bottom-left-radius: var(--card_r, 0);
}

.bl_card_categories {
  display: flex;
  flex: none;
  flex-flow: row wrap;
  gap: 0.3rem;
  padding: 0;
  margin: 0;
  list-style-type: none;
}

.bl_card_categories > li {
  flex: none;
}

.bl_card_categories > li > a {
  padding: 0.1em 1em;
  font-size: 0.75em;
  font-weight: 400;
  line-height: 1.7;
  color: var(--sc_txt_body__small);
  text-align: center;
  text-decoration: none;
  letter-spacing: 0.02em;
  cursor: pointer;
  border: var(--sc_border_divider);
  border-radius: 9999px;
  transition: 0.25s;
}

.bl_card_categories > li > a:hover,
.bl_card_categories > li > a:active {
  background-color: var(--sc_bg_body__secondary);
}
```

:::

## まとめ

このページではカードを扱う `bl_card` について、カード群を格納する要素と、カードのバリエーションについて説明しました。

これで全てのブロックモジュールについて解説したことになります。次項からは レイアウト（`ly_*`）について取り扱っていきます。
