---
title: "📄B203：ツールの設計"
---

## このページについて

ここでは、前項で説明した開発ツールを画面内に表示できるようにしていきます。

### 前項のおさらい

前項で機能に対して必要なコントロールについて解説しました。下記の内容を基に、要素を作っていきます。

- 画面の右端に固定配置
- 「ページトップに戻る」ボタンに被らないように
- 通常時は展開ボタンのみ見えるようにし、ボタンが押された時にツールボックスを表示
- カラーモード切り替えに必要なコントロールは次の通り
  1. 機能有効化のためのトグルボタン
  2. ライト↔ダーク 切り替えのトグルボタン
- フォントサイズ変更に必要なコントロールは次の通り
  1. 機能有効化のためのトグルボタン
  2. フォントサイズを変更するためのスライダー
  3. ユーザーに値を見せるための output

## ツールボックスの作成

ここでは、大枠となるツールボックスを作成していきます。

### html文の作成

下記のような構造にしました。

- ツールボックス全体
  - 展開/格納トグルボタン
  - ツールボックスの中身

```html:ツールボックス（大枠）
<div class="bl_devToolBox" id="js_toolBox">
  <div class="bl_devToolBox_btn">
    <input type="checkbox" name="is-show" id="js_toolBox_chkbox">
  </div>
  <!-- /.bl_devToolBox_btn -->
  <div class="bl_devToolBox_utils">
    <!-- ここにコントロールを配置していく -->
  </div>
  <!-- /.bl_debToolBox_utils -->
</div>
<!-- /.bl_devToolBox -->
```

CSS で `position: fixed;` にするので、`html` 内での場所は（ある程度）どこでも構いません。ですがWordPress化した際の展開を踏まえて `</body>` の前に配置するようにします。

- `head`
- `body`
  - `header`
  - `main`
  - `footer`
  - 「ページトップに戻る」ボタン
  - **ツールボックス**
  - 各種 `script`

上表は一般的なページレイアウト構造になります。`main` の内容はページ種類により変化しますが、`footer` 以降はどのページでも同じものを表示するようになります。なので、共通する部分をまとめて テンプレートパーツ `footer.php` で管理するために、`footer` タグの後ろに配置するよう統一しておきます。

### スタイルの定義

次にスタイルを定義していきます。下記の要素を満たすようにしていきます

- 右下に固定配置
- 他要素に隠れないよう、`z-index` を設定
- フォント含む全てのサイズを`px`で固定（フォントサイズ変更でレイアウトが崩れないように）
- フォント含む全ての色関係は固定（開発ツールにカラーモード変更による反映は不要）
- 構造については下記のようにする
  - 「展開ボタン」「ツールボックスの中身」を行方向に配置
  - 「ツールボックスの中身」の幅は 200px
  - `right: -200px;` にし、「ツールボックスの中身」側を画面外に配置
  - 「展開ボタン」の状態を JavaScript で拾い、`right` 設定を書き換えることで、表示/非表示を切り替える

```css:devUtilities.css
/* ≡≡≡ ❒ bl_devToolBox ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    開発に使うツール
  ■ 構成
    bl_devToolBox : ベース
      bl_devToolBox_btn : ツールボックスを開閉するためのチェックボックス
      bl_devToolBox_utils : ツール集
---------------------------------------------------------------------------- */
.bl_devToolBox {
  --tb_text: #000000;
  --tb_bg: #f8f8fb;
  --tb_accent: #0031d8;

  position: fixed;
  right: -200px;
  bottom: 110px;
  z-index: 999;
  display: flex;
  flex-flow: row nowrap;
  align-items: flex-end;
}

.bl_devToolBox * {
  font-size: 16px;
  color: var(--tb_text);
}

.bl_devToolBox_btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background-color: var(--tb_bg);
  border-top: 2px solid var(--tb_accent);
  border-bottom: 2px solid var(--tb_accent);
  border-left: 2px solid var(--tb_accent);
}

.bl_devToolBox_btn > input {
  width: 24px;
  height: 24px;
}

.bl_devToolBox_utils {
  display: block;
  width: 200px;
  padding: 4px;
  background-color: var(--tb_bg);
  border: 2px solid var(--tb_accent);
}
```

### JavaScriptによる展開ボタンの管理

最後に JavaScript を用いて、ツールボックスの展開/格納できるようにしていきます。

ツールボックス本体は `id="js_toolBox"`、展開ボタンは `id="js_toolBox_chkbox"` にしています。これらの要素をスクリプト上で取得し、展開ボタン（`input type="chkbox"`）の `checked` を条件式に用いてスタイルを上書きすることで、展開/格納を実現します。

```js:devUtilities.js
/* ≡≡≡ ▀▄ ShowToolBox ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    開発用機能をまとめたツールボックス要素を、画面右下に固定する
    必要になったときには画面内に出てくるようにし、それ以外は邪魔にならないよう隠しておく
---------------------------------------------------------------------------- */

// 各種要素の取得
const toolBoxChkbox = document.getElementById('js_toolBox_chkbox');
const toolBox = document.getElementById('js_toolBox');

// チェック時には要素を表示範囲内に移動、そうでなければ外側に隠す
toolBoxChkbox.addEventListener('change', (e) => {
  if(e.target.checked == true){
    toolBox.style.right = '-10px';
  } else {
    toolBox.style.right = '-200px';
  }
})
```

