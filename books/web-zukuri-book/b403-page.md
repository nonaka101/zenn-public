---
title: "📄B403：固定ページ"
---

## このページについて

前項で、`404.html` （404ページ）を作成しました。本項ではこれを用いて、固定ページを作成していきます。

![固定ページのスクリーンショット](/images/books/web-zukuri/page-page-01.png)

## 作成

まずは `404.html` をコピーし、`page.html` にリネームします。`index.html`（フロントページ）ではなく `404.html` を使用しているのは、ページレイアウトがこちらの方が近いからです。フロントページを除く各ページは、全て `404.html` のグリッド構成（左サイドバーなし、右９グリッドをメインコンテンツに使用）となっています。

次に`404.html` で作った `main` 要素内は、あらかじめ消しておきます。その上で、固定ページのメインコンテンツ部を作成していきます。

固定ページのメインコンテンツは、下記のような単純な構成となっています。

- 記事タイトル
- 記事本文

ここからは、この2つの要素を作成していきます。

### 記事タイトル

`main` 内に１つの記事を配置することになるので、`article` でラップします。その中に、見出し要素 `h1` で記事タイトルをつけます。

```html
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <article>
    <h1 class="el_heading_lv1">記事タイトル</h1>
    <!-- 記事本文 -->
  </article>
</main>
<!-- /.ly_mainArea_content -->
```

### 記事本文

これは Wordpress のブロックエディタで作成した内容となります。これを制御する目的で、**B317：wp_blockContents**を使いたいため、`div`要素でラップすることになります。またブロックスキップの移動先として、`id`によるフラグメントと `tabindex="-1"` を付与しておきます。

:::message alert

**B317：wp_blockContents**を使う理由は、コンテンツとカラーモードを対応させるためですが、現状は未解決の状態です。

:::

```html
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

:::message

本テーマのリポジトリでは、暫定的に各コンポーネントをサイズ調整可能な状態でおいています。

:::

## まとめ

https://github.com/nonaka101/web-zukuri/blob/main/docs/page.html

本項では固定ページを、メインコンテンツを調整することで作成しました。

次項の投稿ページに関しても、ここで作成した内容を使い作成していきます。この2つはWordPressのテーマ化する過程で、最終的には1つのページテンプレートとなります。
