---
title: "📄コード：コントローラーからデータを読み取る"
---

## はじめに

本項では、通信が確立したコントローラーから入力情報を引き出していきます。入力情報のイメージとしては下記のような感じです。

![デバッグモードで各種情報がシリアルモニタに出力されている](/images/books/nunchuk-mouse/ide-debug-00.gif)
*スティック(`S`), 加速度(`A`), ボタン(`C`), ボタン(`Z`) が目的とする情報*

## コントローラーからデータ取得する関数

ここでは、コントローラーのデータを利用可能な形に処理する一連の関数 `readNunchuk()` を定義していきます。

この関数には、下記の機能を持たせていきます。

1. データの受信
2. 取得したデータを利用可能な形に整形
3. 正常に処理できたかを `bool` 値にて返す

```cpp:関数定義
bool readNunchuk() {
  // 1. I2C通信でデータを取得
  // 2. 取得したデータを構造体に格納
  // 3. 処理結果を bool 値にて返す
}
```

### データの受信

まずはコントローラーに入力情報を要求し、それを受信できるようにしていきます。

I2C デバイス（≒ コントローラー）にデータを要求するためには、`Wire.requestFrom(アドレス, 要求バイト数)` を使います。これは指定したスレーブ（I2C デバイス）に対しデータを要求するための関数で、要求を受け取った側は 1バイト単位のデータとして Arduino へ送信してきます。

今回は、アドレスは `0x52`（≒ `NUNCHUK_ADDRESS`）、要求するバイト数は 6 になります。

`requestFrom()` で要求を受け取ったコントローラーは、そのタイミングの入力情報を 6 バイトのデータとして Arduino へ送信してきます。そのため バイトデータを格納するための配列を用意し、データを受け取っていきます。

一連の流れを記したのが下記です。通信開始（データ要求）から通信終了までの一連の流れであり、この間でデータを格納しておく必要があります。

```cpp:データ要求から通信終了まで
bool readNunchuk() {
  // I2C通信でデータを要求
  Wire.requestFrom(NUNCHUK_ADDRESS, 6);

  // ここでデータを格納

  // 通信終了の処理
  Wire.beginTransmission(NUNCHUK_ADDRESS);
  Wire.write(0x00);
  Wire.endTransmission();
}
```

データは 6 バイト分あるため、ここでは配列（`rawData[]`）を使います。1 バイト分のデータを配列要素として、計 6 つ格納していきます。

また `while` ループのインデックス管理 `byteIndex` を設け、ループ処理で順々にデータを配列に格納していきます。

:::message

受信したデータは暗号化されているため、このタイミングで複合（データ `X` に対し、`(X ^ 0x17) + 0x17`）しておきます。

:::

```cpp:コントローラーから6バイトのデータ取得
bool readNunchuk() {
  uint8_t rawData[6];  // コントローラーから取得された生データ
  int byteIndex = 0;

  // I2C通信でデータを取得（配列格納時に、信号をデコードしている）
  Wire.requestFrom(NUNCHUK_ADDRESS, 6);
  while (Wire.available() && byteIndex < 6) {
    rawData[byteIndex] = (Wire.read() ^ 0x17) + 0x17;
    byteIndex++;
  }

  // 通信終了の処理
  Wire.beginTransmission(NUNCHUK_ADDRESS);
  Wire.write(0x00);
  Wire.endTransmission();
}
```

これで、`rawData[]` にコントローラーからのデータ（6 バイト）を格納することができました。

### データの整形

ここからは、受信したデータの内訳を確認し、それを使いやすい形に整形していきます。

#### データの内訳

先程取得した `rawData[]` には、コントローラーの入力情報（6 バイト）が格納されています。つまり、この 6 バイトの中に「[📄コントローラー](./210__nunchuk)」で述べた物理機能 全ての情報が詰められているわけです。

各種**物理機能の概要**と**データの内訳**は、下表のようになっています。（**略称**については、本項内での用語となります）

