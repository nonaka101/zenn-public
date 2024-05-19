---
title: "📄B401：フロントページ(6) その他"
---

## このページについて

ここまでで、見える範囲の おおよその部分が完成しました。本項では残っている下記の要素を作成し、フロントページを完成させます。

- ブロックスキップ
- ページトップに戻るボタン
- 開発ツール

## 作成

ここからは各要素を作成していきます。その前に、現在の `index.html` の状況を確認し、各要素をどこに配置するかを確認していきます。

```html:現在のbodyタグ内を簡略して表示
<body>
  <div class="ly_mainArea">
    <header class="ly_mainArea_header"></header>
    <!-- /.ly_mainArea_header -->
    <dialog class="ly_dialog_wrapper" id="menu"></dialog>
    <!-- /.ly_dialog_wrapper -->
    <nav class="ly_mainArea_leftSidebar" id="anchor_desktopMainMenu" tabindex="-1"></nav>
    <!-- /.ly_mainArea_leftSidebar -->
    <main class="ly_mainArea_content ly_mainArea_content__middle" id="anchor_mainContent" tabindex="-1"></main>
    <!-- /.ly_mainArea_content -->
    <aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1"></aside>
    <!-- /.ly_mainArea_rightSidebar -->
  </div>
  <!-- /.ly_mainArea -->
  <footer class="ly_footer" id="anchor_footer" tabindex="-1"></footer>
  <!-- /.ly_footer -->
  <script src="./assets/js/common.js"></script>
  <script src="./assets/js/devUtilities.js"></script>
</body>
```

現在はこのように、「ヘッダー」「ダイアログ」「左サイドバー」「メイン」「右サイドバー」「フッター」という順序になっています。ここに「ブロックスキップ」「ページトップに戻るボタン」「開発ツール」を入れていくことになります。

まず、「ブロックスキップ」は**ヘッダー直下に配置**します。これはその役割（ページ各所に素早く移動できる機能）から、コンテンツの初めの方に持ってくる必要があるためです。

次に、「ページトップに戻るボタン」は**フッター後ろ、つまりコンテンツ末尾**に配置します。これもその役割（コンテンツの先頭に移動する機能）から、末尾に置いておく必要があるためです。

最後に「開発ツール」ですが、これも**コンテンツ末尾、ページトップに戻るボタンより後ろ**に配置しておきます。これは本来開発用で、ユーザーが触れることは想定していない要素です。その上で、コンテンツ閲覧の邪魔にならないよう末尾に置いておくべきでしょう。

これらの話をまとめると、`index.html` 内の各種要素の配置は、最終的には下記のようになります。

```html:headerとfooterの後ろに各種要素を設置した最終形
<body>
  <div class="ly_mainArea">
    <header class="ly_mainArea_header"></header>
    <!-- /.ly_mainArea_header -->

    <nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集"></nav>
    <!-- /.bl_blockSkip_wrapper -->

    <dialog class="ly_dialog_wrapper" id="menu"></dialog>
    <!-- /.ly_dialog_wrapper -->
    <nav class="ly_mainArea_leftSidebar" id="anchor_desktopMainMenu" tabindex="-1"></nav>
    <!-- /.ly_mainArea_leftSidebar -->
    <main class="ly_mainArea_content ly_mainArea_content__middle" id="anchor_mainContent" tabindex="-1"></main>
    <!-- /.ly_mainArea_content -->
    <aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1"></aside>
    <!-- /.ly_mainArea_rightSidebar -->
  </div>
  <!-- /.ly_mainArea -->
  <footer class="ly_footer" id="anchor_footer" tabindex="-1"></footer>
  <!-- /.ly_footer -->

  <button type="button" id="js_pageTop" aria-label="ページトップに戻る"></button>
  <!-- /#js_pagetop -->

  <div class="bl_devToolBox" id="js_toolBox"></div>
  <!-- /.bl_devToolBox -->

  <script src="./assets/js/common.js"></script>
  <script src="./assets/js/devUtilities.js"></script>
</body>
```

### ブロックスキップ

**B309：bl_blockSkip**で作成した要素を使います。

- 移動先要素には、`id` によるフラグメントを仕込んでいる
- 移動先要素には、フォーカスできるよう `tabindex="-1"` に設定している状態
- リンクアイテムには表示/非表示を切り替えるためのモディファイアを使用（本テーマはモバイル↔デスクトップ画面でレイアウトが異なるため）

