---
title: "📄B401：フロントページ(2) カード群"
---

## このページについて

本項では、フロントページのメインコンテンツとなるカード群を作成していきます。

## 作成

**B312：bl_card** の内容を反映します。

今回行う内容は、最終的には `loop-post.php` でプログラム的に出力されるものとなります。つまり、「最新の記事」＋「固定表示設定の記事」を計５件出力します。そのため、ここでは予め作成したカードコンポーネントがきちんと想定通りの形となっているかを確認すべく、下記のものを作成します。

|パターン|リンク(`clickable`)|サムネイル|メタ情報|固定表示(`sticky`)|
|:---:|:---:|:---:|:---:|:---:|
|1|-|-|-|**+**|
|2|-|**+**|**+**|-|
|3|**+**|-|**+**|-|
|4|**+**|**+**|**+**|-|

## 完成形

下図のような形になっていれば、検証できたとして完成とします。

![フロントページのカードを上から４つ](/images/books/web-zukuri/bl_card-05.png)

:::details 全文（長いので格納しています）

```html:今回扱うmainタグ内のみ抽出
<main class="ly_mainArea_content ly_mainArea_content__middle" id="anchor_mainContent" tabindex="-1">
  <div class="bl_cardUnit bl_cardUnit__1col">

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

  </div>
  <!-- /.bl_cardUnit bl_cardUnit__1col -->
</main>
<!-- /.ly_mainArea_content -->
```

:::

## まとめ

本項では、フロントページのメインコンテンツとなるカード群を作成しました。

次項はヘッダーを作成します。
