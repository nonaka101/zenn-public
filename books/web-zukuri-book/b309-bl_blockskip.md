---
title: "📄B309：bl_blockSkip"
---

## このページについて

ここでは、ブロックスキップについて説明します。  

![ブロックスキップのスクリーンショット](/images/books/web-zukuri/bl_blockSkip-01.png)

::::message

本テーマのブロックスキップの扱いについて、個人的な考えで進めたもので、他ではあまり見られないものになっています。そのため、実際に使いやすいのか自信がありません。

このような形にした理由については、下記に記しております。

:::details この形にした理由についてのメモ

- WAI-ARIAでの対処だと、自動翻訳の対象にならない場合がある（最新の主要ブラウザでは翻訳してくれるのが基本です）
- 読み上げ専用の要素（表示しない想定）にすることもできるが、1からの作成なのでそれなら画面に表示してもいいのでは？
- キーボード操作者にも使えるようできるなら、そちらの方がいいだろうと判断
- 二箇所で使う前提だったので、同じコンポーネントを使い回すことを考慮。周りの要素に左右されない形（レイアウト構成から外れる形）が望ましいと考えた

:::

::::

### ブロックスキップについて

マウスやタッチ操作を常用されている方にとって、ブロックスキップは馴染みの薄いものと思われます。なので詳細に入る前に、ブロックスキップとは何かについて説明していきます。

より詳細について知りたい方は、WAIC等を参照してみてください。

https://waic.jp/translations/WCAG21/Understanding/bypass-blocks.html

#### どんなものか？

ブロックスキップは、キーボード操作やスクリーンリーダーの読み上げ時に出てくるものがほとんどです。

試しに、マイクロソフトやアマゾン等の大手サイトを開き、Tabキーによる操作を試してみてください。恐らく、マウスやタッチ操作では見ることがない「本文へスキップする」に類する要素が表示されたと思います。これがブロックスキップです。

Enterキーを押すと、ヘッダーにある要素をスキップして本文にフォーカスが移ると思います。このように、ブロックスキップの役割は「そのページの主となるコンテンツへ直接アクセスする」ためのものです。

#### なぜ必要なのか？

「マウスやタッチ操作」と「キーボード操作、スクリーンリーダー」の大きな違いとしては、コンテンツの順序に縛られるかどうかがあります

:::message

最近は改善が進み、この内容が当てはまらなくなってきました。詳細については後述しますが、ここでは説明の都合上、改善される前の状況を想定します。

:::

視覚による情報が使える場合、ユーザーは必要な情報がどこにあるのかを即座に判断することができます。

![フロントページを見ている画像、右サイドバー（aside）にあるリンクを注視している](/images/books/web-zukuri/about-blockSkip-01.png)
*目視：必要な情報をサイトレイアウトから見つける*

マウス（タッチ）による操作の場合、コンテンツの順序による制限を受けず、必要な場所に直接ポインタを動かせます。

![上図にプラスしてマウス操作を表現している、ページの構造（順序）に捕らわれず、リンク要素に直接カーソルを移動している](/images/books/web-zukuri/about-blockSkip-02.png)
*マウス（タップ）操作：目的場所へ直接ポインタを移動可*

キーボード操作の場合は、コンテンツの順序に従って Tabキー を押下する必要があります。

![これまでの図に対し、キーボードのTabキーで目的のリンクへたどり着く操作を表現している](/images/books/web-zukuri/about-blockSkip-03.png)
*キーボード操作：ページ構成の順序に従いフォーカスインジケーターを移動*

一方、視覚情報が使えない場合を考えます。この時のユーザーは、必要な情報がどこにあるかわかりません。

![これまでの図に対し、目視が使えない場合を表現。どこに目的のリンクがあるかわからない状態](/images/books/web-zukuri/about-blockSkip-04.png)
*目視が使えない場合*

そのため、欲しい情報にたどり着くまでスクリーンリーダーの読み上げを聞く必要があります。

![上図にプラスして、スクリーンリーダーの読み上げを表現。ページの構造（順序）に従い1つずつコンテンツを読み上げている](/images/books/web-zukuri/about-blockSkip-05.png)
*スクリーンリーダー：ページ構成の順序に従いコンテンツを読み上げる*

これは当然ですが、マウス操作と比べて膨大な時間が必要となります。１つ１つのページで欲しい情報にたどり着くまで何分も待つのは、ユーザーにとっては不便に感じることでしょう。

