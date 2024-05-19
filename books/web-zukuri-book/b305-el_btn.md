---
title: "📄B305：el_btn"
---

## このページについて

ここでは、ボタンについて説明します。  
今回で基本コンポーネントのエレメントモジュールは最後となります。

これまでとは違い、今回のボタンは３種類のバリエーションがあります。そのため、**モディファイア**というものを使い対応していくことになります。

## 各種ボタンの説明

各ボタンについて、軽く見ていきましょう。大きな特徴としては、色でなくパターンによる分類であることです。これにより、色覚による差異に関係なく感覚的にとらえることができるようになっています。

:::details 個人的メモ

プライマリ↔セカンダリ↔ターシャリ間での違いはパターンとして見分けがつきやすいが、非活性時に関してはまだ改善の余地がありそうです。

色の彩度を抑えて検証した際 違いが分かりづらかったので、非活性時では**ボタン全体に斜線を引く**など色に頼らない表現を実装できるのが望ましいのかもしれません。

:::

### プライマリボタン

![プライマリボタン、濃紺の背景に白色のテキスト](/images/books/web-zukuri/el_btn__primary-01.png)
*ライトモード時*

![プライマリボタン、水色の背景に白色のテキスト](/images/books/web-zukuri/el_btn__primary-02.png)
*ダークモード時*

![プライマリボタン、灰色の背景に白色のテキスト](/images/books/web-zukuri/el_btn__primary-03.png)
*非活性時*

これは**プライマリボタン**といい最重要なアクションに対して使われるものです。下記に、デザインシステム（Ver1.3.2）から抜粋したものを載せておきます。

> プライマリボタンは、ページ内で最も重要なアクション事に使用します。（例：「同意」「送信」「応募」）
> 『デジタル庁 デザインシステム（Ver1.3.2）』のコンポーネントの項から一部抜粋

### セカンダリボタン

![セカンダリボタン、無色の背景、濃紺の枠、濃紺のテキスト](/images/books/web-zukuri/el_btn__secondary-01.png)
*ライトモード時*

![セカンダリボタン、無色の背景、水色の枠、水色のテキスト](/images/books/web-zukuri/el_btn__secondary-02.png)
*ダークモード時*

![セカンダリボタン、無色の背景、灰色の枠、灰色のテキスト](/images/books/web-zukuri/el_btn__secondary-03.png)
*非活性時*

これは**セカンダリボタン**です。プライマリより重要度の低いアクションに使用します。

> プライマリボタンに比べ優先度が下がるアクションに対して使用します。（例：「問い合わせ」）
> 『デジタル庁 デザインシステム（Ver1.3.2）』のコンポーネントの項から一部抜粋

### ターシャリボタン

![ターシャリボタン、無色の背景と枠、濃紺の下線付きテキスト](/images/books/web-zukuri/el_btn__tertiary-01.png)
*ライトモード時*

![ターシャリボタン、無色の背景と枠、水色の下線付きテキスト](/images/books/web-zukuri/el_btn__tertiary-02.png)
*ダークモード時*

![ターシャリボタン、無色の背景と枠、灰色の下線付きテキスト](/images/books/web-zukuri/el_btn__tertiary-03.png)
*非活性時*

最後に**ターシャリボタン**です。これはセカンダリより重要度の低いアクションに使用します。

> プライマリボタン及びセカンダリボタンと比べて優先度が低い３つ目のアクションに対して使用します（例：「キャンセル」）
> 『デジタル庁 デザインシステム（Ver1.3.2）』のコンポーネントの項から一部抜粋

## 構造の定義

ここでは、どのような形にするかを定義していきます。

- `a` タグ、そして `button` タグに使うことを想定する
- `primary`, `secondary`, `tertiary` の３種類のボタンが存在
- これらの種類はロービジョン用に、色だけでなくパターンの違いによる差異を設ける
- `hover` や `disabled` 等に対しスタイルが変化

また `PRECSS` の考えに則り、CSS設計は下記のようにします。

