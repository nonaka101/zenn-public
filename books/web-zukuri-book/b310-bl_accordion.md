---
title: "📄B310：bl_accordion"
---

## このページについて

ここでは、アコーディオンについて説明します。これは「サマリー」と「格納要素」で構成され、サマリーをクリックすることで要素の展開/格納を行えるものです。  
本テーマでは、「カテゴリ一覧」「年月アーカイブ一覧」に用いています。

![アコーディオンのスクリーンショット１](/images/books/web-zukuri/bl_accordion-01.png)
*格納状態*

![アコーディオンのスクリーンショット２](/images/books/web-zukuri/bl_accordion-02.png)
*展開状態*

## 構造の定義

ここでは、どのような形にするかを考えていきます。

### 基本構造

- `details`, `summary` タグを使用する（JavaScript や ARIA による制御ではなく、`html` で完結させる）
- `summary` 要素をクリックして要素の展開/格納を行う
- `summary` 要素にアローをつける（格納時は下方向、展開時は上方向）

### `details`, `summary` タグについて

アコーディオンを作成する方法としては、他にも JavaScript による制御で実現する方法もあります。自由にスタイルや構造を定義できる利点がありますが、今回はHTMLタグの `details`, `summary` による手法を取っています。これは専ら、Webアクセシビリティによる所が大きいです。

JavaScript で、例えば `div` タグを使ってアコーディオンを実現しようとすると、「それが何であるのか（グループなのか、ボタンなのか）」「展開/格納状態はどうなっているのか」など、マシンリーダブルにするための処理が必要となってきます。それらがあることで、スクリーンリーダーが読み上げる際に「〜 ボタン、展開（折りたたみ）」と、それがインタラクティブ可能な要素であることをユーザーに伝えてくれるのです。  
そのためスクリプトでの制御は複雑なコードが必要となってきます。一方 `details` 要素を使用した場合、これらをブラウザ側が自動的に補ってくれます。

また `details` を使用した場合のメリットとして、格納時はAOM上から隠すことができることも挙げられます。

![AOM上でのdetails タグの扱い、展開時に中身が表示される状況を説明している](/images/books/web-zukuri/about-details-01.png)

見た目上で要素を隠しても、きちんと設定していなければスクリーンリーダーはその内容を読み上げることがあります。`details` を使えばそのあたりも対応してくれるので、大量の情報を必要な時のみ展開したい時に便利です。

上記の問題点は、例えば `WAI-ARIA` を設定するなど、JavaScriptでも対応は可能です。ただ、特段の事情もなくどちらを選んで問題ない状態なのでしたら、HTMLタグの方を使うべきでしょう。

ただし、`details`, `summary` 要素を使う上で注意すべき箇所があります。それは `summary` 要素が `button` ロールであることです。

基本的には `summary` タグの中はテキストノードのみ存在する形が良いでしょう。やってはいけないことは、`summary` 内に `a` タグを入れることです。

本テーマではカテゴリ一覧に使う関係から当初、親子関係のあるカテゴリに対し、下記の構造にしようとしていました。

```html:不適切な例：summary内にaタグを使用している
<details>
  <summary>
    <a href="親カテゴリのアーカイブURL">親カテゴリ名</a>
  </summary>
  <a href="子カテゴリのアーカイブURL">子カテゴリ名</a>
</details>
```

このケースだと、見た目上はあまり問題には見えません。テキスト部にはリンクが仕込まれ、その周辺をクリックすると中身が展開されます。

しかしスクリーンリーダーの場合は違います。例えばVoiceOverでは「親カテゴリ名、リンク」とだけ読み上げます。それはつまり、この要素がアコーディオンであると認識できないということです。

そのため、本テーマで親子関係のあるカテゴリについては、ボタンとなるサマリー要素とリンクとを分ける必要がありました。

## 作成

上記で決めた要件から、マークアップしていきます。

### 基本となる形

`html` タグとして用意されている `details`, `summary` を用います。

```html:基本的な構造
<details>
  <summary>サマリー</summary>
  <div>
    <!-- 格納された要素 -->
  </div>
</details>
```

こうすることで、ARIAやプログラムを用いずともブラウザ上で下記の内容を実現してくれます

- 展開/格納可能な要素であることをブラウザ側が認識
- スクリーンリーダーが下記のことを行える
  - 現在の状況（展開/格納）を伝える
  - 格納時は中の要素を無視する

### クラス付け

次は要素に対してクラス付けをしていきます。  
ベースを `bl_accordion` とし、サマリーを `bl_accordion_summary`、格納要素を `bl_accordion_description` としています。

