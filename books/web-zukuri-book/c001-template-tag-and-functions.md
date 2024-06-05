---
title: "📄C001：主に使うテンプレートタグと関数集"
---

## このページについて

本項では、テーマ作成にあたって使用する主要な関数（テンプレートタグ含む）を説明していきます。

## テンプレートタグと関数集

ここからは各種要素を紹介していきます。見出し前に書かれている T, F, IF は下記の意味を表しています。

|表記|意味|
|:---|:---|
|T|テンプレートタグ|
|F|関数|
|IF|条件分岐タグ|

テンプレートタグの説明は下記にあるように、「WordPressに対し、DBから何かを取得するように指示する WordPress 関数の一部」です。個人的には関数の中で、テンプレートに関する使用を目的に作られているものを、そう呼ぶ印象を持っています。

https://developer.wordpress.org/themes/basics/template-tags/

### テンプレートパーツ呼び出し

ここにあるのは、テンプレートパーツを呼び出すためのものです。

#### T：`get_header()`

https://developer.wordpress.org/reference/functions/get_header/

`get_header( string $name = null, array $args = array() )`

テーマフォルダにある header.php を読み込む

|引数|内容|要求|
|:---|:---|---|
|`$name`|ファイル名。指定すると、任意のヘッダーファイルを読み込むことができる|任意|

例 : `get_header('sample')` で 、`header-sample.php` を読み込む。

#### T：`get_footer()`

https://developer.wordpress.org/reference/functions/get_footer/

`get_footer( string $name = null, array $args = array() )`

テーマフォルダにある footer.php を読み込む

|引数|内容|要求|
|:---|:---|---|
|`$name`|ファイル名。指定すると、任意のフッターファイルを読み込むことができる|任意|

例：`get_footer('sample')` で、`footer-sample.php` を読み込む。

#### T：`get_sidebar()`

https://developer.wordpress.org/reference/functions/get_sidebar/

`get_sidebar( string $name = null, array $args = array() )`

テーマフォルダにある sidebar.php を読み込む

|引数|内容|要求|
|:---|:---|---|
|`$name`|ファイル名。指定すると、任意のサイドバーファイルを読み込むことができる|任意|

例：`get_sidebar('sample')` で、`sidebar-sample.php` を読み込む。

#### T：`get_search_form()`

https://developer.wordpress.org/reference/functions/get_search_form/

`get_search_form( array $args = array() )`

テーマフォルダにある searchform.php を読み込む。テンプレートファイルがない場合は、WordPress標準の検索フォームHTMLを出力する。

|引数 `array()` のキー|内容|
|:---|:---|
|`echo`|検索フォームを `echo` として出力する場合は `true`（初期値）、しない場合は `false`|
|`aria-label`|検索フォームにつける ARIAラベル|

#### T：`get_template_part()`

https://developer.wordpress.org/reference/functions/get_template_part/

`get_template_part( string $slug, string|null $name = null, array $args = array() )`

汎用的に作成されたテンプレートファイルを読み込む

|引数|内容|要求|
|:---|:---|---|
|`$slug`|名称１となる、汎用テンプレート名。名称２と組み合わせてテンプレートファイルを指定できる|必須|
|`$name`|名称２となる、任意の名称。名称１と組み合わせてテンプレートファイルを指定できる|任意|
|`$args`|テンプレートファイルに引き渡す引数|任意|

例：下記のコードで、テーマディレクトリ上の `template-parts/sample-name.php` を読み込み、`array('key'=>'value', ...)` を渡す。

```php
get_template_part(
  'template-parts/sample',
  'name',
  array(
    'key' => 'value',
    'key2' => 'value2'
  )
);
```

### フック関係

ここにあるのは、フックに関係するものです。WordPress に `head` や `footer` 要素がどこにあるかを伝えるもので、これにより WordPress は*適切な場所で必要な処理を行う*ことができるようになります。

#### F：`wp_head()`

https://developer.wordpress.org/reference/functions/wp_head/

`wp_head` アクションを実行する

#### F：`wp_body_open()`

https://developer.wordpress.org/reference/functions/wp_body_open/

`wp_body_open` アクションを実行する

#### F：`wp_footer()`

https://developer.wordpress.org/reference/functions/wp_footer/

`wp_footer` アクションを実行する

### URL、パス関係

ホームURLやテーマディレクトリの場所を取得するもので、リンク要素や、各種ファイルのパス指定に使います。

#### T：`home_url()`

https://developer.wordpress.org/reference/functions/home_url/

