---
title: "📄B314：ly_mainArea(2)"
---

## このページについて

前項では、メイン部のベースとなる `ly_mainArea` について説明しました。２回目となる本項では、中に入る子要素 `header`, `nav`, `main`, `aside` を中心に扱っていきます。

### 前回のおさらい

`ly_mainArea` は、「ページの種類」「画面サイズ（幅）」に応じてレイアウト構成が変化するもの、と前回説明しました。そして `ly_mainArea` が管理する要素は、ランドマークの役割を持つ下記のものでした。

- ヘッダー（`header` タグ）
- 左サイドバー（`nav` タグ）
- メインコンテンツ（`main` タグ）
- 右サイドバー（`aside` タグ）

![フロントページ（各要素に印をつけた状態）](/images/books/web-zukuri/ly_mainArea-01.png)

必要となる要件をまとめると下記になります。

- コンテンツ全体の最大幅は `1024px`
- 横方向に `12fr` に分割したグリッドを用いる
- 「ヘッダー」「左サイドバー」「メインコンテンツ」「右サイドバー」を配置する
- 上記の要素は、ページに応じて配置が変わる（ヘッダーは除く）
  - フロントページのみ、構成比 3:6:3 の形で「左サイドバー」「メインコンテンツ」「右サイドバー」を配置
  - それ以外は構成比 9:3 の形で「メインコンテンツ」「右サイドバー」を配置
- 上記の要素は、画面幅に応じて配置が変わる
  - デスクトップ画面時
    - ヘッダーは左上
    - サイドバーを表示
  - モバイル画面時
    - ヘッダーは上部に固定
    - サイドバーは非表示（メニューボタン＋ダイアログの構成）

## 作成

では、ここからは `ly_mainArea` の子要素を作成していきます。なお、ここからの話は**特段の指定が無い限り、全てフロントページ**を想定した話となります。

### HTML構造

まずは、HTML構造です。前回は下記の所まで作成していました。

```html:前回の完成形
<body>
  <div class="ly_mainArea">
    <header></header>
    <nav></nav>
    <main></main>
    <aside></aside>
  </div>
  <!-- /.ly_mainArea -->
  <footer class="ly_footer"></footer>
  <!-- /.ly_footer -->
</body>
```

ここから、子要素にクラス付けをしていきます。役割を子部品名として、下記のようにしました。

- `div.ly_mainArea`
  - `header.ly_mainArea_header`: ヘッダー
  - `nav.ly_mainArea_leftSidebar`: 左サイドバー
  - `main.ly_mainArea_content`: メインコンテンツ
  - `aside.ly_mainArea_rightSidebar`: 右サイドバー
- `footer.ly_footer`: フッター

ただし、メインコンテンツである `ly_mainArea_content`はこれだけでは不十分です。この要素はページによって下記のように変化するからです。

- フロントページの場合、左サイドバーが存在するので、列位置 `4fr` 〜 `9fr` を使用
- それ以外の場合、左サイドバーがないので、列位置 `1fr` 〜 `9fr` を使用

:::message

左サイドバーに関しては、PHP（WordPress）側で出力する/しないを設定します。

:::

つまり、`ly_mainArea_content` は、ある場合では 中央に `6fr` 、ある場合では 左側 `9fr` のレイアウトとなるわけです。左サイドバーの有無をセレクタに含んでスタイリングする方法もなくはないですが、ここはモディファイアを使うことにします。

- `ly_mainArea_content__middle`：フロントページに使用、中央 `6fr` のレイアウト
- `ly_mainArea_content__left`：フロントページ外に使用、左側 `9fr` のレイアウト

よって、HTML構造は最終的には下記のようになります。

- `div.ly_mainArea`
  - `header.ly_mainArea_header`: ヘッダー
  - `nav.ly_mainArea_leftSidebar`: 左サイドバー
  - `main.ly_mainArea_content`: メインコンテンツ
    - `ly_mainArea_content__middle`: モディファイア（中央）
    - `ly_mainArea_content__left`: モディファイア（左）
  - `aside.ly_mainArea_rightSidebar`: 右サイドバー
- `footer.ly_footer`: フッター

HTML側にこれを反映し、また**B309：bl_blockSkip**で話したように、ランドマークへのリンク機能を持たせたものが下記になります。

```html:フロントページ
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
</body>
```

