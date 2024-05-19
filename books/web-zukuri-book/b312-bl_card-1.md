---
title: "📄B312：bl_card(1)"
---

## このページについて

ここからはブロックモジュール最後のコンポーネントとなる、カード `bl_card` について説明していきます。  
文量の関係から２回に分け、本項では基本となる部分を見ていきます。そしてそれをベースとし、次項で様々なバリエーションについて見ていきます。

![カードコンポーネント（フロントページのもの）](/images/books/web-zukuri/bl_card-01.png =400x)

## カードの役割について

まずは、カードの役割について説明していきます。どこで使われるのか、WordPress と どう関係しているか、などです。

### WordPressの記事データについて

WordPress の記事情報は本文以外にも、様々な情報が含まれています。

![イメージ図、記事本文のデータにタグ、カテゴリ、日付情報がくっついて、DBに格納されている図](/images/books/web-zukuri/flow-create-page-02.png)

WordPress の投稿記事管理画面を見てみましょう。

![WordPressの投稿編集ページ（スラッグやらカテゴリ、タグ等が編集できる）](/images/books/web-zukuri/bl_postData-00.png)

上図から見て分かる通り、記事情報には下記の項目が付与されています。

- タクソノミー（カテゴリ、タグなど）
- スラッグ
- 日付

これらの情報は、サイト管理者が記事を管理するのに役立ちます。またサイト閲覧者にとってもカテゴリやタグ、日付の情報が使えると 検索の指針になったり、同カテゴリや同月の記事を探すのに役立ちそうです。本テーマでは、こうした情報を利用できるようにしており、その代表となるのがカードとなります。

### カードが使用される場所について

カードが使用される場所は、下記のとおりです。

#### フロントページ

![カードコンポーネント（フロントページのもの）](/images/books/web-zukuri/bl_card-01.png)

サイトのフロントページでは、最新記事やスティッキー設定（※）された記事が出力されます。ここでは、データベース上に登録されている下記の記事情報が出力されています。  
（※ スティッキー：WordPress では特定の記事を固定表示するように設定できます）

- サムネイル（なければ、出力しない）
- 記事タイトル
- 記事の要旨（なければ、本文最初の文章）
- 投稿日
- 更新日
- メタ情報（所属するカテゴリ一覧）

これが記事情報を可能な限り使った場合のカードとなります。

:::details タグ情報を使用しない理由について

タグに関しては投稿ページ本体で使用するにとどめ、カードでは使用しません。

カテゴリを使いタグを使用しない理由は、それぞれの位置づけによるものです。本テーマでは、**カテゴリは最小で１、多くても３程度**である一方、**タグは制限なくつけられるもの**と考えています。そのためカードにした際の（読み上げ）文量を考え、タグは含めないようしています。

本テーマでは、カテゴリとは「親子関係を持つ、ツリー構造を持つもの」と考えています。下図は、図書館で用いられる日本十進分類法（NDC）を参考に、カテゴリの関係を説明しようとしたものです。

![カテゴリが木構造となり、そこに記事が付与されているイメージ](/images/books/web-zukuri/about-taxonomy-01.png)
*例：書籍（分野に応じた分類がされており、木構造の形を取る）*

大きな分類から細かな分類に向けて、垂直方向に管理されていることがわかります。この関係性は親子関係であり、例えば「情報工学」に属する記事は、親に「電気工学」「工学」のカテゴリを持つことになります。

一方のタグは、カテゴリとは違った横断的なものと考えています。カテゴリによって垂直方向に分類付けられた記事群を、水平的に繋げているイメージと私は考えています。下図は先程の図書分類を使い、タグを表現しようとしたものです。

![上記のカテゴリ・記事に対し、タグが横断的に繋がりを表している](/images/books/web-zukuri/about-taxonomy-02.png)
*上図のカテゴリに対し、タグは様々な観点から、横断的にリンク構造を取る*

親子関係で分類されるものとは別の視点で、記事間を紐づけていることが見て取れます。ここでは「作者」「発表時期」「キーワード」といった視点で、記事を結びつけています。

ここまでの説明から、カテゴリとタグの関係をもう一度考えます。カテゴリ自体は親子関係を持っていますが、上図では1つの記事に対して1つのカテゴリを割り当てていました。一方のタグは、1つの記事に対して（視点によっては）いくつも紐づけることができます。

そのため、読み上げ量が多くなる可能性のあるタグについては、カードのメタ情報には含めないようしています。

:::

#### アーカイブ・検索ページ

