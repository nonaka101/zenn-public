---
title: "📄B401：フロントページ(4) サイドバー＋ダイアログ"
---

## このページについて

本項では、サイドバー要素を扱っていきます。そして作成したものを流用してダイアログ要素も構成していきます。

## WordPress との関係

作成に移る前に、WordPress テーマとなった際にどのようになるかを説明します。結論を先に述べておくと、下記のようになります。

- 左右サイドバーは、WordPress のカスタムメニューと連携
- 右サイドバーはそれにプラスして、DB上にある記事群からカテゴリと日付データを取得し、各種アーカイブを生成する
- メニューダイアログ内のコンテンツは、その役割上、左右サイドバーの中身をそのまま使う
- （上記から）共通化できる部分はテンプレートパーツ化し、流用しやすい形にする
- よってその土台となる静的ページ（本項で扱う話）は、コピペできるレベルでの構造を意識している

![WordPressのカスタムメニューと左サイドバーが連携していることを示している](/images/books/web-zukuri/about-custom-menu-01.png)
*左サイドバーの構成イメージ*

![上記の話を右サイドバーの場合で。なおこちらはプラスして、DBで管理されている記事群からカテゴリと日付データを抜き出し、アーカイブにしていることを示している](/images/books/web-zukuri/about-custom-menu-02.png)
*右サイドバーの構成イメージ*

![ダイアログは、左右サイドバーを流用していることを示している](/images/books/web-zukuri/about-custom-menu-03.png)
*メニューダイアログの構成イメージ*

ここからは上記（図）に関する詳しい話をしていきます。

### カスタムメニューについて

![WordPress のカスタムメニュー画面１](/images/books/web-zukuri/about-custom-menu-04.png)
*WordPress 管理画面でのメニュー編集*

![WordPress のカスタムメニュー画面２](/images/books/web-zukuri/about-custom-menu-05.png)
*カスタマイザーからライブプレビューで編集することも可能*

WordPress では「カスタムメニュー」という機能があります。これはDB上にある特定のページ（例：会社概要 など）をメニューとして登録できる機能です。カスタムメニューに対応しているテーマでは、この設定を使い、ヘッダーなどに設置されたメニュー要素に 登録内容を反映させることができます。

本テーマでは、３つのメニューをページ内に反映させることができます。

|メニュー名|場所|
|:---:|:---|
|メインメニュー|左サイドバー＋フッターウィジェット左側|
|サブメニュー|右サイドバー（アーカイブを除く）|
|サイトナビゲーション|フッターウィジェット右側にある「サイトナビゲーション」欄|

### アーカイブの出力について

![WordPress 管理画面上での、投稿ページ簡易編集画面。タイトルだけでなくタクソノミーや日付など、記事本文には様々な付加情報がある](/images/books/web-zukuri/bl_postData-00.png)
*投稿記事の簡易編集画面*

![閲覧者が特定記事を要求してからページとして出力されるまでのフロー図。記事が持っているタクソノミーや日付データを使い、ページテンプレートに流し込んでページを生成している](/images/books/web-zukuri/flow-create-page-02.png)
*WordPressが、ページ閲覧要求からHTML生成するまでの大まかな流れ*

上図はこれまでに何度か出てきた、WordPress での記事を説明するための図です。１枚目は WordPress 管理画面にある投稿ページ一覧の簡易編集画面で、記事データにはタイトルや本文だけでなく、タクソノミー（カテゴリやタグなど）や、日付データが付随していることがわかります。２枚目はそうしたデータを使って投稿ページを出力するまでのフロー図です。記事データが持つ付随情報を含め、ページテンプレートに流し込んでページを生成していることを説明しています。

このように、記事データには付随情報（タクソノミー、日付 など）が含まれていますが、これを別の手法で活用するのが、アーカイブ出力機能です。

![アーカイブのイメージ図、DB上にある記事群に含まれるカテゴリと日付データからアーカイブ一覧を生成する](/images/books/web-zukuri/about-custom-menu-06.png)

