---
title: "📄使い方：必要となる機材等"
---

## はじめに

本項では、Wiiヌンチャクをマウスとして機能させるために必要となる機材について説明します。

## 必要となる機材等

![Arduino Lepnardo, ヌンチャクコントローラー, USBケーブル, ジャンプワイヤで接続している状態。画像ではこれらに加えブレッドボードに LED と抵抗を接続している](/images/books/nunchuk-mouse/prototype-01.png)

ここでは必要となる機材等について説明します。上図は開発段階の検証機で、このような簡素な構成でも動かすことができます。

### Arduino IDE

ボードにコードを書き込むための IDE[^1] を PC にインストールしてください。

IDE は [公式ホームページ](https://www.arduino.cc/en/software/)から取得することが可能です。導入に関しての詳細は、公式サイトを参照ください。

[^1]: IDE（Integrated Development Environmen）：統合開発環境、必要となる様々な機能をまとめた開発環境のこと

### Arduinoなどの開発ボード

コードを記録し処理するための開発ボードが必要となります。

開発ボードに必要な要件としては、Arduino IDE を使ってコードを書き込めることの他に、下記の条件を満たす必要があります。

- 必須：HID 対応
- 推奨：3.3V を出力可能

本リポジトリの開発・検証にあたっては、[Arduino Leonardo](https://docs.arduino.cc/hardware/leonardo/) を使用しています。

:::message

「[⭐附録：組み込み例](./910__assembly)」ではコントローラー内部に埋め込むため、Leonardo より省サイズの [Adafruit 社 ItsyBitsy 32u4](https://www.adafruit.com/product/3675)（3V 8MHz 版）を使っています。

本書の内容とは少し異なる手順が必要となってきます。この機材を使う場合は、公式提供の[Introducing ItsyBitsy 32u4](https://learn.adafruit.com/introducting-itsy-bitsy-32u4)を参照してください。

:::

#### 必須：HID 対応

本リポジトリでは マウス機能を使うために、HID[^2] 機能をサポートしたハードウェア（開発ボード）が必要です。

[^2]: HID（Human Interface Device）：コンピューター用デバイスで人が入力する際に使うもの。マウスやキーボードなどが、これに該当する。

公式ドキュメントで、サポートしているボード一覧を確認することができます。

> HID is supported on the following boards:
>
> - Leonardo
> - Micro
> - Due
> - Zero
> - UNO R4 Minima
> - UNO R4 WiFi
> - GIGA R1 WiFi
> - Nano ESP32
> - MKR Family
>
> 『[Mouse | Arduino Documentation](https://docs.arduino.cc/language-reference/en/functions/usb/Mouse/)』より一部抜粋

#### 推奨：3.3Vを出力可能

コントローラーの電源は 3.3V となっています。そのため 3.3V を出力できるピンがついているのが望ましいです。

:::message alert

5V だと、機材への過負荷等の問題が考えられるため非推奨です。

:::

:::message

開発ボードが 5V の場合、レベルコンバーター[^3]を介する方法で対処が可能かもしれません。ただしそこまで検証できていない状態です。

:::

[^3]: レベルコンバーター：3.3V と 5V といった異なる信号の電圧を変換して通信を適正に保つためのモジュールの一種。

### ヌンチャクコントローラー

Wii ヌンチャクを 1 つ準備してください。

製造時期により「シロ」「クロ」の 2 種ありますが、設定で両方対応できるようにしています

:::message

作成に当たってはシロを使っており、クロは入手できていないため検証ができていない状態です。

:::

### 配線

単純に動作を確認するだけなら、**ジャンプ ワイヤー 4本** と **USB ケーブル**（Arduino を PC に接続するためのもの）で動かすことが可能です。

綺麗に使いたい場合は、ケーブルを分解して配線し直したり、市販のアダプターを使ったりしてください。

## まとめ

本項では、リポジトリを動かすに当たって必要となる機材について説明をしました。

- Arduino IDE
- HID対応のArduino開発ボード（Arduino Leonardo推奨、3.3V出力可能ボードが推奨）
- Wiiヌンチャクコントローラー（シロ・クロ両対応可能）
- 配線（USBケーブルに加え、ジャンプワイヤなど）

次項では、用意した機材を使って実際に機器を繋げていきます。
