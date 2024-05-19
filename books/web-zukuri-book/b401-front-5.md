---
title: "📄B401：フロントページ(5) フッター"
---

## このページについて

本項では、フッター要素を扱っていきます。既に**B313：ly_footer**でレイアウトについては構成済みですので、ここではそれを使っていくことになります。その上で、ウィジェットで使う細々した下記の要素を作成していきます。

- ウィジェットトップのタイトル要素 `ly_footer_headerTitle`
- ユーティリティウィジェット内の要素
  - サイトメモとなる、テキストエリア `el_memo`
  - アドレスを構成する `bl_address`
  - コピーライト文 `el_copyright`

## 作成

### B313：ly_footerより

これまでで、フッターのレイアウトに関しては下記のように作成できています。本項では `div.ly_footer_widget` の中身となる、各ウィジェットを作成していくことになります。下記にある構成についての詳細は、**B313：ly_footer**の方を参照ください。

```html
<footer class="ly_footer" id="anchor_footer" tabindex="-1">
  <div class="ly_footer_widgetArea">
    <div class="ly_footer_widget ly_footer_widget__3fr">
      <!-- 左側の 3fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->
    <div class="ly_footer_widget ly_footer_widget__6fr">
      <!-- 中央の 6fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__6fr -->
    <div class="ly_footer_widget ly_footer_widget__3fr">
      <!-- 左側の 3fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->
  </div>
  <!-- /.ly_footer_widgetArea -->
  <div class="ly_footer_copyright">
    <p class="el_copyright">
      2023 Web-Zukuri（コピーライト用テキスト）
    </p>
    <!-- /.el_copyright -->
  </div>
  <!-- /.ly_footer_copyright -->
</footer>
<!-- /.ly_footer -->
```

### 各ウィジェットの共通部

ウィジェットは計３つあり、扱う内容についてはバラバラです。ただし、最初に*タイトル要素を設ける*という構造については共通させるようにします。

なので、各ウィジェットの構成は下記のようになります。

- ウィジェットのタイトル
  1. サイトロゴ＋サイト名
  2. 最新の投稿
  3. ユーティリティ
- コンテンツ本体
  1. ホームへのリンク＋メインメニュー（左サイドバーと同一）
  2. カード３件（小サイズ）
  3. ユーティリティ要素
     - メモ
     - 外部リンク
     - サイトナビゲーション
     - アドレス

この「タイトル要素」については `ly_footer` の一部と考えます。親となる `ly_footer_widget` と結びつきが強く、別モジュール化する意義が薄いからです。よって、その子部品名は `ly_footer_headerTitle` とします。

`ly_footer_headerTitle` の要件は下記のようになります。

- 要素は２つ考えられる
  - １つ目は見出し要素 `h2`
  - ２つ目は `div` 要素で、下記を格納する
    - サイトロゴ画像を扱う `img`
    - サイト名を扱う `span`

```css:style.css
.ly_footer_headerTitle {
  display: flex;
  flex-direction: row;
  gap: 8px;
  margin-top: 0;
  margin-bottom: 1.75rem;
  font-size: 1.75rem;
  color: var(--sc_txt_body);
  text-decoration: none;
}

.ly_footer_headerTitle > img {
  max-height: 1.75rem;
}
```

### 左側ウィジェット

共通部ができたので、ここからは各ウィジェットの中身を作成していきます。

左側ウィジェットに関しては、タイトル欄にヘッダーと同じ「サイトロゴ」と「サイト名」の要素を格納します。コンテンツ自体は、前項の左サイドバーの中身と同一になります。

```html:左側ウィジェット
<div class="ly_footer_widget ly_footer_widget__3fr">
  <div class="ly_footer_headerTitle">
    <img src="./assets/images/Dummy_logo.PNG" alt="Logo">
    <span>Web-Zukuri</span>
  </div>
  <!-- /.ly_footer_headerTitle -->
  <div class="bl_tagMenu">
    <!-- 前項『B401：フロントページ(4) サイドバー＋ダイアログ』の左サイドバーと同一 -->
  </div>
  <!-- /.bl_tagMenu -->
</div>
<!-- /.ly_footer_widget ly_footer_widget__3fr -->
```

:::message

同一であることからも分かるように、WordPressに落とし込む際にはここもテンプレートパーツ化を使うことになります。

:::

### 中央ウィジェット

次に中央ウィジェットです。ここは**B312：bl_card**で作成した「小サイズ」のカードを並べることになります。

