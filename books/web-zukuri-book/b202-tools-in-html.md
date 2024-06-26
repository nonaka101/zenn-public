---
title: "📄B202：ツールの配置"
---

## このページについて

ここでは 前項で説明した開発ツールを、画面内にどのように表示するかについて考えます。  
また、機能を実装するに必要なコントロールの種類や数についても考えていきます。

## ツールの場所

ツールの場所ですが、なるべく邪魔にならないよう画面の右下に配置します。

![ツールボックスの画面（格納状態）](/images/books/web-zukuri/bl_devToolBox-00.png)

本テーマでは「ページトップに戻る」ボタンが画面の右下に表示される関係上、それと被らないようにする必要があります。

そして通常時は展開ボタンだけ見えるようにし、ボタンを押したときだけツールボックス本体が出てくるようにします。

![ツールボックス（展開状態）](/images/books/web-zukuri/bl_devToolBox-01.png)

## 機能に必要となるコントロールについて

次に、ツールボックスに必要なコントロールについて考察していきます。

### カラーモード切り替え

カラーモード切り替えのコントロールですが、結論から先に言うと ２つのトグルボタン（チェックボックス）が必要と考えます。

1. 自動（ブラウザ設定）から手動（JavaScriptによる上書き）に切り替えるためのトグルボタン
2. 「ライト↔ダーク」切り替えのためのトグルボタン

#### カラーモード手動切り替えのトグルボタン

このトグルボタンがオフの状態の時には、ブラウザ設定の状態を優先します。また、カラーモードのトグルボタンは不要なので非表示にしておきます。

![トグルボタンがオフの状態](/images/books/web-zukuri/bl_devToolBox-01.png)

トグルボタンがオンになると、JavaScript の内容を優先、ブラウザ設定は無視するようにします。また、カラーモード切り替えのトグルボタンを表示するようにします。

![トグルボタンがオンの状態](/images/books/web-zukuri/bl_devToolBox-02.png)

「何故 カラーモード切り替えのトグルボタン１つでなく、その前段階である、機能をオンにするボタンが必要なのか？」についてですが、それはデフォルト状態であるブラウザ設定の場合を検証できるようにするためです。

本テーマでは CSS の `@media (prefers-color-scheme: dark) {...}` を使ったカラーモードを前提に設計しています。

完全に JavaScript でモードを制御し、ブラウザの設定を無視してカラーモードを提供する方法もあります。ですが、「ユーザーがブラウザ設定で決めた内容に従えるのならそちらの方が望ましいはず」と考え、採用しませんでした。

そのため、検証用となる開発ツールには「ツールの機能を実行しない、デフォルト（ブラウザ設定）の状態」も確認できるようにする必要があります。

#### カラーモード切り替えトグルボタン

このトグルボタンには、「ダークモード」のラベルをつけ、オンにするとダークモードに切り替わります。逆にオフにすると、ライトモードになります。これは強制的であり、ブラウザで設定した内容を無視します。

![ブラウザ設定と手動設定を対比し、手動側が優先していることを説明する図](/images/books/web-zukuri/about-color-mode-03.png)
*ブラウザ設定は「ライト」の状態で、ダークモードに上書き*

### フォントサイズ切り替え

フォントサイズ切り替えのコントロールですが、結論から先に言うと **１つのトグルボタン（チェックボックス）と、１つのスライダー**、それから１つの`output` が必要です。

1. 自動（ブラウザ設定）から手動（JavaScriptによる上書き）に切り替えるためのトグルボタン
2. フォントサイズを変更するスライダー（10〜32 まで、ステップ数は 1）
3. ２番の内容をユーザーに表示するための `output`

#### フォントサイズ変更の手動切り替えのトグルボタン

このトグルボタンがオフの状態の時には、ブラウザ設定の状態を優先します。また、フォントサイズに関する諸々のコントロールは不要なので非表示にしておきます。

![トグルボタンがオフの状態](/images/books/web-zukuri/bl_devToolBox-01.png)

トグルボタンがオンになると、JavaScript の内容を優先、ブラウザ設定は無視するようにします。また、フォントサイズを変更するためのコントロールを表示します。

![トグルボタンがオンの状態](/images/books/web-zukuri/bl_devToolBox-03.png)

こちらのボタンを設けている理由は、カラーモード手動切り替えのトグルボタンと同じです。デフォルト状態であるブラウザ設定の場合を確認するため、機能で上書きする/しない を切り替えできるようにしています。

#### スライダー

このスライダーは 10 〜 32 の間で、1 ずつ変更することができます。スライダーを変更すると、値を `px` 単位にしてフォントサイズを変更します。デフォルト値は `16px` です。

![スライダー調整 32 とその時の画面](/images/books/web-zukuri/about-font-size-02.png)

#### 値表示用 `output`

これはスライダーの値をユーザーに見せるためのものです。

![output に赤枠つけた画像](/images/books/web-zukuri/bl_devToolBox-04.png)

## まとめ

このページでは、検証ツールを画面内にどう表示させるかについて考えました。  
次項では、ここで考察した内容を基に実装していきます。