```html:クラス設定
<details class="bl_accordion">
  <summary class="bl_accordion_summary">サマリー要素</summary>
  <!-- /.bl_accordion_summary -->
  <div class="bl_accordion_description">
    格納された要素
  </div>
  <!-- /.bl_accordion_description -->
</details>
  <!-- /.bl_accordion -->
```

### スタイルの設定

次に `css` 側でスタイルを規定していきます。

#### ベース部分

まずは `bl_accordion` についてです。ここでは、下記のことを行います。

- 全体のベースとなるフォントサイズを `1rem` に設定
- テキストインデントを `1ch` に設定
- `border` はセマンティックカラーを使用
- `padding` 用のカスタムプロパティ `--padding` を作成
  - 余白は上下で `0.5em`

```css
.bl_accordion {
  --padding: 0.5em 0;

  font-size: 1rem;
  text-indent: 1ch;
  border: var(--sc_border_divider);
}
```

#### サマリー要素

次に、`bl_accordion_summary` に関する部分を設定していきます。

- テキスト色、背景色はセマンティックカラーを用いる
- 領域内に入ったカーソルは、アクション可能であることを伝えるためポインターにする
- ブラウザのデフォルトで出力される `▼` を剥ぐ
- 上記の代わりに `summary` 要素をフレックスボックス化し、擬似要素 `:after` に `∨` マークをつける
  - サイズは幅と高さ共に `1em`
  - 位置は `summary` 要素の右端（`1ch` の余白を取った上で）
  - `∨` マークは CSS の `background-image` でインラインSVGを使用
    - デフォルト（ライトモード）時は黒色
    - ダークモード時は色を反転させ、白色
  - 展開/格納時はこのマークを反転させる、そのためのアニメーションは `0.3s ease` とする

```css
/* デフォルトのスタイル（ ▼ ）を剥ぐ */
.bl_accordion > summary::-webkit-details-marker {
  display: none;
}

.bl_accordion_summary {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  padding: var(--padding);
  color: var(--sc_txt_body);
  list-style-type: none;
  cursor: pointer;
  background-color: var(--sc_bg_body__tertiary);
}

.bl_accordion_summary::after {
  width: 1em;
  height: 1em;
  margin-right: 1ch;
  margin-left: auto;
  content: "";
  background-image: url('data:image/svg+xml;utf-8,<svg viewBox="0 0 24 24" fill="black" xmlns="http://www.w3.org/2000/svg"><path d="M12 17.1L3 8L4 7L12 15L20 7L21 8L12 17.1Z"/></svg>');
  transition: all 0.3s ease;
}

@media (prefers-color-scheme: dark) {
  .bl_accordion_summary::after {
    filter: invert(1);
  }
}
```

#### 展開/格納状態のスタイル

次に展開/格納状態のスタイルを設定していきます。

- 展開→格納 にする際、`∨` マークは180度回転し `∧` マークとなる
- 展開/格納状態を、マークだけでなく背景色でもわかるように変化させる

```css
.bl_accordion[open] > .bl_accordion_summary {
  background-color: var(--sc_bg_body__secondary);
}

.bl_accordion[open] > .bl_accordion_summary::after {
  transform: rotate(180deg);
  translate:0 -12.5%;
}
```

この時、セレクタの指定は子セレクタでなければなりません。子孫セレクタにしてしまうと、アコーディオン要素の中にアコーディオン要素をネストして入れた場合に、関係ない箇所まで反応してしまうからです。

#### 格納要素のスタイル

最後に、格納要素のスタイルです。

```css
.bl_accordion_description {
  padding: 0;
  margin: 0;
  list-style: none;
}
```

### 各種アーカイブ用

ここからは、本テーマで使用しているアーカイブに絞った話となります。

各種アーカイブの場合は、格納要素に「リスト形式のリンク項目」が入ることが確定しています。そのため先ほど説明した基本形からは若干異なり、`bl_accordion_description` クラスの要素が `ul` タグになります。

#### カテゴリアーカイブ

まずはカテゴリアーカイブに関してです。WordPress では、カテゴリに親子関係をつけることができます。例えば、下記のような構成を取ることができます。

- 「旅行」カテゴリ
  - 「国内」カテゴリ
    - 「関東」カテゴリ
    - 「関西」カテゴリ
  - 「海外」カテゴリ
    - 「欧州」カテゴリ
- 「料理」カテゴリ

これをアコーディオンで表現する場合、アコーディオンをネストさせることで親子関係を表せます。自身に子カテゴリがある場合はアコーディオンで格納して、子カテゴリがない場合は単純な要素（リンクアイテム）で良さそうです。よって、構成は下記になります。

