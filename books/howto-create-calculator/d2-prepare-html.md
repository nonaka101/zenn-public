---
title: "応用②：今回使用するHTML文"
---

## 本項について

ここでは、これから JavaScript で操作していくための `html` 要素を掲載します。

:::message

スタイルに関する記述や説明は、本書の趣旨から外れるため除外します。

:::

:::details HTML全文（長いので格納しています）

```html
<div>
  <section aria-label="ディスプレイ">
    <span id="b01js_label"></span>
    <output id="b01js_output"></output>
  </section>
  <section aria-label="機能">
    <button type="button">閉じる</button>
    <button type="button" id="b01js_AllClear">全消去</button>
    <button type="button" id="b01js_BackSpace">1字消去</button>
    <button type="button" id="b01js_Copy">コピー</button>
  </section>
  <section aria-label="テンキー・実行">
    <button type="button" id="b01js_9" data-calc="9">9</button>
    <button type="button" id="b01js_8" data-calc="8">8</button>
    <button type="button" id="b01js_7" data-calc="7">7</button>
    <button type="button" id="b01js_6" data-calc="6">6</button>
    <button type="button" id="b01js_5" data-calc="5">5</button>
    <button type="button" id="b01js_4" data-calc="4">4</button>
    <button type="button" id="b01js_3" data-calc="3">3</button>
    <button type="button" id="b01js_2" data-calc="2">2</button>
    <button type="button" id="b01js_1" data-calc="1">1</button>
    <button type="button" id="b01js_0" data-calc="0">0</button>
    <button type="button" id="b01js_DecimalPoint" data-calc="." aria-label="小数点">.</button>
    <button type="button" id="b01js_Equal" data-calc="=" aria-label="計算実行">=</button>
  </section>
  <section aria-label="四則演算">
    <button type="button" id="b01js_Add" data-calc="+" aria-label="足す">＋</button>
    <button type="button" id="b01js_Subtract" data-calc="-" aria-label="引く">−</button>
    <button type="button" id="b01js_Multiply" data-calc="*" aria-label="掛ける">×</button>
    <button type="button" id="b01js_Divide" data-calc="/" aria-label="割る">÷</button>
  </section>
</div>
```

:::

## 各セクション毎の詳細

ここでは 各 `section` ごとに、中身を簡単に解説していきます。

主に `htmlCalculator` クラスを操作するための `button` 要素を格納しているものが多いです。

:::details button要素についてのメモ

電卓ボタンは、（`span` や `div` などでなく）できれば `button` 要素で実装すべきです。

`button` 要素は、インタラクティブ要素として、下記の特徴を持っています。

- （Tabキーなどで）フォーカスできる
- スペースキーで `click` イベントを発火させられる
- `disabled` 属性の付与で、フォーカスを不可に、スクリーンリーダーの読み上げから除外できる

