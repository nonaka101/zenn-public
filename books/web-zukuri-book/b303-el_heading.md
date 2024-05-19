---
title: "📄B303：el_heading"
---

## このページについて

ここでは、文章中の見出しを扱う `el_heading` について説明していきます。  

![見出し一覧](/images/books/web-zukuri/el_heading-01.png)
*余白がわかるよう`hr`要素で区切りを挿入している*

ここでは、`@media` クエリを使い、デスクトップ↔モバイル画面間で差異を設けることについても扱っていきます。

## 構造の定義

ここでは、どのような形にしていくかについて考えていきます。

- 『デジタル庁デザインシステム（Ver 1.3.2）』の内容に準拠させる
  - `font-size`
  - `font-weight`
  - `line-height`
  - `letter-spacing`
  - 見出し要素上下の `margin`
- （ただし、原典で `px` 単位となっている箇所は `16px = 1rem` で置換）

`rem` 単位への置き換えは、本テーマの特徴となるところです。ブラウザ側の設定値を引き継ぎ、ユーザーに適した大きさになるようする意図があります。  
この際に変化する部分（＝ 相対単位の `rem`, `em` を使用する箇所）は、読みやすさに繋がる下記になります。

- `font-size`
- `letter-spacing`
- `margin-top`
- `margin-bottom`

:::message

イメージとしては「読む」という行為に直接関わる箇所となります。それ以外（例：装飾用スペース）では `px` を使うこともあります。

:::

## 作成

上記で決めた要件から、マークアップしていきます。

なお `html` での使用は、下記のような形を想定します。

```html
<h1 class="el_heading_lv1">見出しレベル１</h1>
<p>段落文</p>

<h2 class="el_heading_lv2">見出しレベル２</h2>
<p>段落文</p>

<h3 class="el_heading_lv3">見出しレベル３</h3>
<p>段落文</p>

<h4 class="el_heading_lv4">見出しレベル５</h4>
<p>段落文</p>

<h5 class="el_heading_lv5">見出しレベル５</h5>
<p>段落文</p>

<!-- ▼ 構成を保ったまま、見た目のみ変更する場合 ▼ -->
<h5 class="el_heading_lv6">見出しレベル５（スタイルは見出しレベル６）</h5>
<p>段落文</p>
```

:::message

上記の一番最後に、「`heading` 要素に直接スタイル付けするのでなく、エレメントモジュールを使用する」理由が表現されています。

前後の流れから見出し文字の大きさを調整したい場合が出てきたとします。その場合、適切な見出し構造を崩して `heading` 要素で調整することは望ましくありません。そうした事態に対処するため、要素セレクタでなくモジュールにスタイリングを施しています。

:::

### カスタムプロパティ

デジタル庁デザインシステムには、スタイルの項に*スペースに関する規定*があります。

> 8px をスペーシングの基本とします。大きなサイズのスペーシングはフィボナッチ数列を用いて定義し、小さなサイズのスペーシングは 2 で割り定義します。
> 『デジタル庁デザインシステム（Ver 1.3.2） スタイル 余白』より抜粋

そこで、再利用できるように `:root` 要素にカスタムプロパティとして用意します。

```css:カスタムプロパティ
:root {
  /* スペース */
  --space_0: 0.25rem;
  --space_1: 0.5rem;
  --space_2: 1rem;
  --space_3: 1.5rem;
  --space_5: 2.5rem;
  --space_8: 4rem;
  --space_13:6.5rem;
}
```

### ６つのバリエーションについて

今回、`h1`〜`h6` までのバリエーションを用意することになります。ただし共通するスタイルは少ないため モディファイア（※） は使わず、下記の６つのクラスを用意します。

- `el_heading_lv1`
- `el_heading_lv2`
- `el_heading_lv3`
- `el_heading_lv4`
- `el_heading_lv5`
- `el_heading_lv6`

（※モディファイア：共通するスタイルを持つ、複数のバリエーションを構成するための手法。こちらはエレメントモジュールの最後**B305：el_btn**で取り扱います）

### 通常時（モバイル画面）のスタイル

本テーマは基本的にモバイルファーストで設計しています。ですのでまずはそちらから規定していきます。