|名称|略称|サイズ|備考|
|---|---|---|---|
|Cボタン|`btnC`|1bit| 0 が押下、1 が開放を表す|
|Zボタン|`btnZ`|1bit| 0 が押下、1 が開放を表す|
|スティック `x`|`sX`|8bit| 128 を中心とし、左に倒すと減少、右に倒すと増加 |
|スティック `y`|`sY`|8bit| 128 を中心とし、下に倒すと減少、上に倒すと増加 |
|加速度 `x`|`aX`|10bit| 512 を中心とし、左に倒すと減少、右に倒すと増加 |
|加速度 `y`|`aY`|10bit| 512 を中心とし、上に立てると減少、下に倒すと増加 |
|加速度 `z`|`aZ`|10bit| デフォルト時（※）で 760、X または Y 方向に 90 度倒すと 512、反転させると 320 に|

※ デフォルトの持ち方とは、スティックが重力に対し垂直方向にある状態（下図）を指します

![ヌンチャクコントローラー](/images/books/nunchuk-mouse/nunchuck_data-00.png =320x)

|**Byte**|bit7|bit6|bit5|bit4|bit3|bit2|bit1|bit0|
|:---:|---:|---:|---:|---:|---:|---:|---:|---:|
|**1**|`sX`7|`sX`6|`sX`5|`sX`4|`sX`3|`sX`2|`sX`1|`sX`0|
|**2**|`sY`7|`sY`6|`sY`5|`sY`4|`sY`3|`sY`2|`sY`1|`sY`0|
|**3**|`aX`9|`aX`8|`aX`7|`aX`6|`aX`5|`aX`4|`aX`3|`aX`2|
|**4**|`aY`9|`aY`8|`aY`7|`aY`6|`aY`5|`aY`4|`aY`3|`aY`2|
|**5**|`aZ`9|`aZ`8|`aZ`7|`aZ`6|`aZ`5|`aZ`4|`aZ`3|`aZ`2|
|**6**|`aZ`1|`aZ`0|`aY`1|`aY`0|`aX`1|`aX`0|`btnC`|`btnZ`|

上表の要点をまとめると、下記のようになります。

- スティック情報は、最初の 2 バイトにきれいにまとまった状態
- 10bit である加速度情報は、上位 8bit と下位 2bit に分割している
- 末尾のバイトには、各加速度下位ビットとボタン情報が詰め込まれた状態

:::details 例

例えば、`11111111 00000000 11111111 10101010 00000000 00101101`（8bit ✕ 6）という 6 バイトの情報が送られてきたとします。

この情報は最終的に下表のデータとなります。

|データ種類|値（2進数 → 10進数）|
|---|---|
|スティック `(x, y)`|`(11111111, 00000000)` → `(255, 0)`|
|加速度 `(x, y, z)`|`(1111111111, 1010101010, 0000000000)` → `(1023, 682, 0)`|
|ボタン `(c, z)`|`(0, 1)` → `(0, 1)`|

:::

#### 構造体の定義

コントローラーからの情報を管理しやすくするため、各種データを構造体の中でまとめるようにしておきます。

```cpp:構造体の定義とインスタンス生成
// コントローラーから取得した生データ
struct NunchukInput {
  // スティック情報（2軸、符号なし 8bit）
  // ボタン情報（2つ、boolean）
  // 加速度情報（3軸、符号なし 10bit）
};

// インスタンス生成
NunchukInput nunchukInput;
```

この際、生成時に 設定項目で設けた値を格納しておくことにします。使う設定項目は、下表となります。

|定数名|型|内容|
|---|---|:---|
|`STICK_CENTER_X`|`int`|スティック X 軸の中心値|
|`STICK_CENTER_Y`|`int`|スティック Y 軸の中心値|
|`ACCEL_CENTER_X`|`int`|加速度 X 軸の中心値|
|`ACCEL_CENTER_Y`|`int`|加速度 Y 軸の中心値|
|`ACCEL_CENTER_Z`|`int`|加速度 Z 軸の中心値|

これらを使いつつ、構造体を定義したものが下記です。

```cpp:構造体の定義
// コントローラーから取得した生データ
struct NunchukInput {
  uint8_t stickX, stickY;  // スティック軸（符号なし8bit）
  bool buttonC, buttonZ;  // ボタン（true: 押下状態, false: 解放状態）
  uint16_t accelX, accelY, accelZ;  // 加速度（符号なし10bit）
  NunchukInput() :
    stickX(STICK_CENTER_X), stickY(STICK_CENTER_Y),
    buttonC(false), buttonZ(false),
    accelX(ACCEL_CENTER_X), accelY(ACCEL_CENTER_Y), accelZ(ACCEL_CENTER_Z)
    {}
};

// 各種構造体のインスタンス
NunchukInput nunchukInput;
```

