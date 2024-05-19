---
title: "📄B304：el_link"
---

## このページについて

ここでは、リンク要素を扱うためのエレメント `el_link` について説明します。

![el_link のスクリーンショット](/images/books/web-zukuri/el_link-01.png =250x)
*上部がライトモード、下部がダークモード時*

ここでは、`:acrive`, `:visited`, `:hover` といった**擬似クラス**を扱っていきます。

## 構造の定義

ここでは、どのような形にしていくかを考えていきます。

- `el_link` を使用するのは、インライン要素としての `a` タグのみを想定
- 基本的には `a` タグのデフォルトスタイルに準ずる（特殊なことはせず、普段見慣れている形に留める）
- ただしデフォルトではダークモード時にコントラスト比が低く見づらいので、適切な色を使うよう対処する

このように、基本的には下記のように `a` タグのデフォルトスタイルに近しいものになります。

- テキスト色は青色
- リンク要素であることがわかるアンダーライン
- 訪問済み状態、ホバー状態で色が変化

## 作成

上記で決めた要件から、マークアップしていきます。  
ここで使用する `html` 文は、下記のようになります。

```html:使用例
<a href="#" class="el_link">Sample</a>
```

### セマンティックカラーの設定

既にリンクに関するセマンティックカラーは設定済みです。下記のカスタムプロパティを使っていくことになります。

```css:セマンティックカラー（一部抜粋）
:root {
  /* プリミティブカラー */
  --sea-1200: #00004F;
  --sea-1100: #000060;
  --sea-1000: #000071;
  --sea-900: #0000be;
  --sea-800: #0024ce;
  --sea-700: #0031d8;
  --sea-600: #003ee5;
  --sea-500: #0946f1;
  --sea-400: #4979f5;
  --sea-300: #7096f8;
  --sea-200: #9db7f9;
  --sea-100: #c5d7fb;
  --sea-50: #e8f1fe;

  /* セマンティックカラー */
  --sc_txt_link: var(--sea-800);
  --sc_txt_link__hover: var(--sea-900);
  --sc_txt_link__active: var(--sea-900);
  --sc_txt_link__visited: var(--sea-900);
}

/* セマンティックカラー（ダークモード時） */
@media (prefers-color-scheme: dark) {
  :root {
    --sc_txt_link: var(--sea-300);
    --sc_txt_link__hover: var(--sea-200);
    --sc_txt_link__active: var(--sea-200);
    --sc_txt_link__visited: var(--sea-200);
  }
}
```

### ベース `el_link`

- 色は `--sc_txt_link`
- リンクとわかるよう、アンダーラインをつける

```css
.el_link {
  color: var(--sc_txt_link);
  text-decoration: underline;
}
```

### 擬似クラスの設定

次に、`:acrive`, `:visited`, `:hover` 状態の色について、用意しているセマンティックカラーを適用していきます。

```css
.el_link {
  color: var(--sc_txt_link);
  text-decoration: underline;
}

.el_link:active {
  color: var(--sc_txt_link__active);
}

.el_link:visited {
  color: var(--sc_txt_link__visited);
}

.el_link:hover {
  color: var(--sc_txt_link__hover);
}
```

## 完成形

```html
<a href="#" class="el_link">Sample</a>
```

```css:style.css
/* ≡≡≡ ☐ el_link ☐ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    リンク要素
---------------------------------------------------------------------------- */
.el_link {
  color: var(--sc_txt_link);
  text-decoration: underline;
}

.el_link:active {
  color: var(--sc_txt_link__active);
}

.el_link:visited {
  color: var(--sc_txt_link__visited);
}

.el_link:hover {
  color: var(--sc_txt_link__hover);
}
```

## まとめ

ここでは、リンク要素である `el_link` について扱いました。  
その中で、セマンティックカラーを扱い、`:acrive`, `:visited`, `:hover` の擬似クラスについても定義をしていきました。

次回はエレメントモジュールの最後となる、ボタン要素 `el_btn` を扱います。
