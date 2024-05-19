---
title: "📄B204：カラーモード変更(1)"
---

## このページについて

ここでは、カラーモード変更機能について、2回に分けて説明します。  
1回目は土台となる、大まかな機能の定義と、スタイルについて解説していきます。

## 機能の定義（全体）

カラーモード変更に付与したい機能は、次のようになります。

1. チェックボックスを用いて、機能の有効無効を切り替えられる
2. 機能を有効化すると、カラーモードを手動変更できるようになる
3. 機能を無効化した場合は、OS やブラウザの設定を優先する
4. 手動モード時になるとチェックボックスがもう1つ出現し、ライト↔ダーク間でカラーモードを切り替えられる
5. OSやブラウザの設定に関わらず、手動モード時は必ずそちらの設定を優先する

これをどう達成するかについて、まずは考えていきます。

### `style.css` でのカラーモード扱いについて

`style.css` でカラーモードに対応する箇所は、下記のようになっています。

```css:セマンティックカラーに関するカスタムプロパティ
:root {
  --sc_txt_body: var(--sumi-900);
  /* 以下、セマンティックカラーに関するカスタムプロパティが続く */
}

@media (prefers-color-scheme: dark) {
  :root {
      --sc_txt_body: var(--white);
      /* 以下、ダークモード時のカスタムプロパティ */
  }
}
```

```css:背景にインラインSVGを使用しているクラス
/* これに当てはまるクラスは、下記の2つ
 * 1. `bl_accordion_summary`
 * 2. `bl_searchForm_inputSelect`
 */

.bl_accordion_summary::after {
  /*
   * 関係ない部分は省略
   * ここでは背景イメージにSVGを使い、`fill="black"` をしていることに注目
   */
  background-image: url('data:image/svg+xml;utf-8,<svg viewBox="0 0 24 24" fill="black" xmlns="http://www.w3.org/2000/svg"><path d="M12 17.1L3 8L4 7L12 15L20 7L21 8L12 17.1Z"/></svg>');
}

/* ダークモード時に反転させることで、`fill="white"` と同等の結果となる */
@media (prefers-color-scheme: dark) {
  .bl_accordion_summary::after {
    filter: invert(1);
  }
}
```

上記の文からわかるように `prefers-color-scheme: dark` を使い、OSやブラウザの設定を基に色を切り替えていることがわかります。

### 手動モード時のスタイルについて

上記を踏まえたうえで、手動モード時のスタイルの挙動について考えていきます。

今回は Javascript を使い、`html` タグにクラス（`is_lightMode`, `is_darkMode`）を付与する手法を使います。これには2つの理由があります。

1. 手法で必要となる Javascript コードが簡単で扱いやすい
2. 既存のスタイルを上書きするために、クラスセレクタによる詳細度上げが利用できる

### 考えるべき場面について

今回の機能の場合、OSやブラウザの設定と、手動モードの設定を両方考える必要があります。
場面分けすると、下記の6パターンです。

|パターン|OSの設定|手動モード|結果|
|:----:|:----:|:----:|:----:|
|1|**ライト**|-|**ライト**|
|2|ライト|**ライト**|**ライト**|
|3|ライト|**ダーク**|**ダーク**|
|4|**ダーク**|-|**ダーク**|
|5|ダーク|**ライト**|**ライト**|
|6|ダーク|**ダーク**|**ダーク**|

ここで気をつけなければならないのは、パターン5です。
OS上ではダークモードなので、 `style.css` 上で `prefers-color-scheme: dark` のスタイルが適用された状態になっています。しかし結果としてはライトモードにしなければならないので、`prefers-color-scheme: dark` のスタイルを打ち消すような定義が必要となります。

### 上書きクラスのスタイル

では、実際のスタイルを見ていきます。
これらのスタイルは開発ツール用になるので、`devUtilities.css` 側に記述していきます。

#### セマンティックカラー

セマンティックカラーに関するスタイルに関しては、詳細度を上げるだけで済むので単純です。
`style.css` 側にある内容と同じものを、`is_lightMode`, `is_darkMode` クラスをつけて用意するだけで完了です。

