---
title: "📄B102：スタイルルール"
---

## このページについて

ここでは、本テーマ上でのスタイルルールについて説明します。また、一部の要素セレクタに対するスタイリングもここで行っておきます。

まずは、ルールの参考となったCSS設計思想である `PRECSS` について簡単に説明していきたいと思います。

## `PRECSS` とは？

`PRECSS`(Prefix CSS) とは、CSS設計思想の１つです。CSS設計思想とは「煩雑になりがちなCSSを扱いやすくするために、一定のルールを設けることで管理しやすくする」ための考え方だと、私は考えています。

CSS設計思想には、他にも `FLOCSS`(Foundation Layout Object CSS) や `BEM`(Block Element Module) などがあります。

`PRECSS` は そのベースに `OOCSS`, `SMACSS`, `BEM` を持ち、下記の特徴を有しています。

- 種類に応じた接頭辞をつける（JavaScript等のプログラム用や、特定ページにのみ使用が限定されているものなども）
- クラスなどの命名に、スネークケースとキャメルケースを一定の規則に基づき使い分ける
- BEM に比べ、ベーススタイルの扱いや、モジュール内のネストに許容的
- モジュール自体は広く使い回せるようにし、一部特殊な扱いが必要なケースはモディファイアやヘルパークラスで対応

具体的な内容については、公式ページや書籍等で取り扱われていますので、そちらをご参照ください。

https://precss.io/ja/

https://gihyo.jp/book/2020/978-4-297-11173-1

本テーマではこの `PRECSS` を参考に、CSSを設計しています。

:::message

参考に留めており、公式ルールに厳密に従ってはいないことをご了承ください。

命名規則、グループ分類、モディファイアなど、大枠となる要素は `PRECSS` の内容に沿っていると思います。一方で細かなルール（例：不必要に詳細度を高めない、レイアウトに関わるスタイルはコンテキスト又はラッパーモジュールで規定、など）からは逸脱している箇所が、多々あります。

:::

## 本テーマにおけるルール

ここからは、本テーマにおけるスタイルのルールを説明していきます。

本節で扱うことになる、スタイルルールをまとめたものが下記になります。

- プロパティの並び方は `stylelint` 準拠
- モバイルファーストで設計
- `box-sizing` は `border-box` を原則とする
- `PRECSS` に関連するルール
  - クラス名には、`el_`, `bl_`, `ly_` 等の接頭辞をつける
    - 接頭辞に応じて、役割の異なるグループを構成する（基本的には `PRECSS` 準拠）
    - `el_`: エレメントモジュール、１つの部品が単一の要素で構成されるもの
    - `bl_`: ブロックモジュール、１つの部品が複数要素で構成されるもの
    - `ly_`: レイアウト、モジュール等のコンテンツを内包する 大枠となるもの
    - `js_`: JavaScript用、操作要素を想定
    - `is_`: JavaScript用、条件式に使用する想定
    - `wp_`: WordPress用、現段階ではブロックエディタで作成されたコンテンツ用となる
  - ベースとなるクラスは `接頭辞_部品名` の形とする
    - 複数単語の場合、命名規則としてローワーキャメルケースを用いる
  - ベースクラスの子要素は `接頭辞_部品名_子要素名` の形とする
  - 上位要素と下位要素との責任関係を明確にするよう心がける
    - 上位の要素は、下位要素のレイアウト（例：`margin`, `padding`）を規定する
    - 下位の要素は、自身の（レイアウト以外の）スタイルを規定する
  - 特定の場面でのみ振る舞いを変えるようなケースは、モディファイアを用いる
    - モディファイアはクラス名末尾に `__モディファイア名` の形とする
    - 詳細度を高めるため `.el_sample.el_sample__modifier{...}` の複合セレクタを使用する
- 色に関するルール
  - ルート上に、プリミティブカラー（カスタムプロパティ）を用意する
  - プリミティブカラーを使用して、カラーモードに対応したセマンティックカラー（カスタムプロパティ）を用意する
  - 原則として、サイトの色に関するスタイルは、このセマンティックカラーから使用する

これらのうち、`PRECSS` に関するルールと、色に関するルールは次項以降で説明します。本項では、それ以外の項目について説明していきます。