こちらは１つ１つの記事に対するものでなく、記事群全体をまとめて使います。DB上にある記事群から「（親子関係含む）カテゴリ情報」と「日付データ」を抽出し、その一覧をアーカイブページへのリンクとして出力します。

:::details 個人メモ（タグ一覧を出力しない理由について）

想定する使い方から、タグ一覧は量が膨大になり扱う理由が薄いと考えているためです。

カテゴリとタグについてですが、同じ「投稿記事に付与できるもの」でありますが、その性質（役割）は別のものと私は考えています。それぞれの想定として、カテゴリの場合は最小で１つ 最多でも３つくらいという一方、タグは３つ以上つけても良いもの という扱いです。

本テーマでは、カテゴリとは「親子関係を持つ、ツリー構造を持つもの」と考えています。下図は、図書館で用いられる日本十進分類法（NDC）を参考に、カテゴリの関係を説明しようとしたものです。

![カテゴリが木構造となり、そこに記事が付与されているイメージ](/images/books/web-zukuri/about-taxonomy-01.png)
*例：書籍（分野に応じた分類がされており、木構造の形を取る）*

大きな分類から細かな分類に向けて、垂直方向に管理されていることがわかります。この関係性は親子関係であり、例えば「情報工学」に属する記事は、親に「電気工学」「工学」のカテゴリを持つことになります。

一方のタグは、カテゴリとは違った横断的なものと考えています。カテゴリによって垂直方向に分類付けられた記事群を、水平的に繋げているイメージと私は考えています。下図は先程の図書分類を使い、タグを表現しようとしたものです。

![上記のカテゴリ・記事に対し、タグが横断的に繋がりを表している](/images/books/web-zukuri/about-taxonomy-02.png)
*上図のカテゴリに対し、タグは様々な観点から、横断的にリンク構造を取る*

親子関係で分類されるものとは別の視点で、記事間を紐づけていることが見て取れます。ここでは「作者」「発表時期」「キーワード」といった視点で、記事を結びつけています。

ここまでの説明から、カテゴリとタグの関係をもう一度考えます。カテゴリ自体は親子関係を持っていますが、上図では1つの記事に対して1つのカテゴリを割り当てていました。一方のタグは、1つの記事に対して（視点によっては）いくつも紐づけることができます。

そのため、カテゴリに対しタグの量は膨大になる可能性があります。カテゴリと違い親子関係で格納することも難しいので、タグ一覧は出力しないようにしています。

:::

内容が膨大になることを踏まえ、ページに出力する際には**B310：bl_accordion**要素を使って格納します。

![bl_accordion のスクリーンショット](/images/books/web-zukuri/bl_accordion-04.png)

格納状態ですとAOM上で出力されることはなく、スクリーンリーダーの読み上げ対象になりません。このように膨大な情報を必要なときに使えるようにするために、`details` 要素は適していると言えるでしょう。

### テンプレートパーツについて

ここまでで説明した**アーカイブ**と**カスタムメニュー**を組み合わせ、左右サイドバーが構成されます。

そしてメニューダイアログは、左右サイドバーが表示できないモバイル画面において、左右サイドバーの代替としての役割を持っています。代替という関係である以上、左右サイドバーとメニューダイアログで提供する内容に違いがあってはなりません。つまり、左右サイドバーとメニューダイアログで同じコンテンツにする必要があるということです。

その上で、**テンプレートパーツ化して流用する**ことには大きな意味があります。

何かしらの変更が生じたとして、個別に同じものを用意していた場合、両者を同じように編集する必要が生じます。それは変更ミスにつながる恐れがあります。一方でテンプレートパーツ化しておいた場合、そこへの変更は、それを使っている箇所全てに自動的に反映してくれます。

なので本テーマでは、カスタムメニューとアーカイブの部分をテンプレートパーツとして独立させ、複数箇所に流用できる構成にしています。

