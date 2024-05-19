---
title: "📄A103：スクリーンリーダー"
---

## このページについて

ここでは検証に使用するスクリーンリーダーについて、説明していきます。

## 種類別スクリーンリーダー

昨今、デバイスに標準でスクリーンリーダーが搭載されることが多くなりました。ここでは主要なものを取り上げていきます。

### iPhone : VoiceOver

「設定」→「アクセシビリティ」にある「**VoiceOver**」というのが、iPhoneでのスクリーンリーダーです。検証の際には、「アクセシビリティ」下にある「ショートカット」を VoiceOver に設定しておけば、*ホームボタンのトリプルクリック*でオン/オフを切り替えられるので便利です。

![VoiceOver で、本サイトの ReadMe を読み上げている画面](/images/books/web-zukuri/iphone-voiceover-01.png =350x)

「現状について」の箇所に黒枠が入っていますが、ここが読み上げている場所です。

![VoiceOver の上下スワイプの設定切替で文字にしている](/images/books/web-zukuri/iphone-voiceover-02.png =350x)

二本指ジェスチャーで回し捻るようにすると、上図のようにスワイプでの移動単位を切り替えたりもできます。

### Mac : VoiceOver

![Mac の VoiceOver で本サイトのReadMeを読み上げている画面](/images/books/web-zukuri/macbook-voiceover-01.png)

MacでもVoiceOverという名前です。上図では、画面下部に読み上げているテキストをボックスとして表示されていることもわかります。

Macでは **VO修飾キー** を使った操作など、キーボードでの操作を覚える必要があります。そのため、「設定」→「アクセシビリティ」→「VoiceOver」の項目では、基本操作を練習することができます。

![VoiceOverの練習画面１、修飾キーについて](/images/books/web-zukuri/macbook-voiceover-02.png)

![VoiceOverの練習画面２，要素操作](/images/books/web-zukuri/macbook-voiceover-03.png)

### Android : TalkBack

![TalkBackで本サイトのReadMeを読み上げている画面](/images/books/web-zukuri/android-talkback-01.png =400x)

Androidでは、**Talkback**というものがスクリーンリーダーになります。

こちらは、使用する指の本数によって様々な操作が可能となっています。下図は、三本指で左右にスワイプすることで、上下スワイプでの移動レベル（文字、単語、段落、など）を切り替えている図です。

![上下スワイプを設定している状態](/images/books/web-zukuri/android-talkback-02.png =400x)

### ChromeBook : ChromeVox

![ChromeVoxを起動した状態で本サイトのReadMeを読み上げている状態](/images/books/web-zukuri/chromebook-chromevox-03.png)
*画面上部に読み上げ内容がテキストでも表示されている*

ChromeBookには、ChromeVoxというものが搭載されています。電源や時刻のスペース（クイック設定パネル と呼ばれます）から「ユーザー補助」→「ChromeVox（音声フィードバック）」で切り替えることができます。

![ChromeBookの画面右下の拡大画面](/images/books/web-zukuri/chromebook-chromevox-01.png)
*画面右下の時計（クイック設定パネル）を開いた状態*

![ユーザー補助一覧、一番上にChromeVoxの項目がある](/images/books/web-zukuri/chromebook-chromevox-02.png)
*上図から「ユーザー補助」を選択した状態*

ChromeVox では、「スピーチ（ピッチ調整など、読み上げ関係の操作）」「見出し（ページ内の各見出しへジャンプできる）」「リンク（ページ内の各リンク要素へジャンプできる）」などの操作を、視覚的にもわかりやすく表示してくれています。

![ChromeVoxのスピーチタブ](/images/books/web-zukuri/chromebook-chromevox-04.png)

![ChromeVoxの見出しタブ](/images/books/web-zukuri/chromebook-chromevox-05.png)
*[ReadMe](https://nonaka101.github.io/web-zukuri/single.html)の見出しが一覧化されている*

![ChromeVoxのリンクタブ](/images/books/web-zukuri/chromebook-chromevox-06.png)
*[ReadMe](https://nonaka101.github.io/web-zukuri/single.html)のリンクが一覧化されている*

### Windows : ナレーター

Windows に標準搭載されているものが ナレーター です。

![ナレーターを起動した状態で、本サイトのReadMeと、ナレーター起動画面を並べた状態](/images/books/web-zukuri/windows-narrator.png)

### Windows : NVDA

https://www.nvda.jp/

NVDA はオープンソースの Windows スクリーンリーダーで、日本語版があります。

### Windows : NetReaderNeo

[株式会社高知システム開発](https://www.aok-net.com/)の製品に、「PC-Talker Neo」というスクリーンリーダーや、「NetReader Neo」という音声ブラウザがあります。

いくつかの制限がありますが、クリエイター（晴眼者）向けの無料版があり、これで実際の挙動を確認しながら作業を行うことができます。

![NetReader Neo の画面](/images/books/web-zukuri/windows-net-reader-neo-01.png)

![右クリックした状態](/images/books/web-zukuri/windows-net-reader-neo-02.png)
*ジャンプなどの様々な機能が搭載されている*

:::message alert

セキュリティソフトによってはインストール時にブロックされることがあります。

なので、利用に際しては ご自身の判断の上で行ってください。

:::

## まとめ

本項では、主要なスクリーンリーダーについて説明をしました。