`home_url( string $path = ”, string|null $scheme = null )`

WordPressの一般設定「サイトアドレス（URL）」の内容を取得する（末尾の `/` は省略）

|引数|内容|要求|
|:---|:---|---|
|`$path`|ホームURLからの相対パスを指定できる|任意|
|`$scheme`|ホームURLに使用するスキーム、`http`, `https`, `relative`, `rest`, `null` が指定できる|任意|

例：`home_url('/')` -> `http://example.com/`  
例：`home_url('section1')` -> `http://example.com/section1`

#### F：`get_template_directory_uri()`

https://developer.wordpress.org/reference/functions/get_template_directory_uri/

現在有効になっているテーマディレクトリのURIを取得する。

子テーマを使用している場合は、親テーマのテーマディレクトリのURIを取得する。子テーマのディレクトリを取得したい場合には、`get_stylesheet_directory_uri()` を使用する

### ファイル読み込み

スタイルシート、及びスクリプトファイルを読み込むためのものです。

#### F：`wp_enqueue_script()`

https://developer.wordpress.org/reference/functions/wp_enqueue_script/

```php
wp_enqueue_script(
  string $handle,
  string $src = ”,
  string[] $deps = array(),
  string|bool|null $ver = false,
  array|bool $args = array()
)
```

JavaScriptファイルの依存関係を考慮し、適切にファイルへのパスを出力する

|引数|内容|要求|
|:---|:---|---|
|`$handle`|ハンドル名、読み込むファイルを区別するための一意な名称|必須|
|`$src`|ファイルパス、JavaScriptファイルのパス|任意|
|`$deps`|依存関係（配列形式）、そのJavaScriptファイルより先に読み込まれる必要のある*ハンドル名*を指定|任意|
|`$ver`|バージョン、JavaScriptファイルのバージョン|任意|
|`$args`|追加のスクリプト読み込みオプション|任意|

|引数 `$args()` のキー|内容|
|:---|:---|
|`strategy`|`defer`, `async` を指定可能|
|`in_footer`|`footer`（≒`wp_footer` の箇所）で出力するか。`false`（初期値）の場合は `header`（≒`wp_header` の箇所）で出力|

#### F：`wp_enqueue_style()`

https://developer.wordpress.org/reference/functions/wp_enqueue_style/

```php
wp_enqueue_style(
  string $handle, 
  string $src = '',
  string[] $deps = array(),
  string|bool|null $ver = false,
  string $media = 'all' 
)
```

CSSファイルの依存関係を考慮し、適切にファイルへのパスを出力する

|引数|内容|要求|
|:---|:---|---|
|`$handle`|ハンドル名、読み込むファイルを区別するための一意な名称|必須|
|`$src`|ファイルパス、CSSファイルのパス|任意|
|`$deps`|依存関係（配列形式）、そのCSSファイルより先に読み込まれる必要のある*ハンドル名*を指定|任意|
|`$ver`|バージョン、CSSファイルのバージョン|任意|
|`$media`|メディア属性、`all`, `screen`, `print` などの `media` 属性を指定する|任意|

### WordPress テーマ機能の拡張

WordPress のテーマで、利用できる機能を拡張するものです。

#### F：`add_theme_support()`

https://developer.wordpress.org/reference/functions/add_theme_support/

`add_theme_support( string $feature, mixed $args )`

テーマサポートを使用する。

|引数|内容|要求|
|:---|:---|---|
|`$feature`|追加する機能名を指定|必須|
|`$args`|追加機能に渡す引数|任意|

追加できる機能一覧

- `post-thumbnails`: 投稿サムネイル
- `custom-header`: カスタムヘッダ
- `custom-logo`: カスタムロゴ機能を使用
- `html5`: HTML5マークアップ（下記）を使用
  - `search-form`
  - `comment-list`
  - `gallery`
  - `caption`
  - `style`
  - `script`

#### F：`add_image_size()`

https://developer.wordpress.org/reference/functions/add_image_size/

`add_image_size( string $name, int $width, int $height, bool|array $crop = false )`

新しい画像サイズを登録する

|引数|内容|要求|
|:---|:---|---|
|`$name`|画像サイズ名、新しく登録する画像サイズの名前|必須|
|`$width`|画像幅、新しく登録する画像サイズの幅（`px`）|任意|
|`$height`|画像高さ、新しく登録する画像サイズの高さ（`px`）|任意|
|`$crop`|切り抜き可否、画像の切り抜きをどうするか|任意|

#### F：`register_nav_menu()`