### WordPress 側での最終的な構成イメージ

最後に、WordPressではどのような形になるかを説明し、その上で静的ページの完成形を見ます。

:::message

ここからPHPの話をしていきますが、ここではその内容を完璧に理解する必要はありません。

「これまで話してきたテンプレートパーツの話を踏まえ、それをここで使っているんだ」ということを掴んでもらえれば十分です。細かな話は第三章で行う予定なので、この段階でのPHPの 文法 や 細かな話 は不要です。

:::

#### その１：左サイドバー

![WordPressのカスタムメニューと左サイドバーが連携していることを示している](/images/books/web-zukuri/about-custom-menu-01.png)

サイドバーに関して、WordPressは特別なテンプレート呼び出し関数（`get_sidebar($name)`）を用意しています。`&name` を指定しない場合は、`sidebar.php` というファイル名を自動的に探し、テンプレートパーツとして読み込んでくれます。本テーマでは左サイドバーの有る無しがあったため、`$name` を使用して個別に呼び出せるようにしています。

左サイドバーは、２つのタグメニューで構成されています。「Homeへのリンク」と「カスタムメニュー（メイン）」の２つです。「Homeへのリンク」についてはベタ打ちで入れて、カスタムメニューはテンプレートパーツ化したものを呼び出しています。

```php:sidebar-left.php
<nav class="ly_mainArea_leftSidebar" id="anchor_desktopMainMenu" tabindex="-1">
  <div class="bl_tagMenu">
    <ul class="bl_tagMenu_list">
      <li class="bl_tagMenu_item">
        <a class="bl_tagMenu_link" href="<?php echo esc_url(home_url()); ?>">Home</a>
      </li>
      <!-- /.bl_tagMenu_item -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->

  <?php get_template_part(
    'template-parts/parts',
    'menu',
    array(
      'loc' => 'main-menu',
      'name' => 'メインメニュー',
      'id' => 'main_menu_in_sidebar',
      )
    );
  ?>
</nav>
<!-- /.ly_mainArea_leftSidebar -->
```

#### その２：右サイドバー

![上記の話を右サイドバーの場合で。なおこちらはプラスして、DBで管理されている記事群からカテゴリと日付データを抜き出し、アーカイブにしていることを示している](/images/books/web-zukuri/about-custom-menu-02.png)

右サイドバーに関しても、特殊な呼び出し関数 `get_sidebar($name)` を使います。中身もほとんど同じです。違うのは、「Homeへのリンクでなく、サイト内検索へのリンクになっていること」と、「アーカイブをテンプレートパーツとして呼び出す内容が追加されていること」です。

```php:sidebar-right.php
<aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1">
  <div class="bl_tagMenu">
    <ul class="bl_tagMenu_list">
      <li class="bl_tagMenu_item">
        <a class="bl_tagMenu_link" href="javascript:searchform.submit()">サイト内検索</a>
      </li>
      <!-- /.bl_tagMenu_item -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->

  <?php get_template_part(
    'template-parts/parts',
    'menu',
    array(
      'loc' => 'sub-menu',
      'name' => 'サブメニュー',
      'id' => 'sub_menu_in_sidebar',
      )
    );
  ?>

  <?php get_template_part(
    'template-parts/parts',
    'categories-and-archives'
    );
  ?>

</aside>
<!-- /.ly_mainArea_rightSidebar -->
```

#### その３：メニューダイアログ

![ダイアログは、左右サイドバーを流用していることを示している](/images/books/web-zukuri/about-custom-menu-03.png)

メニューダイアログは、モバイル画面を想定したものです。画面の余裕がないので左右サイドバーは非表示となり、その代わりとしての役割を持ちます。よって、左右サイドバーの中身を**そのまま**引き継がなければいけません。

なのでここは、**B315：ly_dialog**で作成したダイアログ構成を使い、そのコンテンツ部は左右サイドバーの中身をコピペしたものになっています。