```html:フロントページ以外
<body>
  <div class="ly_mainArea">
    <header class="ly_mainArea_header"></header>
    <!-- /.ly_mainArea_header -->
    <!-- `nav.ly_mainArea_leftSidebar` はフロントページのみ -->
    <main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1"></main>
    <!-- /.ly_mainArea_content.ly_mainArea_content__left -->
    <aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1"></aside>
    <!-- /.ly_mainArea_rightSidebar -->
  </div>
  <!-- /.ly_mainArea -->
  <footer class="ly_footer" id="anchor_footer" tabindex="-1"></footer>
  <!-- /.ly_footer -->
</body>
```

HTML構造が決まったので、ここからは各子要素のスタイルを作っていきます。

### ヘッダー（`header.ly_mainArea_header`）

最初にヘッダーから作ります。まずはモバイル画面ですが、要件は下記のとおりです

- 画面上部に固定表示（≒下にスクロールしても動かない）
- 他要素より前面に表示（メニューダイアログなど一部の例外を除く）
  - `box-shadow` で影を表現し少し浮いたような見た目に（他要素との区切りが目立つように）

```css
.ly_mainArea_header {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 2;
  width: 100%;
  height: 64px;
  padding: 8px 24px;
  background-color: var(--sc_bg_body__primary);
  box-shadow: 0 4px 16px -8px var(--sc_bg_shadow);
}

@media (min-width: 520px) {
  .ly_mainArea_header {
    padding: 8px 40px;
  }
}
```

このように、モバイル画面時のヘッダーは `position: fixed;` のためグリッドレイアウトから外れます。なので後述する `ly_mainArea_content` などの、グリッドで構成される他要素との調整が必要になってきます。

次に、デスクトップ画面です。こちらはモバイル画面とは違い、グリッド構成に組み込みます。

- グリッド構成は、1行目 1列目から 3 列分を使用
- （モバイル画面の）影の表現は不要なので消す

```css
@media (min-width: 960px) {
  .ly_mainArea_header {
    position: inherit;
    grid-area: 1/1/auto/span 3;
    height: auto;
    padding: 0;
    box-shadow: none;
  }
}
```

### サイドバー（`nav.ly_mainArea_leftSidebar`, `aside.ly_mainArea_rightSidebar`）

続いて左右サイドバーです。こちらもモバイル画面から考えていきます。

モバイル画面では、左右サイドバーを表示しません。モバイル画面で表示されるメニューダイアログが、その役割を代行します。

```css
.ly_mainArea_leftSidebar {
  display: none;
}

.ly_mainArea_rightSidebar {
  display: none;
}
```

続いてデスクトップ画面です。

- 左サイドバーのグリッド構成は、2行目 1列目から 3 列分を使用
  - ヘッダーとの間隔を開けるため、`padding-top` を挿入
- 右サイドバーのグリッド構成は、2行目 10列目から 3 列分を使用

```css
@media (min-width: 960px) {
  .ly_mainArea_leftSidebar {
    display: block;
    grid-area: 2/1/auto/span 3;
    padding-top: 64px;
  }
}

@media (min-width: 960px) {
  .ly_mainArea_rightSidebar {
    display: block;
    grid-area: 2/10/auto/span 3;
  }
}
```

### メインコンテンツ（`main.ly_mainArea_content`）

最後にメインコンテンツです。まずはモバイル画面から見ていきます。

- グリッド構成は、2行目 1列目から 12列分を使用（≒グリッド列の全てを使う）
- ヘッダー要素分、上側にマージンを取る

```css
.ly_mainArea_content {
  grid-area: 2/1/auto/span 12;
  margin-top: 64px;
}
```

ここの `margin-top: 64px;` は、`header.ly_mainArea_header` 要素の `height: 64px;` と調整する意味合いがあります。モバイル画面時のヘッダーは `position: fixed;` なので、マージンを設けておかないと、どうやっても見えなくなる部分が発生してしまいます。

次にデスクトップ画面です。これはフロントページとそれ以外とでグリッドレイアウトが変わります。

- モバイル画面で必要だった上側のマージンは不要
- フロントページの場合（`ly_mainArea_content__middle`）
  - グリッド構成は、2行目 4列目から 6 列分を使用
- フロントページ以外の場合（`ly_mainArea_content__left`）
  - グリッド構成は、2行目 1列目から 9 列分を使用

