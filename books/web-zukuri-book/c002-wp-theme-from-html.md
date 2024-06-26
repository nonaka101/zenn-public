---
title: "📄C002：静的ページからのテーマ作成"
---

## このページについて

本項では、第二章で作成した静的ページを基に、WordPressテーマを作成する流れを見ていきます。

## WordPressテーマファイルについて

:::message

WordPress についての大まかな説明は**B001：構成**を参照してください。

:::

ここからは、第三章で作成していくテーマファイルがどういった構成になるかを見ていきます。

### ページテンプレート

WordPress は要求された内容に対し、ページテンプレートに当てはめることでページを生成します。必要となるページ種類については、既に第二章で作成しました。

|ページ種類|静的ページ名|ページテンプレート名|
|:---|:---|:---|
|トップ（フロント）|`index.html`|`front-page.php`|
|404|`404.html`|`404.php`|
|検索|`search.html`|`search.php`|
|アーカイブ|`archive.html`|`archive.php`|
|投稿・固定【※１】|`single.html`, `page.html`|`singular.php`|
|（インデックス）【※２】|（未作成）|`index.php`|

:::message

※１：投稿・固定ページは、構成が ほぼ一緒である関係上 一つのページテンプレートに統合します  
※２：`index.php` は要求内容に合うページテンプレートがない場合の特殊なものなので、静的ページは作成していません

:::

実は 拡張子 `html` を `php` に変更すれば、テーマファイルの3割程度は完成したことになります。ここからは `php` を用いたプログラミングを施していくことになります。

`php` を用いる箇所は、**様々な場所で用いられている部品をテンプレート化する**箇所と、**（DB上のデータを用いて）コンテンツを動的にする**箇所です。

### テンプレートパーツ化による共有

![フロントページをテンプレートパーツごとに区切ったイメージ図](/images/books/web-zukuri/about-template-parts-01.png)

`php` を用いる箇所の1つ目は、各ページ間で共通している場所です。こうした所を**テンプレートパーツ**として独立したファイルにしていくために `php` を使います。上図はページテンプレート `front-page.php` に対し、他ページと共通する部品 `header.php`, `sidebar-left.php`, `sidebar-right.php` を当てはめている図になります。

#### 何故テンプレートパーツとして独立させるか？

![テンプレートパーツの1つであるサイドバーのファイルとコード](/images/books/web-zukuri/about-template-parts-02.png)
*分離し、共有できることがメリット*

テンプレートパーツとして独立させるメリットは、その**保守性**にあると考えます。上図は右サイドバーをテンプレートパーツ化したファイルと そのコードになります。右サイドバーを使うページは、フロントページだけではありません。本テーマに関しては、全てのページに使用しています。

「テンプレートパーツ化せず、全てのページテンプレートにコードを直打ちした場合」と比較し、何故テンプレートパーツとして独立させるかを考えてみます。

まず考えられるのは、変更漏れの恐れです。仮に ここに対し変更が生じた場合、全てのファイルでコードの打ち直しが発生します。もし手打ちでやってる場合は、変更漏れが起きるかもしれません。しかしテンプレートパーツとして1つのファイルとして管理していた場合、修正が必要となるのは そのファイルのみとなります。

また、1ファイルに対するコードの可読性にも貢献してくれます。テンプレートパーツ化しない場合、1画面の中に情報が多量に含まれることになります。コードを見る人は頭の中で「自分が何をしたいか」だけでなく、「どの部分が何を担っているか」をも管理しなければなりません。テンプレートパーツ化した場合、何行にも渡るコードが、呼び出し関数1つに置き換わることになります。役割ごとに切り分けることにより、コードを見る人は、自身の目的とする場所に集中することができるでしょう。

#### 主なテンプレートパーツ

本テーマの HTML構成は、ページ間で ほとんど同じとなっています。もう少し詳しく見てみますと、下記のような構成になっています。

|構成|
|:---|
|ヘッド（スタイルやメタ情報 等）|
|ヘッダー|
|ブロックスキップ|
|メニューダイアログ|
|左サイドバー（※フロントページのみ）|
|メインコンテンツ|
|右サイドバー|
|フッター|
|ページトップへ戻るボタン|
|開発者ツール|
|スクリプト関係|

各ページテンプレートでこの構成を使っていますが、そこから役割に応じ、テンプレートパーツとして独立させていきます。基本的には WordPress が専用で用意している呼び出し関数を使うことを念頭に、独自のものはそれらに含むよう構成しています。