:::details ly_dialog（メニューヘッダーまで）

```html:B315-ly_dialogで作成したダイアログ（メニューヘッダーまで）
<dialog class="ly_dialog_wrapper" id="menu">
  <div class="ly_dialog">
    <div class="bl_menuHeader">
      <h2 class="bl_menuHeader_title">メニュー</h2>
      <button class="bl_menuHeader_btnIcon" type="button" onclick="closeDialog()">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24" width="24" height="24">
          <path stroke-width="1.4286" d="M 22,4.0142857 19.985715,2 12,9.9857149 4.0142857,2 2,4.0142857 9.9857149,12 2,19.985715 4.0142857,22 12,14.014286 19.985715,22 22,19.985715 14.014286,12 Z"/>
        </svg>
        閉じる
      </button>
    </div>
    <!-- /.bl_menuHeader -->
```

:::

:::details コンテンツ本体（左右サイドバーの中身のコピペ）

```php:ダイアログのコンテンツ部（ほぼ左右サイドバーの中身のコピペ）
    <div class="bl_tagMenu">
      <ul class="bl_tagMenu_list">
        <li class="bl_tagMenu_item">
          <a class="bl_tagMenu_link" href="<?php echo esc_url(home_url()); ?>">Home</a>
        </li>
        <!-- /.bl_tagMenu_item -->
      </ul>
      <!-- /.bl_tagMenu_list -->
    </div>
    <!-- /.bl_tagMenu -->

    <?php get_template_part(
      'template-parts/parts',
      'menu',
      array(
        'loc' => 'main-menu',
        'name' => 'メインメニュー',
        'id' => 'main_menu_in_dialog',
        )
      );
    ?>

    <hr>

    <div class="bl_tagMenu">
      <ul class="bl_tagMenu_list">
        <li class="bl_tagMenu_item">
          <a class="bl_tagMenu_link" href="javascript:searchform.submit()">サイト内検索</a>
        </li>
        <!-- /.bl_tagMenu_item -->
      </ul>
      <!-- /.bl_tagMenu_list -->
    </div>
    <!-- /.bl_tagMenu -->

    <?php get_template_part(
      'template-parts/parts',
      'menu',
      array(
        'loc' => 'sub-menu',
        'name' => 'サブメニュー',
        'id' => 'sub_menu_in_dialog',
        )
      );
    ?>
    <?php get_template_part(
      'template-parts/parts',
      'categories-and-archives'
      );
    ?>
```

:::

:::details ly_dialog（終了部）

```html:B315-ly_dialogで作成したダイアログ（終了部）
    <button class="el_btn el_btn__primary" type="button" onclick="closeDialog()">メニューを閉じる</button>
  </div>
  <!-- /.ly_dialog -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

:::

## 作成

ここからは、各要素を作成していきます。

### 左サイドバー

`nav.ly_mainArea_leftSidebar` を使用します。中に入るのは、「ホームへのリンク」と「メインメニュー」の２つのタグメニューです。タグメニューの構成については、**B306：bl_tagMenu**を参照ください。

メインメニューは、最終的にはカスタムメニューに置き換えることになります（中身に関しては、本項では省略します）

```html:左サイドバー
<nav class="ly_mainArea_leftSidebar" id="anchor_desktopMainMenu" tabindex="-1">
  <div class="bl_tagMenu">
    <ul class="bl_tagMenu_list">
      <li class="bl_tagMenu_item">
        <a href="./index.html" class="bl_tagMenu_link">Home</a>
      </li>
      <!-- /.bl_tagMenu_item -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_tagMenu">
    <!-- 省略 -->
  </div>
  <!-- /.bl_tagMenu -->