https://developer.wordpress.org/reference/functions/register_nav_menu/

`register_nav_menu( string $location, string $description )`

**単独の**カスタムメニューを登録する

|引数|内容|要求|
|:---|:---|---|
|`$location`|識別子、カスタムメニューの識別子|必須|
|`$description`|メニュー名、カスタムメニューの名前|必須|

#### F：`register_nav_menus()`

https://developer.wordpress.org/reference/functions/register_nav_menus/

`register_nav_menus( string[] $locations = array() )`

**複数の**カスタムメニューを登録する

|引数|内容|要求|
|:---|:---|---|
|`$locations`|識別子とメニュー名の連想配列、`register_nav_menu()` の内容を配列にしたもの|必須|

例：下記のコードで、`main-menu`, `footer-menu` の２つを登録できる。

```php
register_nav_menus(array(
  'main-menu' => 'メインメニュー'
  'footer-menu' => 'フッターメニュー'
));
```

#### T：`wp_nav_menu()`

https://developer.wordpress.org/reference/functions/wp_nav_menu/

`wp_nav_menu( array $args = array() )`

カスタムメニューを表示する。

|引数|内容|要求|
|:---|:---|---|
|`$args`|表示するメニューを連想配列で指定する|任意|

|引数 `$args()` キーの一部|内容|
|:---|:---|
|`theme_location`|メニューの表示位置、`register_nav_menu()` 等で登録した識別子を使用|
|`menu_class`|メニューの `ul` 要素に指定するクラス|
|`menu_id`|メニューの `ul` 要素に指定するID|
|`container`|メニューの `ul` 要素を囲むコンテナ要素（`div` または `nav`）、不要な場合は `false` を指定|
|`container_class`|コンテナ要素に指定するクラス|
|`container_id`|コンテナ要素に指定するID|
|`items_wrap`|メニューの `ul` 要素のフォーマット（`<ul id="%1$s" class="%2$s">%3$s</ul>`）|

### その他

#### F：`get_month_link()`

https://developer.wordpress.org/reference/functions/get_month_link/

`get_month_link( int|false $year, int|false $month )`

指定した年月の月別アーカイブのURLを返す。  
（引数を空欄にすると、現在の年月を使ってリンクを生成する）

|引数|内容|要求|
|:---|:---|---|
|`$year`|アーカイブの年|必須|
|`$month`|アーカイブの月|必須|

#### F：`language_attributes()`

https://developer.wordpress.org/reference/functions/language_attributes/

`language_attributes( string $doctype = ‘html’ )`

WordPress「サイトの言語」で設定した言語属性をhtmlタグに出力する

|引数|内容|要求|
|:---|:---|---|
|`$doctype`|ドキュメント種類、`html`, `xhtml` を指定できる|任意|

#### T：`bloginfo()`

https://developer.wordpress.org/reference/functions/bloginfo/

`bloginfo( string $show = '' )`

WordPressの情報を出力する

|引数|内容|要求|
|:---|:---|---|
|`$show`|出力する情報名|任意|

出力できる情報名（一部）

- `charset` : ウェブサイトの文字コード
- `description` : 一般設定「キャッチフレーズ」
- `name` : 一般設定「サイトのタイトル」

#### T：`body_class()`

https://developer.wordpress.org/reference/functions/body_class/

`body_class( string|string[] $css_class = ” )`

bodyタグにclass属性を出力する

|引数|内容|要求|
|:---|:---|---|
|`$css_class`|クラス名、任意の文字列（配列可）を渡すと、クラス名として出力される|任意|

#### T：`post_class()`

https://developer.wordpress.org/reference/functions/post_class/

`post_class( string|string[] $css_class = ”, int|WP_Post $post = null )`

|引数|内容|要求|
|:---|:---|---|
|`$css_class`|クラス名、任意の文字列（配列可）を渡すと、クラス名として出力される。|任意|
|`$post`|投稿ID もしくは投稿オブジェクト（初期値は現在の投稿ページ）|任意|

#### T：`wp_list_categories()`

https://developer.wordpress.org/reference/functions/wp_list_categories/

`wp_list_categories( array|string $args = '' )`

カテゴリページへのリンク一覧を表示する

|引数 `$args()` キーの一部|内容|
|:---|:---|
|`title_li`|見出し（省略時は `Categories` ）|
|`show_count`|投稿数を表示するか（省略時はfalse）|
|`depth`|`ul` タグと`li` タグによる階層化をするか、フラット化する場合は `false` を指定|
|`orderby`|ソート対象、`id`, `name`, `slug`, `count`, `term_group` 等がある（省略時は `name`）|
|`order`|ソート順、`ASC` で昇順、`DESC` で降順|
|`hide_empty`|投稿記事がないカテゴリーを表示するかどうか（省略時は `true`）|

