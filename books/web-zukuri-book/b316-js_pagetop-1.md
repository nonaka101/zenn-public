---
title: "📄B316：js_pageTop(1)"
---

## このページについて

本項では JavaScript を使用し「ページトップに戻る」ボタンとして機能させる `js_pageTop` について、2回に分けて解説していきます。

1回目となる本項では、JavaScript を除いた部分を説明します。

## 構造の定義

![ページトップに戻るボタン](/images/books/web-zukuri/js_pageTop-01.png =100x)

このボタンの基本的な特徴は、下記になります。

- 形は、`↑` マークがついた丸ボタン
  - ラベルは画面に出力せず、`aria-label` を使用
  - ボタンであることがわかるようにする（ポインター、ホバーやアクション時など）
  - カラーモードに関わらず、色は固定
- 画面右下に固定表示
- 一定の下スクロールで表示、ページ上部に戻ると非表示
- ボタンを押すとページ最上部にスクロールとフォーカスが移動

## 作成

上記で決めた要件から、ボタンをマークアップしていきます。

### `button` を画面左下へ固定表示

まずは、HTMLで構造を作っていきます。マシンリーダブルであるために、`button` タグを使用します。

:::message

`a`タグをボタンとして使うのではなく `button`タグを使うのは、アクセシビリティのためです。

`a`タグを使って、`type="button"` や `role="button"` でボタンに仕上げたとします。この場合、スクリーンリーダーでボタンであることは認識できます。しかし、`button`タグと違い、スペースキーを使ってボタンを押下（発火）することはできません。

Javascriptでイベントハンドラを設定することで解消できますが、「（同じ結果になるのならば）WAI-ARIAは使わない方を選択する」方針を取り、ここでは `button` タグを使った実装をしていきます。

:::

これまではスタイルを操作するためにクラス名を付与してきましたが、ここでは `id` を使います。理由は下記の2つです。

- 他と違い1つのページに要素は1つしか存在しえず、`class` を使用する意味が薄い
- JavaScript で `Document.getElementById("id名")`（または `Document.querySelector("#id名")`） を使用するため、スタイルとスクリプトで名前を共有できる

```html
<button type="button" id="js_pageTop">
</button>
<!-- /#js_pagetop -->
```

配置する場所は、`footer` タグの下です。**B200：開発者ツール**を除けば、これがドキュメントの最後方になります。最後方に設置する理由として、ページ最後方から最初へと、容易に移動できる手段を提供する意図があります。これはキーボード操作やスクリーンリーダーなどの、コンテンツ順に操作が制限されているユーザーに対しての意味合いが強いです。

次にスタイリングを施していきます。まずは位置に関するものです。画面の右下に固定し、広い画面時には少し内側に寄せるようにします。  
（内に寄せるのは操作性と視認性のためです）

```css:style.css
#js_pageTop {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 10;
}

/* デスクトップ画面では余裕があるので少し内側に寄せる */
@media (min-width: 960px) {
  #js_pageTop {
    right: 40px;
    bottom: 40px;
  }
}
```

:::message

**サイズや位置に関する単位**に `rem` でなく `px` を使っているのがわかります。これはヘッダーで使う考え同様、**コンテンツの閲覧を邪魔しない**ようする意図があります。

ヘッダーや当ボタンに関しては、画面に固定表示する要素です。画面からこれらの要素を除いた領域が、ユーザーがコンテンツを閲覧できる部分となります。`rem` を使うとフォントサイズの変更に追従し、ヘッダーやボタンが大きくなることが考えられます。それはコンテンツ領域を狭めることに繋がり、かえって便益を損ねることになりかねません。なので本テーマでは必要分を固定単位の `px` で指定する方法を取っています。

:::

### インラインSVGの挿入

次に `↑` マークを実装します。ここではインラインSVGを用いて再現しています。

:::message

カラーモードに影響しないのでインライン化する必要性は薄いです（HTTP通信の回数を減らせるなど、利点がないわけではないです）

本テーマでは、他の場所で外部ファイル化したSVGを用いていない関係から、ここもインライン化で済ませています。

:::