### プロパティの並び方は `stylelint` 準拠

本テーマでは、スタイル管理に `stylelint` を使用しています。構文エラーの抽出だけでなく、スタイリングの形式を統一的にしてくれます。

本テーマでは、下記の２つを使用しています。

- `stylelint-config-standard-scss`: 基本設定（＋SCSS拡張にも対応）
- `stylelint-config-recess-order`: プロパティ順序を規定

また、ルール管理用に `stylelintrc.json` を作成しています。

:::message

ルールの設定というより、`PRECSS` 等の運用でエラーになってしまう箇所をその都度解除させているような感じです・・・

:::

```json:stylelintrc.json
{
  "$schema": "https://json.schemastore.org/stylelintrc.json",
  "extends": [
    "stylelint-config-standard-scss",
    "stylelint-config-recess-order"
  ],
  "ignoreFiles": [
    "**/node_modules/**"
  ],
  "rules": {
    "property-no-vendor-prefix": null,
    "comment-empty-line-before": "always",
    "media-feature-range-notation": "prefix",
    "color-named" : "never",
    "custom-property-pattern": null,
    "color-hex-length": null,
    "selector-class-pattern": null,
    "keyframes-name-pattern": null,
    "no-descending-specificity": null,
    "value-no-vendor-prefix": null,
    "selector-id-pattern": null
  }
}
```

### モバイルファーストで設計

本テーマはレスポンシブデザインとなっており、ビューポートに応じてレイアウト等が変化します。

スタイルを規定するには 複数のパターンを考えていくことになりますが、ここではモバイル画面での表示を基準として設計していきます。

具体的には、メディアクエリの使用についてです。本テーマでは 960px をモバイル↔デスクトップ 画面の境界線としています。CSS上では基本的にはモバイル画面のスタイルを書いていき、デスクトップ画面の規定が必要になった場合には `min-width` を使ったメディアクエリを用います。

```css
@media (min-width: 960px) { /* デスクトップ画面時のスタイル */ }
```

### `box-sizing` は `border-box` を原則とする

全ての要素の `box-sizing` を `border-box` にします。デフォルトでは `content-box` となっており、パディングやボーダーの数値は含まれていません。本テーマにおけるレイアウトは、「横幅いっぱいに要素が伸びる」のを基本としています。そのため、計算しやすいようにサイズ基準を「パディングとボーダーを含めたもの」に変更しています。

```css:style.css
*,:after,:before {
  box-sizing: border-box;
}
```

## 要素セレクタへのスタイリング

`PRECSS` や色に関するルールは以降の項で説明するので、ここからは要素セレクタに対するスタイリングを行っていきます。

### `:root`

前項で用意した `Noto Sans JP` をフォントとして使用することを宣言します。

```css:style.css
:root {
  font-family: "Noto Sans JP", "Segoe UI", "Hiragino Kaku Gothic ProN", "BIZ UDPGothic", meiryo, sans-serif;
  line-height: 1.5;
}
```
### `img` 要素

「可能な限り横幅を100%使用する」が原則なので、`img` の `max-width` は 100% にしておきます。

```css:style.css
img {
  max-width: 100%;
}
```

### `svg` 要素

SVG要素に関しては、`fill: currentColor;` を指定します。キーワード `currentColor` は、要素の `color` プロパティの値を表します。

```css:style.css
svg {
  fill: currentColor;
}
```

これは、カラーモードに対応させる意図があります。

本テーマでは、SVG要素はアイコンボタン等に使用しております。そのため、カラーモードの切り替えに対応できなければなりません。

そこで利用しているのが `color` プロパティです。ここは背景に対し適切な文字色を提供するよう工夫を行っている箇所になります。それを流用し、SVGにも同じ色を適用しているわけです。

:::message

ただし、`currentColor` はインラインSVGのみが対象となっており、`<img scr="*.svg">` で呼び出されたものには適用されません。そのため、本テーマではインラインSVGが何度も登場してきます。

:::

## フォーカスのスタイリング

最後に、フォーカスに関するスタイリングを行います。

