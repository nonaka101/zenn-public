---
title: "📄B307：bl_postData"
---

## このページについて

ここでは投稿ページで使われる、記事情報を掲載する `bl_postData` について説明します。

![bl_postData の画像。投稿日、更新日、カテゴリ、タグ一覧、の情報が羅列されている。](/images/books/web-zukuri/bl_postData-01.png)

まずはこれがどのようなものかについて話していきます。

## 記事データについて

![single.htmlの見本](/images/books/web-zukuri/page-single-01.png)

上図は投稿ページを撮ったものです。この投稿ページでは１つの記事を扱っており、下記で構成されています。

- 記事タイトル
- （ブロックスキップ）
- 記事補足情報欄
  - 公開日
  - 更新日
  - カテゴリ
  - タグ
- サムネイル画像
- 本文

:::message

ブロックスキップはマウス操作では表示されないコンテンツです。Tabキーによる操作などで確認することができます。

:::

### WordPress上での記事データの扱い

![WordPress管理画面の投稿一覧の画面。記事の1つをクイック編集状態にし、記事に付随する情報を見せている。そこにはスラッグ、カテゴリ、タグなどの情報がある。](/images/books/web-zukuri/bl_postData-00.png)

これは WordPress 上での投稿記事の管理画面になります。このように投稿記事はタイトルと本文だけでなく、他にも様々な情報が補足情報として含まれます。代表的なものとしては、下記があります

- 作成日
- 更新日
- スラッグ（記事固有のIDのようなもの）
- タクソノミー（カテゴリやタグ等）

これらの情報も含め、投稿ページが作られます。大まかなイメージとして、下図のように考えてください。

![構造図（記事本文、タイトル、タクソノミー、日付データがDB上にあり、それらを拾って、投稿ページのテンプレートに流し込まれ、ページが作られる過程を説明している）](/images/books/web-zukuri/flow-create-page-02.png)
*DB上には記事本文だけでなく補足情報も含まれており、それらも引っ張ってきてページを作ることができる*

またこれらの情報は、「同じ作成月の記事群」「同じカテゴリ（タグ）の記事群」でアーカイブページを作ったりと、様々な場面で使用することができます。

### 本テーマでの記事データの利用

本テーマでは、**投稿データに関する情報が利用できればユーザーにとって利便性が上がる**と考えています。そこでこれらの情報を下記のように利用しています。

- 作成日にリンクを付与（同月の記事をまとめたアーカイブページ）
- 更新日は表示のみ（アーカイブを必要とするものではないと判断）
- カテゴリ・タグにリンクを付与（同じタクソノミーを持つ記事のアーカイブページ）

そしてこれらをまとめているのが、`bl_postData` となります。

#### 補足情報の配置に関する私見

他サイトでは、カテゴリやタグを載せない所も見受けられます。また載せる場合でもその置き方は様々で、本文前に置くパターンや、本文の末尾に置くパターンもあります。  
ここでは、本テーマで*補足情報を記事の先頭に持ってきた理由*について、私の考えを話します。

::::details 私見（格納しています）

端的に言ってしまうと、私の個人的な好みになります。

様々なサイトを利用する中で、「今から見る記事はどのような内容なのか」について先に掲示があったほうが、内容の理解がスムーズに済むことが多く感じられました。また、記事を探している時には「それが目当てのものなのか」についてもアタリがつけやすく、（そうした情報を載せているサイトに対し）ありがたいなと感じているのが大きいです。

:::message

ただし、この手法にはデメリットがあります。それは「記事本文までに余分な情報が詰まってしまう」ことです。マウスやタッチ操作においては、これは大した問題ではないかもしれませんが、スクリーンリーダーやキーボード操作のユーザーにとっては、場合によっては不便に感じてしまうことに繋がります。

そこで本テーマでは、「ブロックスキップ（**B309：bl_blockSkip**）」を使用することでそれを解消しようとしています。詳しくは、そちらで解説することにします。

:::

::::

## 構造の定義

ここでは、どのような形にしていくかを考えていきます。

- キー・バリューの形で要素名、値が横に並ぶ
- （両者を並べるに）画面幅が十分でない場合は縦に並べる
- 日付データは一項目、タクソノミーは複数項目を並べる前提とする
- 要素はliとして出力する想定
- 作成日とタクソノミーは、WordPressのアーカイブページへのリンクを付与する想定（アンダーラインをつけてリンクであることを伝える）
- 更新日についてはリンクを貼らない（更新日でアーカイブを表記することは想定していない）

## 作成

上記で決めた要件から、マークアップしていきます。

### ベース要素

まずは一番外側である、ベース要素からです。ここでは部品名を `postData` としています。