- アーカイブ本体（アコーディオン）
  - 親カテゴリ（リンクアイテム）
  - 親カテゴリ（アコーディオン）
    - 子カテゴリ（リンクアイテム）
    - 子カテゴリ（アコーディオン）

このように、リンクアイテムとアコーディオンが同列に位置することになります。

![カテゴリアーカイブ（展開状態）](/images/books/web-zukuri/bl_accordion-03.png)

見た目をアコーディオンとリンクアイテムで変えるたくないので、ここではモディファイア `bl_accordion__dummy` を作り、アコーディオンに似たスタイルを作ります。

```html:実際のカテゴリアーカイブから一部抜粋
<details class="bl_accordion">
  <summary class="bl_accordion_summary">カテゴリ一覧</summary>
  <ul class="bl_accordion_description">
    <li class="bl_accordion bl_accordion__dummy">
      <a href="./archive.html">親カテゴリ１</a>
    </li>
    <li class="bl_accordion bl_accordion__dummy">
      <a href="./archive.html">親カテゴリ２</a>
    </li>
    <li>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">親カテゴリ３</summary>
        <ul class="bl_accordion_description">
          <li class="bl_accordion bl_accordion__dummy">
            <a href="./archive.html">親カテゴリ３</a>
          </li>
          <li class="bl_accordion bl_accordion__dummy">
            <a href="./archive.html">子カテゴリ１</a>
          </li>
          <li class="bl_accordion bl_accordion__dummy">
            <a href="./archive.html">子カテゴリ２</a>
          </li>
        </ul>
      </details>
    </li>
    <li class="bl_accordion bl_accordion__dummy">
      <a href="./archive.html">親カテゴリ４</a>
    </li>
  </ul>
</details>
<!-- /.bl_accordion -->
```

```css
.bl_accordion.bl_accordion__dummy {
  padding: var(--padding);
  background-color: var(--sc_bg_body__primary);
}

.bl_accordion.bl_accordion__dummy > a {
  color: var(--sc_txt_body);
  text-decoration: none;
}

.bl_accordion.bl_accordion__dummy > a:hover {
  text-decoration: underline;
}
```

#### 年月アーカイブ

年月アーカイブの場合は、下記の構成になります。

- アーカイブ本体（アコーディオン）
  - 年（アコーディオン）
    - 月（リンク要素）

![年月アーカイブ（展開状態）](/images/books/web-zukuri/bl_accordion-04.png)

```html:実際の年月アーカイブから一部抜粋
<details class="bl_accordion">
  <summary class="bl_accordion_summary">年月アーカイブ</summary>
  <ul class="bl_accordion_description">
    <li>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">2023 年</summary>
        <ul class="bl_accordion_description">
          <li><a href="./archive.html">2023 年全体</a></li>
          <li><a href="./archive.html">9月</a></li>
          <li><a href="./archive.html">7月</a></li>
        </ul>
      </details>
    </li>
    <li>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">2022 年</summary>
        <ul class="bl_accordion_description">
          <li><a href="./archive.html">2022 年全体</a></li>
          <li><a href="./archive.html">11月</a></li>
          <li><a href="./archive.html">10月</a></li>
          <li><a href="./archive.html">9月</a></li>
          <li><a href="./archive.html">8月</a></li>
          <li><a href="./archive.html">7月</a></li>
          <li><a href="./archive.html">4月</a></li>
          <li><a href="./archive.html">3月</a></li>
          <li><a href="./archive.html">1月</a></li>
        </ul>
      </details>
    </li>
  </ul>
</details>
<!-- /.bl_accordion -->
```

本テーマの場合、年月アーカイブでこれを使います。そのため、専用のスタイルを設定します。

```css
/* 年月アーカイブ用 */
ul.bl_accordion_description > li > a {
  color: var(--sc_txt_body);
  text-decoration: none;
}

ul.bl_accordion_description > li > a:hover {
  text-decoration: underline;
}
```

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html:基本形
<details class="bl_accordion">
  <summary class="bl_accordion_summary">サマリー要素</summary>
  <!-- /.bl_accordion_summary -->
  <div class="bl_accordion_description">格納された要素</div>
  <!-- /.bl_accordion_description -->
