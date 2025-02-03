---
title: "WCAG備忘録：国際的な立ち位置" # 記事のタイトル
emoji: "🌏" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["wcag", "a11y", "webaccessibility"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## はじめに

ここでは「**WCAG が海外でどういった影響を与えているのか**」を、規則や法律といった観点から見ていきます。記事作成段階では、とりあえず情報が集めやすい アメリカと EUを取り扱います。

なお、各国や各地域での Webアクセシビリティに関する法律やポリシーといったものについては、WAI[^1]が公開しているリストが参考になると思いますので、そちらを参照ください。

[^1]: 正式名は Web Accessibility Initiative 。W3C（*World Wide Web Consortium*）内にある組織で、障害を持つ方の Web アクセシビリティ向上を目的としている

https://www.w3.org/WAI/policies/

:::message alert

本記事は備忘録という立場を取っています、情報の精度は保証できない旨をご了承ください。

翻訳の間違い等も考えられますので、参考にしたソース等は載せるようにしております。可能であれば、そちらをご参照ください。

:::

## 米国のウェブアクセシビリティに関する規則

### Section 508 Amendment to the Rehabilitation Act of 1973：リハビリテーション法第508条改正

https://en.wikipedia.org/wiki/Section_508_Amendment_to_the_Rehabilitation_Act_of_1973

『[リハビリテーション法](https://en.wikipedia.org/wiki/Rehabilitation_Act_of_1973)（1973年）』は、障害者を主な対象とした職業リハビリテーションを推進するための法律です。第501条では 連邦政府（行政）による積極的差別是正措置（*affirmative action*）について、第504条では政府による障害者差別を禁止する規定についてが記されています。

そして 第508条は、連邦政府を代表とする公的機関を対象に、彼らが調達する電子機器や情報技術が、障害を持つ人々にも利用できることを義務づけた法律です。成立から複数回 改正が行われ、現在はウェブ上の情報やアプリケーションも対象となっています。

この法律では、基本的に連邦政府で提供されるウェブサイトについて **WCAG2.0 レベルAA** での準拠を義務付けています。

https://www.access-board.gov/ict/

### Americans with Disabilities Act of 1990：障害を持つアメリカ人法

https://en.wikipedia.org/wiki/Americans_with_Disabilities_Act_of_1990

https://en.wikipedia.org/wiki/ADA_Amendments_Act_of_2008

頭文字を取って ADA とも呼ばれています。2008年には ADA 改正法 が制定されました。

ADA は公民権法の一環であり、心身に障害をもつ人々の権利に関する包括的な法律となっています。

ベースとなるのは[リハビリテーション法第504条](https://en.wikipedia.org/wiki/Section_504_of_the_Rehabilitation_Act)です。この法律は連邦政府による障害者差別を禁止する規定でしたが、ADA ではこれを拡張する形で、全州 及び 全地方自治体を対象とし、更に公共的な役割を持つ民間企業等も対象となります。

ADA では、障害を持つ人々に対する差別を禁止しています。これは**直接的な差別を禁じている**だけでなく、公共的な場所や交通機関の利用において**健常者と差があってはならない**ということも意味します。

昨今では日常生活でウェブサイト等の利用は当たり前のものとなっており、利用できないことによる不利益が大きくなっています。そのため、ADA は その対象（州政府、地方自治体、公的役割を持つ民間企業 など）に対し、Web アクセシビリティの確保を要求しています。

https://www.ada.gov/resources/web-guidance/

## EUのウェブアクセシビリティに関する規則

### Web Accessibility Directive：ウェブアクセシビリティ指令

https://en.wikipedia.org/wiki/Web_Accessibility_Directive

https://eur-lex.europa.eu/eli/dir/2016/2102/oj

原典では "the accessibility of the websites and mobile applications of **public sector bodies**" とあるように**公的機関を対象に**、ウェブサイトとモバイルアプリケーションのアクセシビリティについて規定がされています。アクセシビリティ要件を規定する欧州規格として **EN 301 549**があり、この中で **WCAG2.0 レベルAA** に準拠した要件が求められています。

後述する『欧州アクセシビリティ法（2019年）』もそうですが、各加盟国が個別で制定するのでなく、EU 加盟国全体に適用される共通規則としている理由について、下記のような意図があるのだと思われます。

- 各加盟国で個別対応となると、それぞれで似たような国内規定を採択する必要が生じてしまう
- 個別で規定した場合、加盟国間で法律や規則、行政規定といった部分で格差が拡大することに繋がってしまう
- 『障害者の権利に関する条約[^2]』に対し、EU は集団として批准しており、一体となって促進していく必要がある

[^2]: 2006 年に国連で採択された人権条約。日本も 2014 年に締結しており、[外務省にて和文が掲載されている](https://www.mofa.go.jp/mofaj/fp/hr_ha/page22_000899.html)

#### EN 301 549

https://en.wikipedia.org/wiki/EN_301_549

ETSI[^3] が発行している欧州規格の1つで、"*Accessibility requirements suitable for public procurement of ICT products and services in Europe*"（欧州における ICT 製品およびサービスの公共調達に適したアクセシビリティ要件）を扱っています。

[^3]: European Telecommunications Standards Institute：欧州電気通信標準化機構

ここではウェブ以外にも様々な基準を扱っています。ウェブコンテンツに関しては **WCAG2.0 レベルAA** が、変更なし（≒そのままの形）で適用されています。

### European Accessibility Act：欧州アクセシビリティ法

https://en.wikipedia.org/wiki/European_Accessibility_Act

https://eur-lex.europa.eu/eli/dir/2019/882/oj

原典では *the accessibility requirements for products and services*（製品やサービスに必要なアクセシビリティ）とあるように、この法律は 様々な製品やサービスを対象とし、アクセシビリティの向上を目的としています。

対象範囲は多岐に渡り、**おおよそ生活上で登場する電子通信全般に関わる製品やサービス**と思っていいでしょう。例えば コンピューティングシステム（パソコン, スマートフォン, タブレット等）, ATM, 発券機, 電子書籍リーダー などがあります。物理的なものに限らず、テレビ放送やウェブ上のオンデマンド配信、飛行機や電車といった交通情報提供サービス、金融サービス、電子商取引といったサービス関係も、この法律の対象となります。

適用範囲は公的セクターだけでなく、民間にも適用されます。製品やサービスを製造、輸入、流通させる事業者に対しては、EU 市場内で消費者が一定のアクセシビリティの恩恵に預かれるように義務が課されています。

影響範囲が民間にも及ぶためなのか、実施に至るまでに段階が設けられているようです。公布は 2019 年ですが、加盟国は（運用のための国内規定の採択含め）2025 年 6 月 28 日までに適用できるように定められています。またサービス提供者に対しては、そこから 5 年を限度に経過措置を認められていたりもしているようです。

具体的なアクセシビリティ要件については、[附属書](https://eur-lex.europa.eu/eli/dir/2019/882/oj#anx_I)に規定がなされています。ウェブコンテンツに関しては **WCAG2.0 派生の規定**となっています。といってもその中身を見てみると「複数の感覚器官を通じて利用可能であること」「理解可能な方法で提示されていること」「ユーザーが知覚可能であること」とあることから、基本的には WCAG と同様であると考えていいでしょう。

## まとめ

### その他参考文献

記事内に載せていない参考文献を、ここに載せております。

https://www8.cao.go.jp/shougai/suishin/tyosa/h27kokusai/index.html

『EU のアクセシビリティ指令』（国立国会図書館　調査及び立法考査局）【[PDF](https://dl.ndl.go.jp/view/download/digidepo_11643920_po_02870002.pdf?contentNo=1)】