## 各種機能用のコントロール作成

ここからは、用意したボックス内に必要となるコントロールを作成していきます。

### フォントサイズ切り替え

基本的な構造は、下記のとおりです。

- フォントサイズコントロール群
  - ラベル＋スライダー
  - アウトプット（値表示用）
- 区切り線
- トグルボタン（機能有効化）
  - チェックボックス＋ラベル

このように、「フォントサイズのコントロール群」と「トグルボタン（機能有効化）」を分けています。その上でトグルボタンの状態に応じて、JavaScript で `hidden` 属性をコントロール群の格納要素に付与します。こうすることで、機能有効化の際にコントロールを表示させるようにしています。

```html:bl_devToolBox内
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
  <!-- ここに カラーモード関係の要素が入る -->
</div>
<!-- /.bl_debToolBox_utils -->
```

`id` を使っている下記要素は、JavaScript に渡す要素となっています。

- `js_fontSize_controller`: コントロール群格納要素、表示/非表示を管理
- `js_fontSize_range`: スライダー、フォントサイズ値を管理
- `js_fontSize_result`: アウトプット、フォントサイズ値の表示
- `js_fontSize_chkbox`: トグルボタン、機能有効化を管理

### トグルボタンのスタイル

ここでは、前述したトグルボタンのスタイルを規定していきます。

![トグルボタン](/images/books/web-zukuri/bl_devToolBox-02.png)

トグルボタンの `html` 文は、下記を基本としています。

```html:トグルボタンの構造
<div class="bl_toggleArea">
  <input
    type="checkbox"
    name="*"
    id="js_*_chkbox"
    class="bl_toggleArea_chkbox"
  >
  <label for="js_*_chkbox" class="bl_toggleArea_label">
    <!-- ラベル名 -->
  </label>
</div>
```

`id` は JavaScript で扱うものなので、スタイルに関係するのは `class` の方です。なので、CSSで規定するクラスは下記になります。

- `bl_toggleArea`: ベースとなる `div` 要素
  - `bl_toggleArea_chkbox`: チェックボックス用のスタイル、トグルボタンに変える
  - `bl_toggleArea_label`: ラベル用スタイル

チェックボックスをトグルボタンに変えるために、プロパティ `appearance` が必要となります。ここでは、プレフィックス `-webkit-appearance` と `-moz-appearance` のサポート状況を調べ、サポートされている場合にスタイルを規定していきます。  
（※サポートされていない場合はスタイル規定を行わず、ブラウザデフォルトのスタイルを使います）

```css:devUtilities.css
/* ≡≡≡ ❒ bl_toggleArea ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    トグルボタンとラベルを持つ、トグルの中身はチェックボックス
  ■ 構成
    bl_toggleArea : ベース
      bl_toggleArea_chkbox : トグルボタン
      bl_toggleArea_label : ラベル
---------------------------------------------------------------------------- */
@supports(-webkit-appearance: none) or (-moz-appearance: none) {
  .bl_toggleArea {
    display: flex;
    flex-flow: row nowrap;
    gap: 8px;
    padding: 0;
    margin: 0;
  }

  .bl_toggleArea_chkbox{
    position: relative;
    flex: none;
    width: 50px;
    height: 26px;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    background-color: #f1f1f4;
    border: 1px solid #949497;
    border-radius: 99px;
  }

  .bl_toggleArea_chkbox::after {
    position: absolute;
    top: 0;
    right: auto;
    left: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 24px;
    height: 24px;
    content: "";
    background-color: #ffffff;
    border: 1px solid #414143;
    border-radius: 50%;
    transition: all .3s ease;
  }

  /* 
   * ここのカスタムプロパティは、`.bl_devToolBox` 上で 
   * `--tb_accent: #0031d8;` と規定している。
   */
  .bl_toggleArea_chkbox:checked {
    background-color: var(--tb_accent);
  }

  .bl_toggleArea_chkbox:checked::after {
    top: 0;
    right: 0;
    left: auto;
  }

  .bl_toggleArea_label {
    flex: none;
  }
}
```

### カラーモード切り替え

基本的な構造は、フォントサイズ変更とほぼ同じです。

- トグルボタン（カラーモード切り替え）
  - チェックボックス＋ラベル
- 区切り線
- トグルボタン（機能有効化）
  - チェックボックス＋ラベル

フォントサイズ変更と同様に、「トグルボタン（カラーモード切り替え）」と「トグルボタン（機能有効化）」を分けています。機能有効化の状況に応じて表示/非表示を切り替えるためです。

```html:bl_devToolBox内
<div class="bl_devToolBox_utils">
  <!-- ここに フォントサイズ関係の要素が入る -->
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
```

`id` を使っている下記要素は、JavaScript に渡す要素となっています。

- `js_colorMode_controller`: コントロール群格納要素、表示/非表示を管理
- `js_darkMode`: トグルボタン、カラーモード「ライト↔ダーク」の切り替えを管理
- `js_colorMode_chkbox`: トグルボタン、機能有効化を管理

一点だけフォントサイズ変更と違うのは、`hidden` 属性に関する部分です。

カラーモード側ではコントロール群に `bl_toggleArea` を使用しています。この中で、`display: flex;` を使っているため、`hidden` 属性を加えても非表示にはなりません。これは `hidden` 属性で非表示になるのはプロパティ `display` を規定していない場合であるためです  
（※規定があると`hidden` 属性の `display: none;` の指定が上書きされてしまう）

そのため、`hidden` 属性を付与する要素に対し、専用のスタイル規定が必要になります。

```css:devUtilities.css
#js_colorMode_controller[hidden] {
  display: none;
}
```

## 作成した内容全文

:::details 長いので格納してあります

```html:ツールボックス
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

