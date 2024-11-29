---
title: "Chromebookで環境構築：Java" # 記事のタイトル
emoji: "🏗" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["java", "chromebook", "vscode"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## はじめに

本記事は、Chromebook で Java の開発環境を作る過程を説明していくものです。VSCode の拡張機能を使いつつ、なるべく手軽に導入できる方向を目指します。

### 前提となる条件

本記事は、Chromebook に Linux 開発環境が構築済みであることが前提となります。また Linux 側で VSCode がインストールされた状態で、日本語化や GitHub との連携といった諸々の設定が済んだ状態を想定し、話を進めていきます。

:::message

このあたりは調べると 丁寧にわかりやすく説明してくれる記事が たくさん見つかると思います。そのため本記事では省略させていただきます、ご了承ください。

:::

なお本記事は 2024年 11月に作成したものです。当時の ChromeOS のバージョンは `131.0.6778.96`（Official Build）（64 ビット）、VSCode のバージョンは `1.95.3` となります。

## 導入手順

ここからは Java 環境を作っていく手順を説明していきます。ここで使っているスクリーンショットは、Linux 開発環境を新規に作り最低限の設定のみ整えた状態で撮ったものとなります。

環境構築の手順としては、下記のような流れとなります。

1. 拡張機能のインストール
2. JDK のインストール
3. `settings.json` に JDK へのパスを渡す

### 拡張機能のインストール

拡張機能タブの検索欄に `java` と入力すると、結果一覧に Microsoft の **Extension Pack for Java** が見つかります。「インストール」ボタンを押すと、必要となる諸々の拡張機能とセットでインストールされます。

![VSCodeの検索結果画面。検索結果最上段にMicrosoftの提供する拡張機能が掲載されている。この機能のインストールには追加で複数の拡張機能がパックでインストールされることが記されている](/images/articles/setup-java-on-chromebook/setup-01.png)

インストールが完了すると、下図の画面が表示されます。Java の開発環境を構築する手順や確認方法が紹介されており、「まず最初に **JDK**(Java Development Kit) をインストールしてください」と促されています。

![拡張機能のインストール後に画面遷移した状態。Java の開発環境を整えるための手順やコマンド、JDKのインストールに必要なリンク等が説明されている。JDKのバージョンを 8, 11, 17, 21 と最新のモノを含めて選択することができる](/images/articles/setup-java-on-chromebook/setup-02.png)

今回はこの方法は使わず、別の方法で進めていきます。

:::message

後述しますが、本記事では `apt` 経由でのインストールとなるため、相対的に古いバージョンとなってしまいます。より新しい JDK を使いたい場合は、画面に掲載されている手順に従い、必要なものをダウンロードしてください。

その際は、ダウンロードした圧縮データ（本記事掲載時点だと `OpenJDK21U-jdk_〜.tar.gz`）を任意の箇所に展開したり、`alternatives` でコマンドとして紐づけしたりといった作業が必要となるはずです。今回は手軽に導入できることを目的としているため、別の方法を使うことにします。

:::

#### JDKとは

JDK（Java Development Kit：Java開発キット）は、Java で開発を進めていく際の諸々の要素がセットになったものです。主に下記のものがセットになっています。

- コンパイラ、デバッガ
- **JRE**（Java Runtime Environment：Java実行環境）
  - **JVM**（Java Virtual Machine：Java仮想マシン）
  - API

##### コンパイラとデバッガ

JavaScript や Python, PHP のようなインタプリタ（ソースコード等を逐次解釈しながら実行するプログラム）と違って、Java はコンパイラ（あらかじめソースコードをマシン語へと変換しておくプログラム）が必要な言語と言えます。

JDK にはソースコードである `.java` をマシン語に変換するためのコンパイラや、ソースコードの挙動を確認するためのデバッガが付属されています。

##### JREとは

Java の特徴の 1 つに「**プログラムは仮想マシン上で実行される**」点があります。これによって セキュリティ性を確保し、そして特定のプラットフォームへの依存性を抑えることができます。この**仮想マシン**と**プログラムを動かすための API** をまとめたのが、**JRE**（Java Runtime Environment：Java実行環境）です。

:::details 仮想マシンに関する追加解説

この Java の特徴について、もう少し解説を加えます。

同じコンパイル作業が必要な言語として、C言語があります。どちらもマシン語に変換されますが、C言語と Java が違うのは**別のプラットフォームで実行できるか**という点です。

C 言語の場合、Windows で自身用にコンパイルしたプログラムは、Linux に持っていっても動かすことができません。コンパイルの段階で そのプラットフォーム固有のものとして調整されるためです。

一方の Java は、同じマシン語にコンパイルする手順を踏みますが、その出力は **Javaバイトコード** という Java 仮想マシン上で解釈できる形式となります。

コンパイルしたプログラムは、実行時に仮想マシンに送られます。そしてこの仮想マシンは、各プラットフォームに対応した形でネイティブコードに変換できるようになっています。なので、Windows で作ったプログラムを Linux に持っていっても、同じように実行することが可能となるわけです。

:::

#### JDK未インストール時の挙動

現段階（[拡張機能のインストール](#拡張機能のインストール)）では、まだ JDK がインストールされていない状態です。この時点で Java プロジェクトを作成した場合、どうなるかを試してみます。

まず、コマンドパレットから `>java` で **Create Java Project** を実行します。ここではビルドツールを用いない最も単純な構成 `No build tools` で新規プロジェクトを作成してみます。

![VSCodeのコマンドパレットにjavaと入力し、候補一覧から Create Java Project を選択している図](/images/articles/setup-java-on-chromebook/setup-03.png)

一応この段階でもプロジェクト作成自体は通るので、下図のようにプロジェクト名を聞かれます。

!["Input a Java project name" とのラベルが表示され、テキスト入力欄に "new-project" と打ち込んでいる図](/images/articles/setup-java-on-chromebook/setup-04.png)

これで指定の場所に新規プロジェクトが作成されました。

```text:新規プロジェクト作成時の構成
new-project/
├── .vscode/
│      └── settings.json
├── lib/
├── src/
│      └── App.java
└── README.md
```

:::details 各ファイルの詳細

```json:settings.json
{
  "java.project.sourcePaths": ["src"],
  "java.project.outputPath": "bin",
  "java.project.referencedLibraries": [
    "lib/**/*.jar"
  ]
}
```

```java:App.java
public class App {
  public static void main(String[] args) throws Exception {
    System.out.println("Hello, World!");
  }
}
```

```md:README.md
## Getting Started

Welcome to the VS Code Java world. Here is a guideline to help you get started to write Java code in Visual Studio Code.

## Folder Structure

The workspace contains two folders by default, where:

- `src`: the folder to maintain sources
- `lib`: the folder to maintain dependencies

Meanwhile, the compiled output files will be generated in the `bin` folder by default.

> If you want to customize the folder structure, open `.vscode/settings.json` and update the related settings there.

## Dependency Management

The `JAVA PROJECTS` view allows you to manage your dependencies. More details can be found [here](https://github.com/microsoft/vscode-java-dependency#manage-dependencies).
```

:::

この状態で `App.java` を選択すると、`Run Java`, `Debug Java` が有効になり実行できるようになります。実行してみると下図のようなエラーが表示され、コンパイルに必要な JDK が存在していないことがわかります。

![エラーメッセージ「プロジェクトのコンパイルに必要なJDKをダウンロードし、インストールしてください。ランタイム設定で、プロジェクトに異なるJDKを使うことができます」](/images/articles/setup-java-on-chromebook/setup-05.png)

### JDKのインストール

[JDK未インストール時の挙動](#jdk未インストール時の挙動)の項目でJDKが必要なことが確認できたので、ここで JDK をインストールしていきます。[拡張機能のインストール](#拡張機能のインストール)の項目にあった方法でなく、ここでは `apt` 経由でのインストールを行います。

#### ターミナル上での確認

ターミナルを起動し、`java` 及び コンパイラ `javac` が存在しているかを確認してみます。

```bash:Javaの登録確認
username@penguin:~$ java -version
-bash: java: command not found
username@penguin:~$ javac -version
-bash: javac: command not found
```

本記事では Linux 開発環境を新規に作成した関係から、Java に関するインストールがされていないことが確認できました。

#### default-jdkのインストール

次に `default-jdk` を `apt` 経由でインストールします。`default-jdk` に関する説明は下記のとおりです。

```bash:default-jdkの説明
username@penguin:~$ apt search default-jdk
Sorting... Done
Full Text Search... Done
default-jdk/stable 2:1.17-74 amd64
  Standard Java or Java compatible Development Kit
```

```bash:default-jdkのインストール
username@penguin:~$ sudo apt install default-jdk
```

:::details インストール時の詳細

JDK のインストール時に JRE（`default-jre`）も含まれていること、`apt` 経由なので[拡張機能のインストール](#拡張機能のインストール)と違い OpenJDK 17 と古いことがわかります。

```bash
username@penguin:~$ sudo apt install default-jdk
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  ca-certificates-java default-jdk-headless default-jre default-jre-headless fonts-dejavu-extra java-common libatk-wrapper-java libatk-wrapper-java-jni libgif7 libice-dev libpcsclite1
  libpthread-stubs0-dev libsm-dev libx11-dev libxau-dev libxcb1-dev libxdmcp-dev libxt-dev openjdk-17-jdk openjdk-17-jdk-headless openjdk-17-jre openjdk-17-jre-headless x11proto-dev
  xorg-sgml-doctools xtrans-dev
Suggested packages:
  libice-doc pcscd libsm-doc libx11-doc libxcb-doc libxt-doc openjdk-17-demo openjdk-17-source visualvm libnss-mdns fonts-wqy-microhei | fonts-wqy-zenhei fonts-indic
The following NEW packages will be installed:
  ca-certificates-java default-jdk default-jdk-headless default-jre default-jre-headless fonts-dejavu-extra java-common libatk-wrapper-java libatk-wrapper-java-jni libgif7 libice-dev
  libpcsclite1 libpthread-stubs0-dev libsm-dev libx11-dev libxau-dev libxcb1-dev libxdmcp-dev libxt-dev openjdk-17-jdk openjdk-17-jdk-headless openjdk-17-jre openjdk-17-jre-headless
  x11proto-dev xorg-sgml-doctools xtrans-dev
0 upgraded, 26 newly installed, 0 to remove and 0 not upgraded.
Need to get 122 MB of archives.
After this operation, 290 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
```

:::

#### 再度ターミナル上での確認

`java` 及び コンパイラ `javac` が存在しているかを、再度確認してみます。

```bash:java及びjavacが登録されているのを確認
username@penguin:~$ java -version
openjdk version "17.0.13" 2024-10-15
OpenJDK Runtime Environment (build 17.0.13+11-Debian-2deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.13+11-Debian-2deb12u1, mixed mode, sharing)
username@penguin:~$ javac -version
javac 17.0.13
```

### `settings.json`の編集

ここまでで JDK をインストールすることができました。実際はこの段階で `App.java` の実行ができてしまいます。ターミナルで `java` 及び `javac` も登録済みであることが確認できましたが、VSCode はそれでコンパイルを通すことができるようです。

![App.java を実行した図](/images/articles/setup-java-on-chromebook/setup-06.png)
*ターミナル上で、コードに指示した "Hello World" の出力を確認*

`settings.json` を使えば JDK のバージョンを指定するなどが可能です。ここではその方法を説明します。

#### 設定タブを開く

メニューの「ファイル」から「ユーザー設定」→「設定」へ移動し、設定タブを開きます。「設定の検索」欄に `java:home` と入力すると、下図にあるように `Java › Jdt › Ls › Java: Home` の項目を見つけることができます。

![VSCode設定タブに "java:home" と入力した状態。検索結果の最上段に "Java › Jdt › Ls › Java: Home" がある](/images/articles/setup-java-on-chromebook/setup-07.png)

> **Java › Jdt › Ls › Java: Home**
> Specifies the folder path to the JDK (17 or more recent) used to launch the Java Language Server. This setting will replace the Java extension's embedded JRE to start the Java Language Server.
> On Windows, backslashes must be escaped, i.e.
> "java.jdt.ls.java.home":"C:\\Program Files\\Java\\jdk-17.0_3"

要するに、Java言語サーバーを立てるために必要な JDK（17以降）の**フォルダパス**を指定する必要があるわけです。Windowsの場合はエスケープ処理が必要とありますが、Chromebook想定の今回は無視して大丈夫です。

「`settings.json` で編集」をクリックすると、項目が登録された状態でファイルが開きます。ここにパスを渡してあげれば完成です。

```json:settings.json
{
  "java.jdt.ls.java.home": "",
}
```

#### JDKへのパスを調べて入力

ターミナルで java がインストールされているのを確認できましたが、その場所が必要です。パスを確認したい場合は、例えばシステムが使ってるプログラムのバージョン管理を見るための `update-alternatives` を使って見ることができます。

```bash:JDKのインストール先のパスを確認
username@penguin:~$ sudo update-alternatives --config java
There is 1 choice for the alternative java (providing /usr/bin/java).

  Selection    Path                                         Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-17-openjdk-amd64/bin/java   1711      auto mode
  1            /usr/lib/jvm/java-17-openjdk-amd64/bin/java   1711      manual mode

Press <enter> to keep the current choice[*], or type selection number:
```

つまりターミナルで `java` を実行した場合、それは `/usr/lib/jvm/java-17-openjdk-amd64/bin/java` を見に行っていることを意味してます。今回必要なのは **JDK のフォルダパス**なので、`settings.json` に入力するのは `/usr/lib/jvm/java-17-openjdk-amd64` の部分となります。

```json:settings.json
{
  "java.jdt.ls.java.home": "/usr/lib/jvm/java-17-openjdk-amd64",
}
```

:::details JDKフォルダの中身

JDKフォルダ内を見てみると、`bin` 内に `java`, `javac` などがあり、`lib` や `man` があることが確認できます。

```bash:JDKフォルダの中
username@penguin:/usr/lib/jvm/java-17-openjdk-amd64$ ls -l
合計 8
drwxr-xr-x 1 root root  330 11月 28 19:50 bin
drwxr-xr-x 1 root root  212 11月 28 19:50 conf
lrwxrwxrwx 1 root root   42 10月 18 05:50 docs -> ../../../share/doc/openjdk-17-jre-headless
drwxr-xr-x 1 root root  140 11月 28 19:50 include
drwxr-xr-x 1 root root 2642 11月 28 19:50 jmods
drwxr-xr-x 1 root root 1942 11月 28 19:50 legal
drwxr-xr-x 1 root root 1252 11月 28 19:50 lib
drwxr-xr-x 1 root root    8 11月 28 19:50 man
-rw-r--r-- 1 root root 1230 10月 18 05:50 release

username@penguin:/usr/lib/jvm/java-17-openjdk-amd64$ ls bin/
jar        javac    jcmd      jdeprscan  jhsdb   jlink  jpackage    jshell  jstatd       serialver
jarsigner  javadoc  jconsole  jdeps      jimage  jmap   jps         jstack  keytool
java       javap    jdb       jfr        jinfo   jmod   jrunscript  jstat   rmiregistry
```

:::

変更をかけた際には、下図のように再起動を促されます。

![変更時にVSCodeに表示されるポップアップメッセージ："Java Language Server Configuration is changed, please reload Visual Studio Code."](/images/articles/setup-java-on-chromebook/setup-08.png)

これで VSCode に Java 言語サーバーを立てるうえで必要な JDK フォルダへのパスを渡すことができました。下図のように `App.java` が問題なく動作していることを確認できます。

![App.java を実行している図、ターミナルタブにコードで指示したように "Hello World!" と出力されている](/images/articles/setup-java-on-chromebook/setup-09.png)

## まとめ

本記事では Chromebook 上に立てた Linux 開発環境で、VSCode の拡張機能を使いつつ Java の開発環境を構築する流れについて説明しました。

### Java Projectsペインについて

ここまでの手順の中で、VSCode のエクスプローラー欄に **JAVA PROJECTS** のペインも登録されていることが確認できると思います。物理的なディレクトリ構造を管理しているエクスプローラーと違い、こちらのペインは下図に示されるように、Java プロジェクトに沿った形の管理が可能となります。

![Java Projectペイン、ソースコードやライブラリ等の観点から管理できる](/images/articles/setup-java-on-chromebook/setup-10.png)

### 参考文献

https://qiita.com/keomo/items/a814a36be95033c14207

https://www.shoeisha.co.jp/book/detail/9784798180946