```html:中央ウィジェット
<div class="ly_footer_widget ly_footer_widget__6fr">
  <h2 class="ly_footer_headerTitle">最新の投稿</h2>
  <div class="bl_cardUnit bl_cardUnit__1col">

    <article class="bl_card bl_card__small bl_card__clickable">
      <a class="bl_card_main" href="./single.html">
        <div class="bl_card_body">
          <h3 class="bl_card_title">Read me</h3>
          <div class="bl_card_dates">
            <div class="bl_card_date">
              <span>投稿</span>
              <time datetime="2023-10-16">2023/10/16</time>
            </div>
            <!-- /.bl_card_date -->
            <div class="bl_card_date">
              <span>更新</span>
              <time datetime="2023-11-08">2023/11/08</time>
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
          <li><a href="./archive.html">ReadME</a></li>
        </ul>
        <!-- /.bl_card_categories -->
      </div>
      <!-- /.bl_card_meta -->
    </article>
    <!-- /.bl_card bl_card__small bl_card__clickable -->
    <!-- 以下省略 -->

  </div>
  <!-- /.bl_cardUnit bl_cardUnit__1col -->
</div>
<!-- /.ly_footer_widget ly_footer_widget__6fr -->
```

:::message

このカード出力も、WordPressではテンプレートパーツで扱うことになります。「最新の投稿記事を○件分出力する」と、動的な内容になるためです。

:::

### 右側ウィジェット

最後に、右側のウィジェットになります。ここは「ユーティリティウィジェット」として、下記の役割を持っています。

- サイト閲覧者に対するメモ欄（注意事項など）
- サイト管理者のSNSページなどをまとめた外部リンク一覧
- 各種プライバシーポリシーなどをまとめたサイトナビゲーション
- アドレス（住所や電話番号、メールアドレスなど）

これらは、WordPressのテーマカスタマイザーを使って設定できるようにしています。よってその詳細は第三章で説明することになります。ここでは「WordPressで設定した内容を取得し、それを静的ページに落とし込む」という想定で、HTML構成を考えていきます。

```html:右側ウィジェット
<div class="ly_footer_widget ly_footer_widget__3fr">
  <h2 class="ly_footer_headerTitle">ユーティリティ</h2>
  <p class="el_memo">
    このサイトはテスト用のものです。
  </p>
  <div class="bl_tagMenu">
    <span class="bl_tagMenu_title" aria-hidden="true" id="external_link_in_footer">外部リンク</span>
    <ul class="bl_tagMenu_list" role="list" aria-labelledby="external_link_in_footer">
      <li class="bl_tagMenu_item">
        <a href="#" class="bl_tagMenu_link">YouTube</a>
      </li>
      <!-- /.bl_tagMenu_item -->
      <!-- 以下省略 -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_tagMenu">
    <span class="bl_tagMenu_title" aria-hidden="true" id="footer_menu_in_footer">サイトナビゲーション</span>
    <ul class="bl_tagMenu_list" role="list" aria-labelledby="footer_menu_in_footer">
      <li class="bl_tagMenu_item">
        <a href="#" class="bl_tagMenu_link">サイトマップ</a>
      </li>
      <!-- /.bl_tagMenu_item -->
      <!-- 以下省略 -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_address">
    <h3 class="bl_address_title">アドレス</h3>
    <address class="bl_address_item">
      <span class="bl_address_name">住所</span>
      〒000-0000<wbr>
      A県B市C区D町<wbr>
      123-4
    </address>
    <!-- /.bl_address_item -->
    <p class="bl_address_item">
      <span class="bl_address_name">Tel</span>
      000-0000-0000
    </p>
    <!-- /.bl_address_item -->
    <p class="bl_address_item">
      <span class="bl_address_name">Mail</span>
      sample@sample.jp
    </p>
    <!-- /.bl_address_item -->
  </div>
  <!-- /.bl_address -->
</div>
<!-- /.ly_footer_widget ly_footer_widget__3fr -->
```

「外部リンク」や「サイトナビゲーション」については、**カスタムメニューの内容をタグメニュー化して出力する**という、左右サイドバーの項で説明した内容とほぼ同じになるので、ここではその詳細は省略しています。

「サイト閲覧者に対するメモ欄」と「アドレス」については、タグメニュー化には適さず独自のスタイルを作る必要があります。ここでは、メモ欄は単独要素となるためエレメントモジュール `el_memo` に、アドレスは複数要素の集合体なのでブロックモジュール `bl_address` を新規に作っています。

