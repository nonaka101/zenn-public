---
title: "📄B306：bl_tagMenu"
---

## このページについて

ここでは、メニュー項目をまとめるリスト要素 `bl_tagMenu` について説明します。

![タグメニュー本体。「サンプルページ一覧」というタイトルがあり、その中にリストとして「投稿ページ」「固定ページ」とメニュー項目が並んでいる](/images/books/web-zukuri/bl_tagMenu-01.png)

上図のように、この要素はいくつかのメニュー項目をまとめる役割を持っています。WordPressでは、「メインメニュー」「サブメニュー」などの形でカスタムメニューとして表示させる想定です。

使用される場所は、下記の通りです。

- 左サイドバー
- 右サイドバー
- フッター（左右ウィジェット）
- （モバイル画面の）メニューダイアログ

## 構造の定義

ここでは、どのような形にしていくかを考えていきます。

大まかに、下記のような形をイメージ

- リスト要素 `ul` を使いメニュー項目をまとめる
- リスト前に、そのリストのタイトルを表す要素を入れる
- スタイルについては、旧デジタル庁のサイトにあった内容を参考にする
- メニュー項目内に*メニューがネストされる*可能性を考慮する

:::message

最後の*ネストされたメニュー*については、WordPress のカスタムメニューに関係します。WordPress の管理画面からカスタムメニューを設定できますが、メニュー項目内にサブメニューを配置することができます。

:::

以上のことをまとめると、下図のような形になります。

![bl_tagMenuの構造図を表している。bl_tagMenu:全体, bl_tagMenu_title:タイトル, bl_tagMenu_list:リスト, bl_tagMenu_item:リストアイテム, bl_tagMenu_link:リンク文](/images/books/web-zukuri/bl_tagMenu-02.png)

## 作成

上記で決めた要件から、マークアップしていきます。

### HTML

まずは `html` で作成し、クラス付けを行います。

```html
<div class="bl_tagMenu">
  <span class="bl_tagMenu_title">メニュータイトル</span>
  <ul class="bl_tagMenu_list">
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素１</a>
    </li>
    <!-- /.bl_tagMenu_item -->
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素２</a>
    </li>
    <!-- /.bl_tagMenu_item -->
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素３</a>
    </li>
    <!-- /.bl_tagMenu_item -->
  </ul>
  <!-- /.bl_tagMenu_list -->
</div>
<!-- /.bl_tagMenu -->
```

このように、ブロックモジュールは下記のような形で運用します。

- ベースとなる要素名 `bl_tagMenu` は、接頭辞 `bl` とコンポーネント名 `tagMenu` で構成される（複数単語はローワーキャメル）
- ベース要素の下位に子要素が置かれ、`bl_tagMenu_title`, `bl_tagMenu_list` のように `ベースネーム_子要素名` で構成される
- 子要素の子要素は、更にアンダーバーで繋げることはしない（`bl_tagMenu_item`, `bl_tagMenu_link`）

### CSS

ここから、スタイルを記述します。

タグメニュー自体は、下部にスペースを持つようします。これは、レイアウト上のものなので `px` 単位とします。

```css
.bl_tagMenu {
  margin-bottom: 40px;
}
```

次にタイトル部です。  
`span` 要素を想定しているため、`display: block;` を明示します。

```css
.bl_tagMenu_title {
  display: block;
  margin-bottom: 0.2rem;
  font-size: 0.7rem;
  color: var(--sc_txt_desc);
}
```

次にリスト部（全体）です。

```css
.bl_tagMenu_list {
  padding: 0;
  margin: 0;
  list-style-type: none;
}
```

なお、WordPress側でサブメニュー（メニュー項目内に別のメニューリストを挿入）を使うことを考慮する必要があります。サブメニューは他の項目より右にインデントするようにし、モディファイアである`bl_tagMenu_list__indented` を設けます。

```css
/* `<ul class="bl_tagMenu">` 内で使用することを想定 */
.bl_tagMenu_list.bl_tagMenu_list__indented {
  padding-left: 1em;
}
```

最後に、中にあるリストアイテムとリンク要素です。リンク要素はホバー時にアンダーラインが出るようにします。

```css
.bl_tagMenu_item {
  margin-bottom: 0.75em;
}

.bl_tagMenu_link {
  font-size: 0.9rem;
  font-weight: 700;
  color: var(--sc_txt_body);
  text-decoration: none;
}

.bl_tagMenu_link:hover {
  text-decoration: underline;
}
```