- ３種に共通する基本的な構造は、`el_btn` クラスに規定
- 各ボタンに特有のスタイルについては、モディファイアクラス `el_btn__primary`, `el_btn__secondary`, `el_btn__tertiary` を設ける
- 使用時は `class="el_btn el_btn__primary"` のようにし、複合セレクタによるスタイル上書きを利用する

![説明図（３つのボタンから共通する部分を抜き出してベースクラスである el_btn にする。残った個別のバリエーションのスタイルはモディファイア el_btn__primary, el_btn__secondary, el_btn__tertiary に分けている図）](/images/books/web-zukuri/el_btn-modifier-01.png)

## 作成

上記で決めた要件から、マークアップしていきます。使用される際は、下記のようになります。

```html:使用例
<a href="#" class="el_btn el_btn__primary">
  Primary
</a>

<button type="button" class="el_btn el_btn__secondary" disabled>
  Secondary(disabled)
</button>
```

### 基本構造　`el_btn`

#### 要素の既存スタイルとの調整

全てのボタンに共通するスタイルを、このベースで規定します。

まず注意しなければならないのは、`a`, `button` タグのデフォルトスタイルについてです。これら既存のスタイルは、打ち消す必要があります。

`a` タグのアンダーラインや色についてはモディファイア側で扱う範囲なので、`display:inline;` が変える対象になります。`button` に関しては、背景や枠線などが対象になります。

```css
.el_btn {
  display: block;
  cursor: pointer;
  background: none;
  border: 2px solid transparent;
}
```

`border` に `transparent` を設定しているのは、３種のボタンの大きさを揃えるためです。

今回 `border` で枠線を使うのはセカンダリボタンのみです。そのため他で `border` を設定しなければ、`2px` 分 大きさがずれることになってしまいます。それを防ぐため、枠を使わないボタンにも全て `border: 2px` を設定し、色を透明にすることで、サイズが同じになるよう調整しています。

また本テーマでは、`button` タグを親要素の幅まで広げるため、`button` 要素用にベンダープレフィックスを加えています。

```css
button.el_btn {
  width: -moz-available;
  width: -webkit-fill-available;
}
```

#### 余白・フォント関係

次に、余白やフォント関係も付与していきます。

```css
.el_btn {
  display: block;

  padding: 16px;
  margin: 0;
  font-size: 1rem;
  font-style: normal;
  font-weight: 700;
  line-height: 1.5;
  text-align: center;
  letter-spacing: 0.04em;

  cursor: pointer;
  background: none;
  border: 2px solid transparent;
}

button.el_btn {
  width: -moz-available;
  width: -webkit-fill-available;
}
```

:::details paddingについてのメモ

`padding: 16px` なのは、デザインシステムにある指定です。これを `1rem` に置き換えていないのは、「ボタン内のテキストは大きさを変えることで読みやすくなるが、ボタンにおいては周りを囲むスペースを変化させても読みやすさには貢献しない」という考えで、そうしています。  
（不用意に余白が増えるとテキストの折り返しが発生し、逆に読みづらくなりうる可能性もあります）

文章内の段落間のスペースや、見出しの上下余白など、読みやすさにつながる部分には `rem` を用いるようしていますが、今回はそれには当てはまらないと考えています。

:::

#### `disabled` 属性

次に `disabled` 属性についてです。各ボタンで色や背景が変化するので、共通クラスでは `pointer-events: none;` のみの制御となります。

```css
.el_btn {
  display: block;
  padding: 16px;
  margin: 0;
  font-size: 1rem;
  font-style: normal;
  font-weight: 700;
  line-height: 1.5;
  text-align: center;
  letter-spacing: 0.04em;
  cursor: pointer;
  background: none;
  border: 2px solid transparent;
}

.el_btn:disabled {
  pointer-events: none;
}

button.el_btn {
  width: -moz-available;
  width: -webkit-fill-available;
}
```

#### その他のスタイル

最後にその他のスタイルについてです。これで共通スタイルの完成になります。

```css
.el_btn {
  display: block;
  padding: 16px;
  margin: 0;
  font-size: 1rem;
  font-style: normal;
  font-weight: 700;
  line-height: 1.5;
  text-align: center;
  letter-spacing: 0.04em;
  cursor: pointer;
  background: none;
  border: 2px solid transparent;

  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
  transition: .25s;
}

.el_btn:disabled {
  pointer-events: none;
}

button.el_btn {
  width: -moz-available;
  width: -webkit-fill-available;
}
```