また その特徴（インタラクティブ要素であること）を強調するため、ブラウザに備わっているデフォルトスタイルが特徴的な表現になっています。自由なスタイリングには `appearance` による[固有のスタイル除去](https://developer.mozilla.org/ja/docs/Web/CSS/appearance)が必要になってきますが、アクセシビリティ上のメリットが大きいため、`button` 要素の使用をおすすめします。

:::

### ディスプレイ部

![ディスプレイ部のスクリーンショット](/images/books/howto-create-calculator/ui-display-01.png)

```html
<section aria-label="ディスプレイ">
  <span id="b01js_label"></span>
  <output id="b01js_output"></output>
</section>
```

ここは、`htmlCalculator` クラスで管理している「式」「ラベル」を表示出力させる要素です。

`output` 要素は 暗黙のロールとして `status` を持ち、多くのブラウザで `aria-live="polite"` 領域として認識されます。これによりスクリーンリーダーは、中身が変化した際（例：式への入力時 など）、読み上げに余裕があるタイミング（≒アイドル状態）で中身を読み上げてくれます。

https://developer.mozilla.org/ja/docs/Web/HTML/Element/output

### 各種ボタン

![ボタン配列のスクリーンショット](/images/books/howto-create-calculator/ui-button-01.png)

#### 機能ボタン部

```html
<section aria-label="機能">
  <button type="button">閉じる</button>
  <button type="button" id="b01js_AllClear">全消去</button>
  <button type="button" id="b01js_BackSpace">1字消去</button>
  <button type="button" id="b01js_Copy">コピー</button>
</section>
```

ここでは、`htmlCalculator` クラスの `reset()`, `back()` 処理を行うためのボタン要素を格納しています。  
（※閉じるボタンとコピーボタンもありますが、本書の趣旨から外れる部分のため説明を除外します）

#### 数値ボタン部

```html
<section aria-label="テンキー・実行">
  <button type="button" id="b01js_9" data-calc="9">9</button>
  <button type="button" id="b01js_8" data-calc="8">8</button>
  <button type="button" id="b01js_7" data-calc="7">7</button>
  <button type="button" id="b01js_6" data-calc="6">6</button>
  <button type="button" id="b01js_5" data-calc="5">5</button>
  <button type="button" id="b01js_4" data-calc="4">4</button>
  <button type="button" id="b01js_3" data-calc="3">3</button>
  <button type="button" id="b01js_2" data-calc="2">2</button>
  <button type="button" id="b01js_1" data-calc="1">1</button>
  <button type="button" id="b01js_0" data-calc="0">0</button>
  <button type="button" id="b01js_DecimalPoint" data-calc="." aria-label="小数点">.</button>
  <button type="button" id="b01js_Equal" data-calc="=" aria-label="計算実行">=</button>
</section>
```

ここでは 数値に加え、小数点やイコールボタンを格納しています。

::::details 「=」の aria-label を「イコール」にしていない理由

`img` の `alt` にも共通することですが、私は スクリーンリーダーを使用する方へのアクセシビリティを下記のように考えています。

- **文脈に基づく情報提供**：記号や画像情報は単なる外観でなく、前後の文脈を含めて認知されたものであるべきです
- **適切な情報量**：晴眼者と比較し、情報量は過剰でも不足でもいけません
- **晴眼者と同等の体験**：晴眼者と「同じ情報」より「同じ体験」を目指すべきです

その上で、`=` 記号は**その役割である**「計算実行」の方をラベルにすべきと考えました。

以下の格納されたメモについては、拙いながら私の考えを書いたものになります。

:::details 「外観でなく認知された情報」について

ここでは、ある記号と それを受け取った際の認識について考えてみます。

例えば、UI上に `💾`, `≡`, `⚙` マークがあったとします。この時 晴眼者は「フロッピー」「三本線」「歯車」とは認識しないでしょう。意識にも登らない段階で、「保存」「メニュー」「設定」と認識するはずです。

これは前後のコンテキスト（文化的に共有された認識、言ってしまえば「お約束」のようなもの？）によるものです。

- メニュー欄には通常「保存」や「設定」の項目がある
- 長年多くの Web サイトで3本線のマークをメニューとして扱ってきた経緯がある

画像や記号を説明する際には、こうしたコンテキストを踏まえるべきなのだと思います（最も、フロッピーは昨今使われなくなってきたので、「保存」を表す意味合いもだんだん薄れていくとは思いますが・・・）

:::

:::details 「過剰な情報量」について

ここでは、プログレスバーと [progress 要素](https://developer.mozilla.org/ja/docs/Web/HTML/Element/progress)による、同じ内容を繰り返してしまう例を挙げてみます。

状況として、視覚的なプログレスバーとして `div` を使ったローディングスピナーを用意し、別途スクリーンリーダー用に視覚から隠した `progress` 要素も用意したとします。

この場合、`div` 要素を AOM（Accessibility Object Model）上から消していない場合、同じ内容を2回読み上げることになります。同じ内容を複数回読み上げると、ユーザーは冗長に感じ、集中力が削がれてしまいます。情報が伝わらないケースよりは良いのでしょうが、削れる部分は削ったほうが より良くなります。

スクリーンリーダーの読み上げは、マウスやタップ操作と違い、コンテンツの順番通りに進むという制限が課されます。そのため ユーザーが必要とする情報へ素早くアクセスできるよう、心がけておくべきです。

:::

:::details 「晴眼者と同等の体験」について

ここではその例として、`select` 要素を挙げてみます。これはデフォルトのスタイルとしてボックスに `▼` が出ることが多いですが、これを除去して自作するケースが多いです。

その際、セレクトボックスにある `▼` は、晴眼者にとっては「展開可能」であることを示す視覚情報です。しかしスクリーンリーダーは `select` 要素を「展開可能なボックス」と認識しており、要素にたどり着いた段階でユーザーへその旨を伝えてくれます。

つまり両者は、（`▼` と `select` で）箇所は違うけど同じ体験ができるようになっているわけです。これを「同じ情報を提供しなければならない」という考えに嵌まると、`▼` を「下方向の三角」や「展開可能」とスクリーンリーダに読み上げさせなければなりません。けれどもこれは、恐らくユーザーにとって混乱を生じさせるのではないでしょうか？

同じ体験でにあることを重視するなら、`▼` 要素は AOM 上から消しておき、スクリーンリーダーが読み上げないようした方がいいでしょう。

:::

::::

#### 演算子ボタン部

```html
<section aria-label="四則演算">
  <button type="button" id="b01js_Add" data-calc="+" aria-label="足す">＋</button>
  <button type="button" id="b01js_Subtract" data-calc="-" aria-label="引く">−</button>
  <button type="button" id="b01js_Multiply" data-calc="*" aria-label="掛ける">×</button>
  <button type="button" id="b01js_Divide" data-calc="/" aria-label="割る">÷</button>
</section>
```

ここでは四則演算子のボタンを格納しています。スクリーンリーダーの読み上げを統一するため、記号それぞれに `aria-label` を割り振ってます。