これで大枠は完成しました。

## アクセシビリティへの対応

ここからは、アクセシビリティを踏まえた改良を加えていきます。

### 問題点

2023年の段階では、Safariにおいて `list-style: none;` を指定した `<ul>` はリストと読み上げられない問題があります。

https://developer.mozilla.org/ja/docs/Web/CSS/list-style#%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B7%E3%83%93%E3%83%AA%E3%83%86%E3%82%A3%E3%81%AE%E8%80%83%E6%85%AE

また、リストのタイトルである `span` 要素との紐づけも行っていないため、スクリーンリーダーは順序通り読み上げることになります。Safariにおいては下記の読み上げとなり、少しわかりづらいです。

- 「メニュータイトル」
- 「要素１、リンク」
- 「要素２、リンク」

### 改良

#### `ul` 要素に `role="list"` を付与

Safariも他のブラウザと同じ挙動にするために、`role="list"` を付与します。暗黙のロールと同じなので一見意味がないように見えますが、こうすることでSafariでもリストと認識して読み上げてくれます。

```html
<ul class="bl_tagMenu_list" role="list">
  ...
</ul>
```

#### `span` 要素に `id` をつけ、`ul` 要素に `aria-labelledby` で紐づける

リストのタイトルとして読み上げてもらうため、`aria-labelledby` による紐づけを行います。

```html
<span class="bl_tagMenu_title" id="list-title">メニュータイトル</span>
<ul class="bl_tagMenu_list" role="list" aria-labelledby="list-title">
  ...
</ul>
```

但し、このままだと `span` 要素が２回読み上げられてしまいます（「メニュータイトル」「メニュータイトル、リスト」「要素１、リンク」… ）  
そこで、`aria-hidden` をつけAOM上からは認識できないようにします。

`span`要素に差し掛かった時、スクリーンリーダーはこれを読み上げません。一方で、`ul`要素の `aria-labelledby` は、その参照先である `span` 要素をきちんと読み上げてくれます。

```html
<span class="bl_tagMenu_title" id="list-title" aria-hidden="true">メニュータイトル</span>
<ul class="bl_tagMenu_list" role="list" aria-labelledby="list-title">
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素１</a>
    </li>
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素２</a>
    </li>
</ul>
```

これで、どのブラウザでも望ましいと考える読み上げをしてくれるようになります。

- 「リスト メニュータイトル ２項目」
- 「要素１、２の１」
- 「要素２、２の２」
- 「リストの終わり」

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html
<div class="bl_tagMenu">
  <span
    class="bl_tagMenu_title"
    id="external_link_in_footer"
    aria-hidden="true"
  >
    メニュータイトル
  </span>
  <ul
    class="bl_tagMenu_list"
    role="list"
    aria-labelledby="external_link_in_footer"
  >
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素１</a>
    </li>
    <!-- /.bl_tagMenu_item -->
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素２</a>
    </li>
    <!-- /.bl_tagMenu_item -->
    <li class="bl_tagMenu_item">
      <a href="#" class="bl_tagMenu_link">要素３</a>
    </li>
    <!-- /.bl_tagMenu_item -->
  </ul>
  <!-- /.bl_tagMenu_list -->
</div>
<!-- /.bl_tagMenu -->
```

```css:style.css
.bl_tagMenu {
  margin-bottom: 40px;
}

.bl_tagMenu_title {
  display: block;
  margin-bottom: 0.2rem;
  font-size: 0.7rem;
  color: var(--sc_txt_desc);
}

.bl_tagMenu_list {
  padding: 0;
  margin: 0;
  list-style-type: none;
}

.bl_tagMenu_list.bl_tagMenu_list__indented {
  padding-left: 1em;
}

.bl_tagMenu_item {
  margin-bottom: 0.75em;
}

.bl_tagMenu_link {
  font-size: 0.9rem;
  font-weight: 700;
  color: var(--sc_txt_body);
  text-decoration: none;
}

.bl_tagMenu_link:hover {
  text-decoration: underline;
}
```

:::

## まとめ

このページでは、メニューリストを扱う `bl_tagMenu` について解説しました。  
また今回はブロックエレメントを扱う最初の回だったため、命名規則や運用などについても説明しました。

次回は投稿ページで使用される、投稿データの補足情報 `bl_postData` について扱っていきます。