```html
<button type="button" id="js_pageTop">
  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 16 16">
    <path fill-rule="evenodd" d="M8 15a.5.5 0 0 0 .5-.5V2.707l3.146 3.147a.5.5 0 0 0
      .708-.708l-4-4a.5.5 0 0 0-.708 0l-4 4a.5.5 0 1 0 .708.708L7.5 2.707V14.5a.5.5 0 0 0 .5.5z">
    </path>
  </svg>
</button>
<!-- /#js_pagetop -->
```

この状態は、Webアクセシビリティ上の問題があります。SVGに読み上げる内容がないため、スクリーンリーダーは これを「ボタン」とだけ読み上げることになります。何を行うのかわからないボタンがあるのは、避けなければなりません。

今回の場合、画面に表示するラベル（≒テキストノード）はないので `button` タグに `aria-label` を設定し、`svg` タグはAOM上から除外します。

```html:マシンリーダブルした状態
<button type="button" id="js_pageTop" aria-label="ページトップに戻る">
  <svg
    role="graphics-symbol img"
    aria-hidden="true"
    xmlns="http://www.w3.org/2000/svg"
    width="24"
    height="24"
    viewBox="0 0 16 16"
  >
    <path fill-rule="evenodd" d="M8 15a.5.5 0 0 0 .5-.5V2.707l3.146 3.147a.5.5 0 0 0
      .708-.708l-4-4a.5.5 0 0 0-.708 0l-4 4a.5.5 0 1 0 .708.708L7.5 2.707V14.5a.5.5 0 0 0 .5.5z">
    </path>
  </svg>
</button>
<!-- /#js_pagetop -->
```

SVGタグの `role="graphical-symbol img"` でアイコン（`graphical-symbol` が認識できない場合は `img`）であることを伝えていますが、`aria-hidden="true"` でAOM上からは消しています。これは、`button` の `aria-label` に「ページトップに戻る」があるので、スクリーンリーダーはSVGなしで「ページトップに戻る ボタン」と読み上げてくれるからです。

:::message

このあたりの考えについては、**Z000：附則**の章で取り扱いたいと思っています。

:::

次にCSSでスタイリングしていきます。外観について、下記の内容を盛り込みます。

- 幅と高さは56px
- 背景色は白固定
- 枠線は青色固定
- 円形

```css:style.css
#js_pageTop {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 10;
  width: 56px;
  height: 56px;
  background-color: #fff;
  border: 1px solid #0028b5;
  border-radius: 50%;
}
```

:::message

色に関しては原則セマンティックカラーを使う方針ですが、カラーモードで変化しない要素のため、RGB値の直接入力を許容しています。

デジタル庁にあったボタンと似せることを優先した結果、用意したカラーに近しい色がないためこうなったと記憶しています。あまり良いやり方とは言えないので、どこかのタイミングで更新するかもしれません。

:::

次に内部要素入るSVG要素に向けたスタイリングです。下記の内容を盛り込みます。

- 内部の構造はフレックスで、中心揃え
- 中のSVGの色はセマンティックカラー（`--sea-900`）

```css:style.css
#js_pageTop {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
  color: var(--sea-900);
  cursor: pointer;
  background-color: #fff;
  border: 1px solid #0028b5;
  border-radius: 50%;
}
```

### ホバー時のスタイル設定

次にホバー時の挙動をスタイリングしていきます。

- ホバー時のカーソルはポインター
- ホバーがわかるよう、背景色を少し青く、枠線は青色を濃くする
- `transition` をつけて、挙動の切り替えがわかるようにする

```css:style.css
#js_pageTop {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
  color: var(--sea-900);
  cursor: pointer;
  background-color: #fff;
  border: 1px solid #0028b5;
  border-radius: 50%;
  transition: background-color 0.5s cubic-bezier(0.4, 0.4, 0, 1);
}

#js_pageTop:hover {
  background-color: #e7eefd;
  border-color: #0f41af;
}
```

### スクロール状態による表示非表示の切り替え

最後に、スクロールによる表示非表示の切り替えです。これは、Javascriptを使い `button` タグに `hidden` 属性をつけ外しすることで表現するものと想定します。下へのスクロール量が一定値を超えると、`hidden` 属性が外れ、徐々にボタンが現れてくるといったイメージです。