</nav>
<!-- /.ly_mainArea_leftSidebar -->
```

### 右サイドバー

右サイドバーを構成するのは、`aside.ly_mainArea_rightSidebar` になります。この中は、「サイト内検索へのリンク」「サブメニュー」「各種アーカイブ」計３つのタグメニューとなります。アーカイブを構成するアコーディオンについては、**B310：bl_accordion**を参照ください。

サブメニューと各種アーカイブは、最終的にはカスタムメニューとアーカイブ出力機能に置き換わることになります（中身に関しては、本項では省略します）

```html:右サイドバー
<aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1">
  <div class="bl_tagMenu">
    <ul class="bl_tagMenu_list">
      <li class="bl_tagMenu_item">
        <a href="./search.html" class="bl_tagMenu_link">サイト内検索</a>
      </li>
      <!-- /.bl_tagMenu_item -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_tagMenu">
    <!-- 省略 -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_tagMenu">
    <span class="bl_tagMenu_title">アーカイブ</span>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">カテゴリ一覧</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">年月アーカイブ</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
  </div>
  <!-- /.bl_tagMenu -->
</aside>
<!-- /.ly_mainArea_rightSidebar -->
```

### ダイアログメニュー

最後にダイアログメニューです。これは**B315：ly_dialog**で作成した構成に、先ほど作成した左右サイドバーの中身（一番外側に有る `nav`, `aside` タグを除いたもの）を加えたものとなっています。

```html:ダイアログメニュー
<dialog class="ly_dialog_wrapper" id="menu">
  <div class="ly_dialog">
    <div class="bl_menuHeader">
      <h2 class="bl_menuHeader_title">メニュー</h2>
      <button class="bl_menuHeader_btnIcon" type="button" onclick="closeDialog()">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24" width="24" height="24">
          <path stroke-width="1.4286" d="M 22,4.0142857 19.985715,2 12,9.9857149 4.0142857,2 2,4.0142857 9.9857149,12 2,19.985715 4.0142857,22 12,14.014286 19.985715,22 22,19.985715 14.014286,12 Z"/>
        </svg>
        閉じる
      </button>
    </div>
    <!-- /.bl_menuHeader -->

    <div class="bl_tagMenu">
      <ul class="bl_tagMenu_list">
        <li class="bl_tagMenu_item">
          <a href="./index.html" class="bl_tagMenu_link">Home</a>
        </li>
        <!-- /.bl_tagMenu_item -->
      </ul>
      <!-- /.bl_tagMenu_list -->
    </div>
    <!-- /.bl_tagMenu -->
    <div class="bl_tagMenu">
      <!-- 省略 -->
    </div>
    <!-- /.bl_tagMenu -->

    <hr>
    <div class="bl_tagMenu">
      <ul class="bl_tagMenu_list">
        <li class="bl_tagMenu_item">
          <a href="./search.html" class="bl_tagMenu_link">サイト内検索</a>
        </li>
        <!-- /.bl_tagMenu_item -->
      </ul>
      <!-- /.bl_tagMenu_list -->
    </div>
    <!-- /.bl_tagMenu -->

    <div class="bl_tagMenu">
      <!-- 省略 -->
    </div>
    <!-- /.bl_tagMenu -->

    <div class="bl_tagMenu">
      <span class="bl_tagMenu_title">アーカイブ</span>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">カテゴリ一覧</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">年月アーカイブ</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
    </div>
    <!-- /.bl_tagMenu -->

    <button class="el_btn el_btn__primary" type="button" onclick="closeDialog()">メニューを閉じる</button>
  </div>
  <!-- /.ly_dialog -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

## 完成形

最後に、完成形をまとめておきます。

:::details 左サイドバー

```html:左サイドバー
<nav class="ly_mainArea_leftSidebar" id="anchor_desktopMainMenu" tabindex="-1">
  <div class="bl_tagMenu">
    <ul class="bl_tagMenu_list">
      <li class="bl_tagMenu_item">
        <a href="./index.html" class="bl_tagMenu_link">Home</a>
      </li>
      <!-- /.bl_tagMenu_item -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_tagMenu">
    <!-- 省略 -->
  </div>
  <!-- /.bl_tagMenu -->
</nav>
<!-- /.ly_mainArea_leftSidebar -->
```

