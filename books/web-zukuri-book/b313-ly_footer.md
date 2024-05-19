---
title: "📄B313：ly_footer"
---

## このページについて

ここからは、レイアウト（`ly_*`）について扱っていきます。

このページでは、レイアウトの中でも扱いが単純なフッター部 `ly_footer` について取り扱います。

## 構造の定義

まずは構造から決めていきます。まずはフッターに限らずコンテンツ全体に適用されるものになります。

- コンテンツの最大幅は `1024px` までとする
- 画面幅がそれ以上ある時は、コンテンツ自身は `1024px` のまま、画面中央に配置する

その上で、フッターの基本的な要件を決めていきます。

- 背景色をメイン部と変える
- いくつかのウィジェットで構成される
- 画面幅に余裕がある時は横並びにし、余裕がない時は縦に並べる

本テーマでは `display: grid;` を使い、横方向に `12fr` 分割します。そしてこの `12fr` 内で、各種ウィジェットを振り分けていきます。

![footer部、3つのウィジェットが横方向12のグリッドを使って配置されている](/images/books/web-zukuri/ly_footer-01.png)
*デスクトップ画面*

本テーマは上図のように、構成比 3:6:3 でウィジェットを配置します。モバイル画面などの画面幅が小さい時は、これらのウィジェットは `12fr` 全てを使って縦１列に並びます。

![上図のモバイル画面時、横に並べる余裕がないので3つのウィジェットは縦1列に並んでいる](/images/books/web-zukuri/ly_footer-02.png)
*モバイル画面*

また、ウィジェットとは別に本テーマではコピーライト分を画面最後段に入れられるようにしています。これは画面幅関係なく常に最後段に同じように表示されます。

![コピーライト文、サイト最下段にある](/images/books/web-zukuri/ly_footer-04.png)
*デスクトップ画面*

![上図のモバイル画面時、位置は変わらずサイト最下段にある](/images/books/web-zukuri/ly_footer-03.png)
*モバイル画面*

なので、最終的な要件は下記になります。

- 背景色をメイン部と分ける
- コンテンツの最大幅は `1024px`
- 列を `12fr` に分割したGridを使ってウィジェットを配置
  - デスクトップ画面時は 3:6:3 の構成比でウィジェットを配置
  - モバイル画面時は 12fr 全て使い縦１列にウィジェットを配置
- コピーライト要素をサイト最後段に入れる

## 作成

上述した要件を基に、要素を作成していきます。HTMLの構成は、大まかに下記のようになります。

- ベース部
  - ウィジェット格納要素（12Gridを使用）
    - ウィジェット要素
  - コピーライト要素

### ベース部の作成

まずはベース部です。`footer` 要素にクラス `ly_footer` を設定します。またフッターは**B309：ブロックスキップ**のリンク先となりますので、アンカー用のフラグメントを用意し、`tabindex="-1"` に設定することでフォーカス可能状態（ただし Tabキーによる操作ではフォーカスしない）にしておきます。

```html
<footer class="ly_footer" id="anchor_footer" tabindex="-1">
  <!-- ウィジェット格納要素 -->
  <!-- コピーライト要素 -->
</footer>
<!-- /.ly_footer -->
```

次にスタイルです。ここで設定するものは下記のとおりです。

- コピーライト要素で `position: absolute;` を使うため、`position: relative;` に
- ウィジェット等は画面全体に広がらないようにパディングを設ける
- （上にある）メイン要素との間にマージンを設ける
- 背景色を `--sc_bg_body__tertiary` に設定（メイン要素は `--sc_bg_body__primary`）

```css
.ly_footer {
  position: relative;
  padding: 64px 24px;
  margin-top: 40px;
  background-color: var(--sc_bg_body__tertiary);
}
```

### ウィジェット格納要素

次にウィジェット群を格納する要素 `ly_footer_widgetArea` を作成します。この中に、３つのウィジェットが格納される想定です。

```html
<footer class="ly_footer" id="anchor_footer" tabindex="-1">
  <div class="ly_footer_widgetArea">
    <!-- 左側の 3fr ウィジェット -->
    <!-- 中央の 6fr ウィジェット -->
    <!-- 右側の 3fr ウィジェット -->
  </div>
  <!-- /.ly_footer_widgetArea -->
  <!-- コピーライト要素 -->
</footer>
<!-- /.ly_footer -->
```

