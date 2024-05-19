---
title: "📄B311：bl_searchForm"
---

## このページについて

ここでは、検索ページに使うフォーム `bl_searchForm` について説明します。  

![bl_searchForm通常時](/images/books/web-zukuri/bl_searchForm-01.png)

通常はこのように２段に分かれており、上段が「表示件数」を選択するセレクト要素、下段がテキスト入力欄とサブミットボタンという構成です。

画面幅が小さい場合は、テキスト入力欄の `∨` マークは邪魔になるので消えるようになっています。

![bl_searchFormのセレクト要素にあったマークが消えている](/images/books/web-zukuri/bl_searchForm-02.png)
*画面幅200px以下の状態*

また著しく画面幅が小さく構成が維持できない場合は、下図のように３段に分かれます。

![通常時は２段構成だったものが、入力欄とボタンが横並びで維持できなくなり３段構成へと変化](/images/books/web-zukuri/bl_searchForm-03.png)
*画面幅が極端に小さい状態*

## 構造の定義

まずは、構造の定義からしていきます。

状況によっては３段に分かれたりしていましたが、基本的な構成としては下記のような２段にします。

- フォーム
  - ボックス１
    - 表示件数の選択欄
  - ボックス２
    - 検索テキストの入力欄
    - 検索ボタン

またアクセシビリティの観点から、下記の要件も追加します。

- `form` 要素等の適切な HTML タグを使用する
  - 表示件数選択は `select`
  - 検索テキスト入力は `input type="search"`
  - 決定ボタンは `button type="submit"`
- `label` タグを使用し、各種フォーム要素と関連付けを行う
- 検索ボタンは `🔍` のインラインSVGを使う
  - ただしSVGはスクリーンリーダーが読み上げできないので別途ラベルを用意する

## 作成

上記で決めた要件から、マークアップしていきます。

### 基本的な要素の準備

まずは、クラスやスタイル等を考えず、HTMLタグを使ってフォームを作成してみます。必要となるフォーム関連の要素は、下記のようになります。

```html:HTMLによるフォームの構成
<form action="#">
  <select>
    <option><!-- ここに選択項目 --></option>
  <select>
  <input type="search">
  <button type="submit">検索</button>
</form>
```

更にここに、`label` による関連付けを行うと、最終的には下記のようになります。

```html:上記にlabel付与
<form action="#">
  <label for="num">表示件数</label>
  <select id="num">
    <option><!-- ここに選択項目 --></option>
  <select>
  <label for="txt">検索キーワード</label>
  <input type="search" id="txt">
  <button type="submit">検索</button>
</form>
```

ここから、スタイル等を考慮したものに編集していきます。

### ２段構造化

最初に見たように、検索フォームは「表示件数」と「テキスト入力＋ボタン」の２段構成が基本となっています。そこで、先程のマークアップを `div` を使って２段構成にします。

```html:クラス付け＋２段構成化
<form action="#" class="bl_searchForm_wrapper">
  <div class="bl_searchForm">
    <label for="num">表示件数</label>
    <select id="num">
      <option><!-- ここに選択項目 --></option>
    <select>
  </div>
  <!-- /.bl_searchForm -->
  <div class="bl_searchForm">
    <label for="txt">検索キーワード</label>
    <input type="search" id="txt">
    <button type="submit">検索</button>
  </div>
  <!-- /.bl_searchForm -->
</form>
```

`form` と２段構成用の `div` にクラス付けしたので、スタイルを規定していきます。

- `bl_searchForm_wrapper` について
  - ブロック要素
  - 上下に余白を取る
  - 格段のサイズ基準となるカスタムプロパティ `--searchForm_base` を用意
- `bl_searchForm` について
  - 上下に余白を取る

```css
.bl_searchForm_wrapper {
  display: block;
  margin-top: 1rem;
  margin-bottom: 1rem;

  --searchForm_base: 3rem;
}

.bl_searchForm {
  margin-top: 1rem;
  margin-bottom: 0.75rem;
}
```

`--searchForm_base` については、格段をフレックスボックス化した際のサイズ規定用です。１段の高さについてや、幅の最小値、折り返しのタイミングなどで使用します。

### １段目の設定

２段に分けたので、まずは１段目の表示件数の箇所から作成していきます。

#### `label` と `select` を２段化

図のように、`label` 要素を上に、`select` 要素を下に持ってきたいです。

![searchFormの１段目を切り出して映している、セレクト要素の上に「表示件数」というラベルが表示されている](/images/books/web-zukuri/bl_searchForm-04.png)

