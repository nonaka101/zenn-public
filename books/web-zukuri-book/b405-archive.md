---
title: "📄B405：アーカイブページ"
---

## このページについて

本項では、アーカイブページのテンプレートとなる `archive.html` を作成していきます。

![アーカイブページのスクリーンショット](/images/books/web-zukuri/page-archive-01.png)

## 作成

まずは `404.html` をコピーし、`archive.html` にリネームします。`main` 内にある要素は削除して構いません。

次に、アーカイブページのメインコンテンツを確認します。アーカイブページのメインコンテンツは下記のような構成です。

- アーカイブタイトル
- カード群
  - カード（サムネイル無し、クリッカブル）
- ページネーション（ページ区切りが発生する場合）

ここからは、この構成を再現していきます。

### アーカイブタイトル

まずは、アーカイブタイトルを `h1` 要素で作ります。アーカイブタイトルとは、そのページでアーカイブされている要素の名前になります。例えばカテゴリ「旅行」の記事をアーカイブしている場合は「旅行」になりますし、「2023年12月」作成の記事でアーカイブしている場合は「2023年12月」がタイトルになります。

```html:アーカイブタイトル
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <h1 class="el_heading_lv1">アーカイブ名（年月、カテゴリなど）</h1>
  <!-- カード群 -->
  <!-- ページネーション -->
</main>
<!-- /.ly_mainArea_content -->
```

### カード群

次にカード群です。**B312：bl_card**で作成したものを利用して、カード化した記事を配置します。

```html:カード群の配置（最初の１つのみ配置、残りは省略）
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <h1 class="el_heading_lv1">アーカイブ名（年月、カテゴリなど）</h1>

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

  <!-- ページネーション -->

</main>
<!-- /.ly_mainArea_content -->
```

### ページネーション

![ページネーション要素のスクリーンショット](/images/books/web-zukuri/page-archive-02.png)

最後にページ区切りをするための、ページネーション要素を**B308：bl_pager**を使って配置します。この要素の本来の動きとしては、WordPressでクエリを走らせ、抽出した記事の件数により変化するものです。静的ページである本項ではレイアウトや挙動を見たいので、要素がフルで使われるケースを想定して作っています。

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <h1 class="el_heading_lv1">アーカイブ名（年月、カテゴリなど）</h1>

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

これでアーカイブページのメインコンテンツは完了です。

## まとめ

https://github.com/nonaka101/web-zukuri/blob/main/docs/archive.html#L369-L603

本項では、アーカイブページを作成しました。

次項ではページテンプレートの最後となる、検索ページを作成します。