![カードコンポーネント（アーカイブと検索ページで使われているもの）](/images/books/web-zukuri/bl_card-02.png)

アーカイブページと検索ページでは、同じカード形式となっております。ここでは、フロントページからサムネイルを除いた形となっています。

#### フッター

![カードコンポーネント（フッターにあるもの）](/images/books/web-zukuri/bl_card-03.png)

最後にフッターにある、「最新の投稿」です。ここでは全体的にフォントサイズを小さくしたうえで、下記の情報を載せています。

- 記事タイトル
- 投稿日
- 更新日
- メタ情報（所属するカテゴリ一覧）

## 構造の定義

では、これまでの内容から構造を決めていきます。前述した通り、記事情報をフルで使った場合のカード要素は、下記のようになります。

- サムネイル
- 記事タイトル
- 記事の要旨
- 投稿日
- 更新日
- メタ情報（カテゴリ一覧）

そしてカードに求められる要件は、下記のようになります。

- タイトルはマシンリーダブルにするため `h2`（あるいは `h3` ）を使用
- 日付データはマシンリーダブルにするため `time` タグを使用
- サムネイルがない場合を想定
- メタ情報（カテゴリ一覧）がない場合を想定
- 日付データとメタ情報（カテゴリ一覧）は折り返しを想定

また次項の内容に踏み込んだ話になりますが、クリック可能にした場合の想定も考慮していきます。カードをクリックした際に投稿ページへ移動するようリンクを仕込むのとは別に、メタ情報（カテゴリ一覧）をクリックした際には該当するカテゴリのアーカイブページに移るようしたいです。`a` タグ内に `a` タグをネストさせることはできないので、このあたりの構造を踏まえて作成していく必要があります。

## 作成

上記で決めた要件から、マークアップしていきます。

### 基本的な構成

初めにベースとなる部分から作成していきます。

まず、カード１つで単一記事となるので これを `article` で囲みます。これはどのバリエーションでも同じです。

```html:ベースはarticle
<article class="bl_card">
  <!-- この中に諸要素 -->
</article>
<!-- /.bl_card -->
```

次に `article` の子要素ですが、「メタ情報（カテゴリ一覧）」と「それ以外」に分けます。メタ情報側は `bl_card_meta`、それ以外（≒本体）は `bl_card_main` とし、`div` で分けてみます。

```html
<article class="bl_card">
  <div class="bl_card_main">
    <!-- サムネイル -->
    <!-- タイトル -->
    <!-- 要旨 -->
    <!-- 日付データ -->
  </div>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <!-- カテゴリリスト -->
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card -->
```

この理由は、クリック範囲を分けるためです。

