---
title: "📄B308：bl_pager"
---

## このページについて

ここでは、ページネーション（ページャー）について説明します。  
これは複数ページにまたがるような情報を扱う「アーカイブページ」「検索ページ」で使用され、ページ位置を切り替える役割を持っています。

![ページネーション要素の例。アーカイブページのフッター手前の位置に配置され、ページを切り替える役割を持っている](/images/books/web-zukuri/page-archive-02.png)
*例：[アーカイブページ](https://nonaka101.github.io/web-zukuri/archive.html)のページネーション要素*

## 構造の定義

ここでは、どのような形にするかを考えていきます。

### 基本構造

- `現在のページ数 / 全ページ数` を記した `div`要素を中央に配置（左右のページ数に関わらず、位置は固定）
- 左側には現在のページより前の情報を配置（ない場合は空白）
- 右側には現在のページより後ろの情報を配置（ない場合は空白）
- 左右の情報は、「最初へ/最後へ」「次へ/前へ」「隣接するページ数（最大3つ）」のボタンを配置
- 上記のボタンは、画面幅に応じて表示/非表示が切り替わる（モバイル画面時は「ページ数」ボタンは非表示）
- 上記のボタンは、総ページ数や現在ページ位置の状況に応じて出力が変化する
  - 例：現在のページ位置が `2/2` の場合、「前へ」のみ出力（「最初へ」「1」は出力しない）
  - 例：現在のページ位置が `5/5` の場合、「最初へ」「前へ」「2」「3」「4」を出力

![ページネーション要素の基本的な構造（総ページ数15に対し、現在位置は7の場合）](/images/books/web-zukuri/bl_pager-01.png)
*基本的な構造（デスクトップ画面）*

![ページ総数が少ない場合の構造（総ページ数2に対し、現在位置は2の場合）](/images/books/web-zukuri/bl_pager-05.png)
*現在のページ位置や、総ページ数に合わせて出力要素は調整される*

## 作成

上記で決めた要件から、マークアップしていきます。

### 外枠部

まずは外枠となる部分です。  
ここでは３つのボックス（`bl_pager_left`, `bl_pager_center`, `bl_pager_right`）を用意します。

```html
<div class="bl_pager">
  <div class="bl_pager_left">
    <!-- 「最初へ」「前へ」「ページ数」要素 -->
  </div>
  <!-- /.bl_pager_left -->
  <div class="bl_pager_center">
    <!-- `[現在位置]/[総ページ数]` 形式のテキスト -->
  </div>
  <!-- /.bl_pager_center -->
  <div class="bl_pager_right">
    <!-- 「ページ数」「次へ」「最後へ」要素 -->
  </div>
  <!-- /.bl_pager_right -->
</div>
<!-- /.bl_pager -->
```

次にスタイルの設定です。必要となる条件は、下記になります。

- `bl_pager` 要素の上下に余白を取る
- `bl_pager_center` を左右の中心位置に固定する
- 左右の `bl_pager_left`, `bl_pager_right` に各種ボタンが格納される
- ３つのボックス間に一定の間隔を開ける
- 上記の間隔において、スペースに余裕のあるデスクトップ画面時は間隔を広めに取る

`bl_pager_center` を左右中央に固定するために、３つのクラスを*中央揃えのフレックスボックス*とするのがポイントです。
上記を満たすようスタイルを設定すると、下記のようになります。

```css
.bl_pager {
  display: flex;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: center;
  margin-top: var(--space_5);
  margin-bottom: var(--space_5);
}

@media (min-width: 960px) {
  .bl_pager {
    gap: 8px;
  }
}

/* ここで flex-basis を 48px にしているのは、中身が空の場合の対策 */
.bl_pager_left {
  flex: 1 0 48px;
}

.bl_pager_center {
  flex: none;
}

.bl_pager_right {
  flex: 1 0 48px;
}
```

`bl_pager_left`, `bl_pager_right` において `flex-basis` の値を 48px にしているのは、
*どのような状況下でも中央揃えの位置が同じになるようにするため*です。

例えば「現在のページ位置が 1 で、総ページ数が 10」の場合、`bl_pager_left` の中には出力すべき要素がありません。
`flex-basis: 48px;` が無いと `bl_pager_left` の中身が空となった際、
`bl_pager_center` と `bl_pager_right` 内の要素で中央揃えが行われ、位置がずれることになります。

*中身が空でも最低限のスペースを確保しておく*ことで、どのような場合でも３つ全てを使った中央揃えを行い、位置を固定することができます。

![bl_pager のフレックスボックスを表示した状態、左側は空だが右と同じスペースを使うことで中央揃えを為している](/images/books/web-zukuri/bl_pager-07.png)

### 左右フレックスボックスの定義

次に `bl_pager_left`, `bl_pager_right` 内に格納される要素を、どう配置するかについてです。

- 各ボックス内には「次へ/前へ」「最初へ/最後へ」「各ページ数」を表したボタンが格納される
- 各種ボタンは横並びに、一定程度間隔を開けて配置
- 上記の間隔について、スペースに余裕があるデスクトップ画面時は間隔を少し広めに取る
- 各種ボタンの上下は、中央揃えで配置
- 各種ボタンの左右は、`bl_pager_center` 側に寄せて配置（`bl_pager_left` は右寄せ、`bl_pager_right` は左寄せ）

これらを満たすようスタイルを設定すると、下記のようになります。

```css
.bl_pager {
  display: flex;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: center;
  margin-top: var(--space_5);
  margin-bottom: var(--space_5);
}

@media (min-width: 960px) {
  .bl_pager {
    gap: 8px;
  }
}

.bl_pager_left {
  display: flex;
  flex: 1 0 48px;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: end;
}

@media (min-width: 960px) {
  .bl_pager_left {
    gap: 8px;
  }
}

.bl_pager_center {
  flex: none;
}

.bl_pager_right {
  display: flex;
  flex: 1 0 48px;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: start;
}

@media (min-width: 960px) {
  .bl_pager_right {
    gap: 8px;
  }
}
```

### 各種ボタン要素

次に、左右ボックス内に格納されるボタン要素についてです。基本的な形は、次のようになります。

```html
<a class="bl_pager_btn" href="#">
  <!-- 何かしらの要素 -->
</a>
<!-- /.bl_pager_btn -->
```

`a`タグ内には、ページ数ならそのまま数値が入り、それ以外ならインラインSVGが入ることになります。

```html:例：「最初へ」ボタン
<a class="bl_pager_btn" href="#">
  <svg role="graphics-symbol img" aria-hidden="true"
    xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" height="24" width="24"
  >
    <path fill-rule="evenodd"
      d="M11.854 3.646a.5.5 0 0 1 0 .708L8.207 8l3.647 3.646a.5.5 0 0 1-.708.708l-4-4a.5.5 0 0 1 0-.708l4-4a.5.5 0 0 1 .708 0zM4.5 1a.5.5 0 0 0-.5.5v13a.5.5 0 0 0 1 0v-13a.5.5 0 0 0-.5-.5z">
    </path>
  </svg>
</a>
<!-- /.bl_pager_btn -->
```

次に、スタイルの設定です。

- ボタンの高さと幅は 48px の丸形状
- ボタン内の要素は、上下左右の中央揃え
- 色はセマンティックカラーの「リンクテキスト」を使用
- `a`タグだが、デフォルトにあるアンダーラインは不要
- ホバー、アクティブ、既に訪問済みなど、状態がわかるよう各種セマンティックカラーを使用する

これらの条件を満たすように、スタイルを設定していきます。

```css
.bl_pager_btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  color: var(--sc_txt_link);
  text-decoration: none;
  border: var(--sc_border_divider);
  border-radius: 50%;
}

.bl_pager_btn:hover {
  color: var(--sc_txt_link__hover);
  background-color: var(--sc_bg_btn__secondary);
  border-color: var(--sc_border_selected);
}

.bl_pager_btn:active {
  color: var(--sc_txt_link__active);
}

.bl_pager_btn:visited {
  color: var(--sc_txt_link__visited);
}
```

### ボタン要素（拡張画面用）

次に、画面に応じたボタン表示/非表示についてです。

インラインSVGを用いた「最初へ/最後へ」「前へ/次へ」ボタンは、画面の如何に関わらず必ず表示されます。
一方で「各ページ数」のボタンは、スペースに余裕のあるデスクトップ画面時には表示されますが、モバイル画面時には表示されないようします。

![説明図、画面幅が狭い時には「最初へ」「前へ」「次へ」「最後へ」ボタンのみだが、画面幅が広くなるとそれに加えて近辺のページ数に移動できるボタンが拡張表示される](/images/books/web-zukuri/bl_pager-06.png)

これを実現するために、`bl_pager_btn` クラス用のモディファイア `bl_pager_btn__extended` を用意します。

```html:例：デスクトップ画面時にのみ表示されるボタン
<a class="bl_pager_btn bl_pager_btn__extended" href="#">
  4
</a>
<!-- /.bl_pager_btn bl_pager_btn__extended -->
```

このモディファイアを利用し、スタイルを設定していきます。

```css
/* ※ `.bl_pager_btn` に関するスタイルは省略 */

.bl_pager_btn.bl_pager_btn__extended {
  display: none;
}

@media (min-width: 960px) {
  .bl_pager_btn.bl_pager_btn__extended {
    display: flex;
  }
}
```

### 組み合わせ

これまでの `html`, `css` を組み合わせることで、下図のようなページネーション要素を作ることができます。

![ページネーション要素の基本的な構造（総ページ数15に対し、現在位置は7の場合）](/images/books/web-zukuri/bl_pager-01.png)

:::details HTML文（長いので格納しています）

```html
<div class="bl_pager">
  <div class="bl_pager_left">
    <a class="bl_pager_btn" href="#">
      <svg role="graphics-symbol img" aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" height="24" width="24"
      >
        <path fill-rule="evenodd"
          d="M11.854 3.646a.5.5 0 0 1 0 .708L8.207 8l3.647 3.646a.5.5 0 0 1-.708.708l-4-4a.5.5 0 0 1 0-.708l4-4a.5.5 0 0 1 .708 0zM4.5 1a.5.5 0 0 0-.5.5v13a.5.5 0 0 0 1 0v-13a.5.5 0 0 0-.5-.5z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
    <a class="bl_pager_btn" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        height="24" width="24">
        <path fill-rule="evenodd"
          d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">4</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">5</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">6</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
  </div>
  <!-- /.bl_pager_left -->
  <div class="bl_pager_center">7/15</div>
  <!-- /.bl_pager_center -->
  <div class="bl_pager_right">
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">8</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">9</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">10</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        height="24" width="24">
        <path fill-rule="evenodd"
          d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
    <a class="bl_pager_btn" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        height="24" width="24">
        <path fill-rule="evenodd"
          d="M4.146 3.646a.5.5 0 0 0 0 .708L7.793 8l-3.647 3.646a.5.5 0 0 0 .708.708l4-4a.5.5 0 0 0 0-.708l-4-4a.5.5 0 0 0-.708 0zM11.5 1a.5.5 0 0 1 .5.5v13a.5.5 0 0 1-1 0v-13a.5.5 0 0 1 .5-.5z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
  </div>
  <!-- /.bl_pager_right -->
</div>
<!-- /.bl_pager -->
```

:::

:::details CSS文（長いので格納しています）

```css
.bl_pager {
  display: flex;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: center;
  margin-top: var(--space_5);
  margin-bottom: var(--space_5);
}

@media (min-width: 960px) {
  .bl_pager {
    gap: 8px;
  }
}

.bl_pager_left {
  display: flex;
  flex: 1 0 48px;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: end;
}

@media (min-width: 960px) {
  .bl_pager_left {
    gap: 8px;
  }
}

.bl_pager_center {
  flex: none;
}

.bl_pager_right {
  display: flex;
  flex: 1 0 48px;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: start;
}

@media (min-width: 960px) {
  .bl_pager_right {
    gap: 8px;
  }
}

.bl_pager_btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  color: var(--sc_txt_link);
  text-decoration: none;
  border: var(--sc_border_divider);
  border-radius: 50%;
}

.bl_pager_btn:hover {
  color: var(--sc_txt_link__hover);
  background-color: var(--sc_bg_btn__secondary);
  border-color: var(--sc_border_selected);
}

.bl_pager_btn:active {
  color: var(--sc_txt_link__active);
}

.bl_pager_btn:visited {
  color: var(--sc_txt_link__visited);
}

.bl_pager_btn.bl_pager_btn__extended {
  display: none;
}

@media (min-width: 960px) {
  .bl_pager_btn.bl_pager_btn__extended {
    display: flex;
  }
}
```

:::

## アクセシビリティへの対応

ここからは、アクセシビリティを踏まえた改良を加えていきます。

### 問題点

必要な改良は、下記の２点です

#### `nav` タグを使用していない

ナビゲーション要素として使用できるページネーションですが、現在は `div` タグを使っているのでマシンリーダブルではありません。

```html
<div class="bl_pager">
  <!-- ページネーション要素 -->
</div>
<!-- /.bl_pager -->
```

#### 必要なARIAを使用していない(`aria-label`, `aria-current`)

「最初へ/最後へ」「前へ/次へ」を示すSVGは、`aria-hidden` を使用しAOM上では無視されるようしています。
そのため、スクリーンリーダーはこれらのボタンが何であるのか読み上げることはしません。

そして `bl_pager_center` にある `7/15` の情報ですが、視覚からはこれが何であるか予測することができても、
マシンリーダブルでないのでスクリーンリーダーは認識することができません。

```html
<div class="bl_pager">
  <div class="bl_pager_left">
    <a class="bl_pager_btn" href="#">
      <!-- SVG省略 -->
    </a>
    <a class="bl_pager_btn" href="#">
      <!-- SVG省略 -->
    </a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">4</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">5</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">6</a>
  </div>
  <div class="bl_pager_center">7/15</div>
  <div class="bl_pager_right">
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">8</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">9</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">10</a>
    <a class="bl_pager_btn" href="#">
      <!-- SVG省略 -->
    </a>
    <a class="bl_pager_btn" href="#">
      <!-- SVG省略 -->
    </a>
  </div>
</div>
```

### 改良

#### `nav`タグの使用

外側にある `bl_pager` にナビゲーションであることを指定します。
この際、後述するように `aria-label` を使ってこれが何であるかをラベル付けしておくと良いでしょう。

```html
<nav class="bl_pager" aria-label="ページ割り">
  <!-- ページネーション要素 -->
</nav>
<!-- /.bl_pager -->
```

#### ARIAの使用

不足している箇所に ARIA を用いて、情報が伝わるようにしましょう。

- `nav` タグに `aria-label` を用い、どんなナビゲーション要素であるかを伝える
- SVG要素の親に `aria-label` を用い、ボタンの意味を伝える
- `bl_pager_center` に `aria-current="page"` を用い、「7/15」が現在のページ位置を表していることを伝える

```html
<nav class="bl_pager" aria-label="ページ割り">
  <div class="bl_pager_left">
    <a class="bl_pager_btn" aria-label="最初のページ" href="#">
      <!-- SVG省略 -->
    </a>
    <a class="bl_pager_btn" aria-label="前のページ" href="#">
      <!-- SVG省略 -->
    </a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">4</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">5</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">6</a>
  </div>
  <div class="bl_pager_center" aria-current="page">7/15</div>
  <div class="bl_pager_right">
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">8</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">9</a>
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">10</a>
    <a class="bl_pager_btn" aria-label="次のページ" href="#">
      <!-- SVG省略 -->
    </a>
    <a class="bl_pager_btn" aria-label="最後のページ" href="#">
      <!-- SVG省略 -->
    </a>
  </div>
</nav>
```

## 完成形

![ページネーション要素の基本的な構造（総ページ数15に対し、現在位置は7の場合）](/images/books/web-zukuri/bl_pager-01.png)

:::details HTML（長いので格納しています）

```html
<nav class="bl_pager" aria-label="ページ割り">
  <div class="bl_pager_left">
    <a class="bl_pager_btn" aria-label="最初のページ" href="#">
      <svg role="graphics-symbol img"
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
        viewBox="0 0 16 16"
        height="24" width="24"
      >
        <path fill-rule="evenodd"
          d="M11.854 3.646a.5.5 0 0 1 0 .708L8.207 8l3.647 3.646a.5.5 0 0 1-.708.708l-4-4a.5.5 0 0 1 0-.708l4-4a.5.5 0 0 1 .708 0zM4.5 1a.5.5 0 0 0-.5.5v13a.5.5 0 0 0 1 0v-13a.5.5 0 0 0-.5-.5z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
    <a class="bl_pager_btn" aria-label="前のページ" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        height="24" width="24">
        <path fill-rule="evenodd"
          d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">4</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">5</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">6</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
  </div>
  <!-- /.bl_pager_left -->
  <div class="bl_pager_center" aria-current="page">7/15</div>
  <!-- /.bl_pager_center -->
  <div class="bl_pager_right">
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">8</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">9</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">10</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn" aria-label="次のページ" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        height="24" width="24">
        <path fill-rule="evenodd"
          d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
    <a class="bl_pager_btn" aria-label="最後のページ" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        height="24" width="24">
        <path fill-rule="evenodd"
          d="M4.146 3.646a.5.5 0 0 0 0 .708L7.793 8l-3.647 3.646a.5.5 0 0 0 .708.708l4-4a.5.5 0 0 0 0-.708l-4-4a.5.5 0 0 0-.708 0zM11.5 1a.5.5 0 0 1 .5.5v13a.5.5 0 0 1-1 0v-13a.5.5 0 0 1 .5-.5z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
  </div>
  <!-- /.bl_pager_right -->
</nav>
<!-- /.bl_pager -->
```

:::

![ページ総数が少ない場合の構造（総ページ数2に対し、現在位置は1の場合）](/images/books/web-zukuri/bl_pager-03.png)

:::details HTML（長いので格納しています）

```html:前無しVer
<nav class="bl_pager" aria-label="ページ割り">
  <div class="bl_pager_left"></div>
  <!-- /.bl_pager_left -->
  <div class="bl_pager_center" aria-current="page">1/2</div>
  <!-- /.bl_pager_center -->
  <div class="bl_pager_right">
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">2</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
    <a class="bl_pager_btn" aria-label="次のページ" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16" height="24"
        width="24">
        <path fill-rule="evenodd"
          d="M4.646 1.646a.5.5 0 0 1 .708 0l6 6a.5.5 0 0 1 0 .708l-6 6a.5.5 0 0 1-.708-.708L10.293 8 4.646 2.354a.5.5 0 0 1 0-.708z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
  </div>
  <!-- /.bl_pager_right -->
</nav>
<!-- /.bl_pager -->
```

:::

![ページ総数が少ない場合の構造（総ページ数2に対し、現在位置は2の場合）](/images/books/web-zukuri/bl_pager-05.png)

:::details HTML（長いので格納しています）

```html:後ろ無しVer
<nav class="bl_pager" aria-label="ページ割り">
  <div class="bl_pager_left">
    <a class="bl_pager_btn" aria-label="前のページ" href="#">
      <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 16 16"
        height="24" width="24">
        <path fill-rule="evenodd"
          d="M11.354 1.646a.5.5 0 0 1 0 .708L5.707 8l5.647 5.646a.5.5 0 0 1-.708.708l-6-6a.5.5 0 0 1 0-.708l6-6a.5.5 0 0 1 .708 0z">
        </path>
      </svg>
    </a>
    <!-- /.bl_pager_btn -->
    <a class="bl_pager_btn bl_pager_btn__extended" href="#">1</a>
    <!-- /.bl_pager_btn bl_pager_btn__extended -->
  </div>
  <!-- /.bl_pager_left -->
  <div class="bl_pager_center" aria-current="page">2/2</div>
  <!-- /.bl_pager_center -->
  <div class="bl_pager_right"></div>
  <!-- /.bl_pager_right -->
</nav>
<!-- /.bl_pager -->
```

:::

:::details CSS（長いので格納しています）

```css:style.css
/* ≡≡≡ ❒ bl_pager ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    ページネーション要素
    アーカイブ、検索ページで記事数が1ページに入り切らない際に使用される
  ■構成
    bl_pager : ベース
      bl_pager_center : 現在位置
      bl_pager_left : 左側（前）
      bl_pager_right : 右側（後）
        bl_pager_btn : 「最初（最後）」「前（次）」「各ページ数」用のボタン
        -> bl_pager_btn__extended : モバイル画面時、非表示となる要素（＝「各ページ数」）
---------------------------------------------------------------------------- */
.bl_pager {
  display: flex;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: center;
  margin-top: var(--space_5);
  margin-bottom: var(--space_5);
}

@media (min-width: 960px) {
  .bl_pager {
    gap: 8px;
  }
}

/*
 * flex-basis に適当な数値(48px)を入れているのは中央揃えのため
 *（auto だと、要素無しの場合 左右のズレが生じる）
 */
.bl_pager_left {
  display: flex;
  flex: 1 0 48px;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: end;
}

@media (min-width: 960px) {
  .bl_pager_left {
    gap: 8px;
  }
}

.bl_pager_center {
  flex: none;
}

.bl_pager_right {
  display: flex;
  flex: 1 0 48px;
  flex-flow: row nowrap;
  gap: 4px;
  align-items: center;
  justify-content: start;
}

@media (min-width: 960px) {
  .bl_pager_right {
    gap: 8px;
  }
}

.bl_pager_btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 48px;
  height: 48px;
  color: var(--sc_txt_link);
  text-decoration: none;
  border: var(--sc_border_divider);
  border-radius: 50%;
}

.bl_pager_btn:hover {
  color: var(--sc_txt_link__hover);
  background-color: var(--sc_bg_btn__secondary);
  border-color: var(--sc_border_selected);
}

.bl_pager_btn:active {
  color: var(--sc_txt_link__active);
}

.bl_pager_btn:visited {
  color: var(--sc_txt_link__visited);
}

.bl_pager_btn.bl_pager_btn__extended {
  display: none;
}

@media (min-width: 960px) {
  .bl_pager_btn.bl_pager_btn__extended {
    display: flex;
  }
}
```

:::

## まとめ

本項ではページネーション要素について扱いました。

次項は、ブロックスキップについて説明します。