```css
@media (min-width: 960px) {
  .ly_mainArea_content {
    margin-top: 0;
  }

  .ly_mainArea_content.ly_mainArea_content__left {
    grid-area: 2/1/auto/span 9;
  }

  .ly_mainArea_content.ly_mainArea_content__middle {
    grid-area: 2/4/auto/span 6;
  }
}
```

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html:フロントページ
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
</body>
```

```html:フロントページ以外
<body>
  <div class="ly_mainArea">
    <header class="ly_mainArea_header"></header>
    <!-- /.ly_mainArea_header -->
    <!-- `nav.ly_mainArea_leftSidebar` はフロントページのみ -->
    <main class="ly_mainArea_content ly_mainArea_content__left" id="anchor_mainContent" tabindex="-1"></main>
    <!-- /.ly_mainArea_content.ly_mainArea_content__left -->
    <aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1"></aside>
    <!-- /.ly_mainArea_rightSidebar -->
  </div>
  <!-- /.ly_mainArea -->
  <footer class="ly_footer" id="anchor_footer" tabindex="-1"></footer>
  <!-- /.ly_footer -->
</body>
```

```css:style.css
/* ≡≡≡ ◲ ly_mainArea ◲ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    フッターなどを除く、メインとなるエリアのレイアウトを構成する
    レイアウトは大別すると「トップページ」と「それ以外のページ」の２種
    それぞれでモバイル画面とデスクトップ画面で変化
  ■ 構成
    ly_mainArea : 全体
      ly_mainArea_header : ヘッダーレイアウト（12Grid の 1行 1〜3列目、モバイル画面では画面上部に固定）
      ly_mainArea_leftSidebar : 左サイドバーのレイアウト（12Grid の 2行 1〜3列目）
      ly_mainArea_rightSidebar : 右サイドバーのレイアウト（12Grid の 2行 10〜12列目）
      ly_mainArea_content : メインコンテンツのレイアウト（各ページレイアウトに応じて変化）
      -> ly_mainArea_content__middle : トップページに利用（12Grid の 2行 4〜9列目）
      -> ly_mainArea_content__left : それ以外のページに利用（12Gridの 2行 1〜9列目）
---------------------------------------------------------------------------- */
.ly_mainArea {
  display: grid;
  grid-template-rows: auto auto 1fr;
  grid-template-columns: repeat(12, 1fr);
  grid-auto-flow: row;
  gap: 0 4px;
  max-width: 1024px;
  padding: 0 24px;
  margin-right: auto;
  margin-left: auto;
}

@media (min-width: 560px) {
  .ly_mainArea {
    gap: 0 8px;
    padding: 0 40px;
  }
}

@media (min-width: 960px) {
  .ly_mainArea {
    gap: 0 16px;
    padding: 64px 40px 0;
  }
}

@media (min-width: 1120px) {
  .ly_mainArea {
    gap: 0 32px;
    padding-right: 0;
    padding-left: 0;
  }
}

.ly_mainArea_header {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 2;
  width: 100%;
  height: 64px;
  padding: 8px 24px;
  background-color: var(--sc_bg_body__primary);
  box-shadow: 0 4px 16px -8px var(--sc_bg_shadow);
}

@media (min-width: 520px) {
  .ly_mainArea_header {
    padding: 8px 40px;
  }
}

@media (min-width: 960px) {
  .ly_mainArea_header {
    position: inherit;
    grid-area: 1/1/auto/span 3;
    height: auto;
    padding: 0;
    box-shadow: none;
  }
}

.ly_mainArea_leftSidebar {
  display: none;
}

@media (min-width: 960px) {
  .ly_mainArea_leftSidebar {
    display: block;
    grid-area: 2/1/auto/span 3;
    padding-top: 64px;
  }
}

.ly_mainArea_rightSidebar {
  display: none;
}

@media (min-width: 960px) {
  .ly_mainArea_rightSidebar {
    display: block;
    grid-area: 2/10/auto/span 3;
  }
}

.ly_mainArea_content {
  grid-area: 2/1/auto/span 12;
  margin-top: 64px;
}

@media (min-width: 960px) {
  .ly_mainArea_content {
    margin-top: 0;
  }

  .ly_mainArea_content.ly_mainArea_content__left {
    grid-area: 2/1/auto/span 9;
  }

  .ly_mainArea_content.ly_mainArea_content__middle {
    grid-area: 2/4/auto/span 6;
  }
}
```

:::

## まとめ

本項では、メイン部の構造を担う `ly_mainArea` の各子要素について見ていきました。これで、2回に渡って解説した `ly_mainArea` が完成したことになります。

これと以前やった `ly_footer` を組み合わせれば、サイトに表示される一連の構成を作ることができ、レイアウトとしては一段落になります。

次項では少し特殊なレイアウトを扱います。具体的には、モバイル画面時のメニューダイアログ `ly_dialog` で、上記で作られたサイトレイアウトとは違う箇所を担うものです。