これを短縮できるようするために、２つの方法があります。

- サイト全体で、コンテンツの並び（構造）を共通化する
- コンテンツの並びを無視できるリンクを提供

まずは**コンテンツの並びの共通化**です。「最初にヘッダーがあって、次にメニュー、それから本文、フッターときて・・・」のように、サイト全体でコンテンツの並び方が共通化されていれば、全体に注意を払わず「サイドバーにある情報が見たいから、最初にある内容は無視して大丈夫」と判断することができます。

![これまでの図に対し、不要なランドマーク箇所を黒塗りし、必要な箇所を強調](/images/books/web-zukuri/about-blockSkip-06.png)
*サイト内でページ構造が共通化されていれば、大まかな場所がわかりやすくなる*

次に**コンテンツの並びを無視するリンク**です。ユーザーにとって重要な箇所（例：記事本文、各種ランドマークなど）へのリンクを提供することで、読み上げや移動のコストは大幅に低下します。

![これまでの図に対し、ページ内リンク集を用いて道中にあるコンテンツを無視して移動できる状態](/images/books/web-zukuri/about-blockSkip-07.png)

ここで提供するリンクが、ブロックスキップとなります。

本テーマでは、**コンテンツの並びの共通化**については、B400のページ節で解説していきます。本項ではもう一つの、**コンテンツの並びを無視するリンク**であるブロックスキップを扱っていきます。

:::message

現在はスクリーンリーダーの機能が向上し、「見出し要素」「リンク要素」「ランドマーク要素」「フォーム要素」等に応じた移動機能ができるようになりました。

![VoiceOver の上下スワイプの設定切替で文字にしている](/images/books/web-zukuri/iphone-voiceover-02.png =300x)
*iPhone の VoiceOver*

![上下スワイプを設定している状態](/images/books/web-zukuri/android-talkback-02.png =300x)
*Android の Talkback*

![ChromeVoxの見出しタブ](/images/books/web-zukuri/chromebook-chromevox-05.png)
*ChromeBook の ChromeVox_1*

![ChromeVoxのリンクタブ](/images/books/web-zukuri/chromebook-chromevox-06.png)
*ChromeBook の ChromeVox_2*

そのため、WCAG等のアクセシビリティ要件において、見出しやランドマーク等の構造がマシンリーダブルであれば、ブロックスキップを搭載することは必須ではありません。

https://waic.jp/translations/WCAG21/Understanding/bypass-blocks.html

本テーマでは、下記の理由からブロックスキップ機能を搭載しています。

- 必須ではないが、あっても利便性を損ねるものではないこと
- キーボード操作には依然として有用であること
- 搭載する箇所が明確で、導入に対するリスクやコストが低いこと

:::

## 構造の定義

本テーマでは、ブロックスキップを２箇所に導入しています。

- ヘッダー下に、各ランドマークへの移動（全ページ）
- 記事タイトル下に、補足情報のスキップ（投稿ページのみ）

この２つのケースを確認し、そこから構造や機能について定義していきます。

### ケース１：ヘッダー直下（全ページ）

![ヘッダー直下にあるブロックスキップ](/images/books/web-zukuri/bl_blockSkip-01.png)

ページの初め付近に挿入することを想定しています。

適用するサイトやページの内容によって、例えば「サイドバーに表示されるメニュー項目が多い」「記事本文の文量が多い」ことが想定されます。そうなるとページ後部にある内容にたどり着くまで、読み上げにかかる時間や、Tabキーの操作数がかかってしまいます。

そこでページの最初付近に、各種ランドマーク要素への移動用リンクを配置することで解決します。

### ケース２：記事タイトル下（投稿ページのみ）

![投稿ページにあるブロックスキップ](/images/books/web-zukuri/about-blockSkip-08.png)

本テーマでは、投稿ページの記事レイアウトを下記のようにしています。

- 記事タイトル
- （ブロックスキップ）
- 記事補足情報欄
  - 公開日
  - 更新日
  - カテゴリ
  - タグ
- サムネイル画像
- 記事本文

![投稿記事（記事タイトルから本文までが見えるように、また内容を書いたラベルを貼る）](/images/books/web-zukuri/page-single-01.png)

