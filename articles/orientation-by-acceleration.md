---
title: "加速度センサーを利用した向き判定" # 記事のタイトル
emoji: "🤳" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["arduino"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## はじめに

本記事では**加速度センサーから得られた情報を使ってデバイスの傾きを判定する**ことを主目的として考えていきます。

【2025 年 8 月 13 日追記】  
本記事の内容も含めた 書籍を公開しました。Arduino を使ってコントローラー（Wii ヌンチャク）をマウスとして利用するための諸々（設計段階からコーディングまで）を取り扱っており、本記事で扱っている「加速度情報に関する処理」もアップデートしています。ですので、よければこちらを参照ください。

https://zenn.dev/nonaka101/books/nunchuk-mouse

### 本記事の前提

本記事は、下記「WiiヌンチャクとArduinoを接続してデータを取得する」の続きとなっています。こちらも参照いただけると、本項の内容も掴みやすくなるかもしれません。

https://zenn.dev/nonaka101/articles/i2c-for-nunchuck

### 加速度センサーによる判定について

加速度センサーは文字通り「物体の加速度」を計測するためのものです。これには**物体自身が動く**ことによって生じるものの他に、**重力によって生じる**ものがあります。本記事では後者の「重力による加速度」を用いて、デバイスの傾きを判定することを考えていきます。

下図は、通常の持ち方（スティックが垂直に天井方向を指す状態）を表したものです。灰色の矢印は重力方向で、この重力加速度をセンサーは検知します。

![三軸座標上の中央にヌンチャクコントローラー。計測する向きは奥側を向いており、重力は下方向に発生している](/images/articles/orientation-by-acceleration/axis-01.png)
*デフォルトのポジション*

一方でデバイスはXYZ方向で自身の座標情報を持っています。例えば「デバイスが縦方向にあるか」を判定したい場合、Y座標のマイナス方向（白色の矢印）が計測する向きとなります。この**重力方向と計測する向きの角度**が、今回判定する上で重要な要素となります。上図の場合は、白矢印と灰矢印で およそ 90°の角度があることになります。

そして下図は、デバイスを縦方向に傾けた状態を表しています。

![先程の座標上で、コントローラーを立てた状態。計測する向き及び重力は下方向を指している](/images/articles/orientation-by-acceleration/axis-02.png)
*縦方向のポジション*

重力方向（灰矢印）は変わらず下方向を指しています。一方でデバイス座標上の計測向き（白矢印）は、デフォルトでは奥方向だったのに対し こちらでは下方向に移動しています。両者の角度はデフォルト時の 90°から 0°に近い状態まで変化したことから、「デバイスが縦方向にある」と判定できそうです。

本記事では 空間ベクトル や ベクトルの為す角 といったものを扱い、数式を織り交ぜながら考えていくことになります。しかし やっていることの本質は、上図２枚が表している**重力方向とデバイス計測方向の角度を出して、向き方向を判定する**という単純なものです。

#### 加速度情報についての追加説明

上記で加速度を使った判定方法について説明しました。ここでは いくつかのイメージ図を用いて、もう少し詳しく見ていこうと思います。

下図は FreeCAD で作ったものです。中央にある四角柱がデバイス（デフォルトのポジションを想定）で、そこから球をつけた糸を垂らした状態だとイメージしてください。

![加速度センサーの説明図](/images/articles/orientation-by-acceleration/accel-00.png)
*デバイスと計測される加速度情報*

X軸は赤、Y軸は緑、Z軸は青色で配色しています。各色の弧は、デバイスを傾けて軸を動かした際の球の挙動を表しています。

どういうことかについて、まずはX軸から見ていきます。下図は、デバイスを正面から見た図となります。

![加速度センサーの説明図](/images/articles/orientation-by-acceleration/accel-01.png)
*デバイス正面図*

デバイスを左右に傾けると、X軸（赤色）はそれに沿って動き、球も赤色の弧に沿って動くことになります。  
正位置の状態では図の通り 0 を指します。（このデバイスを持っているとして）右方向に傾けると、球はプラスの方向に移動します。逆に左に傾けると、球はマイナスの方向に移動します。

次に Y軸を見てみます。下図は、デバイスを右から見ている図です。

![加速度センサーの説明図](/images/articles/orientation-by-acceleration/accel-02.png)
*デバイス右側面図*

X軸での説明と同様、デバイスを前後に傾けると球は緑色の弧に沿って動きます。  
正位置の状態では 0 を指しています。デバイスを前方に傾ける（倒す）と、球はプラスの方向に移動します。逆に後方に傾ける（倒す）と、球はマイナスの方向に移動します。

最後に Z軸についても見てみます。下図は、デバイスを 正面と右側面の中間位置から斜めに見ている図です。

![加速度センサーの説明図](/images/articles/orientation-by-acceleration/accel-03.png)

この Z軸では、デバイスの表裏を見る役割があります。表状態（≒ スティックが天井を向いている）ではプラスを指し、裏状態（≒ スティックが床を向いている）ではマイナスを指します。中間状態（例： X もしくは Y 方向に90°傾けた状態）ではプラスマイナスの中間である 0 を指します。

:::message

最初の X および Y の計測に対し、Z 軸の意義についてわかりづらいかもしれません。この Z 軸は、**デバイスの正確なポジション**を断定するために必要な情報になります。

例えば「X 軸での値が 0 で、Y 軸での値が 0 となる」ケースを考えてみます。まず思いつくのが、これまで説明に使ってきたデフォルトのポジションです。

![デフォルトポジションの状態、X及びYは0を指している。一方、Zは1となっている](/images/articles/orientation-by-acceleration/accel-00.png)

しかし **Z 軸を考慮しない場合、同じ値となるポジションがもう一つ発生します**。それは表裏を逆にした状態です。左右にも前後にも傾いていないので、X 及び Y の値は変わらず 0 となります。

このことから、Z 軸を考慮しない場合 デバイスのポジションは 2 択になってしまうことがわかります。正確なポジションを断定するために、Z 軸の情報は必要ということです。

:::

## 理論上の部分

前提となる情報が出揃ったので、ここからは判定の際の過程を考えていきます。この段落では理論的な部分に絞って考え、「どのような手順でデバイス向きを判定していくか」を見ていきます。

### 空間ベクトルの為す角

「デバイスが縦方向にあるか」を判定するのに必要なのは、**重力方向とデバイスの向きを直線で表した際の角度**です。

![三軸座標上の中央に、ヌンチャクコントローラーが立てられた状態にある図。重力方向と計測する向きが それぞれ下方向を指している](/images/articles/orientation-by-acceleration/axis-02.png)

この角度を、2つのベクトル情報を使って計算します。すなわち重力加速度と計測したい向きを、それぞれベクトル情報としたものです。両者のベクトル（**加速度情報**と**計測向き**）を使って**空間ベクトルの為す角**を計測します。

空間ベクトルの為す角とは、2 つの 3 次元ベクトルを 1 つの平面上に とらえた際に計測できる角度を指します。これには公式があり、例えば $\overrightarrow{a} = (a_1, a_2, a_3)$, $\overrightarrow{b} = (b_1, b_2, b_3)$ のベクトルデータがある時、両者の為す角 $\theta (0\degree \leqq \theta \leqq 180\degree)$ とすると、下記の式が成り立ちます。

$$

\cos \theta = \frac{\overrightarrow{a} \cdot \overrightarrow{b}}{|\overrightarrow{a}| \, |\overrightarrow{b}|} = \frac{a_1 b_1 + a_2 b_2 + a_3 b_3}{\sqrt{a_1^2 + a_2^2 + a_3^2} \sqrt{b_1^2 + b_2^2 + b_3^2}}

$$

上記で $\cos \theta$ が算出できれば、後は角度に変換するだけです。$\arccos(\cos \theta)$ にかけてラジアンを取得し、そこに $\frac{180}{\pi}$ を掛ければ、角度を取得できます。

#### 空間ベクトルの為す角をArduino関数に

上記公式を単純にコード化すると、下記のようになります。

```c:2つの空間ベクトルから角度を算出
float angleBetweenTwoVectors(int vecX1, int vecY1, int vecZ1, int vecX2, int vecY2, int vecZ2) {
  float cosTheta = ((vecX1 * vecX2) + (vecY1 * vecY2) + (vecZ1 * vecZ2)) / (sqrt(pow(vecX1, 2) + pow(vecY1, 2) + pow(vecZ1, 2)) * (sqrt(pow(vecX2, 2) + pow(vecY2, 2) + pow(vecZ2, 2))));
  return (acos(cosTheta) * 180) / PI;
}
```

これでも動くことには動くのですが、下記の問題が考慮されていない状態です。

- いずれかがゼロベクトルの場合【角度算出が不能】
- 内積や各ベクトルの大きさの内、いずれかが 0 になった場合【角度算出が不能】
- θが 0°から 180°までという制限【$\arccos(\cos \theta)$ に影響】

それらを踏まえた上で、異常値を弾いたり調整したりしたコードが下記となります。

```c:2つの空間ベクトルから為す角を算出する関数
float angleBetweenTwoVectors(int vecX1, int vecY1, int vecZ1, int vecX2, int vecY2, int vecZ2){
  // ゼロベクトルの場合、NaN を返す
  if ((vecX1 == 0 && vecY1 == 0 && vecZ1 == 0) || (vecX2 == 0 && vecY2 == 0 && vecZ2 == 0)){
    return NAN;
  }

  // 内積とベクトルの大きさを算出
  float dotProduct = (vecX1 * vecX2) + (vecY1 * vecY2) + (vecZ1 * vecZ2);
  float magnitudeV1 = sqrt(pow(vecX1, 2) + pow(vecY1, 2) + pow(vecZ1, 2));
  float magnitudeV2 = sqrt(pow(vecX2, 2) + pow(vecY2, 2) + pow(vecZ2, 2));

  // 問題なければ cosθを算出（内積が 0 の場合、cosθ が 0 となり90°となる）
  if (dotProduct == 0) return 90.0;
  float cosTheta = dotProduct / (magnitudeV1 * magnitudeV2);

  // acos 計算用に、cosTheta の範囲を制限 (-1 <= cosTheta <= 1)
  cosTheta = constrain(cosTheta, -1.0, 1.0);

  // ラジアンを算出し、角度に変換して返す
  return (acos(cosTheta) * 180) / PI;
}
```

### 縦方向の判定のみに限定する場合

上記では「2つのベクトル情報から角度を算出する」という汎用的な関数ができました。しかし今回は「デバイスが縦方向か」という限定された条件であるため、簡略化して計算量を抑えることが可能です。

先程の式では、$\overrightarrow{a} = (a_1, a_2, a_3)$, $\overrightarrow{b} = (b_1, b_2, b_3)$ のベクトルデータを用いていました。しかし今回の縦方向に限定する場合、$\overrightarrow{b}$ については $\overrightarrow{b} = (0, -1, 0)$ で固定することができます。となると 先程の数式はもっと簡略化することができ、最終的には下記のようになります。  
※ $\theta$ は $(0\degree \leqq \theta \leqq 180\degree)$ とする

$$

\cos \theta = \frac{-a_2}{\sqrt{a_1^2 + a_2^2 + a_3^2}}

$$

:::details この数式の意味

この数式は、「**ベクトルの大きさに対して y の割合がどの程度あるか**」を意味しています。つまり、ベクトルの大きさに対して y の割合が 100% であれば 1 を算出し、無関係（0%）であれば 0 を算出します。

下図は高校数学等で見る、単位円上の $\cos \theta$ 値です。角度 $\theta$ が 0°から180°へ変化するに伴って、$\cos \theta$ は 1 から -1 へと変化します。

![コサインθのグラフ、0から180まで](/images/articles/orientation-by-acceleration/graph-of-cosine-theta.png)

ここから、2つのことが読み取れます。

- 2つの直線が一致（≒ 角度が 0°）の場合、$\cos \theta$ は 1 となる
- 2つの直線が直交（≒ 角度が 90°）の場合、$\cos \theta$ は 0 となる

その上で、下記の式を見てみます。

$$

\cos \theta = \frac{-a_2}{\sqrt{a_1^2 + a_2^2 + a_3^2}}

$$

この式において、（基本的には）分母が分子より小さくなることはありません。そのため この式は -1 から 1 の間で推移することになります。

分母はベクトルの大きさを表します。この大きさは、三平方の定理（ピタゴラスの定理）より $\sqrt{a_1^2 + a_2^2 + a_3^2}$ となります。

そして分子はそのまま y 値を意味しています（マイナス方向）。つまりこれは、ベクトルの大きさに対して（マイナス方向の）y の割合がどれだけかを示していることになります。100%の割合（$\cos \theta = 1$）なら求めたい縦方向との角度が 0°であることを意味し、0%の割合（$\cos \theta = 0$）なら縦方向との角度が 90°であることを意味します。

:::

#### 縦方向との角度をArduino関数に

**デバイスを垂直に立てた状態との角度**がわかれば、縦方向にあるかを判定できます。上記の式を単純にコード化したものが、下記となります。

```c:1つの空間ベクトルから縦方向との角度を算出
float angleToPortrait(int vecX, int vecY, int vecZ) {
  float cosTheta = (-vecY) / (sqrt(pow(vecX, 2) + pow(vecY, 2) + pow(vecZ, 2)));
  return (acos(cosTheta) * 180) / PI;
}
```

ここから異常値を弾いたり調整したりすると、下記のようになります。

```c:1つの空間ベクトルから縦方向との角度を算出
float angleToPortrait(int vecX, int vecY, int vecZ) {
  // ゼロベクトルの場合、NaN を返す
  if (vecX == 0 && vecY == 0 && vecZ == 0) return NAN;

  // Y方向が 0（cosθ が 0）の場合は、90°となる
  if (vecY == 0) return 90.0;

  // cosθの算出（acos 計算用に範囲を制限）
  float cosTheta = (-vecY) / (sqrt(pow(vecX, 2) + pow(vecY, 2) + pow(vecZ, 2)));
  cosTheta = constrain(cosTheta, -1.0, 1.0);

  // ラジアンを算出し、角度に変換して返す
  return (acos(cosTheta) * 180) / PI;
}
```

## 実機への実装部分

ここからは [前記事](https://zenn.dev/nonaka101/articles/i2c-for-nunchuck)の内容を引き継いだ上で、[理論上の部分](#理論上の部分)を実機に落とし込む作業を行っていきます。そのためには、**実機で運用する際に考慮しなければならない要素**を考える必要があります。

それらを踏まえ、最終的に「**誤差◯度以内で垂直状態を判定し、条件を満たした場合はLEDを光らせる**」という動きを実装していきます。

### 実機へ落とし込むにあたって考慮すべき要素

本項の指す「実機」とは、 [前記事](https://zenn.dev/nonaka101/articles/i2c-for-nunchuck)で扱ったヌンチャク型のコントローラーとなります。詳しくはそちらを参照いただくとして、この実機が取り扱う加速度センサーの情報は、下記のようになっています。

- 各軸データ共通して**中央が512となる符号なし10bitの情報**
- ±3g まで計測し、静止時(1g)は だいたい 200 前後の振れ幅となる
- 計測誤差は 10% 前後を想定

これらを踏まえると、下記の事柄に対処していかなければなりません。

- 各軸データの中央値を、0 を中心とした符号付きデータに置き換える
- 計測誤差の大きさに対処
  - 垂直であることを検知する角度に、一定のマージンを設ける
  - 1回あたりのブレ幅を抑えるために、平滑化処理を挟む

これらに対処しながら、「**誤差◯度以内で垂直状態を判定し、条件を満たした場合はLEDを光らせる**」という動きを実装していきます。

### 改修していくコードについて

ここから使うコードについては、前記事「[WiiヌンチャクとArduinoを接続してデータを取得する](https://zenn.dev/nonaka101/articles/i2c-for-nunchuck)」で作成したものを用います。「コントローラーから各種入力データを取得する」「入力データの内訳」などの詳しい話は、前記事を参照してください。

:::details コード（長いので格納しています）

```c:コントローラーからスティック、ボタン、加速度情報を取得し表示
#include <Wire.h>

// 復号化
#define DECODE(X) ((X ^ 0x17) + 0x17)

// バイトデータ(6) を格納するための諸要素
uint8_t signalNunchuck[6];
int byteCount;

// 各種入力データ（加速度のみ 10bit 必要）
uint8_t stickX, stickY;
uint8_t buttonC, buttonZ;
uint16_t accelX, accelY, accelZ;

void setup() {
  Serial.begin(9600);

  // シェイクハンド通信（シロ用）
  Wire.begin();
  Wire.beginTransmission(0x52);
  Wire.write((uint8_t)0x40);
  Wire.write((uint8_t)0x00);
  Wire.endTransmission();
}

void loop() {
  // コントローラーに入力データ(6byte) を要求、復号化しつつ配列に格納
  byteCount = 0;
  Wire.requestFrom(0x52, 6);
  while(Wire.available()){
    signalNunchuck[byteCount] = DECODE(Wire.read());
    byteCount++;
  }

  // 入力データが適正に受け取れた場合に、諸処理に進む
  if(byteCount >= 5){

    // スティック情報（中心は 128）
    stickX = signalNunchuck[0];
    stickY = signalNunchuck[1];

    // ボタン情報（0 が押下状態）
    buttonC = bitRead(signalNunchuck[5], 1);
    buttonZ = bitRead(signalNunchuck[5], 0);

    // 加速度情報（中心は 512）
    //（下位 2bit は末尾バイトデータに分割して格納されているので、組み合わせて 10bit 情報に）
    accelX = ((uint16_t) signalNunchuck[2] << 2) | ((signalNunchuck[5] & B00001100) >> 2);
    accelY = ((uint16_t) signalNunchuck[3] << 2) | ((signalNunchuck[5] & B00110000) >> 4);
    accelZ = ((uint16_t) signalNunchuck[4] << 2) | ((signalNunchuck[5] & B11000000) >> 6);

    // 出力
    char buffer[100];
    sprintf(
      buffer, 
      "| Stick(%3d, %3d) | BtnC: %1d | BtnZ: %1d | Accel(%4d, %4d, %4d) |",
      stickX, stickY, buttonC, buttonZ, accelX, accelY, accelZ
    );
    Serial.println(buffer);
  }

  // リクエスト分の通信終了処理
  Wire.beginTransmission(0x52);
  Wire.write(0x00);
  Wire.endTransmission();

  // データ入力の頻度を抑えるための遅延処理
  delay(100);
}
```

:::

#### データ取得をvoid関数化

今回、`void setup()` 側でもコントローラー情報を取得したい場面が出てきます。そのため上記コードを修正し、コントローラーの情報を取得する関数として独立させます。

```c:入力データを取得する処理をvoid関数として独立
#include <Wire.h>

// 復号化
#define DECODE(X) ((X ^ 0x17) + 0x17)

// バイトデータ(6) を格納するための諸要素
uint8_t signalNunchuck[6];
int byteCount;

// 各種入力データ（加速度のみ 10bit 必要）
uint8_t stickX, stickY;
uint8_t buttonC, buttonZ;
uint16_t accelX, accelY, accelZ;

// I2C通信を用いてコントローラーから各種入力情報を取得
void getDataFromController(){

  // コントローラーに入力データ(6byte) を要求、復号化しつつ配列に格納
  byteCount = 0;
  Wire.requestFrom(0x52, 6);
  while(Wire.available()){
    signalNunchuck[byteCount] = DECODE(Wire.read());
    byteCount++;
  }

  // 入力データが適正に受け取れた場合に、諸処理に進む
  if(byteCount >= 5){

    // スティック情報（中心は 128）
    stickX = signalNunchuck[0];
    stickY = signalNunchuck[1];

    // ボタン情報（0 が押下状態）
    buttonC = bitRead(signalNunchuck[5], 1);
    buttonZ = bitRead(signalNunchuck[5], 0);

    // 加速度情報（中心は 512）
    //（下位 2bit は末尾バイトデータに分割して格納されているので、組み合わせて 10bit 情報に）
    accelX = ((uint16_t) signalNunchuck[2] << 2) | ((signalNunchuck[5] & B00001100) >> 2);
    accelY = ((uint16_t) signalNunchuck[3] << 2) | ((signalNunchuck[5] & B00110000) >> 4);
    accelZ = ((uint16_t) signalNunchuck[4] << 2) | ((signalNunchuck[5] & B11000000) >> 6);
  }

  // リクエスト分の通信終了処理
  Wire.beginTransmission(0x52);
  Wire.write(0x00);
  Wire.endTransmission();
}

void setup() {
  Serial.begin(9600);

  // シェイクハンド通信（シロ用）
  Wire.begin();
  Wire.beginTransmission(0x52);
  Wire.write((uint8_t)0x40);
  Wire.write((uint8_t)0x00);
  Wire.endTransmission();
}

void loop() {

  // コントローラーの情報取得（データは各種グローバル変数に）
  getDataFromController();

  // データ入力の頻度を抑えるための遅延処理
  delay(100);
}
```

### 各軸データの中央値を移動

まずは、各種加速度データを見てみます。下表は実機で計測した、各種ポジション時の加速度情報となります。

| ポジション＼各軸データ | `x` | `y` | `z` |
| :--- | --- | --- | --- |
| デフォルト | 508 | 515 | 719 |
| 左に90°傾ける | 314 | 502 | 541 |
| 右に90°傾ける | 716 | 500 | 503 |
| 前に90°傾ける | 515 | 721 | 520 |
| 後ろに90°傾ける | 511 | 306 | 548 |
| 表裏反転（※） | 512 | 513 | 308 |

※ 左右もしくは前後に 180°回転した状態を指す

ここから `x`, `y`, `z` は 512 前後を中央とし、そこから 200 前後の幅で変動していることがわかります。これは静止時（1g）の場合なので、勢いをつけて振れば 200 を越える形で変動します。

さて、まず しなければならないのは「512 を中心とした符号なしデータ」から「0 を中心とした符号ありデータ」に置き換えることです。こうすることで、ベクトルデータとして扱えるようにします。

`x`, `y`, `z` ともに 512 周辺を中央としているので 1 つの定数で管理してもいいですが、ここでは各軸で調整できるように `CENTER_ACCEL_X`, `CENTER_ACCEL_Y`, `CENTER_ACCEL_Z` の計 3 つ用意することにします。各種加速度情報から これら定数を引けば、0 を中心としたデータにすることができます。

```c:加速度情報を0中心のデータに置き換え
#include <Wire.h>

// 省略：復号化関数
// 省略：バイトデータ(6) を格納するための諸要素
// 省略：各種入力データ（加速度のみ 10bit 必要）
uint16_t accelX, accelY, accelZ;

// 加速度データの中央値
const int CENTER_ACCEL_X = 512;
const int CENTER_ACCEL_Y = 512;
const int CENTER_ACCEL_Z = 512;

// I2C通信を用いてコントローラーから各種入力情報を取得
void getDataFromController(){ }

void setup() {
  Serial.begin(9600);

  // 省略：シェイクハンド通信
}

void loop() {

  // コントローラーの情報取得（データは各種グローバル変数に）
  getDataFromController();

  // 加速度情報を 0 を中央にした符号付きに置き換え、ベクトルデータとして扱う
  int vecX = accelX - CENTER_ACCEL_X;
  int vecY = accelY - CENTER_ACCEL_Y;
  int vecZ = accelZ - CENTER_ACCEL_Z;

  // データ入力の頻度を抑えるための遅延処理
  delay(100);
}

```

### 検知する角度に一定のマージンを設ける

[理論上の部分](#理論上の部分)では、[2つの空間ベクトルの為す角度を算出する関数](#空間ベクトルの為す角をarduino関数に)を作成しました。また「縦方向にあるか」という限定された状況用に、（計算量を抑えた）[縦方向との角度を算出する関数](#縦方向との角度をarduino関数に)も作成しました。

これら**角度**を算出している理由は、許容するマージンを定数として用意しておいて状況により調整できるようするためです。マージン値については、グローバル部に `const float MARGIN_DEGREE = 20.0;` と定数の形で設定するようにします。

そして[理論の部分で作成した、縦方向との角度を算出する関数](#縦方向との角度をarduino関数に)を持ってきて、組み合わせて条件分岐するようにしてみます。また Arduino に付属しているLED（ピン番号 13）を使い、条件を満たせば光るようにしてみます。

```c:角度判定のマージンと角度計算関数を設け、LEDを使った条件分岐
#include <Wire.h>

// LED
#define LEDPIN 13

// 省略：復号化関数
// 省略：バイトデータ(6) を格納するための諸要素
// 省略：各種入力データ（加速度のみ 10bit 必要）
uint16_t accelX, accelY, accelZ;

// 加速度データの中央値
const int CENTER_ACCEL_X = 512;
const int CENTER_ACCEL_Y = 512;
const int CENTER_ACCEL_Z = 512;

// 許容する角度のマージン値
const float MARGIN_DEGREE = 20.0;

// 1つの空間ベクトルから縦方向との角度を算出する
float angleToPortrait(int vecX, int vecY, int vecZ) {
  // ゼロベクトルの場合、NaN を返す
  if (vecX == 0 && vecY == 0 && vecZ == 0) return NAN;

  // Y方向が 0（cosθ が 0）の場合は、90°となる
  if (vecY == 0) return 90.0;

  // cosθの算出（acos 計算用に範囲を制限）
  float cosTheta = (-vecY) / (sqrt(pow(vecX, 2) + pow(vecY, 2) + pow(vecZ, 2)));
  cosTheta = constrain(cosTheta, -1.0, 1.0);

  // ラジアンを算出し、角度に変換して返す
  return (acos(cosTheta) * 180) / PI;
}

// I2C通信を用いてコントローラーから各種入力情報を取得
void getDataFromController(){ }

void setup() {
  Serial.begin(9600);

  // 省略：シェイクハンド通信

  // LED
  pinMode(LEDPIN, OUTPUT);
}

void loop() {

  // コントローラーの情報取得（データは各種グローバル変数に）
  getDataFromController();

  // 加速度情報を 0 を中央にした符号付きに置き換え、ベクトルデータとして扱う
  int vecX = accelX - CENTER_ACCEL_X;
  int vecY = accelY - CENTER_ACCEL_Y;
  int vecZ = accelZ - CENTER_ACCEL_Z;

  // 縦方向との角度を算出し、マージン内の場合かで分岐
  float deg = angleToPortrait(vecX, vecY, vecZ);
  if(deg < MARGIN_DEGREE){
    // 縦方向と判定
    digitalWrite(LEDPIN, HIGH);
  } else {
    // 縦方向でないと判定
    digitalWrite(LEDPIN, LOW);
  }

  // データ入力の頻度を抑えるための遅延処理
  delay(100);
}
```

これで「平滑化処理が必要なブレ幅」を除く内容に対し、実機での運用を考慮できました。

### ブレ幅を抑える平滑化処理

次に、残った「実機運用の際に考慮すべき箇所」である**取得する加速度情報にブレ幅が大きい**問題を解決していきます。この問題を解決するにあたって、今回は**移動平均による平滑化処理**を使います。

:::message

ここで扱う「移動平均による手法」は、単純移動平均を想定しています。

これは複数回数分の情報を保持しておき、そこから平均して 1 回分のデータを算出することで、ブレ幅を抑える手法となります。

:::

加速度に絞って説明したコードが、下記になります。

```c:移動平均を使った加速度情報の平滑化
#include <Wire.h>

// 省略：復号化関数
// 省略：バイトデータ(6) を格納するための諸要素
// 省略：各種入力データ（加速度のみ 10bit 必要）
uint16_t accelX, accelY, accelZ;

// 移動平均に使うデータ数
const int SAMPLE = 100;

// 移動平均で貯めておくデータ（SAMPLE 回数分のデータを保管）
uint16_t bufAccelX[SAMPLE], bufAccelY[SAMPLE], bufAccelZ[SAMPLE];
int bufIndex = 0;

// I2C通信を用いてコントローラーから各種入力情報を取得
void getDataFromController(){ }

void setup() {
  Serial.begin(9600);

  // 省略：シェイクハンド通信

  // 各軸の移動平均用のデータを準備
  for (int i = 0; i < SAMPLE; i++) {
    getDataFromController();
    bufAccelX[i] = accelX;
    bufAccelY[i] = accelY;
    bufAccelZ[i] = accelZ;
  }
}

void loop() {
  getDataFromController();

  // 新しいデータに更新
  bufAccelX[bufIndex] = accelX;
  bufAccelY[bufIndex] = accelY;
  bufAccelZ[bufIndex] = accelZ;

  // インデックスを更新
  bufIndex = (bufIndex + 1) % SAMPLE;

  // バッファ内のデータを合計
  uint32_t sumX = 0, sumY = 0, sumZ = 0;
  for (int i = 0; i < SAMPLE; i++) {
    sumX += bufAccelX[i];
    sumY += bufAccelY[i];
    sumZ += bufAccelZ[i];
  }

  // 移動平均の貯蔵データからデータ数を割ることで、1回の平均値を算出
  int aveX = sumX / SAMPLE;
  int aveY = sumY / SAMPLE;
  int aveZ = sumZ / SAMPLE;
  
  // データ表示
  Serial.print(aveX);
  Serial.print("\t");
  Serial.print(aveY);
  Serial.print("\t");
  Serial.print(aveZ);
  Serial.print("\n");

  // データ入力の頻度を抑えるための遅延処理
  delay(5);
}
```

ここで扱った変数や定数について、下表で補足を入れておきます。

|変数・定数|型|説明|
|---|---|:---|
|`SAMPLE`|整数|移動平均に使うデータ数|
|`bufAccelX`|整数（配列）|`SAMPLE` 数分の要素を持つ、加速度情報を格納した配列。最も過去のデータが最新のデータに更新されていく|
|`bufIndex`|整数|`bufAccelX` のインデックスを管理するための変数。0 から `SAMPLE` まで変化し、また 0 に戻る|
|`sumX`|整数|`bufAccelX` にあるデータの合計値。ここから `SAMPLE` で割ることで、1 回分となる平均値を算出する|
|`aveX`|整数|`bufAccelX` の平均値で、平滑化された値となる。この状態はまだ 512 を中心としたデータなので、符号付きに置換する必要がある|

`SAMPLE` が 10 程度だと まだブレ幅を感じます。手に持った際の微妙な動きや、計測誤差による変動が 1 回の処理毎に現れるのが確認できます。

一方 `SAMPLE` を 100 まで増やすと かなり滑らかになります。一気に動かした場合に それが反応するまで少しラグを感じますが、手に持った状態の微妙な動きや計測誤差は ほとんど感じられなくなります。

## まとめ

本記事では**加速度センサーから得られた情報を使ってデバイスの傾きを判定する**ことについて考えてみました。

まずは理論を考え、2つの空間ベクトルで為す角度について算出できるようにしました。そしてそこから計算量を抑えるために、片方の空間ベクトルを固定した簡略化した形での関数を作成しました。

そして上記理論を用いつつ、「512を中心としたデータ」「誤差による振れ幅が大きい」といった実機の事情に対処する形で実装を行いました。そこでは角度判定のマージンを設けたり、移動平均を用いた際のデータ数を設定したりと、個別の機材に応じた調整ができるように心がけてみました。

### コード全文

:::details 関数本体

```c:2つの空間ベクトルから為す角を算出する関数
float angleBetweenTwoVectors(int vecX1, int vecY1, int vecZ1, int vecX2, int vecY2, int vecZ2){
  // ゼロベクトルの場合、NaN を返す
  if ((vecX1 == 0 && vecY1 == 0 && vecZ1 == 0) || (vecX2 == 0 && vecY2 == 0 && vecZ2 == 0)){
    return NAN;
  }

  // 内積とベクトルの大きさを算出
  float dotProduct = (vecX1 * vecX2) + (vecY1 * vecY2) + (vecZ1 * vecZ2);
  float magnitudeV1 = sqrt(pow(vecX1, 2) + pow(vecY1, 2) + pow(vecZ1, 2));
  float magnitudeV2 = sqrt(pow(vecX2, 2) + pow(vecY2, 2) + pow(vecZ2, 2));

  // 問題なければ cosθを算出（いずれかが 0 の場合は NaN を返す）
  if (dotProduct == 0 || magnitudeV1 == 0 || magnitudeV2 ==0) return NAN;
  float cosTheta = dotProduct / (magnitudeV1 * magnitudeV2);

  // acos 計算用に、cosTheta の範囲を制限 (-1 <= cosTheta <= 1)
  cosTheta = constrain(cosTheta, -1.0, 1.0);

  // ラジアンを算出し、角度に変換して返す
  return (acos(cosTheta) * 180) / PI;
}
```

```c:1つの空間ベクトルから縦方向との角度を算出
float angleToPortrait(int vecX, int vecY, int vecZ) {
  // ゼロベクトルの場合、NaN を返す
  if (vecX == 0 && vecY == 0 && vecZ == 0) return NAN;

  // Y方向が 0（cosθ が 0）の場合は、90°となる
  if (vecY == 0) return 90.0;

  // cosθの算出（acos 計算用に範囲を制限）
  float cosTheta = (-vecY) / (sqrt(pow(vecX, 2) + pow(vecY, 2) + pow(vecZ, 2)));
  cosTheta = constrain(cosTheta, -1.0, 1.0);

  // ラジアンを算出し、角度に変換して返す
  return (acos(cosTheta) * 180) / PI;
}
```

:::

:::details コントローラーに組み込む場合

```c:各種入力データを出力、縦方向を判定しLEDを点灯
#include <Wire.h>

// 復号化
#define DECODE(X) ((X ^ 0x17) + 0x17)

// LED
#define LEDPIN 13

// バイトデータ(6) を格納するための諸要素
uint8_t signalNunchuck[6];
int byteCount;

// 各種入力データ（加速度のみ 10bit 必要）
uint8_t stickX, stickY;
uint8_t buttonC, buttonZ;
uint16_t accelX, accelY, accelZ;

// 加速度データの中央値
const int CENTER_ACCEL_X = 512;
const int CENTER_ACCEL_Y = 512;
const int CENTER_ACCEL_Z = 512;

// 許容する角度のマージン値
const float MARGIN_DEGREE = 20.0;

// 移動平均に使うデータ数
const int SAMPLE = 100;

// 移動平均で貯めておくデータ（SAMPLE 回数分のデータを保管）
uint16_t bufAccelX[SAMPLE], bufAccelY[SAMPLE], bufAccelZ[SAMPLE];
int bufIndex = 0;

// 1つの空間ベクトルから縦方向との角度を算出する
float angleToPortrait(int vecX, int vecY, int vecZ) {
  // ゼロベクトルの場合、NaN を返す
  if (vecX == 0 && vecY == 0 && vecZ == 0) return NAN;

  // Y方向が 0（cosθ が 0）の場合は、90°となる
  if (vecY == 0) return 90.0;

  // cosθの算出（acos 計算用に範囲を制限）
  float cosTheta = (-vecY) / (sqrt(pow(vecX, 2) + pow(vecY, 2) + pow(vecZ, 2)));
  cosTheta = constrain(cosTheta, -1.0, 1.0);

  // ラジアンを算出し、角度に変換して返す
  return (acos(cosTheta) * 180) / PI;
}

// I2C通信を用いてコントローラーから各種入力情報を取得
void getDataFromController(){

  // コントローラーに入力データ(6byte) を要求、復号化しつつ配列に格納
  byteCount = 0;
  Wire.requestFrom(0x52, 6);
  while(Wire.available()){
    signalNunchuck[byteCount] = DECODE(Wire.read());
    byteCount++;
  }

  // 入力データが適正に受け取れた場合に、諸処理に進む
  if(byteCount >= 5){

    // スティック情報（中心は 128）
    stickX = signalNunchuck[0];
    stickY = signalNunchuck[1];

    // ボタン情報（0 が押下状態）
    buttonC = bitRead(signalNunchuck[5], 1);
    buttonZ = bitRead(signalNunchuck[5], 0);

    // 加速度情報（中心は 512）
    //（下位 2bit は末尾バイトデータに分割して格納されているので、組み合わせて 10bit 情報に）
    accelX = ((uint16_t) signalNunchuck[2] << 2) | ((signalNunchuck[5] & B00001100) >> 2);
    accelY = ((uint16_t) signalNunchuck[3] << 2) | ((signalNunchuck[5] & B00110000) >> 4);
    accelZ = ((uint16_t) signalNunchuck[4] << 2) | ((signalNunchuck[5] & B11000000) >> 6);
  }

  // リクエスト分の通信終了処理
  Wire.beginTransmission(0x52);
  Wire.write(0x00);
  Wire.endTransmission();
}

void setup() {
  Serial.begin(9600);

  // シェイクハンド通信（シロ用）
  Wire.begin();
  Wire.beginTransmission(0x52);
  Wire.write((uint8_t)0x40);
  Wire.write((uint8_t)0x00);
  Wire.endTransmission();

  // LED
  pinMode(LEDPIN, OUTPUT);

  // 各軸の移動平均用のデータを準備
  for (int i = 0; i < SAMPLE; i++) {
    getDataFromController();
    bufAccelX[i] = accelX;
    bufAccelY[i] = accelY;
    bufAccelZ[i] = accelZ;
  }
}

void loop() {

  // コントローラーの情報取得（データは各種グローバル変数に）
  getDataFromController();

  // 新しいデータに更新
  bufAccelX[bufIndex] = accelX;
  bufAccelY[bufIndex] = accelY;
  bufAccelZ[bufIndex] = accelZ;

  // インデックスを更新
  bufIndex = (bufIndex + 1) % SAMPLE;

  // バッファ内のデータを合計
  uint32_t sumX = 0, sumY = 0, sumZ = 0;
  for (int i = 0; i < SAMPLE; i++) {
    sumX += bufAccelX[i];
    sumY += bufAccelY[i];
    sumZ += bufAccelZ[i];
  }

  // 移動平均の貯蔵データからデータ数を割ることで、1回の平均値を算出
  int aveX = sumX / SAMPLE;
  int aveY = sumY / SAMPLE;
  int aveZ = sumZ / SAMPLE;

  // 加速度情報を 0 を中央にした符号付きに置き換え、ベクトルデータとして扱う
  int vecX = aveX - CENTER_ACCEL_X;
  int vecY = aveY - CENTER_ACCEL_Y;
  int vecZ = aveZ - CENTER_ACCEL_Z;

  // 縦方向との角度を算出し、マージン内の場合かで分岐
  float deg = angleToPortrait(vecX, vecY, vecZ);
  if(deg < MARGIN_DEGREE){
    // 縦方向と判定
    digitalWrite(LEDPIN, HIGH);
  } else {
    // 縦方向でないと判定
    digitalWrite(LEDPIN, LOW);
  }

  // 確認用に出力
  char buffer[100];
  sprintf(
    buffer, 
    "| Stick(%3d, %3d) | BtnC: %1d | BtnZ: %1d | Accel(%4d, %4d, %4d) | Deg: %3d |",
    stickX, stickY, buttonC, buttonZ, aveX, aveY, aveZ, (int) deg
  );
  Serial.println(buffer);

  // データ入力の頻度を抑えるための遅延処理
  delay(5);
}
```

:::

### 参考文献

下記は本記事を作成するに参考にした資料等になります。

#### I2C関係

https://ja.wikipedia.org/wiki/I2C

https://www.nxp.com/docs/ja/user-guide/UM10204.pdf

#### Arduino

https://docs.arduino.cc/

http://www.musashinodenpa.com/arduino/ref/

https://www.oreilly.co.jp/books/9784814400232/

#### ヌンチャク関係

https://wiibrew.org/wiki/Wiimote/Extension_Controllers/Nunchuck

https://github.com/madhephaestus/WiiChuck

https://github.com/todbot/wiichuck_adapter

https://github.com/rkrishnasanka/ArduinoNunchuk

##### 加速度情報からデバイス向きを判定

https://qiita.com/kobayashi_ryo/items/48db56e62f7a76532d38

空間ベクトルの為す角を算出する式については、追加で下記を参考にしています。

https://mathwords.net/bekutorunasukaku

https://w3e.kanazawa-it.ac.jp/math/category/vector/henkan-tex.cgi?target=/math/category/vector/naiseki-wo-fukumu-kihonsiki.html&pcview=2