```html
<div class="bl_postData">
  <!-- この中に、各種「名前 - 値」が並ぶ -->
</div>
<!-- /.bl_postData -->
```

スタイルについては、下方向のマージンのみ設定します。

```css
.bl_postData {
  margin-bottom: var(--space_2);  /* -> 1rem */
}
```

### 各種項目を格納する要素

`bl_postData` の下位に、「公開日」「更新日」「カテゴリ」「タグ」の項目要素が並ぶようにします。ここでは、これらの項目要素を格納する子部品名を `bl_postData_item` とします。

```html
<div class="bl_postData">
  <div class="bl_postData_item">
    <!-- 「公開日」 -->
  </div>
  <!-- /.bl_postData_item -->
  <div class="bl_postData_item">
    <!-- 「更新日」 -->
  </div>
  <!-- /.bl_postData_item -->
  <div class="bl_postData_item">
    <!-- 「カテゴリ」 -->
  </div>
  <!-- /.bl_postData_item -->
  <div class="bl_postData_item">
    <!-- 「タグ」 -->
  </div>
  <!-- /.bl_postData_item -->
</div>
<!-- /.bl_postData -->
```

次にスタイルです。この `bl_postData_item` 内には「項目名」と「値」が入ります。画面幅に余裕があるときは両者を横並びに置き、余裕がないときは縦に配置します。

![記事補足情報欄は、項目ごとに1行ずつ使用している](/images/books/web-zukuri/bl_postData-02.png)
*画面幅に余裕がある場合*

![記事補足情報欄は、タグ一覧の項目は1行にまとめられないので行を分けて表示できるようにしている](/images/books/web-zukuri/bl_postData-03.png)
*タグ要素を横並びにできる余裕がない*

![記事補足情報欄は、項目の名前も値も行を分割し、縦長に情報を表示している](/images/books/web-zukuri/bl_postData-04.png)
*両者とも横並びにできる余裕がない*

そのため、`bl_postData_item` を `display: flex;` にし、`flex-flow: row wrap;` にすることで実現します。

```css
.bl_postData {
  margin-bottom: var(--space_2);
}

.bl_postData_item {
  display: flex;
  flex-flow: row wrap;
  gap: 0.125rem 0.25rem;
  align-items: baseline;
  margin-bottom: 0.5rem;
}
```

### 各種項目

ここからは、`bl_postData_item` の中身を作っていきます。

`bl_postData_item` 内は、`display: flex;` の `flex-flow: row wrap;` となっているので、子要素には「項目名」と「値」を置きます。

この「値」には、公開日や更新日などのように単一であることが確定しているものもありますが、カテゴリやタグは複数あることが予測されます。そこで、「値」には複数入れるように `ul` 要素を使います。

```html
<div class="bl_postData">
  <div class="bl_postData_item">
    <div class="bl_postData_name">項目名</div>
    <ul class="bl_postData_values">
      <li class="bl_postData_value">
        <!-- この中に a 要素や日付を表すための time 要素が入る -->
      </li>
    </ul>
    <!-- /.bl_postData_values -->
  </div>
  <!-- /.bl_postData_item -->
</div>
<!-- /.bl_postData -->
```

クラスの名前と構造を示すと、下記のようになります。

- `bl_postData`: ベース
  - `bl_postData_item`: 各種項目を格納
    - `bl_postData_name`: 項目名
    - `bl_postData_values`: 値を格納するリスト
      - `bl_postData_value`: 値

次にスタイルです。

- 項目名と値の間には `:` をつける
- 横並びの際、各項目の値の開始位置は揃える
  - → 項目名は長くても４文字なので、項目名に `8ch` の幅を持たせて実現
- 値が複数ある場合は、各値を横並びに並べる

```css
.bl_postData_name {
  flex: 0 0 8ch;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: var(--sc_txt_body);
}

.bl_postData_name::after {
  content: ":";
}

.bl_postData_values {
  display: flex;
  flex: auto;
  flex-flow: row wrap;
  gap: 0.1rem 0.3rem;
  padding: 0;
  margin: 0;
  font-size: 0.875rem;
  font-weight: 500;
  line-height: 1.5;
  list-style-type: none;
}

.bl_postData_value {
  flex: none;
}
```

### 値

最後に値について見ていきます。リスト要素に絞って見てみると、「値」に入る内容は、下記のようになります。