```css:devUtilities.css
/* ≡≡≡ ❒ bl_devToolBox ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    開発に使うツール
  ■ 構成
    bl_devToolBox : ベース
      bl_devToolBox_btn : ツールボックスを開閉するためのチェックボックス
      bl_devToolBox_utils : ツール集
---------------------------------------------------------------------------- */
.bl_devToolBox {
  --tb_text: #000000;
  --tb_bg: #f8f8fb;
  --tb_accent: #0031d8;

  position: fixed;
  right: -200px;
  bottom: 110px;
  z-index: 999;
  display: flex;
  flex-flow: row nowrap;
  align-items: flex-end;
}

.bl_devToolBox * {
  font-size: 16px;
  color: var(--tb_text);
}

.bl_devToolBox_btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background-color: var(--tb_bg);
  border-top: 2px solid var(--tb_accent);
  border-bottom: 2px solid var(--tb_accent);
  border-left: 2px solid var(--tb_accent);
}

.bl_devToolBox_btn > input {
  width: 24px;
  height: 24px;
}

.bl_devToolBox_utils {
  display: block;
  width: 200px;
  padding: 4px;
  background-color: var(--tb_bg);
  border: 2px solid var(--tb_accent);
}





/* ≡≡≡ ❒ bl_toggleArea ❒ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    トグルボタンとラベルを持つ、トグルの中身はチェックボックス
  ■ 構成
    bl_toggleArea : ベース
      bl_toggleArea_chkbox : トグルボタン
      bl_toggleArea_label : ラベル
---------------------------------------------------------------------------- */
@supports(-webkit-appearance: none) or (-moz-appearance: none) {
  .bl_toggleArea {
    display: flex;
    flex-flow: row nowrap;
    gap: 8px;
    padding: 0;
    margin: 0;
  }

  .bl_toggleArea_chkbox{
    position: relative;
    flex: none;
    width: 50px;
    height: 26px;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    background-color: #f1f1f4;
    border: 1px solid #949497;
    border-radius: 99px;
  }

  .bl_toggleArea_chkbox::after {
    position: absolute;
    top: 0;
    right: auto;
    left: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 24px;
    height: 24px;
    content: "";
    background-color: #ffffff;
    border: 1px solid #414143;
    border-radius: 50%;
    transition: all .3s ease;
  }

  .bl_toggleArea_chkbox:checked {
    background-color: var(--tb_accent);
  }

  .bl_toggleArea_chkbox:checked::after {
    top: 0;
    right: 0;
    left: auto;
  }

  .bl_toggleArea_label {
    flex: none;
  }
}

#js_colorMode_controller[hidden] {
  display: none;
}
```

```js:devUtilities.js
/* ≡≡≡ ▀▄ ShowToolBox ▀▄ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    開発用機能をまとめたツールボックス要素を、画面右下に固定する
    必要になったときには画面内に出てくるようにし、それ以外は邪魔にならないよう隠しておく
---------------------------------------------------------------------------- */

// 各種要素の取得
const toolBoxChkbox = document.getElementById('js_toolBox_chkbox');
const toolBox = document.getElementById('js_toolBox');

// チェック時には要素を表示範囲内に移動、そうでなければ外側に隠す
toolBoxChkbox.addEventListener('change', (e) => {
  if(e.target.checked == true){
    toolBox.style.right = '-10px';
  } else {
    toolBox.style.right = '-200px';
  }
})
```

:::

## まとめ

このページでは、画面上に開発ツールを作成しました。  
次項ではここで作成した下記コントロールを基に、JavaScript を使って機能を実装していきます。

- フォントサイズ変更
  - `js_fontSize_controller`: コントロール群格納要素、表示/非表示を管理
  - `js_fontSize_range`: スライダー、フォントサイズ値を管理
  - `js_fontSize_result`: アウトプット、フォントサイズ値の表示
  - `js_fontSize_chkbox`: トグルボタン、機能有効化を管理
- カラーモード変更
  - `js_colorMode_controller`: コントロール群格納要素、表示/非表示を管理
  - `js_darkMode`: トグルボタン、カラーモード「ライト↔ダーク」の切り替えを管理
  - `js_colorMode_chkbox`: トグルボタン、機能有効化を管理
