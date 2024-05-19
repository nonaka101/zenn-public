---
title: "📄B404：投稿ページ"
---

## このページについて

本項では、前項で作成した `page.html` を流用して新たに投稿ページのテンプレートとなる `single.html` を作成します。

![投稿ページのスクリーンショット](/images/books/web-zukuri/page-single-01.png)

## 作成

まずは `page.html` をコピーし、`single.html` にリネームします。

次に、固定ページと投稿ページの差異を確認します。固定ページのメインコンテンツは下記の構成になっていました。

- 記事タイトル
- 本文

```html:前項で作成したメインコンテンツ
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <article>
    <h1 class="el_heading_lv1">記事タイトル</h1>
    <div id="anchor_mainArticle" class="wp_blockContent" tabindex="-1">
      <!-- この中にブロックエディタによるコンテンツが入る -->
    </div>
  </article>
</main>
<!-- /.ly_mainArea_content -->
```

一方、投稿ページはこれにプラスする形で、構造としては下記のようになります。

- 記事タイトル
- （ブロックスキップ）
- 記事補足情報欄
  - 公開日
  - 更新日
  - カテゴリ
  - タグ
- サムネイル画像
- 記事本文

ここからは、これを模したページにすることで、`single.html` を作成していきます。「記事タイトル」と「記事本文」は、前項のをそのまま流用します。

### ブロックスキップ

本テーマでは、記事タイトルと本文の間に「記事補足情報欄」や「サムネイル画像」が入っています。

- 記事タイトル
- （ブロックスキップ）
- 記事補足情報欄
  - 公開日
  - 更新日
  - カテゴリ
  - タグ
- サムネイル画像
- 本文

これは「記事本文を見る前に、内容のあたりをつける」上ではメリットとなります。一方で、「記事本文までに余分な内容が詰まっている」ことがデメリットです。想定では記事補足情報欄が長くなることはないはずですが、テーマの利用者によっては、タグやカテゴリを大量につけて管理している場合も考えられます。

ここで問題となるのが、スクリーンリーダーやキーボード操作などの、コンテンツの閲覧が その順序に制限される方々です。カテゴリやタグが大量に有る場合、記事本文にたどり着くまでの読み上げ時間や操作数が多くなってしまう可能性があります。

そこで、記事タイトルから直接 記事本文に移動できるようにリンクを挿入しています。

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <article>
    <h1 class="el_heading_lv1">記事タイトル</h1>
    
    <nav class="bl_blockSkip_wrapper" aria-label="記事本文へのスキップ">
      <div class="bl_blockSkip">
        <p class="bl_blockSkip_title">ブロックスキップ</p>
        <p class="bl_blockSkip_description">カテゴリ、タグ情報等を無視して本文までスキップします</p>
        <ul class="bl_blockSkip_linkList">
          <li class="bl_blockSkip_linkItem">
            <a href="#anchor_mainArticle" class="el_link">本文へ移動</a>
          </li>
          <!-- /.bl_blockSkip_linkItem -->
        </ul>
        <!-- /.bl_blockSkip_linkList -->
      </div>
      <!-- /.bl_blockSkip -->
    </nav>
    <!-- /.bl_blockSkip_wrapper -->

    <!-- 記事補足情報欄 -->
    <!-- サムネイル画像 -->

    <div id="anchor_mainArticle" class="wp_blockContent" tabindex="-1">
      <!-- この中にブロックエディタによるコンテンツが入る -->
    </div>
</main>
<!-- /.ly_mainArea_content -->
```

### 記事補足情報欄

**B307：bl_postData**で作成した記事補足情報欄を、ブロックスキップ後方につけます。

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <article>
    <h1 class="el_heading_lv1">記事タイトル</h1>
    
    <nav class="bl_blockSkip_wrapper" aria-label="記事本文へのスキップ">
      <!-- 省略 -->
    </nav>
    <!-- /.bl_blockSkip_wrapper -->

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

    <!-- サムネイル画像 -->

    <div id="anchor_mainArticle" class="wp_blockContent" tabindex="-1">
      <!-- この中にブロックエディタによるコンテンツが入る -->
    </div>
  </article>
</main>
<!-- /.ly_mainArea_content -->
```

### サムネイル画像

最後に**B302：el_thumbnail**を記事補足情報欄の後ろにつけます。

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <article>
    <h1 class="el_heading_lv1">記事タイトル</h1>
    
    <nav class="bl_blockSkip_wrapper" aria-label="記事本文へのスキップ">
      <!-- 省略 -->
    </nav>
    <!-- /.bl_blockSkip_wrapper -->

    <div class="bl_postData">
      <!-- 省略 -->
    </div>
    <!-- /.bl_postData -->

    <div class="el_thumbnail">
      <img src="./images_readme/screenshot.png" alt="ダミーのサムネイル:このテーマのスクリーンショット">
    </div>

    <div id="anchor_mainArticle" class="wp_blockContent" tabindex="-1">
      <!-- この中にブロックエディタによるコンテンツが入る -->
    </div>
  </article>
</main>
<!-- /.ly_mainArea_content -->
```

## まとめ

https://github.com/nonaka101/web-zukuri/blob/main/docs/single.html#L369-L443

本項では、投稿ページのベースとなる `single.html` を作成しました。

次項では、アーカイブページのベース `archive.html` を作成します。