そのため、`div` を挟んで２段構成にします。

```html:１段目
<div class="bl_searchForm">
  <label for="num" class="bl_searchForm_label">表示件数</label>
  <div class="bl_searchForm_inputSelect">
    <select name="n" id="num">
      <option value="5" selected>5</option>
      <option value="10">10</option>
      <option value="20">20</option>
      <option value="50">50</option>
      <option value="100">100</option>
    </select>
  </div>
</div>
<!-- /.bl_searchForm -->
```

```css:label要素をブロック化して２段構成に
.bl_searchForm_label {
  display: block;
  margin-bottom: 0.1rem;
  font-size: 0.75rem;
  font-weight: 500;
  line-height: 1.5;
  color: var(--sc_txt_body__small);
}
```

#### `select` 要素のベース規定

次に `select` 要素のベースを作っていきます。ここでやるのは下記の内容です。

- ブラウザデフォルトの `select` スタイルは消しておく
- 幅は 100%
- 高さは `--searchForm_base` を使用
- テキスト開始位置は `1ch` 分右にずらす
- テキスト色、背景色はセマンティックカラーを使用
- 要素の形を角丸にし、ボーダーはセマンティックカラーを使用

```css
.bl_searchForm_inputSelect > select {
  width: 100%;
  height: var(--searchForm_base, 3rem);
  color: var(--sc_txt_body);
  text-indent: 1ch;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
}

/* デフォルトのスタイルを剥ぐ */
.bl_searchForm_inputSelect > select::-ms-expand {
  display: none;
}
```

上記では、既存のスタイルを剥ぐために `appearance` をベンダープレフィックスと合わせて使用し、それとは別に `-ms-expand` も使用しています。

:::message

`border-radius` の計算内容については、**B305:el_btn**で説明した内容と同じです

:::

あとは、`disabled` 状態になった場合の動作（ポインター反応を起こさないようにし、非活性状態がわかるようボーダーを変更）すれば、ベース部分は完成です。

```css
.bl_searchForm_inputSelect > select:disabled {
  pointer-events: none;
  border: var(--sc_border_disabled);
}
```

#### `∨` マークの作成

次に、ボックス右に表示させる `∨` マークを作っていきます。これは、`bl_searchForm_inputSelect` をフレックスボックス化し、擬似要素 `::after` で描画することで実現させます。

```html:bl_searchForm_inputSelect部
<div class="bl_searchForm_inputSelect">
  <select name="n" id="num">
    <option value="5" selected>5</option>
    <!-- 省略 -->
  </select>
</div>
```

```css
.bl_searchForm_inputSelect {
  position: relative;
  display: flex;
  flex-flow: row nowrap;
  cursor: pointer;
}
```

`position: relative;` に指定したことで、子要素に `position: absolute;` を使い位置を指定しやすくしています。

次に疑似要素 `::after` を作っていきます。要件は下記のとおりです。

- サイズは幅高さ共に `1rem`
- 親要素の高さが `3rem` なので、中心に来るよう縦位置は `top: 1rem` に
- 横位置は右から `1rem` 話した場所に
- `∨` マークは CSS の `background-image` でインラインSVGを使用

```css
.bl_searchForm_inputSelect::after {
  position: absolute;
  top: 1rem;
  right: 1rem;
  display: flexbox;
  width: 1rem;
  height: 1rem;
  content: "";
  background-image: url('data:image/svg+xml;utf-8,<svg viewBox="0 0 24 24" fill="black" xmlns="http://www.w3.org/2000/svg"><path d="M12 17.1L3 8L4 7L12 15L20 7L21 8L12 17.1Z"/></svg>');
}
```

用意したインラインSVGは、`fill="black"` となっているので、黒色の `∨` マークとなります。ダークモードにした場合このままでは見えなくなってしまうので、色を反転させる処理が必要です。

```css
@media (prefers-color-scheme: dark) {
  .bl_searchForm_inputSelect::after {
    filter: invert(1);
  }
}
```

また、横幅があまりに狭い場合、このマークが内容に被ってしまうのはよくありません。そうした場合は消えるようにしておきます。

![bl_searchFormのセレクト要素にあったマークが消えている](/images/books/web-zukuri/bl_searchForm-02.png)
*画面幅200px以下の状態*

```css
@media (max-width: 200px) {
  .bl_searchForm_inputSelect::after {
    display: none;
  }
}
```

これで、１段目に関するスタイル設定が完了しました。

:::message

