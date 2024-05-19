---
title: "📄B406：検索ページ"
---

## このページについて

本項はページテンプレート最後となる、検索ページを作成します。

![検索ページのスクリーンショット](/images/books/web-zukuri/page-search-01.png)

## 作成

まずは `archive.html` をコピーし、`search.html` にリネームします。`archive.html` で作成した、カード群やページネーションはそのまま活用できますので `main` 内の要素はそのままにしておいてください。

次に、検索ページのメインコンテンツの構成を確認します。

- ページタイトル「サイト内検索」
- サイト内検索の説明テキスト
- 検索フォーム
  - 表示件数（セレクト）
  - 検索キーワード（テキスト）
  - 検索ボタン
- 区切り線
- 見出し「検索結果」
- 検索キーワードとヒット件数
- カード群
  - カード（サムネイル無し、クリッカブル）
- ページネーション（ページ区切りが発生する場合）

前項のアーカイブページで、既に「ページタイトル」「カード群」「ページネーション」は作成済みです。なので、これら上下端の要素を除いた真ん中の要素を作成すれば、検索ページが作れそうです。

```html:アーカイブページから検索ページへ編集していく
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <h1 class="el_heading_lv1">アーカイブ名（年月、カテゴリなど）</h1>

  <!-- サイト内検索の説明テキスト -->
  <!-- 検索フォーム -->
  <!-- 区切り線 -->
  <!-- 見出し「検索結果」 -->
  <!-- 検索キーワードとヒット件数 -->

  <div class="bl_cardUnit bl_cardUnit__1col">

    <article class="bl_card bl_card__clickable">
      <a class="bl_card_main" href="./single.html">
        <div class="bl_card_body">
          <h2 class="bl_card_title">Read me</h2>
          <p class="bl_card_content">
            ここでは、このプロジェクトの主旨や注意事項等についてまとめてあります。
            また、投稿ページのサンプルも兼ねています。
          </p>
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
    <!-- /.bl_card bl_card__clickable -->

    <!-- 以降のカードは省略 -->

  </div>
  <!-- /.bl_cardUnit bl_cardUnit__1col -->

  <nav class="bl_pager" aria-label="ページ割り">
    <div class="bl_pager_left">
      <a class="bl_pager_btn" aria-label="最初のページ" href="#">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" height="24" width="24">
          <path fill-rule="evenodd"
            d="M11.854 3.646a.5.5 0 0 1 0 .708L8.207 8l3.647 3.646a.5.5 0 0 1-.708.708l-4-4a.5.5 0 0 1 0-.708l4-4a.5.5 0 0 1 .708 0zM4.5 1a.5.5 0 0 0-.5.5v13a.5.5 0 0 0 1 0v-13a.5.5 0 0 0-.5-.5z">
          </path>
        </svg>
      </a>
      <!-- /.bl_pager_btn -->
      <a class="bl_pager_btn" aria-label="前のページ" href="#">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" height="24" width="24">
          <path fill-rule="evenodd"
            d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z">
          </path>
        </svg>
      </a>
      <!-- /.bl_pager_btn -->
      <a class="bl_pager_btn bl_pager_btn__extended" href="#">4</a>
      <!-- /.bl_pager_btn bl_pager_btn__extended -->
      <a class="bl_pager_btn bl_pager_btn__extended" href="#">5</a>
      <!-- /.bl_pager_btn bl_pager_btn__extended -->
      <a class="bl_pager_btn bl_pager_btn__extended" href="#">6</a>
      <!-- /.bl_pager_btn bl_pager_btn__extended -->
    </div>
    <!-- /.bl_pager_left -->
    <div class="bl_pager_center" aria-current="page">7/15</div>
    <!-- /.bl_pager_center -->
    <div class="bl_pager_right">
      <a class="bl_pager_btn bl_pager_btn__extended" href="#">8</a>
      <!-- /.bl_pager_btn bl_pager_btn__extended -->
      <a class="bl_pager_btn bl_pager_btn__extended" href="#">9</a>
      <!-- /.bl_pager_btn bl_pager_btn__extended -->
      <a class="bl_pager_btn bl_pager_btn__extended" href="#">10</a>
      <!-- /.bl_pager_btn bl_pager_btn__extended -->
      <a class="bl_pager_btn" aria-label="次のページ" href="#">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" height="24" width="24">
          <path fill-rule="evenodd"
            d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708z">
          </path>
        </svg>
      </a>
      <!-- /.bl_pager_btn -->
      <a class="bl_pager_btn" aria-label="最後のページ" href="#">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" height="24" width="24">
          <path fill-rule="evenodd"
            d="M4.146 3.646a.5.5 0 0 0 0 .708L7.793 8l-3.647 3.646a.5.5 0 0 0 .708.708l4-4a.5.5 0 0 0 0-.708l-4-4a.5.5 0 0 0-.708 0zM11.5 1a.5.5 0 0 1 .5.5v13a.5.5 0 0 1-1 0v-13a.5.5 0 0 1 .5-.5z">
          </path>
        </svg>
      </a>
      <!-- /.bl_pager_btn -->
    </div>
    <!-- /.bl_pager_right -->
  </nav>
  <!-- /.bl_pager -->

</main>
<!-- /.ly_mainArea_content -->
```

