---
title: "概要①：この本について"
---

## 本書の趣旨

本書は、簡易的な電卓を作成した際のメモを整理したものです。
コードは JavaScript を使ってますが、設計に関する部分は他の言語でも使えると思います。

本書で説明する電卓は、下記サイトにて動作を確認することができます。

https://nonaka101.github.io/jig-a/

### 参考文献など

ここでは、作成に際し参考にさせていただいたサイト等を掲載しております。

#### JavaScript の小数計算について

https://zenn.dev/riya_amemiya/articles/5b50a5ca4d8997

JavaScript は小数計算をする際に丸め誤差が生じます。これを解決する方法として、上記 記事が大いに参考になりました。

#### 状態遷移について

[Calculator HSM | Florida State University](https://www.cs.fsu.edu/~lacher/courses/COP4380/lectures/QP1/slide08.html)

電卓をモデルとした HSM(*Hierarchical State Machine*)についてのページのようです。電卓の仕組みについて調べている際、上記ページにある画像を見つけ、状態遷移による設計を思いつきました。

### ライセンスについて

本書の記事、画像に関しては[クリエイティブ・コモンズ 表示 4.0 国際](https://creativecommons.org/licenses/by/4.0/deed.ja)に準拠します。

本書内のコード（マークアップ言語を含む）に関しては、[MIT License](https://opensource.org/license/mit) とします。