メモ欄に関するスタイル要件は、複雑なものは不要です。ここではフォントサイズや余白に関する内容にとどめています。

```css:style.css
.el_memo {
  margin: 1rem 0;
  font-size: 0.8em;
}
```

アドレス欄については、フォントや余白にプラスし、「名前：値」の形式で各要素を横並びになるようフレックスボックスとして管理しています。スタイルの大部分は、**B307：bl_postData**で作った内容から流用して作ったと記憶しています。

```css:style.css
.bl_address {
  margin-bottom: 40px;
}

.bl_address_title {
  margin: 0 0 0.5rem;
  font-size: 1.25rem;
}

.bl_address_item {
  display: flex;
  flex-flow: row wrap;
  margin: 0.5rem 0;
  font-size: 0.8rem;
  color: var(--sc_txt_body__small);
}

.bl_address_name {
  flex: none;
}

.bl_address_name::after {
  content: " : ";
}
```

## 完成形

最後に、完成形を載せておきます。

:::details HTML（長いので格納しています）

```html
<footer class="ly_footer" id="anchor_footer" tabindex="-1">
  <div class="ly_footer_widgetArea">
    <div class="ly_footer_widget ly_footer_widget__3fr">
      <div class="ly_footer_headerTitle">
        <img src="./assets/images/Dummy_logo.PNG" alt="Logo">
        <span>Web-Zukuri</span>
      </div>
      <!-- /.ly_footer_headerTitle -->
      <div class="bl_tagMenu">
        <!-- 前項『B401：フロントページ(4) サイドバー＋ダイアログ』の左サイドバーと同一 -->
      </div>
      <!-- /.bl_tagMenu -->

    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->

    <div class="ly_footer_widget ly_footer_widget__6fr">
      <h2 class="ly_footer_headerTitle">最新の投稿</h2>
      <div class="bl_cardUnit bl_cardUnit__1col">

        <article class="bl_card bl_card__small bl_card__clickable">
          <a class="bl_card_main" href="./single.html">
            <div class="bl_card_body">
              <h3 class="bl_card_title">Read me</h3>
              <div class="bl_card_dates">
                <div class="bl_card_date">
                  <span>投稿</span>
                  <time datetime="2023-10-16">2023/10/16</time>
                </div>
                <!-- /.bl_card_date -->
                <div class="bl_card_date">
                  <span>更新</span>
                  <time datetime="2023-11-08">2023/11/08</time>
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
              <li><a href="./archive.html">ReadME</a></li>
            </ul>
            <!-- /.bl_card_categories -->
          </div>
          <!-- /.bl_card_meta -->
        </article>
        <!-- /.bl_card bl_card__small bl_card__clickable -->
        <!-- 以下省略 -->

      </div>
      <!-- /.bl_cardUnit bl_cardUnit__1col -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__6fr -->

    <div class="ly_footer_widget ly_footer_widget__3fr">
      <h2 class="ly_footer_headerTitle">ユーティリティ</h2>
      <p class="el_memo">
        このサイトはテスト用のものです。
      </p>
      <div class="bl_tagMenu">
        <span class="bl_tagMenu_title" aria-hidden="true" id="external_link_in_footer">外部リンク</span>
        <ul class="bl_tagMenu_list" role="list" aria-labelledby="external_link_in_footer">
          <li class="bl_tagMenu_item">
            <a href="#" class="bl_tagMenu_link">YouTube</a>
          </li>
          <!-- /.bl_tagMenu_item -->
          <!-- 以下省略 -->
        </ul>
        <!-- /.bl_tagMenu_list -->
      </div>
      <!-- /.bl_tagMenu -->
      <div class="bl_tagMenu">
        <span class="bl_tagMenu_title" aria-hidden="true" id="footer_menu_in_footer">サイトナビゲーション</span>
        <ul class="bl_tagMenu_list" role="list" aria-labelledby="footer_menu_in_footer">
          <li class="bl_tagMenu_item">
            <a href="#" class="bl_tagMenu_link">サイトマップ</a>
          </li>
          <!-- /.bl_tagMenu_item -->
          <!-- 以下省略 -->
        </ul>
        <!-- /.bl_tagMenu_list -->
      </div>
      <!-- /.bl_tagMenu -->
      <div class="bl_address">
        <h3 class="bl_address_title">アドレス</h3>
        <address class="bl_address_item">
          <span class="bl_address_name">住所</span>
          〒000-0000<wbr>
          A県B市C区D町<wbr>
          123-4
        </address>
        <!-- /.bl_address_item -->
        <p class="bl_address_item">
          <span class="bl_address_name">Tel</span>
          000-0000-0000
        </p>
        <!-- /.bl_address_item -->
        <p class="bl_address_item">
          <span class="bl_address_name">Mail</span>
          sample@sample.jp
        </p>
        <!-- /.bl_address_item -->
      </div>
      <!-- /.bl_address -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->
  </div>
  <!-- /.ly_footer_widgetArea -->
  <div class="ly_footer_copyright">
    <p class="el_copyright">
      2023 Web-Zukuri（コピーライト用テキスト）
    </p>
    <!-- /.el_copyright -->
  </div>
  <!-- /.ly_footer_copyright -->
</footer>
<!-- /.ly_footer -->
```