:::

:::details 右サイドバー

```html:右サイドバー
<aside class="ly_mainArea_rightSidebar" id="anchor_desktopSubMenu" tabindex="-1">
  <div class="bl_tagMenu">
    <ul class="bl_tagMenu_list">
      <li class="bl_tagMenu_item">
        <a href="./search.html" class="bl_tagMenu_link">サイト内検索</a>
      </li>
      <!-- /.bl_tagMenu_item -->
    </ul>
    <!-- /.bl_tagMenu_list -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_tagMenu">
    <!-- 省略 -->
  </div>
  <!-- /.bl_tagMenu -->
  <div class="bl_tagMenu">
    <span class="bl_tagMenu_title">アーカイブ</span>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">カテゴリ一覧</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">年月アーカイブ</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
  </div>
  <!-- /.bl_tagMenu -->
</aside>
<!-- /.ly_mainArea_rightSidebar -->
```

:::

:::details メニューダイアログ

```html:ダイアログメニュー
<dialog class="ly_dialog_wrapper" id="menu">
  <div class="ly_dialog">
    <div class="bl_menuHeader">
      <h2 class="bl_menuHeader_title">メニュー</h2>
      <button class="bl_menuHeader_btnIcon" type="button" onclick="closeDialog()">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24" width="24" height="24">
          <path stroke-width="1.4286" d="M 22,4.0142857 19.985715,2 12,9.9857149 4.0142857,2 2,4.0142857 9.9857149,12 2,19.985715 4.0142857,22 12,14.014286 19.985715,22 22,19.985715 14.014286,12 Z"/>
        </svg>
        閉じる
      </button>
    </div>
    <!-- /.bl_menuHeader -->

    <div class="bl_tagMenu">
      <ul class="bl_tagMenu_list">
        <li class="bl_tagMenu_item">
          <a href="./index.html" class="bl_tagMenu_link">Home</a>
        </li>
        <!-- /.bl_tagMenu_item -->
      </ul>
      <!-- /.bl_tagMenu_list -->
    </div>
    <!-- /.bl_tagMenu -->
    <div class="bl_tagMenu">
      <!-- 省略 -->
    </div>
    <!-- /.bl_tagMenu -->

    <hr>
    <div class="bl_tagMenu">
      <ul class="bl_tagMenu_list">
        <li class="bl_tagMenu_item">
          <a href="./search.html" class="bl_tagMenu_link">サイト内検索</a>
        </li>
        <!-- /.bl_tagMenu_item -->
      </ul>
      <!-- /.bl_tagMenu_list -->
    </div>
    <!-- /.bl_tagMenu -->

    <div class="bl_tagMenu">
      <!-- 省略 -->
    </div>
    <!-- /.bl_tagMenu -->

    <div class="bl_tagMenu">
      <span class="bl_tagMenu_title">アーカイブ</span>
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">カテゴリ一覧</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
      <details class="bl_accordion">
        <summary class="bl_accordion_summary">年月アーカイブ</summary>
        <ul class="bl_accordion_description">
          <!-- 省略 -->
        </ul>
      </details>
      <!-- /.bl_accordion -->
    </div>
    <!-- /.bl_tagMenu -->

    <button class="el_btn el_btn__primary" type="button" onclick="closeDialog()">メニューを閉じる</button>
  </div>
  <!-- /.ly_dialog -->
</dialog>
<!-- /.ly_dialog_wrapper -->
```

:::

## まとめ

本項では、左右サイドバーとメニューダイアログを作成しました。全体として流用できる箇所が多く、またWordPressの機能を使った箇所もあり、最終的な形を想定した構成にする必要がありました。といってもそれは難しい考えがあるわけではなく、大雑把に言ってしまうと「コピペで楽できるような、できるだけ使い回せる形に」といった考えに基づいています。

次項はフッター部を扱っていきます。その中には今回用意した内容が一部出てくることになります。