ここでは新たに `border-radius` とホバー時の色変化用に `transition` を設定しました。  
（※ `border-radius` でやっていることの内容については、下で補足として説明します）

これで共通スタイルが規定できたので、次は各ボタン用のモディファイアを作成していきます。

#### 補足：`border-radius` の内容について

下記のテクニックは `el_btn` の他に `bl_card`, `bl_searchForm` にも使われていますので、何をしているのかについて解説します。

```css
border-radius: max(
  min(4px, calc(40px - 100%) * 9999),
  min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
  min(12px, calc(100% - 120px) * 9999)
);
```

上記の設定は、ボックスの形に丸みを持たせるためのものです。そしてその丸みとなる R値 ですが、コンテナクエリを使わず、自身の幅を参照して `4px`, `8px`, `12px` で変化するようになっています。

具体的に説明すると、下記のようになります。

- 自身が `40px` より小さい場合、`border-radius: 4px;`
- 自身が `40px` 以上かつ `120px` より小さい場合、`border-radius: 8px;`
- 自身が `120px` 以上の場合、`border-radius: 12px;`

:::details 計算の過程（長いので格納しています）

計算の過程を見ていきましょう。仮に、自身の幅が `20px` とします。すると、`max()` 内にある各 `min()` を計算していくと、下記のようになります。  
（※ `9999` は計算の簡略化のため、`10000` として扱います。`9999` は単に巨大な数というだけで、この数値自体に特殊な意味はありません）  
（※ マイナス値は、`0` として計算されます）

- `min(4px, calc(40px - 20px) * 9999)` -> `min(4px, 200000px)` -> `4px`
- `min(8px, calc(20px - 40px) * 9999, calc(120px - 20px) * 9999)` -> `min(8px, 0px, 1000000px)` -> `0px`
- `min(12px, calc(20px - 120px) * 9999)` -> `min(12px, 0px)` -> `0px`

そして、これら３つの数値は、`max(4px, 0px, 0px)` として計算されます。よって、自身が `20px` の要素は、`border-radius: 4px;` となります。

---

もう一つ例を見ていきましょう。今度は自身の幅が `100px` の時です。

- `min(4px, calc(40px - 100px) * 9999)` -> `min(4px, 0px)` -> `0px`
- `min(8px, calc(100px - 40px) * 9999, calc(120px - 100px) * 9999)` -> `min(8px, 600000px, 200000px)` -> `8px`
- `min(12px, calc(100px - 120px) * 9999)` -> `min(12px, 0px)` -> `0px`

結果としては `max(0px, 8px, 0px)` となるので、自身が `100px` の場合は `border-radius: 8px;` となりました。

---

最後に `160px` の場合を見てみましょう。

- `min(4px, calc(40px - 160px) * 9999)` -> `min(4px, 0px)` -> `0px`
- `min(8px, calc(160px - 40px) * 9999, calc(120px - 160px) * 9999)` -> `min(8px, 1200000px, 0px)` -> `0px`
- `min(12px, calc(160px - 120px) * 9999)` -> `min(12px, 400000px)` -> `12px`

結果としては `max(0px, 0px, 12px)` となるので、自身が `160px` の場合は `border-radius: 12px;` となりました。

:::

コンテナクエリを使用すれば、もっと単純な形で自身の幅を変化させることができます。ですがコンテナクエリの実装が比較的新しいこともあり、全てのブラウザで想定どおりの動きをするか保証ができませんでした。なので古いブラウザでも動くよう、この方式を採用しています。

### モディファイア `el_btn__primary`

では、各ボタンのモディファイアを作成していきます。まずはプライマリボタンからです。

![プライマリボタン（ライト/ダーク）](/images/books/web-zukuri/el_btn__primary-00.png)

![プライマリボタン（非活性状態）](/images/books/web-zukuri/el_btn__primary-03.png)

