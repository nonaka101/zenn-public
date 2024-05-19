---
title: "📄B401：フロントページ(1) 準備"
---

## このページについて

本項では静的ページを作成していくにあたり、全てのベースとなる フロントページ（GitHub Pages の関係で、名前は `index.html`）作成を行っていきます。

**B001：構成について**で話した WordPress のページテンプレートやテンプレートパーツなどを見据え、ここまで作成してきた基本コンポーネントを使って構成していくことになります。

## 作成

ではここからは、フロントページを作成していきます。大まかな作成の流れについては、下記のようになります。

1. レイアウト等の構築
2. 各レイアウト単位で、中身の作り込み
   1. フッター
   2. 右サイドバー
   3. ヘッダー
   4. ダイアログ
   5. メインコンテンツ

ここまでで フロントページが完成する見込みです。フロントページ以外については、フロントページをコピペ＆要所を変更して作っていくことになります。

### 構築(1) フォルダ構成

まずは土台となる部分からページを構築していきます。

ルートフォルダに `index.html` を作成し、`asset` フォルダに各種ファイルを用意します。構成は下図のようになります。

```text
.
├── index.html
└── assets/
       ├── css/
       │     ├── style.css
       │     ├── normalize.css
       │     └── devUtilities.css
       ├── js/
       │     ├── common.js
       │     └── devUtilities.js
       └── image/
             ├── dummy_logo.png
             └── m_02_white.png
```

`style.css`, `devUtilities.css`, `common.js`, `devUtilities.js` は空の状態で構いません。それらはこれから埋めていくことになります。

`normalize.css` は下記から取得します。

https://necolas.github.io/normalize.css/

`dummy_logo.png`（サイトロゴ画像）や `m_02_white.png`（サムネイル画像）は自身で用意しても構いませんし、リポジトリのものを使うこともできます。

#### `style.css`

**B102：スタイルルール**から、**B317：wp_blockContents**まで行ってきた諸々のスタイル文をまとめます。最終的には下記のようになります

https://github.com/nonaka101/web-zukuri/blob/main/docs/assets/css/style.css

:::message

一部まだ説明していないスタイルもありますが、それらはここから解説していくことになります

:::

#### `devUtilities.css`

**B200：概要 開発ツール**の節で行ったスタイル文をまとめます。最終的には下記のようになります。

https://github.com/nonaka101/web-zukuri/blob/main/docs/assets/css/devUtilities.css

#### `common.js`

**B315：ly_dialog**と**B316：js_pageTop**で作成したコードをまとめます。最終的には下記のようになります。

https://github.com/nonaka101/web-zukuri/blob/main/docs/assets/js/common.js

#### `devUtilities.js`

**B200：概要 開発ツール**の節で作成したコードをまとめます。最終的には下記のようになります。

https://github.com/nonaka101/web-zukuri/blob/main/docs/assets/js/devUtilities.js

### 構築(2) ページ土台部分、レイアウト

ファイルが用意できたため、ここからは `index.html` の中身を扱っていきます。まずは土台となる部分として、用意したファイル群をページ内に取り込んでいきます。

#### ファイル関係の読み込み

**B101：全体の構成**で少し触れましたが、スタイル関係はファイルとして用意した３つに加え、Googleフォントも使用します。

```html:B101：全体の構成_より
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

これに JavaScript の読み込みとランドマーク要素を加えると、下記のような形になります。

```html:土台部分の完成形
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>トップページ</title>
    <meta name="description" content="アクセシビリティに配慮したサイト設計を目指して">
    <link rel="stylesheet" href="./assets/css/normalize.css">
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link
      href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&display=swap"
      rel="stylesheet"
    >
    <link rel="stylesheet" href="./assets/css/style.css">
    <!-- 開発用 -->
    <link rel="stylesheet" href="./assets/css/devUtilities.css">
  </head>
  <body>
    <header></header>
    <nav></nav>
    <main></main>
    <aside></aside>
    <footer></footer>
    <script src="./assets/js/common.js"></script>
    <!-- 開発用 -->
    <script src="./assets/js/devUtilities.js"></script>
  </body>
</html>
```

#### レイアウトの設定

次にレイアウト関係です。`body` タグ内には現在ランドマーク要素を素置きしている状態ですが、これを**B313：ly_footer**や**B314：ly_mainArea**で説明した形にします。今回のフロントページは他と違い、左サイドバーが存在し、メインコンテンツは横１２グリッドに対し中央６を使用するレイアウトとなります。

```html:土台部分＋レイアウト
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>トップページ</title>
    <meta name="description" content="アクセシビリティに配慮したサイト設計を目指して">
    <link rel="stylesheet" href="./assets/css/normalize.css">
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link
      href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;500;700&display=swap"
      rel="stylesheet"
    >
    <link rel="stylesheet" href="./assets/css/style.css">
    <!-- 開発用 -->
    <link rel="stylesheet" href="./assets/css/devUtilities.css">
  </head>
  <body>
    <div class="ly_mainArea">
      <header class="ly_mainArea_header"></header>
      <!-- /.ly_mainArea_header -->
      <nav class="ly_mainArea_leftSidebar" id="anchor_desktopMainMenu" tabindex="-1"></nav>
      <!-- /.ly_mainArea_leftSidebar -->
      <main class="ly_mainArea_content ly_mainArea_content__middle" id="anchor_mainContent" tabindex="-1"></main>
      <!-- /.ly_mainArea_content.ly_mainArea_content__middle -->
      <aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1"></aside>
      <!-- /.ly_mainArea_rightSidebar -->
    </div>
    <!-- /.ly_mainArea -->
    <footer class="ly_footer" id="anchor_footer" tabindex="-1"></footer>
    <!-- /.ly_footer -->
    <script src="./assets/js/common.js"></script>
    <!-- 開発用 -->
    <script src="./assets/js/devUtilities.js"></script>
  </body>
</html>
```

ここでのおさらい事項は、下記のとおりです。

- メイン要素とフッター要素とで、12グリッドは別個に管理
  - その関係で、`header`, `nav`, `main`, `aside` は `div` でラップしている
  - `main` 要素は左サイドバーのあるなしでレイアウトが変化する関係上、モディファイア（今回は `ly_mainArea_content__middle`）で対応
- `id` と `tabindex` はブロックスキップ用
  - `id` はブロックスキップに仕込むリンクのフラグメント
  - `tabindex="-1"` はフォーカス可能状態（ただし Tabキーによる手動フォーカスは不可）

## まとめ

本項では実際の静的ページを作るための準備段階となる、フォルダ構成や土台となるページレイアウトを作成しました。

次項からは各ランドマーク要素内の作り込みを行っていきます。最初はメインコンテンツから作成していきます。
