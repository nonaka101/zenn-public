---
title: "📄B104：色について(2)"
---

## このページについて

２回目となる本項では、前回の内容を踏まえ、実際の `style.css` を参照しながら説明します。

## 前回のおさらい

### カスタムプロパティについて

カスタムプロパティは、接頭辞 `--` を付けることで利用できるプロパティです。プロパティ名と何らかのCSS値のセットで構成され、`var()` 関数でアクセスすることができます。カスタムプロパティの利用は、下記のメリットがあります。

#### 複数のまとまりを一括管理できる

個別にRGB値を扱っていると管理が大変ですし、変更が生じた際に漏れなどといったミスに繋がりやすいです。

サイト全体でデザインを統一する場合、「テキスト色は これ」「こういった線の色は これ」「このボタンの背景色は これ」といったように、使用する色は要素に応じて必然と固まってくるはずです。それらをカスタムプロパティ化できれば、管理にスタイルシート全体を見回す必要はないですし、変更の際のミスを防止しやすいはずです。

#### 上書きと継承という特徴がある

カスタムプロパティは、ルート上に設定すると どこからでも `var()` を使い参照することができます。これはルート上で規定された内容が子要素へと継承されているからです。

一方で特定の状況や要素内に対し、その値を上書きすることもできます。今回の話では、この特徴を使って「ライト↔ダーク間で色を変更（上書き）する」ことを実現させます。

### プリミティブカラーとセマンティックカラー

デジタル庁デザインシステムに登場するこれらの用語について、下記のように話しました。

- プリミティブカラーは、カラーパレットのようなもの
- セマンティックカラーは、役割に応じた色を管理するためのシステム
- 両者を組み合わせて運用する

![プリミティブとセマンティックの関係図（３段階あり、１段階目に色のグラデーション、第二段階にプリミティブカラー、第三段階にセマンティックカラーとなる）](/images/books/web-zukuri/about-color-system-09.png)
*多数ある色から使用するものをプリミティブカラーという形で抽出、そこから各要素毎に使う色をセマンティックカラーという形で管理する*

## 本テーマでの色に関するスタイルの仕組み

本テーマの `style.css` を使い、どういったことを行っているかについて説明します。

### カスタムプロパティ化したプリミティブ/セマンティックカラー

下記のサンプルは、プリミティブカラーとセマンティックカラーをカスタムプロパティとして設定しています。

```css:サンプル：プリミティブカラーとセマンティックカラー
:root {
  /* プリミティブカラー
   * ここでは 白↔黒 間の13(+1)色を設定
   * 数値が大きいほど黒く、少ないほど白に近い
   */
  --sumi-1200: #000000;
  --sumi-1100: #080808;
  --sumi-1000: #111111;
  --sumi-900: #1a1a1c;
  --sumi-800: #414143;
  --sumi-700: #626264;
  --sumi-600: #757578;
  --sumi-500: #949497;
  --sumi-400: #b4b4b7;
  --sumi-300: #d8d8db;
  --sumi-200: #e8e8eb;
  --sumi-100: #f1f1f4;
  --sumi-50: #f8f8fb;
  --white: #ffffff;

  /* セマンティックカラー
   * ここでは 「テキスト」「境界線」「背景」を設定
   */
  --color_text: var(--sumi-900);
  --color_border: var(--sumi-900);
  --color_background: var(--white);
}
```

### カラーモードによる切り替え

下記のサンプルでは、`@madia (prefers-color-scheme: dark){...}` が追加されています。ここでカラーモードがダーク時のセマンティックカラーの値を上書きする設定を作っています。

こうすることで、最後にある `p`, `div`, `body` タグの色は、ブラウザのカラーモードに応じて変化するようになります。

```css:サンプル：カラーモードによる切り替え
:root {
  /* プリミティブカラー
   * ここでは 白↔黒 間の13(+1)色を設定
   * 数値が大きいほど黒く、少ないほど白に近い
   */
  --sumi-1200: #000000;
  --sumi-1100: #080808;
  --sumi-1000: #111111;
  --sumi-900: #1a1a1c;
  --sumi-800: #414143;
  --sumi-700: #626264;
  --sumi-600: #757578;
  --sumi-500: #949497;
  --sumi-400: #b4b4b7;
  --sumi-300: #d8d8db;
  --sumi-200: #e8e8eb;
  --sumi-100: #f1f1f4;
  --sumi-50: #f8f8fb;
  --white: #ffffff;

  /* セマンティックカラー
   * ここでは 「テキスト」「境界線」「背景」を設定
   */
  --color_text: var(--sumi-900);
  --color_border: var(--sumi-700);
  --color_background: var(--white);
}

/* ダークモード時は、色関係が白黒逆転する */
@media (prefers-color-scheme: dark) {
  :root {
    --color_text: var(--white);
    --color_border: var(--sumi-200);
    --color_background: var(--sumi-900);
  }
}

/* カラーモードによって、色が変化する */
p { color: var(--color_text); }
div { border-color: var(--color_border); }
body { background-color: var(--color_background); }
```