[構造の定義](#構造の定義)の最後で説明した通り、`a` タグ内に `a` タグをネストすることはできません。ここでは記事本体をクリックした場合は投稿ページへ、カテゴリリンクをクリックした場合はアーカイブページへリンクを構成したいので、この段階で要素を分けることでネストしないようにしています。

```html:aタグをつけた場合
<article class="bl_card">
  <a class="bl_card_main" href="投稿ページへのリンク">
    <!-- 諸要素 -->
  </a>
  <!-- /.bl_card_main -->
  <div class="bl_card_meta">
    <ul>
      <li><a href="アーカイブページへのリンク">カテゴリリンク１</a></li>
      <li><a href="アーカイブページへのリンク">カテゴリリンク２</a></li>
    </ul>
  </div>
  <!-- /.bl_card_meta -->
</article>
<!-- /.bl_card -->
```

:::message

`bl_card_meta` 部は、カテゴリリストを入れるのは確定しているため `div` でなく直接 `ul` で記述することもできます。ただし、今後拡張してそれ以外の要素を入れる可能性があるため、`div` を噛ませています。

:::

### ベース部のスタイリング

現在の構成は下記のようになっています。

- `bl_card`: ベース要素
  - `bl_card_main`: サムネイル、タイトル、要旨、日付データ
  - `bl_card_meta`: カテゴリリスト

各子要素の中身に入る前に、ここまでのクラスに対するスタイリングを行っていきます。まずは全体を構成する `bl_card` です。

ここでは主に、下位要素のベースとなる設定を行います。

- 余白を管理するカスタムプロパティ `--card_space`
- カードの角丸R値を管理するカスタムプロパティ `--card_r`
- デフォルトの `a` タグスタイルを打ち消す意味でも、`color` はセマンティックカラーを使用
- ベースとなる `font-size` を `1rem` に設定
  - ベースクラスで全体を管理する（下位要素の `font-size` には `em` を使用）
  - 次項のバリエーションで小サイズを作る際はここを `0.8rem` にすることで表現

```css
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
```

次に子要素となる `bl_card_main`, `bl_card_meta` です。

- 両者の組み合わせで角丸を表現
  - `bl_card_main` 側で左上右上の角丸
  - `bl_card_meta` 側で左下右下の角丸
- `bl_card_main` について
  - `a` タグ使用を踏まえ、下線は打ち消しておく
- `bl_card_meta` について
  - 縦方向折り返しなしの `flex` 要素
  - 余白を設定
    - 左右余白に `--card_space` を使用（`main` 側はサムネイルの関係から下位で設定）
  - `border` を設定
    - 上側は `main` とのつながりがあるので実線でなく点線にする

```css
/* メイン部 */
.bl_card_main {
  display: block;
  text-decoration: none;
  border-top-left-radius: var(--card_r, 0);
  border-top-right-radius: var(--card_r, 0);
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
```

`border` に関して、`bl_card_meta` 側はこれで完了となりますが、`bl_card_main` 側はもう少し工夫が必要です。

まず、またカードはサムネイルを含む場合と含まない場合があります。詳細は後述しますが、`bl_card_main` の子要素は下記のようにしていく予定です。

- `bl_card`
  - `bl_card_main`
    - `bl_card_thumbnail`: サムネイル画像の要素
    - `bl_card_body`: テキスト要素
      - タイトル
      - 要旨
      - 日付データ
  - `bl_card_meta`

そのため、画像を格納する `bl_card_thumbnail` の有無に関わらず角丸を表現する必要があります。これは擬似クラス `:first-child` を使い、最初にくる要素に対してスタイリングするようにします。

次に WordPress テーマ化した際の出力内容です。メタ情報がない場合、`bl_card_meta` 要素自体が出力されないようにしなければなりません。そうなると`bl_card_main` 単体でも、四隅で角丸が作れるようにスタイルを規定する必要があります。これは擬似クラス関数 `:has()` と `:not()` を使い、`bl_card_meta`**を兄弟に持たない場合**という指定をすることで表現します。

これらのケースを踏まえ、最終的は**カードをborderで囲み、四隅が丸められるように**という要件を達成するようスタイリングしていきます。もう一度構成を見てみます。

- `bl_card`
  - `bl_card_main`
    - `bl_card_thumbnail`
    - `bl_card_body`
  - `bl_card_meta`

ここで必要なスタイルは、`bl_card_main` の子要素に対する下記の要件です。

- 最初の要素の場合
  - 上、左、右に `border` を設定
  - 左上右上の角を丸める
- 最初の要素*でない*場合
  - 左、右に `border` を設定
- `bl_card_meta` がない場合の最後の要素
  - 左、右、下に `border` を設定
  - 左下右下の角を丸める

```css:main側のborder設定（metaが無いことを踏まえて）
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
```

ベース部に関しては、これで完了です。

### `bl_card_main` 要素

ここからは `bl_card_main` 内を見ていきます。

- 左右の余白について、テキスト要素にのみ設定
  - このため、サムネイル要素 `bl_card_thumbnail` とテキスト要素 `bl_card_body` に分ける
- `bl_card_thumbnail` について
  - 中に `img` タグが入る想定（これは WordPress の `the_post_thumbnail` で出力）
- `bl_card_body` について
  - タイトル要素には `h2` もしくは `h3` を使用（場所によって使い分ける）
  - 要旨要素には `p` タグを使用
  - 日付データは、それが何であるかのラベルと関連付ける
    - `span` を用いて、「投稿」「更新」と表記する
    - `time` を用いてマシンリーダブルな形にする
    - 両者の間に `:` で区切りを設ける（スタイルで行う）

::::details 個人的メモ

`bl_card_thumbnail` については、`div` を挟まず直接 `img` を使うこともできそうです。WordPress 側で出力する際、`the_post_thumbnail()` の引数に`['class'=>'bl_card_thumbnail']` を渡せば、クラス付きの `img` タグを生成できます。

今後カードコンポーネントは変更する可能性があります。その際に、ここも合わせて変更するかもしれません。

:::details カード変更案

スクリーンリーダーの移動機能を考慮し、並び順を変える。

現在の構成では見出し要素が中間にあります。そのため見出しによる拾い読みをしている場合、上方にあるサムネイル画像の `alt` 文を読むのに手間がかかってしまいます。

`bl_card_body` を `bl_card_thumbnail` の前に持ってくれば、`article` 内で最初に読み上げられるのが見出し要素となります。また情報の順番を考えると、下記の順番にしたほうが重要度に沿っているような感じです。

1. タイトル
2. 要旨
3. 日付
4. サムネイル
5. メタ情報

:::

::::

上記の要件を基に、構造を作っていきます。

- `div.bl_card_main`
  - `div.bl_card_thumbnail`: サムネイル要素を格納
    - `img`: WordPress で出力を想定
  - `div.bl_card_body`: テキスト諸要素を格納
    - `h2.bl_card_title`: 記事タイトル
    - `p.bl_card_content`: 要旨
    - `div.bl_card_dates`: 日付データ群を格納
      - `div.bl_card_date`: 単一の日付データ
        - `span`: ラベル
        - `time`: 日付

`div.bl_card_dates` で「投稿日」「更新日」の日付データをラップしているのは、中の要素を `flex-flow: row wrap;` で横並びにする想定だからです。またその中で `div.bl_card_date > span::after{...}` を使い、`time` との間に `:` を追加できるようにします。

```html:bl_card_main内の構成
<div class="bl_card_main">
  <div class="bl_card_thumbnail">
    <img src="画像パス">
  </div>
  <!-- /.bl_card_thumbnail -->
  <div class="bl_card_body">
    <h2 class="bl_card_title">タイトル</h2>
    <p class="bl_card_content">要旨</p>
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
</div>
<!-- /.bl_card_main -->
```

構造ができたので、次はそれに対するスタイルを設定していきます。

```css:bl_card_main内のスタイル
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
```

これで、`bl_card_main` 側は完成です。

### `bl_card_meta` 要素

次に、`bl_card_meta` 部を作っていきます。

- 現段階では**所属するカテゴリ一覧**をリストとして出力する
  - 今後の変更・拡張の可能性は残しておく（≒リスト要素を格納する形で `bl_card_meta` を用意）
- カテゴリ一覧について
  - `ul` を使い、カテゴリの一覧であることを伝えるラベルを用意
  - 中は `li` と `a` タグによる構成
  - ボタンのような形にして、横並びに羅列する
    - ホバー時に色を変えるなど、押せることがわかるように表現する

以上の要件から、構造を作っていきます。

```html:bl_card_meta内の構成
<div class="bl_card_meta">
  <ul class="bl_card_categories" aria-label="カテゴリ">
    <li><a href="アーカイブのURL">カテゴリ１</a></li>
    <li><a href="アーカイブのURL">カテゴリ２</a></li>
  </ul>
  <!-- /.bl_card_categories -->
</div>
<!-- /.bl_card_meta -->
```

次はこれに対するスタイルです。

```css:bl_card_meta内のスタイル
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

これで `bl_card_meta` 側が完成し、`bl_card` の基本形が完成しました。

## 完成形

最後に完成形をまとめておきます。

:::details フロントページのカード（HTML）

```html:フロントページのカード
<article class="bl_card">
  <div class="bl_card_main">
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
  </div>
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
<!-- /.bl_card -->
```

:::

:::details メタ情報なし（HTML）

```html:メタ情報なし
<article class="bl_card">
  <div class="bl_card_main">
    <div class="bl_card_thumbnail">
      <img src="./assets/images/m_02_white.png">
    </div>
    <!-- /.bl_card_thumbnail -->
    <div class="bl_card_body">
      <h2 class="bl_card_title">このサイトについて</h2>
      <p class="bl_card_content">このサイトは、デジタル庁の「デザインシステム」を基に試作したものになります。</p>
      <div class="bl_card_dates">
        <div class="bl_card_date">
          <span>投稿</span>
          <time datetime="2023-10-16">2023/10/16</time>
        </div>
        <!-- /.bl_card_date -->
        <div class="bl_card_date">
          <span>更新</span>
          <time datetime="2023-11-15">2023/11/15</time>
        </div>
        <!-- /.bl_card_date -->
      </div>
      <!-- /.bl_card_dates -->
    </div>
    <!-- /.bl_card_body -->
  </div>
  <!-- /.bl_card_main -->
</article>
<!-- /.bl_card -->
```

:::

:::details サムネイルがない、フロントページ外のカード（HTML）

```html:サムネイルなし（例：フロントページ外のカード）
<article class="bl_card">
  <div class="bl_card_main">
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
  </div>
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
    bl_card : ベース
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

このページでは、カードを扱う `bl_card` について、基本的な形式のものを扱いました。  
次回は様々なところで使われるカードのバリエーションを、今回の内容に加えていく作業を行います。