### ページタイトル＋ページ説明文

まずはページタイトルと、サイト内検索の説明テキストを設置します。

:::message

以降のHTML文では、カード群とページネーション要素のマークアップは、説明に不要なため省略します。

:::

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <h1 class="el_heading_lv1">サイト内検索</h1>
  <p>検索結果は検索フォームの下に表示されます。</p>

  <!-- 検索フォーム -->
  <!-- 区切り線 -->
  <!-- 見出し「検索結果」 -->
  <!-- 検索キーワードとヒット件数 -->

  <!-- カード群（省略） -->
  <!-- ページネーション（省略） -->
</main>
<!-- /.ly_mainArea_content -->
```

### 検索フォーム＋区切り線

**B311：bl_searchForm**で作成した検索フォームを挿入します。また、後方の要素とを区切るため、`hr` で区切り線を設けます。

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <h1 class="el_heading_lv1">サイト内検索</h1>
  <p>検索結果は検索フォームの下に表示されます。</p>

  <form action="#" class="bl_searchForm_wrapper">
    <div class="bl_searchForm">
      <label for="num" class="bl_searchForm_label">表示件数</label>
      <div class="bl_searchForm_inputSelect">
        <select name="n" id="num">
          <option value="5" selected>5</option>
          <option value="10">10</option>
          <option value="20">20</option>
          <option value="50">50</option>
          <option value="100">100</option>
        </select>
      </div>
    </div>
    <!-- /.bl_searchForm -->
    <div class="bl_searchForm">
      <label for="txt" class="bl_searchForm_label">検索キーワード</label>
      <div class="bl_searchForm_input">
        <input type="search" name="s" id="txt" class="bl_searchForm_inputText">
        <button type="submit" class="bl_searchForm_btn">
          <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" height="24" width="24">
            <path
              d="M21 20.5L15 14.5C16.2 13.2 16.9 11.4 16.9 9.5C17 5.4 13.6 2 9.5 2C5.4 2 2 5.4 2 9.5C2 13.6 5.3 17 9.5 17C11.2 17 12.7 16.4 14 15.5L20 21.5L21 20.5ZM3.5 9.5C3.5 6.2 6.2 3.5 9.5 3.5C12.8 3.5 15.5 6.2 15.5 9.5C15.5 12.8 12.8 15.5 9.5 15.5C6.2 15.5 3.5 12.8 3.5 9.5Z">
            </path>
          </svg>
          <span>検索</span>
        </button>
      </div>
    </div>
    <!-- /.bl_searchForm -->
  </form>

  <hr>

  <!-- 見出し「検索結果」 -->
  <!-- 検索キーワードとヒット件数 -->

  <!-- カード群（省略） -->
  <!-- ページネーション（省略） -->
</main>
<!-- /.ly_mainArea_content -->
```

### 検索結果見出し＋検索キーワードとヒット件数

最後に「検索結果」の見出しと、検索キーワードとヒット件数を表示します。見出し構成による移動をしやすいよう、見出しは `h2` を使用します。

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <h1 class="el_heading_lv1">サイト内検索</h1>
  <p>検索結果は検索フォームの下に表示されます。</p>

  <form action="#" class="bl_searchForm_wrapper">
    <!-- 検索フォーム（省略） -->
  </form>

  <hr>

  <h2 class="el_heading_lv2">検索結果</h2>
  <p>
  「***」の検索結果<span>（全75件中、1～5件を表示）</span>
  </p>

  <!-- カード群（省略） -->
  <!-- ページネーション（省略） -->
</main>
<!-- /.ly_mainArea_content -->
```

:::message

検索キーワードやヒット件数については、WordPress 側で動的に決まったものを出力する動きになります。

:::

## まとめ

https://github.com/nonaka101/web-zukuri/blob/main/docs/search.html#L369-L618

これで全てのページテンプレート化の土台となる静的ページが完成しました。第三章ではこれらを基に、ページテンプレートに書き換えていくことになります。