```css:devUtilities.css
/* セマンティックカラー（ライトモード時） */
:root.is_lightMode {
  --sc_txt_body: var(--sumi-900);
  /* 以下省略 */
}

/* セマンティックカラー（ダークモード時） */
:root.is_darkMode {
  --sc_txt_body: var(--white);
  /* 以下省略 */
}
```

#### SVG背景を使用したクラス

`background-image` でSVGを使用しているクラスに関しては、`style.css` 側でどのようにスタイルが適用されているかを考える必要があります。
`style.css` 側では `filter: invert();` を使って色の調整を行っています。
ですので `devUtilities.css` 側ではそれを上書きするようなスタイルを定義していきます。

- OSがダークモードで `is_lightMode` の場合、`style.css` で適用されている `filter:invert(1);` を打ち消す
- OSがライトモードで `is_darkMode` の場合、`style.css` で適用されていない `filter:invert(1);` を付与

```css:devUtilities.css
/* style.css 側で適用される `filter: invert(1);` を上書きする */
@media (prefers-color-scheme: dark) {
  .is_lightMode .bl_accordion_summary::after {
    filter: invert(0);
  }
}

.is_darkMode .bl_accordion_summary::after {
  filter: invert(1);
}

@media (prefers-color-scheme: dark) {
  .is_lightMode .bl_searchForm_inputSelect::after {
    filter: invert(0);
  }
}

.is_darkMode .bl_searchForm_inputSelect::after {
  filter: invert(1);
}
```

結果として、前述したパターン表に当てはめると、下表のようになります。  
（※横幅の関係で表記を一部省略しています。ライト = "L"、ダーク = "D"）

||OS|style.css|手動|devUtilities.css|結果|style|
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
|1|**L**|-|-|-|**L**|-|
|2|L|-|**L**|-|**L**|-|
|3|L|-|**D**|**`invert(1)`**|**D**|**`invert(1)`**|
|4|**D**|**`invert(1)`**|-|-|**D**|**`invert(1)`**|
|5|D|`invert(1)`|**L**|**`invert(0)`**|**L**|**`invert(0)`**|
|6|D|`invert(1)`|**D**|**`invert(1)`**|**D**|**`invert(1)`**|

パターン5については、他と違い `invert(0)` で上書きしています。

## スタイル全文

最後に、ここまでで作られたスタイル文をまとめておきます

:::details 全文（長いので格納しています）

```css:devUtilities.css
/* ≡≡≡ ▀▄ カラーシステム（DesignSystem 1.3.2 準拠）▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
■ 概要
  カラーモードを切り替えられるよう、カスタムプロパティでセマンティックカラーを再現している
  ここでは Javascript によるカラーモード変更機能で付与するクラス `is_lightMode`, `is_darkMode` を使用する
--------------------------------------------------------------------------- */

/* セマンティックカラー（ライトモード時） */
:root.is_lightMode {
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
:root.is_darkMode {
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


/* style.css 側で適用される `filter: invert(1);` を上書きする */
@media (prefers-color-scheme: dark) {
  .is_lightMode .bl_accordion_summary::after {
    filter: invert(0);
  }
}

.is_darkMode .bl_accordion_summary::after {
  filter: invert(1);
}

@media (prefers-color-scheme: dark) {
  .is_lightMode .bl_searchForm_inputSelect::after {
    filter: invert(0);
  }
}

.is_darkMode .bl_searchForm_inputSelect::after {
  filter: invert(1);
}
```

:::

## まとめ

このページでは、カラーモード変更機能に関する基本部分について説明しました。  
1回目の今回は、どのような機能を作るかを決め、それを実現するためのスタイルについて決めました。

ここまでで、上書きするためのクラス `is_lightMode`, `is_darkMode` を用意できれば、
`devUtilities.css` に記述したスタイルに上書きされるようになりました。
これによりOS設定を反映しながらも、必要に応じて手動でカラーモードを切り替えることが可能になっています。

次回は、クラスを付与するための Javascript コードを中心に解説していきます。