ここで問題となるのが、記事の補足情報が記事本文より前にあることです。カテゴリやタグは WordPress 上で複数登録することができるので、場合によっては記事本文にたどり着くまでの読み上げ時間や操作数が多くなってしまう可能性があります。

そこで、記事タイトルから直接 記事本文に移動できるようにリンクを挿入しています。

### ケースのまとめ

以上のケースから、本テーマのブロックスキップは下記のようにしたいと考えます。

- マウスやタッチ操作では（通常）見えない
- スクリーンリーダーで読み上げられる
- Tabキーによる操作で表示される
  - モーダルウィンドウのような形で、画面中央にボックスを表示
  - （実際にはモーダルではないので）ボックス外クリックや Tabキーによる移動で容易に抜けられる
  - ボックス外はブラーをかけて、ブロックスキップに注意が向くようにする
  - ボックスの内容は「タイトル」「説明文」「移動用リンク」

## 作成

上記で決めた要件から、マークアップしていきます。ここでは、ヘッダー直下に配置される各種ランドマーク移動のブロックスキップを想定して作成します。

### ラッパー要素

まずはラッパー要素を作成します。配置する場所は、`header` の直後です。こうすることでコンテンツの順番から、`header` の次にスクリーンリーダーや Tabキー操作の対象となります。

```html:前後関係
<header>
  <!-- 省略 -->
</header>

<nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集">
</nav>
<!-- /.bl_blockSkip_wrapper -->

<main>
  <!-- 省略 -->
</main>
```

次にスタイルです。必要となる要件は下記の２つです。

- 内部要素は、画面中央に表示される
- 通常時は見えない状態である

ここでは見えない状態にするために、スクリーンリーダー用の要素としてよく使われるテクニック（サイズを `1px` にし、`clip-path: inset(100%)`）を使用しています。

```css:通常時は見えないようにする
.bl_blockSkip_wrapper {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 1px;
  height: 1px;
  clip-path: inset(100%);
}
```

ただしこの状態は、中の要素をスクリーンリーダーでは読み上げてくれますが、スクリーン上には表示されません。そこで、キー操作者に中の要素を見せるため、**内部要素にフォーカスが当たっている**時に表示を切り替えるようします。使う擬似クラスは `:focus-within` です。

これと下記の要件を踏まえ、スタイルを規定していきます。

- 配置場所は画面の中央
- 最前面に表示
- 内部に入る要素の外側にブラーをかける

```css:フォーカスが内部要素にある時は、画面中央に表示
.bl_blockSkip_wrapper:focus-within {
  /* 100 でないのは横スクロールが発生するため */
  width: 95dvw;
  height: 95dvh;

  /* 画面中央に配置するためのテクニック */
  top: 50dvh;
  left: 50dvw;
  transform: translate(-51%, -51%);

  /* デフォルトのスタイルを剥ぐ */
  clip-path: none;

  /* 内部ボックスの外側（画面全体）にブラー */
  backdrop-filter: blur(4px);

  z-index: 100;
}
```

`95dvw` や `-51%` のような中途半端な数値になっているのは、横スクロールを発生させないための調整です。

`100dvw` にするとスクロールバーの関係から、画面より少し大きくなってしまうことがあります。ここで必要なのは「画面全体にブラーをかけて注意をブロックスキップに向ける」ことなので、画面の端まできっちりする必要はありません。そこで `95dvw` という少し小さめの指定をしています。

### ベース要素

次にベース要素です。これはブロックスキップ本体のボックスとなります。

```html
<nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集">
  <div class="bl_blockSkip">
    <!-- 「タイトル」要素 -->
    <!-- 「説明文」要素 -->
    <!-- 「リンク集」要素 -->
  </div>
  <!-- /.bl_blockSkip -->
</nav>
<!-- /.bl_blockSkip_wrapper -->
```

スタイルは下記の要件を踏まえて作ります。

- 内部要素用に `padding` を入れる
  - 画面の大きいデスクトップ画面では、余白は大きく取っていい
- ボックスが画面から浮き出ているような形にする
- 背景色やボーダーはセマンティックカラーで用意しているものを使用

```css
.bl_blockSkip{
  padding: 12px;
  margin-right: 24px;
  margin-left: 24px;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
  border-radius: 12px;
  box-shadow: 0 19px 38px rgba(0 0 0 0.30),
              0 15px 12px rgba(0 0 0 0.22);
}

@media (min-width: 560px) {
  .bl_blockSkip {
    padding: 12px 16px;
  }
}
```