```html
<!-- 「公開日」：a + time タグ -->
<ul class="bl_postData_values">
  <li class="bl_postData_value">
    <a href="作成月アーカイブのリンク"><time datetime="2023-10-16">2023年10月16日</time></a>
  </li>
</ul>

<!-- 「更新日」：time タグのみ（更新月アーカイブは想定しないため） -->
<ul class="bl_postData_values">
  <li class="bl_postData_value">
    <time datetime="2023-11-08">2023年11月08日</time>
  </li>
</ul>

<!-- 「カテゴリ」：a タグのみ -->
<ul class="bl_postData_values">
  <li class="bl_postData_value">
    <a href="カテゴリアーカイブのリンク">カテゴリA</a>
  </li>
</ul>

<!-- 「タグ」：a タグのみ -->
<ul class="bl_postData_values">
  <li class="bl_postData_value">
    <a href="カテゴリアーカイブのリンク">タグA</a>
  </li>
</ul>
```

使われるタグは `time`, `a` であることがわかりました。次はそれらへのスタイルです。

```css
.bl_postData_value > a,
.bl_postData_value > time {
  color: var(--sc_txt_body);
}
```

両者とも文字を黒色にしています。その上で `a` タグにはデフォルトでアンダーラインがつくので、リンクであることもわかります。

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html
<div class="bl_postData">
  <div class="bl_postData_item">
    <div class="bl_postData_name">公開日</div>
    <ul class="bl_postData_values">
      <li class="bl_postData_value">
        <a href="./archive.html"><time datetime="2023-10-16">2023年10月16日</time></a>
      </li>
    </ul>
    <!-- /.bl_postData_values -->
  </div>
  <!-- /.bl_postData_item -->

  <div class="bl_postData_item">
    <div class="bl_postData_name">更新日</div>
    <ul class="bl_postData_values">
      <li class="bl_postData_value">
        <time datetime="2023-11-08">2023年11月08日</time>
      </li>
    </ul>
    <!-- /.bl_postData_values -->
  </div>
  <!-- /.bl_postData_item -->

  <div class="bl_postData_item">
    <div class="bl_postData_name">カテゴリ</div>
    <ul class="bl_postData_values">
      <li class="bl_postData_value"><a href="./archive.html">サンプルページ</a></li>
      <li class="bl_postData_value"><a href="./archive.html">ReadME</a></li>
    </ul>
    <!-- /.bl_postData_values -->
  </div>
  <!-- /.bl_postData_item -->

  <div class="bl_postData_item">
    <div class="bl_postData_name">タグ一覧</div>
    <ul class="bl_postData_values">
      <li class="bl_postData_value"><a href="./archive.html">WordPress</a></li>
      <li class="bl_postData_value"><a href="./archive.html">デジタル庁</a></li>
      <li class="bl_postData_value"><a href="./archive.html">デザインシステム</a></li>
      <li class="bl_postData_value"><a href="./archive.html">Webアクセシビリティ</a></li>
      <li class="bl_postData_value"><a href="./archive.html">試作版</a></li>
      <li class="bl_postData_value"><a href="./archive.html">GitHub</a></li>
      <li class="bl_postData_value"><a href="./archive.html">はじめに</a></li>
    </ul>
  </div>
  <!-- /.bl_postData_item -->
</div>
<!-- /.bl_postData -->
```

```css:style.css
/* ≡≡≡ ❒ bl_postData ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    投稿ページのタイトル下に表示される情報欄
    記事の補足情報である「公開日」「カテゴリ」「タグ」を表示
  ■ 構成
    bl_postData : ベーススタイル
      bl_postData_item : 公開日、カテゴリ、タグ それぞれの情報スペース
        bl_postData_name : item の名前「カテゴリ」「タグ」
        bl_postData_values : item の中身欄
          bl_postData_value : item の中身
---------------------------------------------------------------------------- */
.bl_postData {
  margin-bottom: var(--space_2);
}

.bl_postData_item {
  display: flex;
  flex-flow: row wrap;
  gap: 0.125rem 0.25rem;
  align-items: baseline;
  margin-bottom: 0.5rem;
}

.bl_postData_name {
  flex: 0 0 8ch;
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.5;
  color: var(--sc_txt_body);
}

.bl_postData_name::after {
  content: ":";
}

.bl_postData_values {
  display: flex;
  flex: auto;
  flex-flow: row wrap;
  gap: 0.1rem 0.3rem;
  padding: 0;
  margin: 0;
  font-size: 0.875rem;
  font-weight: 500;
  line-height: 1.5;
  list-style-type: none;
}

.bl_postData_value {
  flex: none;
}

.bl_postData_value > a,
.bl_postData_value > time {
  color: var(--sc_txt_body);
}
```

:::

## まとめ

本項では、投稿ページに使用する記事の補足情報 `bl_postData` について扱いました。

次項はページネーション要素（アーカイブや検索等で記事数がページ内に収まらない場合に、ページを区切るために用いられるもの）について扱います。
