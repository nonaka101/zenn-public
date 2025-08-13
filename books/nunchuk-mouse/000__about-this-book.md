---
title: "🔰この本について"
---

## この本について

本書は Arduino の学習過程で作成した、下記リポジトリ について扱います。リポジトリでは、コントローラー（Wii ヌンチャク）をマウスとして機能させるためのコードを管理しています。

https://github.com/nonaka101/nunchuk-mouse

本書は備忘録としての位置づけから、下記の事柄について取り扱っています。

- リポジトリの使い方、操作方法など
- コードを作る前段階となる、設計についての考え
- コードの作成段階で、何に注目し、どう対処したか

## 本書の構成

本書は 4 章構成となっています。

1. [概要](#概要)
2. [使い方](#使い方)
3. [設計](#設計)
4. [コーディング](#コーディング)

### 概要

本項がこれにあたります。

本書やリポジトリに関する大まかな説明、注意事項、ライセンス等を説明します。

### 使い方

『[📁使い方](./100__how-to-use)』の章では、リポジトリの使い方を説明します。

:::message

この章では、特段の知識は必要としません。

:::

### 設計

『[📁設計](./200__architecture)』の章では、コードを作成する前段階である設計部分を説明します。

- 物理面となる、コントローラーの機能（C, Z ボタンやスティックなど）
- ソフト面となる、マウス機能（カーソルや左右中ボタンなど）

上記 2 つを大まかに解説した後、そこから両者を組み合わせる際の問題点を見ていきます。

- 物理機能が不足しており、単純に割り当てることが困難である状態
- 加速度情報は、マウス機能としては扱いづらい問題

最後に「モード切替」の考えを説明し、現状抱えている問題に対し どう対処していくかを考えていきます。

- 加速度情報からコントローラーの姿勢を算出し、モード切替に利用
- 実際に運用できるよう、加速度情報に平滑化処理を施す
- ユーザーごとの好みに対応できるよう、調整可能な設定項目を設ける

:::message

この章では、原則として特段の知識は必要としません。

ただし一部で Arduino のライブラリについてや関数についての話が出てくるので、プログラミングについての基礎的な知識があると、より理解しやすいかもしれません。

:::

### コーディング

『[📁コーディング](./300__coding)』の章では、[設計](#設計)で説明した内容を基に、コードとして実装していきます。

:::message

この章では、実際にコーディングを行う関係上、Arduino 言語の理解が前提となります。

プログラミング的な話では、下記の要素についての「基礎的な理解」が必要となります。

- 変数、定数、列挙体、構造体、関数など
- 条件分岐やループなどの基本的な構文

:::

## ライセンス

ここでは、リポジトリ及び本書のライセンスについて説明します。

### リポジトリのライセンス

[リポジトリ](https://github.com/nonaka101/nunchuk-mouse)内の `README` に記載している通り、MIT ライセンスとなっています。

### 本書のライセンス

本書の記事、画像に関しては[クリエイティブ・コモンズ 表示 4.0 国際](https://creativecommons.org/licenses/by/4.0/deed.ja)に準拠するものとします。コードに関しては、リポジトリ同様 MITライセンスとします。

Copyright (C) 2024 nonaka101

## 参考文献

下記は 本書を作成するにあたり、参考にした資料等になります。

### I2C関係

https://ja.wikipedia.org/wiki/I2C

https://www.nxp.com/docs/ja/user-guide/UM10204.pdf

### Arduino

https://docs.arduino.cc/

http://www.musashinodenpa.com/arduino/ref/

https://www.oreilly.co.jp/books/9784814400232/

http://www.sotechsha.co.jp/pc/html/1219.htm

### ヌンチャク関係

https://wiibrew.org/wiki/Wiimote/Extension_Controllers/Nunchuck

https://github.com/madhephaestus/WiiChuck

https://github.com/todbot/wiichuck_adapter

https://github.com/rkrishnasanka/ArduinoNunchuk

### 加速度情報からデバイス向きを判定

https://qiita.com/kobayashi_ryo/items/48db56e62f7a76532d38

空間ベクトルの為す角を算出する式については、追加で下記を参考にしています。

https://mathwords.net/bekutorunasukaku

https://w3e.kanazawa-it.ac.jp/math/category/vector/henkan-tex.cgi?target=/math/category/vector/naiseki-wo-fukumu-kihonsiki.html&pcview=2