### ボックス内要素

次にボックス内要素です。ここでは、下記の要素を作っていきます。

- タイトル
- 説明文
- リンクリスト

#### タイトル

```html
<nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集">
  <div class="bl_blockSkip">
    <p class="bl_blockSkip_title">ブロックスキップ</p>
    <!-- 「説明文」要素 -->
    <!-- 「リンク集」要素 -->
  </div>
  <!-- /.bl_blockSkip -->
</nav>
<!-- /.bl_blockSkip_wrapper -->
```

スタイルは、下記のようにします。

- 上下の余白を取る
- フォントは大きめに
  - デスクトップ画面時は、もう少し大きく

```css
.bl_blockSkip_title {
  margin-top: 16px;
  margin-bottom: 16px;
  font-size: 1.25rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .bl_blockSkip_title {
    font-size: 1.75rem;
    font-weight: 400;
  }
}
```

#### 説明文

```html
<nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集">
  <div class="bl_blockSkip">
    <p class="bl_blockSkip_title">ブロックスキップ</p>
    <p class="bl_blockSkip_description">各種ページ位置に移動できます</p>
    <!-- 「リンク集」要素 -->
  </div>
  <!-- /.bl_blockSkip -->
</nav>
<!-- /.bl_blockSkip_wrapper -->
```

スタイルは下記のようにします。

```css
.bl_blockSkip_description {
  margin-bottom: 24px;
  font-size: 0.75rem;
  font-weight: 400;
  color: var(--sc_txt_body);
}

@media (min-width: 560px) {
  .bl_blockSkip_description {
    margin-bottom: 20px;
  }
}
```

#### リンクリスト

最後にリンクリストです。

##### 基本形

まずは、大枠となる `ul` 要素と、中の `li` を規定していきます。

```html
<nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集">
  <div class="bl_blockSkip">
    <p class="bl_blockSkip_title">ブロックスキップ</p>
    <p class="bl_blockSkip_description">各種ページ位置に移動できます</p>
    <ul class="bl_blockSkip_linkList">
      <li class="bl_blockSkip_linkItem">
        <!-- この中に、`a` タグでリンクを貼る想定 -->
      </li>
      <!-- /.bl_blockSkip_linkItem -->
    </ul>
    <!-- /.bl_blockSkip_linkList -->
  </div>
  <!-- /.bl_blockSkip -->
</nav>
<!-- /.bl_blockSkip_wrapper -->
```

スタイルは下記のようにします。

- リストの開始位置は、タイトルや説明文より少し右にずらす
- リンクアイテム同士で上下の余白を取る
- リンク要素は `a` タグを使用する
  - テキスト色は、セマンティックカラーで規定したものを使用

```css
.bl_blockSkip_linkList {
  padding-inline-start: 1rem;
}

.bl_blockSkip_linkItem {
  padding: 4px;
  margin: 0;
}

.bl_blockSkip_linkItem > a {
  color: var(--sc_txt_link);
  text-decoration: none;
}
```

##### 場面によるリンクアイテムの差異について

基本となる形は上記で完了しました。ここからは、ページの種別や、画面サイズによる差異について話していきます。

まず考えるのは、ページ種類による違いです。現在考えているブロックスキップは「ヘッダー直下に配置する、各種ランドマーク要素へのリンク集」を想定しています。ここでいうランドマーク要素とは、下記のことを指します。

- `main`: ページ本文
- `footer`: フッター
- `nav`: 左サイドバー（メインメニュー）
- `aside`: 右サイドバー（サイドメニュー）

ここで特殊なのは、**`nav`: 左サイドバー** です。これは、フロントページには存在しますが、その他のページにはない要素となります。

![フロントページには左サイドバーが存在する](/images/books/web-zukuri/page-index-01.png)

![投稿ページなどの他のページには左サイドバーがない](/images/books/web-zukuri/page-single-01.png)

なので、フロントページのみ左サイドバーのリンクアイテムを用意する必要があります。これは WordPress テーマ側で制御するため、ここではあまり問題にはなりません。

次に、画面サイズによる差異についてです。もう一度、ランドマーク要素を見てみます。

- `main`: ページ本文
- `footer`: フッター
- `nav`: 左サイドバー（メインメニュー）
- `aside`: 右サイドバー（サイドメニュー）