#### IF：`has_post_thumbnail()`

https://developer.wordpress.org/reference/functions/has_post_thumbnail/

`has_post_thumbnail( int|WP_Post $post = null )`

現在の投稿もしくは固定ページに、アイキャッチ画像が登録されているかを判定する

|引数|内容|要求|
|:---|:---|---|
|`$post`|投稿IDもしくは投稿オブジェクトを指定、省略時は現在の投稿となる|任意|

#### T：`single_term_title()`

https://developer.wordpress.org/reference/functions/single_term_title/

`single_term_title( string $prefix = ”, bool $display = true )`

現在のアーカイブページのタクソノミーを表示、または取得する

|引数|内容|要求|
|:---|:---|---|
|`$prefix`|接頭辞、タクソノミー名の前に出力するテキスト|任意|
|`$display`|出力可否、タクソノミー名を表示する場合は `true`（初期値）、PHPで使う値として取得する場合は `false`|任意|

#### IF：`is_month()`

https://developer.wordpress.org/reference/functions/is_month/

クエリ処理が月別アーカイブかどうかを判定する（≒現在のページが 月アーカイブ かを判定）

#### F：`get_the_excerpt()`

https://developer.wordpress.org/reference/functions/get_the_excerpt/

`get_the_excerpt( int|WP_Post $post = null )`

現在の投稿の抜粋を取得する

|引数|内容|要求|
|:---|:---|---|
|``||必須|
|`$post`|任意の 投稿ID もしくはオブジェクトを指定できる|任意|

:::message

純粋な文字列のみが返されるので、HTML上に出力する場合は `echo` と `esc_html` を使用する。

pタグで囲んだ形で出力が欲しい場合には、`the_excerpt()` を使用する。

:::

#### F：`get_the_category()`

https://developer.wordpress.org/reference/functions/get_the_category/

`get_the_category( int $post_id = false )`

現在の投稿が属するカテゴリの情報を配列として取得する。投稿がカテゴリに属していない場合は、空の配列を返す。

|引数|内容|要求|
|:---|:---|---|
|`$post`|任意の 投稿ID もしくはオブジェクトを指定できる|任意|

#### T：`the_search_query()`

https://developer.wordpress.org/reference/functions/the_search_query/

検索フォームに入力された検索ワードを表示する

#### T：`single_post_title()`

https://developer.wordpress.org/reference/functions/single_post_title/

`single_post_title( string $prefix = ”, bool $display = true )`

記事・固定ページのタイトルをループの外で表示する

|引数|内容|要求|
|:---|:---|---|
|`$prefix`|タイトル前の文字列、ページタイトルの前に表示する文字列|任意|
|`$display`|出力可否、出力する場合は `true` を選択|任意|

#### F：`WP_Query()`

https://developer.wordpress.org/reference/classes/wp_query/

クエリ用のクラス。定義された引数から、要求された投稿データを取得できる。

|引数（一部） | 説明 | 値の例|
|:---|:---|:---|
|`post_type`| 出力する投稿タイプ | `post`|
|`category_name`| 投稿カテゴリのスラッグ名、複数指定も可能 | `category1`|
|`post_per_page`| 表示数、-1を指定すると全件表示 | 1|
|`order`| 出力の並び順、`ASC` は昇順、`DESC` は降順 | `ASC`|
|`orderby`| どの引数を基準として並び替えるか | `ID`, `date`, `modified`, `title`, `rand`|

#### F：`get_the_ID()`

https://developer.wordpress.org/reference/functions/get_the_id/

現在表示している投稿のIDを取得する、必ずループ中で使用する

#### T：`the_title()`

https://developer.wordpress.org/reference/functions/the_title/

`the_title( string $before = ”, string $after = ”, bool $display = true )`

現在の投稿のタイトルを表示/取得する。必ずループ中で使用する。

|引数|内容|要求|
|:---|:---|---|
|`$before`|タイトル前の文字列、タイトルの前に出力する文字列|任意|
|`$after`|タイトル後の文字列、タイトルの後に出力する文字列|任意|
|`$display`|出力可否、タイトルを出力するかどうか|任意|

#### T：`the_content()`

https://developer.wordpress.org/reference/functions/the_content/

`the_content( string $more_link_text = null, bool $strip_teaser = false )`

