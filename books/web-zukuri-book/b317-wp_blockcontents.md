---
title: "📄B317：wp_blockContents"
---

## このページについて

本項では WordPress 専用モジュールである `wp_blockContents` について説明します。これは WordPress のブロックエディタで作成した記事を、投稿ページで問題なく表示させることを目的としています。

:::message alert

本項で行っている内容は暫定的なものであり、ブロックエディタに対して上手く対応できていません。

実装が難しい理由は、私がブロックエディタにまだ習熟していないのが大きく、それにより下記の要因が解決できていないためです。

- ブロックエディタの自由度が高く、どこまで対応すればいいのか掴めていない。
- 特にカラーモードに関する部分が難しい状態
  - エディタ側でテキスト色や背景色を指定していた場合にどうするか？
  - 背景を白と想定してテキスト色を指定している場合、ダークモードにすると見えなくなる恐れがある
  - こちらで色を反転させてしまうと、背景色とセットで見えるようにしていた場合に、逆に見えなくなる恐れがある
- `theme.json` でも制御ができそうだが、CSS側で制御するのとどちらがいいのか不明
  - 各種ブロックごとにスタイルを制御できるみたいだが、コアブロックだけでも1つずつ制御するのは非常に膨大になる
  - 更に自作ブロックやプラグインによるブロックにはどう対処すべきか・・・

:::

## 作成

上記にあるように、暫定的な対応にとどめている状態です。ここでは、見出し要素に限って制御を試みています。

### HTML構造について

本モジュールが使われるのは、投稿ページと固定ページの2つです。下図は投稿ページとなりますが、記事タイトル、補足情報、サムネイル画像があり、その下に記事本文が入っています。この「記事本文」が、モジュールの範囲となります。

![投稿ページの上部を映しており、記事に関する諸々の情報の下に、「概要」から続く記事本文が入っている](/images/books/web-zukuri/page-single-01.png)
*サムネイルより下が記事本文*

これまで作成してきた基本コンポーネントを組み合わせて、`main` タグを構成したのが下記になります。本項では一番下にある、記事本文を格納する要素のスタイリングを行っていきます。

```html:投稿ページと固定ページ
<main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1">
  <article>
    <h1 class="el_heading_lv1">記事タイトル（WordPress側のタイトルを出力）</h1>

    <!-- ブロックスキップ（キー操作用） -->
    <nav class="bl_blockSkip_wrapper" aria-label="記事本文へのスキップ">
      <!-- ブロックスキップ要素 -->
    </nav>
    <!-- /.bl_blockSkip_wrapper -->

    <!-- 記事補足情報 -->
    <div class="bl_postData">
      <!-- この中に WordPress の DB が持っている「日付情報」や「タクソノミー」を出力 -->
    </div>
    <!-- /.bl_postData -->

    <!-- サムネイル画像 -->
    <div class="el_thumbnail">
      <img src="*" alt="サムネイル画像の説明文">
    </div>
    <!-- 記事本文（ブロックコンテンツ） -->
    <div id="anchor_mainArticle" class="wp_blockContent" tabindex="-1">
      <!-- この中に WordPress の DB が持っている「記事本文」を出力 -->
    </div>
  </article>
</main>
```

### 見出し要素

**B303：el_heading** で行っている見出し要素のスタイルを、ここでは試みます。あちらでは `el_heading_lv*` というクラスに対してスタイリングを施していましたが、ここではブロック要素が入る `wp_blockContents` 内に対し、`h1` 〜 `h6` までの要素タグをセレクタとしています。

## 完成形

```css:style.css
/* ≡≡≡ ⚠ wp_blockContent ⚠ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    WordPressのブロックエディタ用のスタイル
    （投稿、固定ページの `the_content()` 箇所を囲う形で使用）
  ■ 注意
    - ブロックテーマに対応させる方法が思い浮かばないための暫定的な処置となっている。
    - 目標としては、他のテーマで使っていたデータを、問題なくこのテーマにも適用できること。
    - ただし、そのためにはダークモードに関する色、レイアウト等に対応させる必要がある。
    - theme.json 側での処理も考えているが、各ブロックに直接書いていくのは現実的でないと判断。
    - そのためここでは、必要最小限となる要素にのみスタイルをつけている。
    - 上記より、エディタ側とのスタイルの違いが生じるという問題が生じている。
---------------------------------------------------------------------------- */
.wp_blockContent h1 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 2rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .wp_blockContent h1 {
    font-size: 2.25rem;
    font-weight: 400;
    line-height: 1.4;
  }
}

.wp_blockContent h2 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 1.75rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .wp_blockContent h2 {
    font-size: 2rem;
    font-weight: 400;
  }
}

.wp_blockContent h3 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_3);
  font-size: 1.5rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .wp_blockContent h3 {
    font-size: 1.75rem;
    font-weight: 400;
  }
}

.wp_blockContent h4 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1.25rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .wp_blockContent h4 {
    font-size: 1.5rem;
    font-weight: 400;
  }
}

.wp_blockContent h5 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 500;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .wp_blockContent h5 {
    font-size: 1.25rem;
    font-weight: 400;
    line-height: 1.5;
  }
}

.wp_blockContent h6 {
  margin-top: var(--space_3);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .wp_blockContent h6 {
    font-size: 1rem;
    font-weight: 500;
  }
}

```

## まとめ

本項では、WordPress専用モジュールとなる `wp_blockContents` を扱いました。これで基本コンポーネントは全て完了しました。

次項からは、各種ページを作成していくことになります。