|構成|テンプレートパーツ名|
|:---|:---|
|ヘッド（スタイルやメタ情報）|（`header.php` 側に統合）|
|ヘッダー|`header.php`|
|ブロックスキップ|（`header.php` 側に統合）|
|メニューダイアログ|（`header.php` 側に統合）|
|左サイドバー|`sidebar-left.php`|
|メインコンテンツ|なし（基本的に各ページテンプレート内で記述）|
|右サイドバー|`sidebar-right.php`|
|フッター|`footer.php`|
|ページトップへ戻るボタン|（`footer.php` 側に統合）|
|開発者ツール|（`footer.php` 側に統合）|
|スクリプト関係|（`footer.php` 側に統合）|

:::message

開発者ツールに関しては、更に `parts-devutils.php` へと分離したりします。これはテーマカスタマイザー設定で、開発者モードの切り替えを行うようするためです。

:::

これにより、例えばフロントページのテンプレートは、下記のような形へと変化します。

```php:front-page.php
<?php get_header(); // ヘッダー部品の呼び出し ?>
<?php get_sidebar('left');  // 左サイドバーの呼び出し ?>

<main class="ly_mainArea_content ly_mainArea_content__middle" id="anchor_mainContent" tabindex="-1">
  <!-- メインコンテンツの中身（省略） -->
</main>
<!-- /.ly_mainArea_content -->

<?php get_sidebar('right'); // 右サイドバーの呼び出し ?>
<?php get_footer(); // フッターの呼び出し ?>
```

元のHTMLデータでは、ダミーデータを入れているにしても 1178 行ありました。それがテンプレートパーツで役割ごとに別ファイル化し、呼び出す形にすることで、その行数は1桁程度まで短縮することができました（メインコンテンツのコード（省略された箇所）があるので実際はもう少しありますが）。

### 動的コンテンツの生成

`php` を用いる箇所の2つ目は、動的コンテンツにしたい所です。

第二章で作成した `html` は**静的**ページなので、記事内容やリンクはそのままです。WordPress 上で作成した記事等を**動的に**反映させていくためには、`php` 言語を用いたプログラミングを施していく必要があります。

下記は、動的コンテンツの例となります。

#### 例：カスタムメニューの設定

![WordPress カスタムメニュー編集画面と、その反映イメージ](/images/books/web-zukuri/about-custom-menu-01.png)
*WordPress の テーマカスタマイザー画面*

WordPress 側で設定した情報をページに反映させるために、`php` は使われます。上図は設定したカスタムメニューの内容を左サイドバーに反映させている状況です。

#### 例：ページネーション要素

![アーカイブページ下部にあるページネーション要素](/images/books/web-zukuri/page-archive-02.png)

アーカイブページや検索ページなどで、条件に一致する記事がたくさんある場合、ページ区切りが挿入されます。この要素は20ページにまたがることもあれば、2ページに収まる場合もあり、その内容は一定ではありません。

DBから取得した記事件数に沿ったページネーション要素にするために、`php` による制御が必要となります。

#### 例：各種カードコンポーネントへの出力

![DBからの記事を、使用する場所に応じたカード形式に変換している図](/images/books/web-zukuri/about-template-parts-03.png)

WordPress から取得した記事データを どのような形に落とし込むか、`php` を使って制御することになります。上図は記事データを（場所ごとの）カードコンポーネントに落とし込む図です。記事データのタイトルやカテゴリ情報を HTML 上の適切な箇所に当てはめ、更に要旨やサムネイルなどはカードの場所に応じて出力を切り替える必要があります。

#### 動的コンテンツを担うテンプレートパーツ

[主なテンプレートパーツ](#主なテンプレートパーツ)で説明したものは、ページ間の共通部品としての役割が強いものでした。ここで紹介するテンプレートパーツは、どちらかと言うと動的コンテンツを生成するためのものになります。

|テンプレートパーツ名|役割|
|:---|:---|
|`loop-post.php`|投稿記事をカードに当てはめる|
|`parts-categories-and-archives.php`|カテゴリと月別のアーカイブを生成する|
|`parts-menu.php`|メニュー（サイドバー等に使われているタグメニュー）を生成する|
|`parts-pagenation.php`|ページネーション要素を生成する|
|`searchform.php`|検索フォーム要素を生成する|

## まとめ

本項では第二章で作成した静的ページを基に、どのようにWordPress テーマを作っていくのかについて、その概要を説明しました。  
WordPress テーマは主に `php` を使って記述していくことになりますが、それは静的ページではできなかったこと（動的なコンテンツ、部品の共通化）を行うためになります。

次項からは、実際に `html` ファイルをテーマ化していくことになります。