```html:ブロックスキップ（headerとdialogの間に設置）
<nav class="bl_blockSkip_wrapper" aria-label="ページ内リンク集">
  <div class="bl_blockSkip">
    <p class="bl_blockSkip_title">ブロックスキップ</p>
    <p class="bl_blockSkip_description">各種ページ位置に移動できます</p>
    <ul class="bl_blockSkip_linkList">
      <li class="bl_blockSkip_linkItem">
        <a href="#anchor_mainContent" class="el_link">このページの本文</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
      <li class="bl_blockSkip_linkItem">
        <a href="#anchor_footer" class="el_link">フッター</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
      <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isMobile">
        <a href="#anchor_mobileMenu" class="el_link">メインメニュー</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
      <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isDesktop">
        <a href="#anchor_desktopMainMenu" class="el_link">メインメニュー</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
      <li class="bl_blockSkip_linkItem bl_blockSkip_linkItem__isDesktop">
        <a href="#anchor_desktopSubMenu" class="el_link">サイドメニュー</a>
      </li>
      <!-- /.bl_blockSkip_linkItem -->
    </ul>
    <!-- /.bl_blockSkip_linkList -->
  </div>
  <!-- /.bl_blockSkip -->
</nav>
<!-- /.bl_blockSkip_wrapper -->
```

### ページトップに戻るボタン

**B316：js_pageTop**で作成した内容を使います。

- リンク先はヘッダーの「サイトロゴ＋サイト名」要素
- 移動先要素には、`id` によるフラグメントを仕込んでいる（JavaScriptでの操作）
- こちらは手動フォーカスの必要があるので `tabindex="-1"` は なし

```html:ページトップに戻るボタン（footer後ろに配置）
<button type="button" id="js_pageTop" aria-label="ページトップに戻る">
  <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" width="24" height="24"
    viewBox="0 0 16 16">
    <path fill-rule="evenodd" d="M8 15a.5.5 0 0 0 .5-.5V2.707l3.146 3.147a.5.5 0 0 0
      .708-.708l-4-4a.5.5 0 0 0-.708 0l-4 4a.5.5 0 1 0 .708.708L7.5 2.707V14.5a.5.5 0 0 0 .5.5z">
    </path>
  </svg>
</button>
<!-- /#js_pagetop -->
```

### 開発ツール

**B200：開発ツール**の節で説明した内容を使います。

```html:開発ツール（footer後ろに配置）
<div class="bl_devToolBox" id="js_toolBox">
  <div class="bl_devToolBox_btn">
    <input type="checkbox" name="is-show" id="js_toolBox_chkbox">
  </div>
  <!-- /.bl_devToolBox_btn -->
  <div class="bl_devToolBox_utils">
    <div id="js_fontSize_controller">
      <label for="js_fontSize_range">サイズ(px)</label>
      <input type="range" name="font-size" id="js_fontSize_range" value="16" max="32" min="10" step="1">
      <output id="js_fontSize_result"></output>
    </div>
    <hr style="border-top: 1px dotted #aaaaaa;">
    <div class="bl_toggleArea">
      <input
        type="checkbox"
        name="enabled-font-size"
        id="js_fontSize_chkbox"
        class="bl_toggleArea_chkbox"
      >
      <label for="js_fontSize_chkbox" class="bl_toggleArea_label">
        font-size 手動
      </label>
    </div>
    <hr style="border-top: 2px solid #0000be;">
    <div id="js_colorMode_controller" class="bl_toggleArea">
      <input
        type="checkbox"
        name="is-dark-mode"
        id="js_darkMode"
        class="bl_toggleArea_chkbox"
      >
      <label for="js_darkMode" class="bl_toggleArea_label">ダークモード</label>
    </div>
    <hr style="border-top: 1px dotted #aaaaaa;">
    <div class="bl_toggleArea">
      <input
        type="checkbox"
        name="enable-color-mode"
        id="js_colorMode_chkbox"
        class="bl_toggleArea_chkbox"
      >
      <label for="js_colorMode_chkbox" class="bl_toggleArea_label">
        color-mode 手動
      </label>
    </div>
  </div>
  <!-- /.bl_debToolBox_utils -->
</div>
<!-- /.bl_devToolBox -->
```

## まとめ

これで `index.html` が完成しました。

https://github.com/nonaka101/web-zukuri/blob/main/docs/index.html

次項からは、今回作成した `index.html` を用いて残りのページを作成していくことになります。