本テーマではデスクトップ画面とモバイル画面とで、左右サイドバーの扱いが違ってきます。なので各種画面構成を考えると、リンクアイテムは下記のように変化します。

- `main`: ページ本文
- `footer`: フッター
- デスクトップ画面時
  - `nav`: 左サイドバー（メインメニュー、フロントページのみ）
  - `aside`: 右サイドバー（サイドメニュー）
- モバイル画面時
  - `nav`: ヘッダーメニュー（メインメニュー）

先程までの内容はデスクトップ画面のものだったので、それにモバイル画面の要素を追加した形です。モバイル画面では左右サイドバーはなくなり、ヘッダー上にメニューが表示されるようになります。

![モバイル画面時のヘッダー](/images/books/web-zukuri/about-blockSkip-10.png)

![ブロックスキップではこれを反映して項目が変化する](/images/books/web-zukuri/about-blockSkip-09.png)

よって、「ページ本文」「フッター」「左サイドバー」「右サイドバー」「ヘッダーメニュー」の５種類のリンクが必要となります。そしてその一部は、デスクトップ↔モバイル画面間で表示/非表示を切り替える必要があります。

そこで、`bl_blockSkip_linkItem` にモディファイアを追加します。追加するのは下記の２つです。

|モディファイア|説明|
|---|:---|
|`bl_blockSkip_linkItem__isMobile`|モバイル画面時に表示|
|`bl_blockSkip_linkItem__isDesktop`|デスクトップ画面時に表示|

よって、構成としては下記のようになります。

```html:リンクリスト内の構成
<ul class="bl_blockSkip_linkList">
  <li class="bl_blockSkip_linkItem">
    <a href="*">基本形</a>
  </li>
  <!-- /.bl_blockSkip_linkItem -->
  <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isMobile">
    <a href="*">モバイル画面時に表示</a>
  </li>
  <!-- /.bl_blockSkip_linkItem -->
  <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isDesktop">
    <a href="*">デスクトップ画面時に表示</a>
  </li>
  <!-- /.bl_blockSkip_linkItem -->
</ul>
<!-- /.bl_blockSkip_linkList -->
```

これを実現するため、画面サイズにより `display` プロパティを切り替えるようにします。

```css
.bl_blockSkip_linkItem.bl_blockSkip_linkItem__isMobile {
  display: list-item;
}

@media (min-width: 960px) {
  .bl_blockSkip_linkItem.bl_blockSkip_linkItem__isMobile {
    display: none;
  }
}

.bl_blockSkip_linkItem.bl_blockSkip_linkItem__isDesktop {
  display: none;
}

@media (min-width: 960px) {
  .bl_blockSkip_linkItem.bl_blockSkip_linkItem__isDesktop {
    display: list-item;
  }
}
```

これでボックス内要素が完成し、ブロックスキップを構成することができました。

### リンク先について

ここでは細かい説明を省きますが、リンク先のナビゲーション要素には `id="anchor_*"` の形でリンクできるようにしておきます。

また、`nav` や `footer` などは通常 Tabキーによるフォーカスができません。また、iOS の VoiceOver で、フォーカスできない要素へ移動した際、ページの最初に戻ってしまうという事象がありました。

そのため、移動先の要素は、`tabindex="-1"` に設定しています。`-1` にしているので、通常の Tab操作ではフォーカスできませんが、リンクや JavaScript による制御によってフォーカスすることが可能となります。

https://developer.mozilla.org/ja/docs/Web/HTML/Global_attributes/tabindex