投稿又は固定ページの本文を全て、または一部を出力する。必ずループ中で使用する。

|引数|内容|要求|
|:---|:---|---|
|`$more_link_text`|続きリンクの文字列、アーカイブテンプレートで使用する場合、「続き」ブロック（`<!--more-->`）で分割された本文の続きを読むためのリンクに表示する文字列。省略時は「さらに・・・」と表示される|任意|
|`$strip_teaser`|分割点前の表示（オプション）：分割された本文の続きに移動した時、分割点より前の内容を表示するかどうか|任意|

#### T：`the_permalink()`

https://developer.wordpress.org/reference/functions/the_permalink/

投稿（引数指定しない場合は現在の投稿）のパーマリンクを表示する、必ずループ中で使用する

`the_permalink( int|WP_Post $post )`

|引数|内容|要求|
|:---|:---|---|
|`$post`|任意の 投稿ID もしくはオブジェクトを指定できる|任意|

#### T：`the_excerpt()`

https://developer.wordpress.org/reference/functions/the_excerpt/

現在の投稿の抜粋を文末に `[...]` をつけて出力する。抜粋文からはHTMLタグと画像は取り除かれる。必ずループ中で使用する

:::message

pタグがついた形で出力される。

純粋に抜粋文の文字列のみ欲しい場合には、`get_the_excerpt()` を使用する。

:::

#### T：`the_category()`

https://developer.wordpress.org/reference/functions/the_category/

`the_category( string $separator = '', string $parents = '', int $post_id = false )`

現在の投稿が属するカテゴリへのリンクを表示、必ずループ中で使用する。

|引数|内容|要求|
|:---|:---|---|
|`$separator`|区切り文字、カテゴリリンクを区切る文字や記号|任意|
|`$parents`|親カテゴリの表示方法、`multiple` の場合、親と子のカテゴリリンクをそれぞれ出力する。`single` の場合、子カテゴリへのリンクのみを出力|任意|
|`$post_id`|投稿ID、カテゴリを取得する投稿のID。初期値は現在の投稿のカテゴリ|任意|

#### T：`the_tags()`

https://developer.wordpress.org/reference/functions/the_tags/

`the_tags( string $before = null, string $sep = ‘, ‘, string $after = ” )`

現在の投稿が属するタグへのリンクを表示、必ずループ中で使用する。

|引数|内容|要求|
|:---|:---|---|
|`$before`|タグ一覧**前の**文字列、タグ一覧の前に出力する文字列。初期値は「タグ : 」|任意|
|`$sep`|区切り文字、タグリンクを区切る文字や記号|任意|
|`$after`|最後のタグ後の文字列、**最後の**タグリンクの後に出力する文字列|任意|

#### T：`the_ID()`

https://developer.wordpress.org/reference/functions/the_id/

現在の投稿または固定ページのIDを出力する。必ずループ中で使用する。

#### F：`the_post_thumbnail()`

https://developer.wordpress.org/reference/functions/the_post_thumbnail/

`the_post_thumbnail( string|int[] $size = ‘post-thumbnail’, string|array $attr = ” )`

投稿または固定ページのアイキャッチ画像を出力する。必ずループ中で使用する。

|引数|内容|要求|
|:---|:---|---|
|`$size`|サイズ、画像サイズをキーワード、または幅と高さの配列 で指定|任意|
|`$attr`|属性、アイキャッチ画像のimg要素に付加する属性や値を配列で記述|任意|

#### T：`get_the_date()`

https://developer.wordpress.org/reference/functions/get_the_date/

`get_the_date( string $format = ”, int|WP_Post $post = null )`

現在の投稿の投稿日を表示する、必ずループ中で使用する。

|引数|内容|要求|
|:---|:---|---|
|`$format`|日付書式。時間を表示する書式、初期値はWordPressで設定された形式が表示される|任意|
|`$post`|任意の 投稿ID もしくはオブジェクトを指定できる|任意|

:::message

同じく日付を出力するテンプレートタグに `the_date()` がある。しかしこのテンプレートタグはループ中で同じ日付を**１回しか表示できない**。

:::

#### F：`function_exist()`

https://www.php.net/manual/ja/function.function-exists.php

`function_exists(string $function)`

引数に設定した関数が定義されているかチェックするPHP関数

|引数|内容|要求|
|:---|:---|---|
|`$function`|関数名、定義状況をチェックする関数の名前|必須|

## まとめ

本項では、テーマ作成にあたって使用するPHP関数を説明しました。
