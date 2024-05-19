---
title: "📄B101：全体の構成"
---

## このページについて

ここでは、`html` 上で使用している `css` について、それぞれどのような役割を持っているかについて簡単に解説していきます。

## 本テーマで使用する `css` ファイルについて

本テーマでは、主に４つの `css` ファイルを扱います。GitHub Pages 側の `<head>` タグ内を参照してみましょう。

```html:使用するスタイルシートを抜粋
<head>
  <!-- ノーマライズ CSS -->
  <link rel="stylesheet" href="./assets/css/normalize.css">

  <!-- Google Font -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link
    href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&display=swap"
    rel="stylesheet"
  >

  <!-- テーマのスタイル本体 -->
  <link rel="stylesheet" href="./assets/css/style.css">

  <!-- 開発ツール用 -->
  <link rel="stylesheet" href="./assets/css/devUtilities.css">
</head>
```

このように、本テーマでは ４つの CSS を使用しています。

- `normalize.css`: ノーマライズCSS
- `fonts.googleapis.com`: Google Font
- `style.css`: スタイル本体
- `devUtilities.css`: 開発ツール用

この内、Google Font のみAPIを使って取得していますが、残り３つに関してはファイルを用意します。ディレクトリ上の構成は、下記のようになります。

```text
.
├── index.html
└── assets/
        ├── css/
        │      ├── style.css
        │      ├── normalize.css
        │      └── devUtilities.css
        ├── js/
        └── image/
```

ここからは、それぞれの内容について見ていきます。

### `normalize.css`

このテーマでは、ブラウザ間でのスタイル差異をなくすために**ノーマライズCSS**を使用しています。

https://necolas.github.io/normalize.css/

:::details 個人的メモ（格納しています）

本テーマにおいてリセット系でなく、ノーマライズ系を採用したのは、ブロックエディタ関係で問題を抱えているためです。この記事を作成している現段階でも、適切な解決策が思いついていない状態です。

リセットCSSを使用する場合、デフォルトのスタイルを気にせず、必要なスタイルを記述するだけで良いというメリットがあります。リストタグを使用することを踏まえて `list-style-type: none;` でリスト項目要素のマーカーを剥いだりする必要がなくなるのは魅力的です。

今回は、「ブロックエディタによる記事データをどうするか？」という問題がありました。デフォルトのスタイルを全て剥いでしまうと、ブロックに関するスタイルも規定していく必要があるかもしれません。考えた結果、「少し複雑になりすぎて現実的ではない」とし、デフォルトのスタイルが残るノーマライズを採用することとなりました。

:::

### GoogleFont関係

本テーマでメインとなるフォントは、`Noto Sans JP` です。ページ全体で、Google Fontで提供されているこのWebフォントを使用しています。

https://fonts.google.com/noto/specimen/Noto+Sans+JP

### `style.css`

本テーマのスタイルシート本体です。色に関する規定や、**B300:基本コンポーネント** で扱うコンポーネント等、テーマで扱う ほとんどのスタイルは ここで管理します。

### `devUtilities.css`

開発・検証用のツールを扱うためのスタイルシートです。詳細については **B200:開発ツール** の節で取り扱います。

## まとめ

このページでは、スタイルに関する全体的な話について説明しました。

ここからは `style.css` について解説していきます。次項では CSS設計手法となる `PRECSS` について簡単に話した後、本テーマで使用しているルールについて触れていきます。