</details>
<!-- /.bl_accordion -->
```

```html:実際のカテゴリアーカイブから一部抜粋
<details class="bl_accordion">
  <summary class="bl_accordion_summary">カテゴリ一覧</summary>
  <ul class="bl_accordion_description">
    <li class="bl_accordion bl_accordion__dummy">
      <a href="./archive.html">親カテゴリ１</a>
    </li>
    <li class="bl_accordion bl_accordion__dummy">
      <a href="./archive.html">親カテゴリ２</a>
    </li>
    <li>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">親カテゴリ３</summary>
        <ul class="bl_accordion_description">
          <li class="bl_accordion bl_accordion__dummy">
            <a href="./archive.html">親カテゴリ３</a>
          </li>
          <li class="bl_accordion bl_accordion__dummy">
            <a href="./archive.html">子カテゴリ１</a>
          </li>
          <li class="bl_accordion bl_accordion__dummy">
            <a href="./archive.html">子カテゴリ２</a>
          </li>
        </ul>
      </details>
    </li>
    <li class="bl_accordion bl_accordion__dummy">
      <a href="./archive.html">親カテゴリ４</a>
    </li>
  </ul>
</details>
<!-- /.bl_accordion -->
```

```html:実際の年月アーカイブから一部抜粋
<details class="bl_accordion">
  <summary class="bl_accordion_summary">年月アーカイブ</summary>
  <ul class="bl_accordion_description">
    <li>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">2023 年</summary>
        <ul class="bl_accordion_description">
          <li><a href="./archive.html">2023 年全体</a></li>
          <li><a href="./archive.html">9月</a></li>
          <li><a href="./archive.html">7月</a></li>
        </ul>
      </details>
    </li>
    <li>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">2022 年</summary>
        <ul class="bl_accordion_description">
          <li><a href="./archive.html">2022 年全体</a></li>
          <li><a href="./archive.html">11月</a></li>
          <li><a href="./archive.html">10月</a></li>
          <li><a href="./archive.html">9月</a></li>
          <li><a href="./archive.html">8月</a></li>
          <li><a href="./archive.html">7月</a></li>
          <li><a href="./archive.html">4月</a></li>
          <li><a href="./archive.html">3月</a></li>
          <li><a href="./archive.html">1月</a></li>
        </ul>
      </details>
    </li>
  </ul>
</details>
<!-- /.bl_accordion -->
```

```css:style.css
/* ≡≡≡ ❒ bl_accordion ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    アコーディオンブロック
    detailsタグとsummaryタグを用いるため、開閉動作にJavascriptを必要としない
  ■ 構成
    bl_accordion : ベース
    -> bl_accordion__dummy : アーカイブ一覧用のダミー要素
      bl_accordion_summary : サマリー要素
      bl_accordion_description : 格納されている要素
---------------------------------------------------------------------------- */
.bl_accordion {
  --padding: 0.5em 0;

  font-size: 1rem;
  text-indent: 1ch;
  border: var(--sc_border_divider);
}

.bl_accordion.bl_accordion__dummy {
  padding: var(--padding);
  background-color: var(--sc_bg_body__primary);
}

.bl_accordion.bl_accordion__dummy > a {
  color: var(--sc_txt_body);
  text-decoration: none;
}

.bl_accordion.bl_accordion__dummy > a:hover {
  text-decoration: underline;
}

/* デフォルトのスタイル（ ▼ ）を剥ぐ */
.bl_accordion > summary::-webkit-details-marker {
  display: none;
}

.bl_accordion_summary {
  display: flex;
  flex-flow: row nowrap;
  align-items: center;
  padding: var(--padding);
  color: var(--sc_txt_body);
  list-style-type: none;
  cursor: pointer;
  background-color: var(--sc_bg_body__tertiary);
}

.bl_accordion[open] > .bl_accordion_summary {
  background-color: var(--sc_bg_body__secondary);
}

.bl_accordion_summary::after {
  width: 1em;
  height: 1em;
  margin-right: 1ch;
  margin-left: auto;
  content: "";
  background-image: url('data:image/svg+xml;utf-8,<svg viewBox="0 0 24 24" fill="black" xmlns="http://www.w3.org/2000/svg"><path d="M12 17.1L3 8L4 7L12 15L20 7L21 8L12 17.1Z"/></svg>');
  transition: all 0.3s ease;
}

@media (prefers-color-scheme: dark) {
  .bl_accordion_summary::after {
    filter: invert(1);
  }
}

/* WordPressブロック化の際、インナーブロックを考慮して子結合子にしている */
.bl_accordion[open] > .bl_accordion_summary::after {
  transform: rotate(180deg);
  translate:0 -12.5%;
}

.bl_accordion_description {
  padding: 0;
  margin: 0;
  list-style: none;
}

/* 年月アーカイブ用 */
ul.bl_accordion_description > li > a {
  color: var(--sc_txt_body);
  text-decoration: none;
}

ul.bl_accordion_description > li > a:hover {
  text-decoration: underline;
}
```

:::

## まとめ

本項ではアコーディオン要素となる `bl_accordion` について扱いました。

次項では、検索ページで使用する検索フォーム `bl_searchForm` について扱っていきます。