*B102：スタイルルール* の項にある「モバイルファーストで設計」の例外的処理になります。 `@media(min-width: *){...}` で記述するのが基本ですが、ここではそれができなかったため、例外的に `max-width` による条件を使ってます。

:::

### ２段目の設定

次に２段目の「検索テキスト入力欄＋ボタン」を作成していきます。

#### `label` と `input`, `button` を２段化

１段目同様、２段目も `label` とそれ以外とで２段構成にします。なので同じように `div` を挟んでいます。

```html:２段目
<div class="bl_searchForm">
  <label for="txt" class="bl_searchForm_label">検索キーワード</label>
  <div class="bl_searchForm_input">
    <input type="search" id="txt">
    <button type="submit">検索</button>
  </div>
</div>
<!-- /.bl_searchForm -->
```

```css:１段目で用意したCSS文
.bl_searchForm_label {
  display: block;
  margin-bottom: 0.1rem;
  font-size: 0.75rem;
  font-weight: 500;
  line-height: 1.5;
  color: var(--sc_txt_body__small);
}
```

#### `bl_searchForm_input` の規定

次に、`input` と `button` を格納している `div` 要素 `bl_searchForm_input` を規定していきます。

- 高さの最小値は `3rem`、これは `--searchForm_base` から引っ張ってくる
  - （最小値なのは、２段になることがあるため）
- `input` と `button` は基本的には１行で並ぶ
- `button` の幅は `3rem`、これは `--searchForm_base` から引っ張ってくる
- `input` の幅は `button` が確保したサイズの残りを使用
- 画面幅が小さく これを維持できない場合は、折り返しを発生させて２段に分ける

ここでは `bl_searchForm_input` 内をフレックスボックス化することで、折り返しできるようしています。

![通常時は２段構成だったものが、入力欄とボタンが横並びで維持できなくなり３段構成へと変化](/images/books/web-zukuri/bl_searchForm-03.png)
*画面幅が極端に小さい状態*

```css
.bl_searchForm_input {
  display: flex;
  flex-flow: row wrap;
  gap: 4px;
  min-height: var(--searchForm_base, 3rem);
  margin: 0;
}
```

#### テキスト入力欄の規定

テキスト入力欄を作成していきます。クラスは `bl_searchForm_inputText` とします。

```html:bl_searchForm_input部
<div class="bl_searchForm_input">
  <input type="search" name="s" id="txt" class="bl_searchForm_inputText">
  <button type="submit">検索</button>
</div>
```

スタイル要件は下記のとおりです。

- サイズ関係
  - `flex-basis` を `3rem`（`--searchForm_base`）に指定
  - `flex-grow` に 999 を指定し、ボタンを除いた行の余白をテキスト入力欄側が埋める
  - 最小幅を `3rem`（`--searchForm_base`）とすることで、維持できなくなったら折り返しを発生させる
- その他
  - テキスト色と背景色はセマンティックカラーを使用
  - テキスト開始位置は `1ch` 分右にずらす
  - 形状を角丸にし、ボーダーはセマンティックカラーを使用
  - 非活性状態時は、ポインター反応を消し、ユーザーにわかるようボーダーを変化させる

```css
.bl_searchForm_inputText {
  flex: 999 0 var(--searchForm_base, 3rem);
  min-width: var(--searchForm_base, 3rem);
  height: var(--searchForm_base, 3rem);
  color: var(--sc_txt_body);
  text-indent: 1ch;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
}

.bl_searchForm_inputText:disabled {
  pointer-events: none;
  border: var(--sc_border_disabled);
}
```

#### ボタンの規定

次にボタン要素です。クラスは `bl_searchForm_btn` とします。

```html:bl_searchForm_input部
<div class="bl_searchForm_input">
  <input type="search" name="s" id="txt" class="bl_searchForm_inputText">
  <button type="submit" class="bl_searchForm_btn">
    <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" height="24" width="24">
      <path
        d="M21 20.5L15 14.5C16.2 13.2 16.9 11.4 16.9 9.5C17 5.4 13.6 2 9.5 2C5.4 2 2 5.4 2 9.5C2 13.6 5.3 17 9.5 17C11.2 17 12.7 16.4 14 15.5L20 21.5L21 20.5ZM3.5 9.5C3.5 6.2 6.2 3.5 9.5 3.5C12.8 3.5 15.5 6.2 15.5 9.5C15.5 12.8 12.8 15.5 9.5 15.5C6.2 15.5 3.5 12.8 3.5 9.5Z">
      </path>
    </svg>
    <span>検索</span>
  </button>
</div>
```