```css:通常スタイル
.el_heading_lv1 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 2rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

.el_heading_lv2 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 1.75rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

.el_heading_lv3 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_3);
  font-size: 1.5rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

.el_heading_lv4 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1.25rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

.el_heading_lv5 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 500;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

.el_heading_lv6 {
  margin-top: var(--space_3);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.7;
  letter-spacing: 0.04em;
}
```

### デスクトップ画面時のスタイル

デザインシステムでは、画面スペースに余裕の少ないモバイル画面と、余裕のあるデスクトップ画面とで、スタイルを分けています。基本的にデスクトップ画面では、`font-size` を大きくし、`font-weight` を小さくしているようです。

ですので、ビューポート `960px` 以上をデスクトップ画面とし、`@media` クエリを使って反映していきます。

```css:960px以上をデスクトップ画面としている
.el_heading_lv1 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 2rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv1 {
    font-size: 2.25rem;
    font-weight: 400;
    line-height: 1.4;
  }
}

.el_heading_lv2 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 1.75rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv2 {
    font-size: 2rem;
    font-weight: 400;
  }
}

.el_heading_lv3 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_3);
  font-size: 1.5rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv3 {
    font-size: 1.75rem;
    font-weight: 400;
  }
}

.el_heading_lv4 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1.25rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv4 {
    font-size: 1.5rem;
    font-weight: 400;
  }
}

.el_heading_lv5 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 500;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv5 {
    font-size: 1.25rem;
    font-weight: 400;
    line-height: 1.5;
  }
}

.el_heading_lv6 {
  margin-top: var(--space_3);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv6 {
    font-size: 1rem;
    font-weight: 500;
  }
}
```

これで完成となります。

## 完成形

最後に `html`, `css` をまとめたものを載せておきます。

:::details 全文（長いので格納しています）

```html
<h1 class="el_heading_lv1">見出しレベル１</h1>
<h2 class="el_heading_lv2">見出しレベル２</h2>
<h3 class="el_heading_lv3">見出しレベル３</h3>
<h4 class="el_heading_lv4">見出しレベル５</h4>
<h5 class="el_heading_lv5">見出しレベル５</h5>

<!-- ▼ 構成を保ったまま、見た目のみ変更する場合 ▼ -->
<h5 class="el_heading_lv6">見出しレベル５（スタイルは見出しレベル６）</h5>
```

```css:style.css
/* ≡≡≡ ☐ el_heading_lv* ☐ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    見出し（投稿、固定ページのブロックエディタコンテンツと同じスタイル）
    これは、他のページと投稿、固定ページとでスタイルを統一させるためのもの
    ※なので投稿、固定ページ以外で使用することを想定
---------------------------------------------------------------------------- */
.el_heading_lv1 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 2rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv1 {
    font-size: 2.25rem;
    font-weight: 400;
    line-height: 1.4;
  }
}

.el_heading_lv2 {
  margin-top: var(--space_8);
  margin-bottom: var(--space_3);
  font-size: 1.75rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv2 {
    font-size: 2rem;
    font-weight: 400;
  }
}

.el_heading_lv3 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_3);
  font-size: 1.5rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv3 {
    font-size: 1.75rem;
    font-weight: 400;
  }
}

.el_heading_lv4 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1.25rem;
  font-weight: 500;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv4 {
    font-size: 1.5rem;
    font-weight: 400;
  }
}

.el_heading_lv5 {
  margin-top: var(--space_5);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 500;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv5 {
    font-size: 1.25rem;
    font-weight: 400;
    line-height: 1.5;
  }
}

.el_heading_lv6 {
  margin-top: var(--space_3);
  margin-bottom: var(--space_2);
  font-size: 1rem;
  font-weight: 400;
  line-height: 1.7;
  letter-spacing: 0.04em;
}

@media (min-width: 960px) {
  .el_heading_lv6 {
    font-size: 1rem;
    font-weight: 500;
  }
}
```

:::

## まとめ

ここでは、見出し要素である `el_heading` について扱いました。  
その中で、カスタムプロパティを使ったスペースの再現、`@media` クエリを使った モバイル↔デスクトップ 画面の差異についても説明しました。

次回はリンク要素となる `el_link` を扱っていきます。