```html:ランドマーク要素へのアンカー設置
<nav id="anchor_mobileMenu" tabindex="-1">
  <!-- ヘッダーメニュー -->
</nav>
<nav id="anchor_desktopMainMenu" tabindex="-1">
  <!-- 左サイドバー -->
</nav>
<main id="anchor_mainContent" tabindex="-1">
  <!-- ページ本文 -->
</main>
<aside id="anchor_desktopSubMenu" tabindex="-1">
  <!-- 右サイドバー -->
</aside>
<footer id="anchor_footer" tabindex="-1">
  <!-- フッター -->
</footer>
```

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html
<nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集">
  <div class="bl_blockSkip">
    <p class="bl_blockSkip_title">ブロックスキップ</p>
    <p class="bl_blockSkip_description">各種ページ位置に移動できます</p>
    <ul class="bl_blockSkip_linkList">
      <li class="bl_blockSkip_linkItem">
        <a href="#anchor_mainContent" class="el_link">このページの本文</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
      <li class="bl_blockSkip_linkItem">
        <a href="#anchor_footer" class="el_link">フッター</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
      <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isMobile">
        <a href="#anchor_mobileMenu" class="el_link">メインメニュー</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
      <!-- このリンクはトップページのみ
      <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isDesktop">
        <a href="#anchor_desktopMainMenu" class="el_link">メインメニュー</a>
      </li>
      -->
      <!-- /.bl_blockSkip_linkItem -->
      <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isDesktop">
        <a href="#anchor_desktopSubMenu" class="el_link">サイドメニュー</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
    </ul>
    <!-- /.bl_blockSkip_linkList -->
  </div>
  <!-- /.bl_blockSkip -->
</nav>
<!-- /.bl_blockSkip_wrapper -->
```

```css:style.css
/* ≡≡≡ ❒ bl_blockSkip ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    ブロックスキップ
    スクリーンリーダーだけでなくキー操作者にも見えるよう、モーダルダイアログっぽく画面中央に表示する
    使用する場面は 2箇所
    - ヘッダー直後に配置、各種ランドマークへ移動可能
    - 投稿ページの記事タイトル直後、記事に関する補足情報を飛ばして記事本文に移動可能
  ■ 構成
    共通パーツ
    bl_blockSkip_wrapper : ラッパー
      bl_blockSkip : ベース
        bl_blockSkip_title : タイトル
        bl_blockSkip_description : 説明
        bl_blockSkip_linkList : リンク集
          bl_blockSkip_linkItem : リンクアイテム
          -> bl_blockSkip_linkItem__isMobile : モバイル画面のみ表示
          -> bl_blockSkip_linkItem__isDesktop : デスクトップ画面のみ表示
---------------------------------------------------------------------------- */

/* フォーカスを受け取るまで外に配置し、フォーカス時は周囲をぼかすフィルターを展開 */
.bl_blockSkip_wrapper {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 1px;
  height: 1px;
  clip-path: inset(100%);
}

.bl_blockSkip_wrapper:focus-within {
  top: 50dvh;
  left: 50dvw;
  z-index: 100;
  width: 95dvw;
  height: 95dvh;
  clip-path: none;
  backdrop-filter: blur(4px);
  transform: translate(-51%, -51%);
}

.bl_blockSkip{
  padding: 12px;
  margin-right: 24px;
  margin-left: 24px;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
  border-radius: 12px;
  box-shadow: 0 19px 38px rgba(0 0 0 0.30),
              0 15px 12px rgba(0 0 0 0.22);
}

@media (min-width: 560px) {
  .bl_blockSkip {
    padding: 12px 16px;
  }
}

.bl_blockSkip_title {
  margin-top: 16px;
  margin-bottom: 16px;
  font-size: 1.25rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .bl_blockSkip_title {
    font-size: 1.75rem;
    font-weight: 400;
  }
}

.bl_blockSkip_description {
  margin-bottom: 24px;
  font-size: 0.75rem;
  font-weight: 400;
  color: var(--sc_txt_body);
}

@media (min-width: 560px) {
  .bl_blockSkip_description {
    margin-bottom: 20px;
  }
}

.bl_blockSkip_linkList {
  padding-inline-start: 1rem;
}

.bl_blockSkip_linkItem {
  padding: 4px;
  margin: 0;
}

.bl_blockSkip_linkItem > a {
  color: var(--sc_txt_link);
  text-decoration: none;
}

.bl_blockSkip_linkItem.bl_blockSkip_linkItem__isMobile {
  display: list-item;
}

@media (min-width: 960px) {
  .bl_blockSkip_linkItem.bl_blockSkip_linkItem__isMobile {
    display: none;
  }
}

.bl_blockSkip_linkItem.bl_blockSkip_linkItem__isDesktop {
  display: none;
}

@media (min-width: 960px) {
  .bl_blockSkip_linkItem.bl_blockSkip_linkItem__isDesktop {
    display: list-item;
  }
}
```

:::

## まとめ

本項ではブロックスキップについて解説しました。

次項では、展開/格納可能となるアコーディオン要素について説明していきます。