`svg` 要素は、サイズ `24px` の `🔍` マークです。`role` は `graphics-symbol`（適用できない場合は `img`）としていますが、この `button` はラベル付きアイコンで `<span>検索</span>` と読み上げる要素があるため、`aria-hidden="true"` でAOM上から消しています。スクリーンリーダーは「検索 ボタン」と読み上げてくれるのでアクセシビリティ上の問題はありません。

`button` 要素にかかるスタイルは、下記のとおりです。

- 高さは `3rem`（`--searchForm_base`）
- 幅となる `flex-basis` は `3rem`（`--searchForm_base`）
- 余白として、`padding: 0.1rem`
- 内部をフレックスボックス化
  - 中の要素は `🔍` の `svg` と、ラベルの `span`
  - 両者は列方向に中央揃えで配置
  - `gap` は `0.1rem`

```css
.bl_searchForm_btn {
  display: flex;
  flex: 1 1 var(--searchForm_base, 3rem);
  flex-direction: column;
  gap: 0.1rem;
  align-items: center;
  justify-content: center;
  height: var(--searchForm_base, 3rem);
  padding: 0.1rem;
  text-decoration: none;
  cursor: pointer;
  background-color: var(--sc_bg_btn__primary);
  border: 2px solid transparent;
  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
}
```

更に、状態に応じたスタイルを追加します。

- ホバー・アクション時に少し暗くする
- 非活性時はポインタの反応を消し、背景色を灰色（セマンティックカラー）に

```css
.bl_searchForm_btn:hover,
.bl_searchForm_btn:active {
  filter: brightness(87%);
}

.bl_searchForm_btn:disabled {
  pointer-events: none;
  background-color: var(--sc_bg_disabled);
}
```

最後に、`button` 内の各要素です。`button` 要素に割り当てられたサイズ `3rem` を超えないように規定していきます。

- `svg`: `🔍` マーク
  - サイズは `2rem`
  - テキスト色（この場合はSVG要素）はプライマリボタンのセマンティックカラーを使用
- `span`: ラベル「検索」
  - フォントサイズは `0.75rem`、`line-height` は `1.25`（合計およそ `0.94rem`）
  - テキスト色はプライマリボタンのセマンティックカラーを使用

```css
.bl_searchForm_btn > svg {
  width: 2rem;
  height: 2rem;
  color: var(--sc_txt_btn__primary);
}

.bl_searchForm_btn > span {
  font-size: 0.75rem;
  font-style: normal;
  font-weight: 700;
  line-height: 1.25;
  color: var(--sc_txt_btn__primary);
  letter-spacing: 0.04em;
}
```

これでボタン要素が完成、２段目の要素全てを規定することができました。

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html
<form action="#" class="bl_searchForm_wrapper">
  <div class="bl_searchForm">
    <label for="num" class="bl_searchForm_label">表示件数</label>
    <div class="bl_searchForm_inputSelect">
      <select name="n" id="num">
        <option value="5" selected>5</option>
        <option value="10">10</option>
        <option value="20">20</option>
        <option value="50">50</option>
        <option value="100">100</option>
      </select>
    </div>
  </div>
  <!-- /.bl_searchForm -->
  <div class="bl_searchForm">
    <label for="txt" class="bl_searchForm_label">検索キーワード</label>
    <div class="bl_searchForm_input">
      <input type="search" name="s" id="txt" class="bl_searchForm_inputText">
      <button type="submit" class="bl_searchForm_btn">
        <svg role="graphics-symbol img" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" height="24" width="24">
          <path
            d="M21 20.5L15 14.5C16.2 13.2 16.9 11.4 16.9 9.5C17 5.4 13.6 2 9.5 2C5.4 2 2 5.4 2 9.5C2 13.6 5.3 17 9.5 17C11.2 17 12.7 16.4 14 15.5L20 21.5L21 20.5ZM3.5 9.5C3.5 6.2 6.2 3.5 9.5 3.5C12.8 3.5 15.5 6.2 15.5 9.5C15.5 12.8 12.8 15.5 9.5 15.5C6.2 15.5 3.5 12.8 3.5 9.5Z">
          </path>
        </svg>
        <span>検索</span>
      </button>
    </div>
  </div>
  <!-- /.bl_searchForm -->