次はスタイルです。ウィジェットを格納するために `display: grid;` 設定しているのが大きな特徴です。

- 要素の最大幅は `1024px`
  - 画面幅がそれ以上ある場合は `1024px` のまま中央配置
- 横方向 `12fr` に分割したグリッド構成
  - 下位要素は横方向に配置
  - `gap` は画面幅に応じて変更
    - モバイル画面時は縦方向に `1rem`
    - デスクトップ画面時は横方向に `2rem`
- モバイル画面でも画面幅に余裕がある場合（`560px` 以上）は、パディングを設ける

```css
.ly_footer_widgetArea {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  grid-auto-flow: row;
  gap: 1rem 0;
  max-width: 1024px;
  margin-right: auto;
  margin-left: auto;
}

@media (min-width: 560px) {
  .ly_footer_widgetArea {
    padding: 0 40px;
  }
}

@media (min-width: 960px) {
  .ly_footer_widgetArea {
    gap: 0 2rem;
    padding: 0;
  }
}
```

### ウィジェット要素

次にウィジェットを作成していきます。クラス名は `ly_footer_widget` をベースとし、使用するフレーム数（例：`3fr`）をモディファイアとして設定します。

```html
<footer class="ly_footer" id="anchor_footer" tabindex="-1">
  <div class="ly_footer_widgetArea">
    <div class="ly_footer_widget ly_footer_widget__3fr">
      <!-- 左側の 3fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->
    <div class="ly_footer_widget ly_footer_widget__6fr">
      <!-- 中央の 6fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__6fr -->
    <div class="ly_footer_widget ly_footer_widget__3fr">
      <!-- 左側の 3fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->
  </div>
  <!-- /.ly_footer_widgetArea -->
  <!-- コピーライト要素 -->
</footer>
<!-- /.ly_footer -->
```

`ly_footer_widget__Left` のように位置指定でなくフレーム数をクラス名に使っているのは、今後の拡張の余地を残すためです。本テーマではモディファイアは `3fr`, `6fr` の２種類のみですが、`12fr` に収まるのなら「`9fr` と `3fr`」あるいは「`4fr` × 3」 と拡張できるよう、命名にフレーム数を使用しています。

次にスタイルです。格納要素 `ly_footer_widgetArea` で「下位要素は横方向に並べる」と指定していたことがここで活かされます。これにより開始位置を指定せず「`3fr` のサイズ」のような使い方ができます。要素は自動的に横方向に並べられ、横に並べられない場合は縦に配置してくれます。

- `.ly_footer_widget.ly_footer_widget__3fr` の横サイズ
  - モバイル画面時は `12fr`
  - デスクトップ画面時は `3fr`
- `.ly_footer_widget.ly_footer_widget__6fr` の横サイズ
  - モバイル画面時は `12fr`
  - デスクトップ画面時は `6fr`

つまり、`ly_footer_widget` には `12fr` を使用する規定をし、`@media (min-width: 960px){...}` を使った時にモディファイア側でグリッドエリアを上書きする処理を行えばいいというわけです。

```css
.ly_footer_widget {
  grid-area: span 1/span 12;
}

@media (min-width: 960px) {
  .ly_footer_widget.ly_footer_widget__3fr {
    grid-area: span 1/span 3;
  }

  .ly_footer_widget.ly_footer_widget__6fr {
    grid-area: span 1/span 6;
  }
}
```

開始位置の指定がないため、同じような形で例えば `ly_footer_widget__4fr` や `ly_footer_widget__9fr` を作って組み合わせても、問題なく動作します。  
（全体を `12fr` に揃える必要はありますが）

### コピーライト要素

最後にコピーライト要素です。ここでは配置を決めるための要素 `div.ly_footer_copyright` と、テキスト要素 `p.el_copyright` を分けています。  
（まとめても問題ないのですが、コピーライトでなく、各種プライバシー条項などのコーポレートリンク集に変更しやすいよう余地を残すためです）