- テキスト色、及び背景色はセマンティックカラーを使用
- `a` タグ等に含まれるアンダーラインは不要
- ホバー、アクションの判別用に、色を少し濃くする

```css:モディファイア：プライマリボタン
.el_btn.el_btn__primary {
  color: var(--sc_txt_btn__primary);
  text-decoration: none;
  background-color: var(--sc_bg_btn__primary);
}

.el_btn.el_btn__primary:hover,
.el_btn.el_btn__primary:active {
  filter: brightness(87%);
}

.el_btn.el_btn__primary:disabled {
  background-color: var(--sc_bg_disabled);
}
```

注目してほしいのは、モディファイアに複合セレクタを使用していることです。こうすることで `el_btn` より `el_btn__primary` の詳細度を上げています。これは `PRECSS` のルールに則った方法になります。

`PRECSS` では `BEM` と異なり、モディファイアのスタイルで上書きする方法を認めています。

> 注意点としてモディファイアでスタイルを上書きする際は、基本的に複数クラスを使用して、詳細度を高めることを推奨しています。「スタイルを上書きする」ということは意図的なアクションであり、であればCSSの読み込み順でスタイルに変化が出てしまうのは好ましくありません。
> 『 [PRECSS](https://precss.io/ja/) 3-3.モディファイア』 より一部抜粋

### モディファイア `el_btn__secondary`

次にセカンダリボタンです。

![セカンダリボタン（ライト/ダーク）](/images/books/web-zukuri/el_btn__secondary-00.png)

![セカンダリボタン（非活性状態）](/images/books/web-zukuri/el_btn__secondary-03.png)

- テキスト色はセマンティックカラーを使用
- `a` タグ等に含まれるアンダーラインは不要
- 背景色は通常時は透過色、ホバーやアクション時は指定のセマンティックカラーを使用

```css:モディファイア：セカンダリボタン
.el_btn.el_btn__secondary {
  color: var(--sc_txt_btn__secondary);
  text-decoration: none;
  background-color: transparent;
  border-color: currentcolor;
}

.el_btn.el_btn__secondary:hover,
.el_btn.el_btn__secondary:active {
  background-color: var(--sc_bg_btn__secondary);
}

.el_btn.el_btn__secondary:disabled {
  color: var(--sc_txt_disabled);
}
```

### モディファイア `el_btn__tertiary`

最後にターシャリボタンからです。

![ターシャリボタン（ライト/ダーク）](/images/books/web-zukuri/el_btn__tertiary-00.png)

![ターシャリボタン（非活性状態）](/images/books/web-zukuri/el_btn__tertiary-03.png)

- テキスト色はセマンティックカラーを使用、背景色は常時透過色
- アンダーラインを使用、これは `button` タグでも同様
- ホバーやアクション時は指定のセマンティックカラーを使用

```css:モディファイア：ターシャリボタン
.el_btn.el_btn__tertiary {
  color: var(--sc_txt_btn__tertiary);
  text-decoration: underline;
  background-color: transparent;
}

.el_btn.el_btn__tertiary:hover,
.el_btn.el_btn__tertiary:active {
  color: var(--sc_txt_link__active);
}

.el_btn.el_btn__tertiary:disabled {
  color: var(--sc_txt_disabled);
}
```

## アクセシビリティについての補足

スタイルについてはこれで完成ですが、最後に `html` で使用する際の注意点について補足いたします。それは `a` タグと `button` タグは適切に使用すべきということです。

今回、`el_btn` の使用は `a`, `button` タグの２つを想定していると説明しました。ですが使用の際には、「ボタンとして扱う場合は `button` タグを使用」「リンクとして使う場合は `a` タグを使用」のように明確に分ける必要があります。同じスタイルになるからと言って、ボタンとして振る舞う要素を `a` タグで書くことは推奨しません。  
（※ 本テーマにおいては `el_btn` の使用は全て `button` タグのみとなっています）

これはキーボード操作やスクリーンリーダーでの振る舞いに影響するからです。

- キーボード操作の際、`button` タグは `space` キーでクリックイベントを発火させることができます。一方 `a` タグは発火しません
- スクリーンリーダーでは見た目に関わらず `button` タグは「ボタン」と読み上げますし、`a` タグは「リンク」と読み上げます

JavaScript の `Element.onkeydown` プロパティや、WAI-ARIA を使用すれば対処することは可能です。ですがそれは最後の手段、つまり他に手段がない場合に限定すべきだと考えます。今回のように１から作成するような場合には、適切なタグを使う方がアクセシビリティにとって望ましいことだと思われます。

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html
<a href="#" class="el_btn el_btn__primary">Primary</a>

<button type="button" class="el_btn el_btn__primary" disabled>Primary</button>

<a href="#" class="el_btn el_btn__secondary">Secondary</a>

<button type="button" class="el_btn el_btn__secondary" disabled>Secondary</button>

<a href="#" class="el_btn el_btn__tertiary">Tertiary</a>

<button type="button" class="el_btn el_btn__tertiary" disabled>Tertiary</button>
```

```css:style.css
/* ≡≡≡ ☐ el_btn ☐ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    ボタン
    aタグ、buttonタグで使うことを想定
  ■ 構成
    el_btn : ベース（下記のモディファイアと組み合わせることを想定）
    -> el_btn__primary : プライマリボタン、ページ内で最も重要なアクションに使用（例：同意、送信、応募）
    -> el_btn__secondary : セカンダリボタン、プライマリより優先度が低いアクション（例：問い合わせ）
    -> el_btn__tertiary : ターシャリボタン、セカンダリより優先度が低いアクション（例：キャンセル）
---------------------------------------------------------------------------- */
.el_btn {
  display: block;
  padding: 16px;
  margin: 0;
  font-size: 1rem;
  font-style: normal;
  font-weight: 700;
  line-height: 1.5;
  text-align: center;
  letter-spacing: 0.04em;
  cursor: pointer;
  background: none;
  border: 2px solid transparent;
  border-radius: max(
    min(4px, calc(40px - 100%) * 9999),
    min(8px, calc(100% - 40px) * 9999, calc(120px - 100%) * 9999),
    min(12px, calc(100% - 120px) * 9999)
  );
  transition: .25s;
}

.el_btn:disabled {
  pointer-events: none;
}

.el_btn.el_btn__primary {
  color: var(--sc_txt_btn__primary);
  text-decoration: none;
  background-color: var(--sc_bg_btn__primary);
}

.el_btn.el_btn__primary:hover,
.el_btn.el_btn__primary:active {
  filter: brightness(87%);
}

.el_btn.el_btn__primary:disabled {
  background-color: var(--sc_bg_disabled);
}

.el_btn.el_btn__secondary {
  color: var(--sc_txt_btn__secondary);
  text-decoration: none;
  background-color: transparent;
  border-color: currentcolor;
}

.el_btn.el_btn__secondary:hover,
.el_btn.el_btn__secondary:active {
  background-color: var(--sc_bg_btn__secondary);
}

.el_btn.el_btn__secondary:disabled {
  color: var(--sc_txt_disabled);
}

.el_btn.el_btn__tertiary {
  color: var(--sc_txt_btn__tertiary);
  text-decoration: underline;
  background-color: transparent;
}

.el_btn.el_btn__tertiary:hover,
.el_btn.el_btn__tertiary:active {
  color: var(--sc_txt_link__active);
}

.el_btn.el_btn__tertiary:disabled {
  color: var(--sc_txt_disabled);
}

button.el_btn {
  width: -moz-available;
  width: -webkit-fill-available;
}
```

:::

## まとめ

このページでは、ボタン用のコンポーネント `el_btn` について扱いました。  
その中で、複数のバリエーションを扱う上で重要となる**モディファイア**についても説明しました。

モディファイアで重要なこととしては、下記の事項がありました。

- バリエーション間で共通するスタイルは、ベースクラス側で規定する
- バリエーション特有のスタイルは、モディファイア側で規定する
- モディファイア側のスタイルを優先させるため、複合セレクタによる詳細度上げを利用する

今回で基本コンポーネントのエレメントモジュールは全て説明しました。次回からは複数の要素を組み合わせて作る**ブロックモジュール**を扱っていきます。