:::message

カラーモード問わず変化しないもの（例：ページトップボタン）については、直接プリミティブカラーや RGB 値を使って指定しているものがあります。

:::

## 本テーマでの `style.css`

### プリミティブカラー

:::message

現行のデザインシステムでは、この方法は最新のものではありません。詳細については、下記を参照してください。

[[バージョン1.4.0]スタイルやコンポーネントの追加・修正・更新を行いました｜デジタル庁](https://www.digital.go.jp/policies/servicedesign/designsystem/20231018#sec-01)

:::

:::details コード全文（長いので格納しています）

```css:style.css
:root {
  /* プリミティブカラー */
  --white: #ffffff;
  /* sea */
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
  /* sumi */
  --sumi-1200: #000000;
  --sumi-1100: #080808;
  --sumi-1000: #111111;
  --sumi-900: #1a1a1c;
  --sumi-800: #414143;
  --sumi-700: #626264;
  --sumi-600: #757578;
  --sumi-500: #949497;
  --sumi-400: #b4b4b7;
  --sumi-300: #d8d8db;
  --sumi-200: #e8e8eb;
  --sumi-100: #f1f1f4;
  --sumi-50: #f8f8fb;
  /* forest */
  --forest-1200: #032213;
  --forest-1100: #08351f;
  --forest-1000: #0c472a;
  --forest-900: #115a36;
  --forest-800: #197a4b;
  --forest-700: #1a8b56;
  --forest-600: #259d63;
  --forest-500: #2cac6e;
  --forest-400: #51b883;
  --forest-300: #71c598;
  --forest-200: #9bd4b5;
  --forest-100: #c2e5d1;
  --forest-50: #e6f5ec;
  /* wood */
  --wood-1200: #662e00;
  --wood-1100: #833b00;
  --wood-1000: #9c4600;
  --wood-900: #b65200;
  --wood-800: #c16800;
  --wood-700: #c87504;
  --wood-600: #cd820a;
  --wood-500: #d18d0f;
  --wood-400: #d69c2b;
  --wood-300: #dcac4d;
  --wood-200: #e5c47f;
  --wood-100: #efdbb1;
  --wood-50: #f8f1e0;
  /* sun */
  --sun-1200: #640101;
  --sun-1100: #890101;
  --sun-1000: #af0000;
  --sun-900: #d50000;
  --sun-800: #ec0000;
  --sun-700: #fa1606;
  --sun-600: #ff220d;
  --sun-500: #ff4b36;
  --sun-400: #ff5838;
  --sun-300: #ff7b5c;
  --sun-200: #ffa28b;
  --sun-100: #ffc8b8;
  --sun-50: #ffe7e6;
}
```

:::

### セマンティックカラー

本テーマにおけるセマンティックカラーの命名規則は、`--sc_{分類}_{部品/要素名}(__{モディファイア})` となります。具体的な内容については、下記のようになります。

第一に、接頭辞に `sc` (semantic color) を用います。

第二に、接頭辞の `sc` の後ろにアンダーバーを1つ挟み、 `--sc_{分類}` の形となります。例えば、テキストに関するものは、`--sc_txt` となります。  
分類の要素は下記の3種類です。

- txt : テキスト
- border : ボーダー
- bg : 背景(background)

第三に、分類の後ろにアンダーバーを1つ挟み、`--sc_{分類}_{部品/要素名}` の形となります。例えば、テキストのリンクに関するものは、`--sc_txt_link` となります。

なお、部品/要素名が複数の単語の場合は、ローワーキャメルケースで繋げます。例えば、テキストで背景色が塗られている要素に使う（on fill）の場合は、`--sc_txt_onFill` となります。

最後に、部品/要素名 の中でも更に複数の振る舞いがある場合には、アンダーバーを*2つ*挟み、`--sc_{分類}_{部品/要素名}__{モディファイア}` の形となります。

これらを踏まえ、本テーマで使うセマンティックカラーは下記のようになります。

:::details 各種説明（長いので格納しています）

テキスト関係

- `--sc_txt_body` : 基本
- `--sc_txt_body__small` : 基本（文字が小さい場合、コントラストを上げるためのもの）
- `--sc_txt_desc` : 説明文などに
- `--sc_txt_placeholder` : プレースホルダーに
- `--sc_txt_onFill` : 背景色が `--sc_bg_body__*` 以外で、塗られているもの
- `--sc_txt_alert` : 警告文に使用
- `--sc_txt_disabled` : 不活性状態のものに
- `--sc_txt_link` : aタグのリンク文に
- `--sc_txt_link__hover` : リンク文のホバー時
- `--sc_txt_link__active` : リンク文のアクティブ時
- `--sc_txt_link__visited` : リンク先に既に訪問済みの時
- `--sc_txt_icon` : アイコンのラベル
- `--sc_txt_btn__primary` : ボタンのラベル（プライマリ）
- `--sc_txt_btn__secondary` : ボタンのラベル（セカンダリ）
- `--sc_txt_btn__tertiary` : ボタンのラベル（ターシャリ）

ボーダー関係

- `--sc_border_field` : フィールド
- `--sc_border_divider` : 分割線
- `--sc_border_focused` : フォーカス時
- `--sc_border_selected` : 選択時
- `--sc_border_disabled` : 不活性時
- `--sc_border_alert` : 警告

背景関係

- `--sc_bg_body__primary` : 基本（プライマリ）
- `--sc_bg_body__secondary` : 基本（セカンダリ）
- `--sc_bg_body__tertiary` : 基本（ターシャリ）
- `--sc_bg_btn__primary` : ボタン背景（プライマリ）
- `--sc_bg_btn__secondary` : ボタン背景（セカンダリ）
- `--sc_bg_disabled` : 不活性時
- `--sc_bg_alert` : 警告
- `--sc_bg_shadow` : 影

:::

### カラーに関するコード全文

今後使用するカスタムプロパティに関するコード全文となります。  

:::details コード全文（長いので格納しています）

```css:style.css
:root {
  /* ≡≡≡ ▀▄ カラーシステム（DesignSystem 1.3.2 準拠）▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
    ■ 概要
      カラーモードを切り替えられるよう、カスタムプロパティでセマンティックカラーを再現している。
      原則として、クラスの色に関してはセマンティックカラーを使用すること。
      （※カラーモードに関係ない要素に関しては、プリミティブカラーからも可とする）
  -------------------------------------------------------------------------- */

  /* プリミティブカラー */
  --white: #ffffff;
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
  --sumi-1200: #000000;
  --sumi-1100: #080808;
  --sumi-1000: #111111;
  --sumi-900: #1a1a1c;
  --sumi-800: #414143;
  --sumi-700: #626264;
  --sumi-600: #757578;
  --sumi-500: #949497;
  --sumi-400: #b4b4b7;
  --sumi-300: #d8d8db;
  --sumi-200: #e8e8eb;
  --sumi-100: #f1f1f4;
  --sumi-50: #f8f8fb;
  --forest-1200: #032213;
  --forest-1100: #08351f;
  --forest-1000: #0c472a;
  --forest-900: #115a36;
  --forest-800: #197a4b;
  --forest-700: #1a8b56;
  --forest-600: #259d63;
  --forest-500: #2cac6e;
  --forest-400: #51b883;
  --forest-300: #71c598;
  --forest-200: #9bd4b5;
  --forest-100: #c2e5d1;
  --forest-50: #e6f5ec;
  --wood-1200: #662e00;
  --wood-1100: #833b00;
  --wood-1000: #9c4600;
  --wood-900: #b65200;
  --wood-800: #c16800;
  --wood-700: #c87504;
  --wood-600: #cd820a;
  --wood-500: #d18d0f;
  --wood-400: #d69c2b;
  --wood-300: #dcac4d;
  --wood-200: #e5c47f;
  --wood-100: #efdbb1;
  --wood-50: #f8f1e0;
  --sun-1200: #640101;
  --sun-1100: #890101;
  --sun-1000: #af0000;
  --sun-900: #d50000;
  --sun-800: #ec0000;
  --sun-700: #fa1606;
  --sun-600: #ff220d;
  --sun-500: #ff4b36;
  --sun-400: #ff5838;
  --sun-300: #ff7b5c;
  --sun-200: #ffa28b;
  --sun-100: #ffc8b8;
  --sun-50: #ffe7e6;

  /* セマンティックカラー */
  --sc_txt_body: var(--sumi-900);
  --sc_txt_body__small: var(--sumi-1200); /* 小さな文字用 */
  --sc_txt_desc: var(--sumi-700);
  --sc_txt_placeholder: var(--sumi-600);
  --sc_txt_onFill: var(--white);
  --sc_txt_link: var(--sea-800);
  --sc_txt_link__hover: var(--sea-900);
  --sc_txt_link__active: var(--sea-900);
  --sc_txt_link__visited: var(--sea-900);
  --sc_txt_alert: var(--sun-800);
  --sc_txt_disabled: var(--sumi-500);
  --sc_txt_icon: var(--sumi-900);
  --sc_txt_btn__primary: var(--white);
  --sc_txt_btn__secondary: var(--sea-800);
  --sc_txt_btn__tertiary: var(--sea-800);
  --sc_border_field: 1px solid var(--sumi-900);
  --sc_border_divider: 1px solid var(--sumi-300);
  --sc_border_focused: 2px solid var(--wood-600);
  --sc_border_selected: 2px solid var(--sea-800);
  --sc_border_disabled: 1px solid var(--sumi-300);
  --sc_border_alert: 2px solid var(--sun-800);
  --sc_bg_body__primary: var(--sumi-50);
  --sc_bg_body__secondary: var(--sumi-200);
  --sc_bg_body__tertiary: var(--sumi-100);
  --sc_bg_btn__primary: var(--sea-800);
  --sc_bg_btn__secondary: var(--sea-50);
  --sc_bg_disabled: var(--sumi-500);
  --sc_bg_alert: var(--sun-800);
  --sc_bg_shadow: var(--sumi-700);
}

/* セマンティックカラー（ダークモード時） */
@media (prefers-color-scheme: dark) {
  :root {
    --sc_txt_body: var(--white);
    --sc_txt_body__small: var(--white);
    --sc_txt_desc: var(--sumi-400);
    --sc_txt_placeholder: var(--sumi-500);
    --sc_txt_onFill: var(--white);
    --sc_txt_link: var(--sea-300);
    --sc_txt_link__hover: var(--sea-200);
    --sc_txt_link__active: var(--sea-200);
    --sc_txt_link__visited: var(--sea-200);
    --sc_txt_alert: var(--sun-500);
    --sc_txt_disabled: var(--sumi-600);
    --sc_txt_icon: var(--white);
    --sc_txt_btn__primary: var(--white);
    --sc_txt_btn__secondary: var(--sea-300);
    --sc_txt_btn__tertiary: var(--sea-300);
    --sc_border_field: 1px solid var(--white);
    --sc_border_divider: 1px solid var(--sumi-700);
    --sc_border_focused: 2px solid var(--wood-600);
    --sc_border_selected: 2px solid var(--sea-300);
    --sc_border_disabled: 1px solid var(--sumi-300);
    --sc_border_alert: 2px solid var(--sun-500);
    --sc_bg_body__primary: var(--sumi-900);
    --sc_bg_body__secondary: var(--sumi-700);
    --sc_bg_body__tertiary: var(--sumi-800);
    --sc_bg_btn__primary: var(--sea-300);
    --sc_bg_btn__secondary: var(--sea-1100);
    --sc_bg_disabled: var(--sumi-600);
    --sc_bg_alert: var(--sun-500);
    --sc_bg_shadow: var(--white);
  }
}
```

![プリミティブカラーの一覧](/images/books/web-zukuri/about-color-system-02.png)
*プリミティブカラーの一部*

![セマンティックカラーの一覧１、テキスト関係の色指定を、「カラー名」「ライト時」「ダーク時」の列を持つ表形式で表されている](/images/books/web-zukuri/semantic-color-01.png)
*テキスト関係のセマンティックカラー*

![セマンティックカラーの一覧２、主に枠線に関する色指定を表形式で表している](/images/books/web-zukuri/semantic-color-02.png)
*テキスト関係と、枠線に関するセマンティックカラー*

![セマンティックカラーの一覧３、主に背景に関する色指定を表形式で表している](/images/books/web-zukuri/semantic-color-03.png)
*背景に関するセマンティックカラー*

:::

## 色の設定

ここで用意したセマンティックカラーは、主に「基本コンポーネント」内で使うことになります。どのようなものになるかは、そちらでお話することになります。

ここではそれと別に、要素セレクタにセマンティックカラーをつけていきます。下記では、テキストノードが付きうる要素の `color` プロパティ、背景となる `body` 要素の `background-color` プロパティに、それぞれ設定しています。

これは、カラーモードとして背景、テキストの最低限の見栄えを確保しておくためです。

```css:style.css
/* カラーモードでの最低限の見栄えを確保 */
h1,h2,h3,h4,h5,h6,
li,
th,
td,
div,
span,
code,
pre,
figcaption,
caption,
time,
address,
small,
p {
  color: var(--sc_txt_body);
}

body {
  background-color: var(--sc_bg_body__primary);
}
```

## まとめ

色に関するスタイルについて、2回に分けて解説してきました。

第二回となる今回では、実際の `style.css` で使われているプリミティブ/セマンティック カラーを見ました。今後でてくるコンポーネントでは、色に関するものはほぼここのセマンティックカラーを使っていくので、必要に応じて見返したりしてください。

本項でスタイルに関する節は終了となります。

次項からは、テーマの特徴である「フォントサイズ」「カラーモード」を検証するための、開発ツールについて解説していきます。