:::

:::details CSS（長いので格納しています）

```css:style.css
/* ≡≡≡ ◲ ly_footer ◲ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    フッター部のレイアウトを構成する
    デスクトップ画面の時は、横に `3:6:3` の比率で3つのウィジェットを並べる。
    モバイル画面の時は、ウィジェットは縦1列に並べる
  ■ 構成
    ly_footer : ベース
      ly_footer_widgetArea : ウィジェット格納要素
        ly_footer_widget : ウィジェットのベース（モバイル画面で 12fr ）
        -> ly_footer_widget__3fr : デスクトップ画面で 3fr 使うモディファイア
        -> ly_footer_widget__6fr : デスクトップ画面で 6fr 使うモディファイア
          ly_footer_headerTitle : ウィジェット先頭のタイトル要素
      ly_footer_copyright : 最下部のコピーライト用
        el_copyright : コピーライトテキスト
---------------------------------------------------------------------------- */
.ly_footer {
  position: relative;
  padding: 64px 24px;
  margin-top: 40px;
  background-color: var(--sc_bg_body__tertiary);
}

.ly_footer_copyright {
  position: absolute;
  right: 8px;
  bottom: 4px;
  left: 8px;
  padding: 0;
  margin: 0;
}

.el_copyright {
  font-size: 0.75em;
  color: var(--sc_txt_body__small);
  text-align: center;
}

.ly_footer_widgetArea {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  grid-auto-flow: row;
  gap: 1rem 0;
  max-width: 1024px;
  margin-right: auto;
  margin-left: auto;
}

@media (min-width: 560px) {
  .ly_footer_widgetArea {
    padding: 0 40px;
  }
}

@media (min-width: 960px) {
  .ly_footer_widgetArea {
    gap: 0 2rem;
    padding: 0;
  }
}

.ly_footer_widget {
  grid-area: span 1/span 12;
}

@media (min-width: 960px) {
  .ly_footer_widget.ly_footer_widget__3fr {
    grid-area: span 1/span 3;
  }

  .ly_footer_widget.ly_footer_widget__6fr {
    grid-area: span 1/span 6;
  }
}

.ly_footer_headerTitle {
  display: flex;
  flex-direction: row;
  gap: 8px;
  margin-top: 0;
  margin-bottom: 1.75rem;
  font-size: 1.75rem;
  color: var(--sc_txt_body);
  text-decoration: none;
}

.ly_footer_headerTitle > img {
  max-height: 1.75rem;
}



/* ≡≡≡ ❒ bl_address ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    アドレス用
  ■ 構成
    bl_address : ベース
      bl_address_title : このモジュール全体のタイトル
      bl_address_item : アイテム
        bl_address_name : アイテム名
---------------------------------------------------------------------------- */
.bl_address {
  margin-bottom: 40px;
}

.bl_address_title {
  margin: 0 0 0.5rem;
  font-size: 1.25rem;
}

.bl_address_item {
  display: flex;
  flex-flow: row wrap;
  margin: 0.5rem 0;
  font-size: 0.8rem;
  color: var(--sc_txt_body__small);
}

.bl_address_name {
  flex: none;
}

.bl_address_name::after {
  content: " : ";
}


/* ≡≡≡ ☐ el_memo ☐ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    メモ用
    ユーティリティウィジェットに使用
---------------------------------------------------------------------------- */
.el_memo {
  margin: 1rem 0;
  font-size: 0.8em;
}
```

:::

## まとめ

本項では、フッター要素を扱いました。

ここまでで、フロントページの大部分が完成しました。次項はその完結にあたり、残った部分（ブロックスキップ、ページトップに戻るボタン、開発ツール）を盛り込んでいきます。