:::message

`display: flex;` を使用している関係上、`hidden` 属性をつけただけでは非表示にはならないので注意が必要です。

ボタンを非表示にするには、`display: none;` で上書きするか、`visibility: hidden;` で消すか、`opacity: 0;` で見えなくする必要があります（※ `opacity` は透過するだけなので、操作することが可能な点にも注意が必要です）

:::

大まかな要件を、下記のようにします。

- レンダリング時はページトップにいるので、`hideden` 属性がついた状態
- 一定の下スクロールで `hidden` 属性が外れ、0.5秒かけてボタンがフェードイン
- ページトップに戻ると `hidden` 属性がつき、0.5秒かけてボタンがフェードアウト
- 非表示時にボタンが触れないよう、`opacity` だけでなく `visibility` を使う

上記を達成するために、`@keyframes` を使い、`fade_in`, `fade_out` のアニメーションを作ります。  
`opacity` は一定速度で変化するのに対し、`visibility` はフェードイン時は効果終了時、フェードアウトは効果開始時で変化するようにします。こうすることで、`visibility` の効果を `opacity` による見た目の変化と一致して提供することができます。

```css:説明簡略化のためアニメーションに関する部分のみ
#js_pageTop {
  animation-name: fade_in;
  animation-duration: 0.5s;
}

@keyframes fade_in {
  0% {
    visibility: hidden;
    opacity: 0;
  }

  1% {
    visibility: visible;
    opacity: 0.01;
  }

  100% {
    visibility: visible;
    opacity: 1;
  }
}

#js_pageTop[hidden] {
  visibility: hidden;
  animation-name: fade_out;
  animation-duration: 0.5s;
}

@keyframes fade_out {
  0% {
    visibility: visible;
    opacity: 1;
  }

  99% {
    visibility: visible;
    opacity: 0.01;
  }

  100% {
    visibility: hidden;
    opacity: 0;
  }
}
```

## 完成形

:::details 全文（長いので格納しています）

```html
<button type="button" id="js_pageTop" aria-label="ページトップに戻る">
  <svg
    role="graphics-symbol img"
    aria-hidden="true"
    xmlns="http://www.w3.org/2000/svg"
    width="24"
    height="24"
    viewBox="0 0 16 16"
  >
    <path fill-rule="evenodd" d="M8 15a.5.5 0 0 0 .5-.5V2.707l3.146 3.147a.5.5 0 0 0
      .708-.708l-4-4a.5.5 0 0 0-.708 0l-4 4a.5.5 0 1 0 .708.708L7.5 2.707V14.5a.5.5 0 0 0 .5.5z">
    </path>
  </svg>
</button>
<!-- /#js_pagetop -->
```

```css:style.css
#js_pageTop {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
  color: var(--sea-900);
  cursor: pointer;
  background-color: #fff;
  border: 1px solid #0028b5;
  border-radius: 50%;
  transition: background-color 0.5s cubic-bezier(0.4, 0.4, 0, 1);
  animation-name: fade_in;
  animation-duration: 0.5s;
}

#js_pageTop:hover {
  background-color: #e7eefd;
  border-color: #0f41af;
}

@keyframes fade_in {
  0% {
    visibility: hidden;
    opacity: 0;
  }

  1% {
    visibility: visible;
    opacity: 0.01;
  }

  100% {
    visibility: visible;
    opacity: 1;
  }
}

#js_pageTop[hidden] {
  visibility: hidden;
  animation-name: fade_out;
  animation-duration: 0.5s;
}

@keyframes fade_out {
  0% {
    visibility: visible;
    opacity: 1;
  }

  99% {
    visibility: visible;
    opacity: 0.01;
  }

  100% {
    visibility: hidden;
    opacity: 0;
  }
}

/* デスクトップ画面では余裕があるので少し内側に寄せる */
@media (min-width: 960px) {
  #js_pageTop {
    right: 40px;
    bottom: 40px;
  }
}
```

:::

## まとめ

本項では、「ページトップに戻る」ボタンとなる `js_pageTop` について、機能を除いた部分を解説しました。

次項では、JavaScript を用いて本項で作ったボタンに機能を付与していきます。