</form>
<!-- /.bl_searchForm_wrapper -->
```

```css:style.css
/* ≡≡≡ ❒ bl_searchForm ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    検索ページ用の専用フォーム
    （※デザインシステムの他のフォームと共通化できるスタイルが少ないため）
  ■ 構成
    bl_searchForm_wrapper : ラッパー
      bl_searchForm : ベース
        bl_searchForm_label : 各種フォームのラベル
        bl_searchForm_inputSelect : 表示件数
        bl_searchForm_input : 検索キーワードと検索ボタン
          bl_searchForm_inputText : テキスト入力
          bl_searchForm_btn : ボタン
---------------------------------------------------------------------------- */
.bl_searchForm_wrapper {
  display: block;
  margin-top: 1rem;
  margin-bottom: 1rem;

  --searchForm_base: 3rem;
}

.bl_searchForm {
  margin-top: 1rem;
  margin-bottom: 0.75rem;
}

.bl_searchForm_label {
  display: block;
  margin-bottom: 0.1rem;
  font-size: 0.75rem;
  font-weight: 500;
  line-height: 1.5;
  color: var(--sc_txt_body__small);
}

.bl_searchForm_inputSelect {
  position: relative;
  display: flex;
  flex-flow: row nowrap;
  cursor: pointer;
}

.bl_searchForm_inputSelect > select {
  width: 100%;
  height: var(--searchForm_base, 3rem);
  color: var(--sc_txt_body);
  text-indent: 1ch;
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
}

/* デフォルトのスタイルを剥ぐ */
.bl_searchForm_inputSelect > select::-ms-expand {
  display: none;
}

.bl_searchForm_inputSelect > select:disabled {
  pointer-events: none;
  border: var(--sc_border_disabled);
}

/*
 * after要素について
 * デフォルトのスタイルは剥いだうえで、背景としてSVGの矢印を使用
 * （アクセシビリティとしては無視される）
 * 画面幅がおおよそ 200px 以下になると、矢印がoption要素に衝突してしまうので、
 * 暫定的ではあるが `display:none;` にしている
 */
.bl_searchForm_inputSelect::after {
  position: absolute;
  top: 1rem;
  right: 1rem;
  display: flexbox;
  width: 1rem;
  height: 1rem;
  content: "";
  background-image: url('data:image/svg+xml;utf-8,<svg viewBox="0 0 24 24" fill="black" xmlns="http://www.w3.org/2000/svg"><path d="M12 17.1L3 8L4 7L12 15L20 7L21 8L12 17.1Z"/></svg>');
}

@media (prefers-color-scheme: dark) {
  .bl_searchForm_inputSelect::after {
    filter: invert(1);
  }
}

@media (max-width: 200px) {
  .bl_searchForm_inputSelect::after {
    display: none;
  }
}

.bl_searchForm_input {
  display: flex;
  flex-flow: row wrap;
  gap: 4px;
  min-height: var(--searchForm_base, 3rem);
  margin: 0;
}

.bl_searchForm_inputText {
  flex: 999 0 var(--searchForm_base, 3rem);
  min-width: var(--searchForm_base, 3rem);
  height: var(--searchForm_base, 3rem);
  color: var(--sc_txt_body);
  text-indent: 1ch;
  background-color: var(--sc_bg_body__primary);
  border: var(--sc_border_field);
  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
}

.bl_searchForm_inputText:disabled {
  pointer-events: none;
  border: var(--sc_border_disabled);
}

.bl_searchForm_btn {
  display: flex;
  flex: 1 1 var(--searchForm_base, 3rem);
  flex-direction: column;
  gap: 0.1rem;
  align-items: center;
  justify-content: center;
  height: var(--searchForm_base, 3rem);
  padding: 0.1rem;
  text-decoration: none;
  cursor: pointer;
  background-color: var(--sc_bg_btn__primary);
  border: 2px solid transparent;
  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
}

.bl_searchForm_btn:hover,
.bl_searchForm_btn:active {
  filter: brightness(87%);
}

.bl_searchForm_btn:disabled {
  pointer-events: none;
  background-color: var(--sc_bg_disabled);
}

.bl_searchForm_btn > svg {
  width: 2rem;
  height: 2rem;
  color: var(--sc_txt_btn__primary);
}

.bl_searchForm_btn > span {
  font-size: 0.75rem;
  font-style: normal;
  font-weight: 700;
  line-height: 1.25;
  color: var(--sc_txt_btn__primary);
  letter-spacing: 0.04em;
}
```

:::

## まとめ

本項では、検索フォームとなる `bl_searchForm` について解説しました。

次項では、ブロックモジュールの最後となる、カードコンポーネント `bl_card` について説明します。