```html:ウィジェット要素は省略
<footer class="ly_footer" id="anchor_footer" tabindex="-1">
  <!-- ウィジェット格納要素 -->
  <div class="ly_footer_copyright">
    <p class="el_copyright">
      2023 Web-Zukuri（コピーライト用テキスト）
    </p>
    <!-- /.el_copyright -->
  </div>
  <!-- /.ly_footer_copyright -->
</footer>
<!-- /.ly_footer -->
```

次にスタイルを規定します。先程申した通り、レイアウトに関するスタイルとテキストに関するスタイルが分かれていることがわかります。

```css
.ly_footer_copyright {
  position: absolute;
  right: 8px;
  bottom: 4px;
  left: 8px;
  padding: 0;
  margin: 0;
}

.el_copyright {
  font-size: 0.75em;
  color: var(--sc_txt_body__small);
  text-align: center;
}
```

これで完成です。

## 完成形

最後に完成形をまとめておきます。

:::details 全文（長いので格納しています）

```html
<footer class="ly_footer" id="anchor_footer" tabindex="-1">
  <div class="ly_footer_widgetArea">
    <div class="ly_footer_widget ly_footer_widget__3fr">
      <!-- 左側の 3fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->
    <div class="ly_footer_widget ly_footer_widget__6fr">
      <!-- 中央の 6fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__6fr -->
    <div class="ly_footer_widget ly_footer_widget__3fr">
      <!-- 左側の 3fr ウィジェット -->
    </div>
    <!-- /.ly_footer_widget ly_footer_widget__3fr -->
  </div>
  <!-- /.ly_footer_widgetArea -->
  <div class="ly_footer_copyright">
    <p class="el_copyright">
      2023 Web-Zukuri（コピーライト用テキスト）
    </p>
    <!-- /.el_copyright -->
  </div>
  <!-- /.ly_footer_copyright -->
</footer>
<!-- /.ly_footer -->
```

```css:style.css
/* ≡≡≡ ◲ ly_footer ◲ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡
  ■ 概要
    フッター部のレイアウトを構成する
    デスクトップ画面の時は、横に `3:6:3` の比率で3つのウィジェットを並べる。
    モバイル画面の時は、ウィジェットは縦1列に並べる
  ■ 構成
    ly_footer : ベース
      ly_footer_widgetArea : ウィジェット格納要素
        ly_footer_widget : ウィジェットのベース（モバイル画面で 12fr ）
        -> ly_footer_widget__3fr : デスクトップ画面で 3fr 使うモディファイア
        -> ly_footer_widget__6fr : デスクトップ画面で 6fr 使うモディファイア
      ly_footer_copyright : 最下部のコピーライト用
        el_copyright : コピーライトテキスト
---------------------------------------------------------------------------- */
.ly_footer {
  position: relative;
  padding: 64px 24px;
  margin-top: 40px;
  background-color: var(--sc_bg_body__tertiary);
}

.ly_footer_copyright {
  position: absolute;
  right: 8px;
  bottom: 4px;
  left: 8px;
  padding: 0;
  margin: 0;
}

.el_copyright {
  font-size: 0.75em;
  color: var(--sc_txt_body__small);
  text-align: center;
}

.ly_footer_widgetArea {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  grid-auto-flow: row;
  gap: 1rem 0;
  max-width: 1024px;
  margin-right: auto;
  margin-left: auto;
}

@media (min-width: 560px) {
  .ly_footer_widgetArea {
    padding: 0 40px;
  }
}

@media (min-width: 960px) {
  .ly_footer_widgetArea {
    gap: 0 2rem;
    padding: 0;
  }
}

.ly_footer_widget {
  grid-area: span 1/span 12;
}

@media (min-width: 960px) {
  .ly_footer_widget.ly_footer_widget__3fr {
    grid-area: span 1/span 3;
  }

  .ly_footer_widget.ly_footer_widget__6fr {
    grid-area: span 1/span 6;
  }
}
```

:::

## まとめ

本項ではレイアウトの１つであるフッター `ly_footer` を解説しました。

このように、レイアウトは画面幅に応じて大きく変化するものとなっています。またレイアウトは、その名の通り、画面に対する要素のレイアウト（幅や高さといったサイズ）を規定するのが特徴です。これはコンテナ要素一杯に要素を使うブロックモジュールやエレメントモジュールとは性質が異なっています。

次項ではメイン要素となる `ly_mainArea` について扱っていきます。