データ構造をまとめたものが、下表になります。

| 名前 | データ型 | 備考 |
| --- | --- | --- |
| `stickX` | `uint8_t` | スティック軸、符号なし 8bit |
| `stickY` | `uint8_t` | スティック軸、符号なし 8bit |
| `buttonC` | `bool` | true: 押下 / false: 解放 |
| `buttonZ` | `bool` | true: 押下 / false: 解放 |
| `accelX` | `uint16_t` | 加速度、符号なし 10bit |
| `accelY` | `uint16_t` | 加速度、符号なし 10bit |
| `accelZ` | `uint16_t` | 加速度、符号なし 10bit |

#### 構造体へ格納

あとは この構造体に、「[データの内訳](#データの内訳)」で述べた各種データを格納していきます。

そのため、関数 `readNunchuk()` の引数に構造体のポインタを受け取れるようにしておきます。

```cpp:構造体を引数に取り、ここに各種データを格納していく
bool readNunchuk(NunchukInput& nunchukInput) {...}
```

まずは、受け取ったデータが正常か（≒ 6 バイト受信できているか）を確認し、扱いやすいスティック情報を構造体に格納していきます。

```cpp:受信状況を確認し、スティック情報を構造体に格納
bool readNunchuk(NunchukInput& nunchukInput) {
  uint8_t rawData[6];  // コントローラーから取得された生データ

  // 省略：一連の通信処理
  // （この段階で、`rawData[]` に6バイト分のデータが格納済）

  // 取得したデータを構造体に格納
  if (byteIndex >= 5) {
    nunchukInput.stickX = rawData[0];
    nunchukInput.stickY = rawData[1];
  }
}
```

残る加速度情報とボタン情報は、末尾のバイトデータを分解しないと得られないものです。そのため、末尾データを `packedData` と切り分け、下記のように処理します。

- ボタン情報
  - バイトデータ内の特定位置を `bitRead()` で読み取る
  - （この際、押下/解放状態を感覚と一致させるために**反転**）
- 各種加速度
  - 上位 8bit と下位 2bit を繋げて 10bit を生成する
  - 上位 8bit は、`uint16_t` でキャストした上でビットシフト
  - 下位 2bit は、末尾データの特定箇所を抽出した上でビット位置を調整

```cpp:ボタンや加速度情報を構造体へ格納
bool readNunchuk(NunchukInput& nunchukInput) {
  uint8_t rawData[6];  // コントローラーから取得された生データ

  // 省略：一連の通信処理
  // （この段階で、`rawData[]` に6バイト分のデータが格納済）

  // 取得したデータを構造体に格納
  if (byteIndex >= 5) {
    nunchukInput.stickX = rawData[0];
    nunchukInput.stickY = rawData[1];

    // 末尾のバイトデータに複数データが詰め込まれている（物理ボタンの状態、各軸の加速度情報下位ビット）
    uint8_t packedData = rawData[5];
    nunchukInput.buttonZ = !bitRead(packedData, 0);  // true が押下になるように反転している
    nunchukInput.buttonC = !bitRead(packedData, 1);
    nunchukInput.accelX = ((uint16_t)rawData[2] << 2) | ((packedData & B00001100) >> 2);
    nunchukInput.accelY = ((uint16_t)rawData[3] << 2) | ((packedData & B00110000) >> 4);
    nunchukInput.accelZ = ((uint16_t)rawData[4] << 2) | ((packedData & B11000000) >> 6);
  }
}
```

:::message

加速度情報の部分で、`((uint16_t)rawData[2] << 2)` とキャストしてるのは、その後の 2 bit シフト移動で情報が抜け落ちてしまうことを防ぐためです。

:::

これで構造体 `NunchukInput` に各種データを格納することができました。

#### 返り値の設定

最後に、関数としての返り値を設定しておきます。データを適切に処理できていれば `true`、そうでなければ `false` なので、最終的な形は下記のようになります。

```cpp:返り値の設定
bool readNunchuk(NunchukInput& nunchukInput) {
  uint8_t rawData[6];  // コントローラーから取得された生データ

  // 省略：一連の通信処理
  // （この段階で、`rawData[]` に6バイト分のデータが格納済）

  // 取得したデータを構造体に格納
  if (byteIndex >= 5) {
    nunchukInput.stickX = rawData[0];
    nunchukInput.stickY = rawData[1];

    // 末尾のバイトデータに複数データが詰め込まれている（物理ボタンの状態、各軸の加速度情報下位ビット）
    uint8_t packedData = rawData[5];
    nunchukInput.buttonZ = !bitRead(packedData, 0);  // true が押下になるように反転している
    nunchukInput.buttonC = !bitRead(packedData, 1);
    nunchukInput.accelX = ((uint16_t)rawData[2] << 2) | ((packedData & B00001100) >> 2);
    nunchukInput.accelY = ((uint16_t)rawData[3] << 2) | ((packedData & B00110000) >> 4);
    nunchukInput.accelZ = ((uint16_t)rawData[4] << 2) | ((packedData & B11000000) >> 6);

    return true;
  }
  return false;
}
```

## 関数を動作に組み込む

「[コントローラーからデータ取得する関数](#コントローラーからデータ取得する関数)」にて、`readNunchuk()` を作成しました。この関数は構造体 `NunchukInput` を受け取り、正常に処理できたかを `bool` 値にて返すよう定義しました。

ここからは、この関数を動作の中に組み込んでいきます。

### `loop()`への組み込み

`readNunchuk()` を組み込む先は、`loop()` になります。

```cpp:前項でのloop関数
void loop() {
  if(isNunchukConnected){

    // 通信状態での処理

  } else {
    // エラー時（未接続・通信断）の処理

    // LEDの高速点滅で、ユーザーにエラーを通知
    if(ENABLE_LED) digitalWrite(LEDPIN, !digitalRead(LEDPIN));
    delay(100);

    // 再度初期化を施し、接続できないかを試す
    isNunchukConnected = initializeNunchuk();
    if (ENABLE_LED && isNunchukConnected) digitalWrite(LEDPIN, LOW);
  }
  delay(MOUSE_DELAY);
}
```

`isNunchukConnected` が `true` であれば、通信が確立している状況ですので、この `if` 文の中で各種処理を行うことになります。

そして `readNunchuk()` は処理結果を `bool` で返すよう定義しているので、ここでも `if` 文を挟んで条件を分岐させておきます。

```cpp:コントローラーのデータ取得判定で更に分岐
void loop() {
  if(isNunchukConnected){
    // コントローラーから入力を読み取る（正常に読み取れたかで分岐）
    if(readNunchuk(nunchukInput)){

      // 正常に読み取れた状態

    } else {

      // 読み取り失敗

    }
  } else {
    // 省略：エラー（未接続・通信断）での処理
  }
  delay(MOUSE_DELAY);
}
```

正常に読み取れた場合には `nunchukInput` にはコントローラーの各種情報が格納された状態です。

一方読み取り失敗では、コントローラーからのデータが扱えない状態であるため、何かしら別の対処が必要になりそうです。

### エラー対処

コントローラーからの読み取りが失敗した場合、どのように処理していくべきかを考えます。

ここでの致命的な状態とは、**マウスボタンが押され続けた状態になってしまう**ことです。

カーソルやホイールは `loop()` 処理の度に取得し反映していますが、マウスボタンに関しては**直近の押下/解放を使った状態管理**を行なっています。

そのため コントローラーからのデータ取得で失敗した場合、カーソルやホイールは動かなくなるだけですみますが、マウスボタンに関しては直近の状態を反映し続ける（≒ボタンを押し続ける）という可能性が生じます。

そこで、コントローラーからのデータ取得失敗時（≒ `readNunchuk()` で `false` 判定）となった場合は、マウスボタン関係を全て解放状態にするようにします。

```cpp:取得失敗時は全てのボタン状態を解放
// セットアップ後、常時処理される関数
void loop() {
  if(isNunchukConnected){
    // コントローラーから入力を読み取る（正常に読み取れたかで分岐）
    if(readNunchuk(nunchukInput)){

      // 正常に読み取れた状態

    } else {
      // 読み取り失敗したので、一度マウスボタン関係を全停止させる
      Mouse.release(MOUSE_LEFT);
      Mouse.release(MOUSE_RIGHT);
      Mouse.release(MOUSE_MIDDLE);
    }
  } else {
    // 省略：エラー（未接続・通信断）での処理
  }
  delay(MOUSE_DELAY);
}
```

:::message

後述しますが、これでマウスボタンに関しては問題ない状態となります。

構造体を使ったマウス操作の反映は `readNunchuk()` が `true` になった場合に処理されることになるため、データ取得がおかしな状態ではその処理までたどり着くことはありません。

:::

更に、[前項](./320__handshake)でも出てきた通信状況の管理を使い、この状況から復旧できるようにしておきます。

`readNunchuk()` が `false` になったということは、何かしら問題が起きているということであり、もう一度 1 からやり直すべき状況ということです。

そのため、前項で使った `isNunchukConnected` を `false` に変更し、もう一度初期化からやり直すようにします。

```cpp:コントローラー取得失敗時の対応
// セットアップ後、常時処理される関数
void loop() {
  if(isNunchukConnected){
    // コントローラーから入力を読み取る（正常に読み取れたかで分岐）
    if(readNunchuk(nunchukInput)){

      // 正常に読み取れた状態

    } else {
      // 読み取り失敗したので、一度マウスボタン関係を全停止させる
      isNunchukConnected = false;
      Mouse.release(MOUSE_LEFT);
      Mouse.release(MOUSE_RIGHT);
      Mouse.release(MOUSE_MIDDLE);
    }
  } else {
    // エラー時（未接続・通信断）の処理

    // LEDの高速点滅で、ユーザーにエラーを通知
    if(ENABLE_LED) digitalWrite(LEDPIN, !digitalRead(LEDPIN));
    delay(100);

    // 再度初期化を施し、接続できないかを試す
    isNunchukConnected = initializeNunchuk();
    if (ENABLE_LED && isNunchukConnected) digitalWrite(LEDPIN, LOW);
  }
  delay(MOUSE_DELAY);
}
```

`isNunchukConnected` を `false` に設定したので、次の `loop()` にて最初の `if` 文で弾かれ、初期化処理からやり直すという寸法です。

## まとめ

本項では、通信が確立したコントローラーから入力情報を引き出すための関数 `readNunchuk()` を作成しました。

次項では、扱いの難しい加速度情報に対処するための**平滑化処理**について説明します。

:::details ここまでのコード

```cpp
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// ライブラリ (Libraries)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

#include <Wire.h>
#include <Mouse.h>
#include <Keyboard.h>
#include <math.h>
#include <stdio.h>





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 内部定義 (Internal definitions)
// ※ 通常、このセクションを変更する必要はありません
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// 向き判定に使う 6 方向：XYZ軸(3) ✕ 正負(2)
enum TargetAxis {
  AXIS_X_NEG, AXIS_X_POS,  // X 軸方向（-/+）: 左傾斜 / 右傾斜
  AXIS_Y_NEG, AXIS_Y_POS,  // Y 軸方向（-/+）: 後傾 / 前傾
  AXIS_Z_NEG, AXIS_Z_POS  // Z 軸方向（-/+）: 表向き / 裏向き
};





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 設定項目 (User-configurable settings)
// ※ このセクション内の値は、好みに応じて調整してください
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// === デバッグ & LED =============================================================

// デバッグモードを有効化する（デフォルト：false）
// ※ デバッグ時は、IDE シリアルモニター上に状態を出力します
const bool ENABLE_DEBUG_MODE = false;

// 状態を可視化するため LED を使用する（デフォルト：true）
// ※ 点灯はホイールモード、点滅は接続エラーを示します
const bool ENABLE_LED = true;

// 使用する LED ピン番号（デフォルト：13）
// ※ ENABLE_LED 有効時に使用します。デフォルトの 13 は、Arduino のオンボード LED です。
const int LEDPIN = 13;

// === デバイス関係 ===============================================================

// ヌンチャク種別、クロの場合は false に変更（デフォルト：true）
const bool IS_NUNCHUK_SHIRO = true;

// Macintosh の場合は true に変更（デフォルト：false）
const bool IS_MACINTOSH = false;

// === ヌンチャク物理特性のデフォルト値 ===========================================

// スティック X 軸の中心値（デフォルト：128）
const int STICK_CENTER_X = 128;

// スティック Y 軸の中心値（デフォルト：128）
const int STICK_CENTER_Y = 128;

// スティック X 軸の遊び範囲（デフォルト：10）
const int STICK_DEADZONE_X = 10;

// スティック Y 軸の遊び範囲（デフォルト：10）
const int STICK_DEADZONE_Y = 10;

// スティック X 軸の可動範囲（デフォルト：90）
const int STICK_RANGE_X = 90;

// スティック Y 軸の可動範囲（デフォルト：90）
const int STICK_RANGE_Y = 90;

// 加速度 X 軸の中心値（デフォルト：512）
const int ACCEL_CENTER_X = 512;

// 加速度 Y 軸の中心値（デフォルト：512）
const int ACCEL_CENTER_Y = 512;

// 加速度 Z 軸の中心値（デフォルト：512）
const int ACCEL_CENTER_Z = 512;

// 誤差が大きい加速度を平滑化するためのサンプリング数（デフォルト：50）
// ※ 10 〜 100 の間での運用を推奨します。範囲外はメモリ不足等が発生する恐れがあります。
const int ACCEL_SMOOTHING_SAMPLES = 50;

// === マウスの挙動 ===============================================================

// マウス操作のディレイ（デフォルト：10 [ミリ秒単位]）
// 増減することで、全体の速さを調整することができます。
const int MOUSE_DELAY = 10;

// マウスカーソルの最大移動量（デフォルト：5, 最小：3）
const int MOVEMENT_CURSOR = 5;

// ホイールの最大移動量（デフォルト：1）
const int MOVEMENT_WHEEL = 1;

// ホイールによるスクロール方向を反転する（デフォルト：false）
const bool ENABLE_SCROLL_REVERSE = false;

// ホイールモードのスティック左右に「戻る」「進む」機能を割り当てる（デフォルト：true）
const bool ENABLE_SHORTCUT_FUNCTIONS = true;

// ショートカットボタン（進む、戻る）の待機回数（`ENABLE_SHORTCUT_FUNCTIONS` 有効時）
// ※ 待機時間は MOUSE_DELAY を掛けた値（ミリ秒）となる。
const int SHORTCUT_WAIT_TIME = 100;

// === モード切替ロジック ============================================================

// モード切替で判定する向き（デフォルト：AXIS_Y_NEG）
// - AXIS_X_NEG : X 軸方向マイナス、左傾斜
// - AXIS_X_POS : X 軸方向プラス、右傾斜
// - AXIS_Y_NEG : Y 軸方向マイナス、後傾
// - AXIS_Y_POS : Y 軸方向プラス、前傾
// - AXIS_Z_NEG : Z 軸方向マイナス、裏向き
// - AXIS_Z_POS : Z 軸方向プラス、表向き
const TargetAxis TARGET_AXIS = AXIS_Y_NEG;

// 向き判定で許容する角度（デフォルト：30.0）
// ※ 大きいほど判定が緩くなるので注意
const float MODE_SWITCH_ANGLE_MARGIN = 30.0;

// === ボタンマッピング ===============================================================

// モードに応じたボタンマッピング（デフォルト：{MOUSE_LEFT,MOUSE_RIGHT},{MOUSE_LEFT,MOUSE_MIDDLE}）
// 対応関係は下記となっています、`MOUSE_LEFT`, `MOUSE_RIGHT`, `MOUSE_MIDDLE` を割り当ててください
//
// { カーソルモードのCボタン , カーソルモードのZボタン },
// { ホイールモードのCボタン , ホイールモードのZボタン }
const int MOUSE_BUTTON_MAPS[2][2] = {
  { MOUSE_LEFT, MOUSE_RIGHT },
  { MOUSE_LEFT, MOUSE_MIDDLE }
};





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 構造体定義 (Data structures)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// コントローラーから取得した生データ
struct NunchukInput {
  uint8_t stickX, stickY;  // スティック軸（符号なし8bit）
  bool buttonC, buttonZ;  // ボタン（true: 押下状態, false: 解放状態）
  uint16_t accelX, accelY, accelZ;  // 加速度（符号なし10bit）
  NunchukInput() :
    stickX(STICK_CENTER_X), stickY(STICK_CENTER_Y),
    buttonC(false), buttonZ(false),
    accelX(ACCEL_CENTER_X), accelY(ACCEL_CENTER_Y), accelZ(ACCEL_CENTER_Z)
    {}
};





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// グローバル変数・インスタンス (Global variables & instances)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// 各種構造体のインスタンス
NunchukInput nunchukInput;

// ヌンチャクのアドレス
const int NUNCHUK_ADDRESS = 0x52;

// ヌンチャクの通信判定
bool isNunchukConnected = false;





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 関数プロトタイプ宣言 (Function prototypes)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

bool initializeNunchuk();
bool readNunchuk(NunchukInput& nunchukInput);





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// メイン処理 (Main: setup, loop)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// 起動時に１度だけ処理される関数
void setup() {
  if (ENABLE_DEBUG_MODE) Serial.begin(9600);
  Wire.begin();
  Mouse.begin();
  Keyboard.begin();
  if(ENABLE_LED) pinMode(LEDPIN, OUTPUT);

  isNunchukConnected = initializeNunchuk();
}

// セットアップ後、常時処理される関数
void loop() {
  if(isNunchukConnected){
    // コントローラーから入力を読み取る（正常に読み取れたかで分岐）
    if(readNunchuk(nunchukInput)){

      // 正常に読み取れた状態

    } else {
      // 読み取り失敗したので、一度マウスボタン関係を全停止させる
      isNunchukConnected = false;
      Mouse.release(MOUSE_LEFT);
      Mouse.release(MOUSE_RIGHT);
      Mouse.release(MOUSE_MIDDLE);
    }
  } else {
    // エラー時（未接続・通信断）の処理

    // LEDの高速点滅で、ユーザーにエラーを通知
    if(ENABLE_LED) digitalWrite(LEDPIN, !digitalRead(LEDPIN));
    delay(100);

    // 再度初期化を施し、接続できないかを試す
    isNunchukConnected = initializeNunchuk();
    if (ENABLE_LED && isNunchukConnected) digitalWrite(LEDPIN, LOW);
  }
  delay(MOUSE_DELAY);
}





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 機能ごとの関数 (Functions)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// === Nunchuk 通信関連 =========================================================

// コントローラーとの通信確立（ハンドシェイク）
bool initializeNunchuk() {
  // 返り値、`endTransmission()` が 1 度でも失敗すると `false`
  bool result = true;

  // 種類別に通信
  if(IS_NUNCHUK_SHIRO){
    // ヌンチャク・シロの初期化
    Wire.beginTransmission(NUNCHUK_ADDRESS);
    Wire.write(0x40);
    Wire.write(0x00);
    if (Wire.endTransmission() != 0) result = false;
  } else {
    // ヌンチャク・クロの初期化
    Wire.beginTransmission(NUNCHUK_ADDRESS);
    Wire.write(0xF0);
    Wire.write(0x55);
    if (Wire.endTransmission() != 0) result = false;
    delay(10);

    Wire.beginTransmission(NUNCHUK_ADDRESS);
    Wire.write(0xFB);
    Wire.write(0x00);
    if (Wire.endTransmission() != 0) result = false;
  }
  delay(10);
  return result;
}



/**
 * コントローラーからデータを読み取る
 *
 * @param nunchukInput `NunchukInput` 構造体への参照
 * @return 読み取り成功時は `true`、失敗時は `false`
 * @note 読み取ったデータは、`NunchukInput` 構造体に格納される
 */
bool readNunchuk(NunchukInput& nunchukInput) {
  uint8_t rawData[6];  // コントローラーから取得された生データ
  int byteIndex = 0;

  // I2C通信でデータを取得（配列格納時に、信号をデコードしている）
  Wire.requestFrom(NUNCHUK_ADDRESS, 6);
  while (Wire.available() && byteIndex < 6) {
    rawData[byteIndex] = (Wire.read() ^ 0x17) + 0x17;
    byteIndex++;
  }
  Wire.beginTransmission(NUNCHUK_ADDRESS);
  Wire.write(0x00);
  Wire.endTransmission();

  // 取得したデータを構造体に格納
  if (byteIndex >= 5) {
    nunchukInput.stickX = rawData[0];
    nunchukInput.stickY = rawData[1];

    // 末尾のバイトデータに複数データが詰め込まれている（物理ボタンの状態、各軸の加速度情報下位ビット）
    uint8_t packedData = rawData[5];
    nunchukInput.buttonZ = !bitRead(packedData, 0);  // true が押下になるように反転している
    nunchukInput.buttonC = !bitRead(packedData, 1);
    nunchukInput.accelX = ((uint16_t)rawData[2] << 2) | ((packedData & B00001100) >> 2);
    nunchukInput.accelY = ((uint16_t)rawData[3] << 2) | ((packedData & B00110000) >> 4);
    nunchukInput.accelZ = ((uint16_t)rawData[4] << 2) | ((packedData & B11000000) >> 6);

    return true;
  }
  return false;
}
```

:::