フォーカス可能な要素にフォーカスが当たる時、デフォルトでインジケーターが表示されるようになっています。フォーカスが可視化されることで「自身が今どこにいるのか」がわかるようになり、これは例えばキーボード操作のユーザーにとって重要な情報となります。

### `:focus` と `:focus-visible` について

`:focus-visible` が登場するまで、フォーカススタイルを扱うためには 擬似クラス `:focus` を使うのが一般的でした。しかしこの手法には、「意図していない操作にもスタイルが適用されてしまう」という問題がありました。

`:focus` の意図している状況とは、キーボードを用いた Tabキー操作です。これによりキーボード操作者は「自分が今どこにいるのか」把握することができます。一方 意図していない状況とは、マウスクリックやタップ操作です。`:focus` はこれらの操作にも適用されてしまい、クリックやタップの都度、フォーカスインジケータが表示されることになります。

すると「ボタンを押すたびにインジケーターが出てしまうのは見栄えが悪い」と考え、`:focus { outline: none; }` でインジケーターの表示を消してしまうサイトがいくつも発生してしまいました。

なので現在は、`:focus` に変わる擬似クラスとして `:focus-visible` が登場しました。本テーマでもこちらを利用しています。

https://developer.mozilla.org/ja/docs/Web/CSS/:focus-visible

> `:focus-visible` 擬似クラスは、要素が `:focus` 擬似クラスに一致している時で、ユーザーエージェントが要素にフォーカスを明示するべきであると推測的に判断した場合に適用されます
> 『`:focus-visible` - CSS: カスケーディングスタイルシート | MDN』より一部抜粋

これにより、必要とするケースにてフォーカスインジケータを表示できるよう、本テーマではスタイリングしています。

```css:本テーマで使用するフォーカス用スタイル１
*:focus-visible{ }
```

### エフェクトについて

まずはアウトラインについてです。ここでは「アウトラインはオレンジ色の実線」とします。また、実線がフォーカス要素と被って文字が読めなくなることを避けるため、少しだけ外側にオフセットしておきます。

```css:本テーマで使用するフォーカス用スタイル２
*:focus-visible{
  outline: 2px solid #cd820a;
  outline-offset: 2px;
}
```

次にエフェクトについてです。華美なものは不要ですが、見つけやすいよう ちょっとした動きをつけておきたいです。

ここでは、「フォーカス要素に対し、大きく外側からフォーカスインジケーターが発生、その後 内側に移動する」ようなエフェクトを作成します。

```css:本テーマで使用するフォーカス用スタイル３
*:focus-visible{
  outline: 2px solid #cd820a;
  outline-offset: 2px;
}

@keyframes focusEffect {
  from {
    outline-width: 1px;
    outline-offset: 8px;
  }

  to {
    outline-width: 2px;
    outline-offset: 2px;
  }
}
```

最後に、アニメーション設定でフォーカス要素にエフェクトを適用します。エフェクトは一瞬あれば十分なので、ここでは 0.3秒としています。

```css:本テーマで使用するフォーカス用スタイル４
*:focus-visible{
  outline: 2px solid #cd820a;
  outline-offset: 2px;
  animation-name: focusEffect;
  animation-duration: 0.3s;
}

@keyframes focusEffect {
  from {
    outline-width: 1px;
    outline-offset: 8px;
  }

  to {
    outline-width: 2px;
    outline-offset: 2px;
  }
}
```

これでフォーカスに関するスタイリングは完了です。

### 実際のスタイル文

```css:style.css
*:focus-visible{
  /* 実際のデータでは、カラーモードに対応したセマンティックカラーを使用 */
  outline: var(--sc_border_focused);
  outline-offset: 2px;
  animation-name: focusEffect;
  animation-duration: 0.3s;
}

@keyframes focusEffect {
  from {
    outline-width: 1px;
    outline-offset: 8px;
  }

  to {
    outline-width: 2px;
    outline-offset: 2px;
  }
}
```

## まとめ

本項では、テーマで運用されているスタイルルールについて説明しました。  
また、一部の要素セレクタに対しスタイリングを行いました。

次項では、最初に軽く説明した `PRECSS` を参考にしたルールについて説明していきます。
