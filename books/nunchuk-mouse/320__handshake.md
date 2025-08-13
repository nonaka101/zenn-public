---
title: "📄コード：コントローラーとの通信確立"
---

## はじめに

本項では、コントローラーと Arduino 間の通信を確立するプロセス（ハンドシェイク）について説明します。

## コントローラーとの通信確立

ここではコントローラーと通信できる状態まで進めていきます。

また、通信状況を監視し 通信がおかしくなった際には復旧できるようにもしていきます。

### 各種準備

まずは `setup()` にて、各種ライブラリを初期化して利用可能な状態にします。

```cpp:初期化
void setup() {
  Wire.begin();
  Mouse.begin();
  Keyboard.begin();
}
```

さらに、設定項目として設けている「デバッグ」「LED」関係の処理を行います。使用する項目は下表のとおりです。

|定数名|型|内容|
|---|---|:---|
|`ENABLE_DEBUG_MODE`|`bool`|デバッグモードの制御|
|`ENABLE_LED`|`bool`|状態通知に LED を利用するか|
|`LEDPIN`|`int`|状態通知に利用する LED ピン番号|

これらを使って、下記の処理を行います。

- デバッグモード時は、シリアル通信を初期化する
- LED で状態通知する場合は、ピンを出力に設定する

```cpp:デバッグやLEDに関する初期設定
void setup() {
  if (ENABLE_DEBUG_MODE) Serial.begin(9600);
  Wire.begin();
  Mouse.begin();
  Keyboard.begin();
  if(ENABLE_LED) pinMode(LEDPIN, OUTPUT);
}
```

これで諸々の準備が完了しました。

### 通信状態の監視

コントローラーと通信を確立する前に、正常に通信ができているかを監視できるようにしておきます。

- 通信状態を `isNunchukConnected`（`bool`）で管理
- コントローラーとの初期通信を関数 `initializeNunchuk()` として設ける
  - 正常に通信できたかの判定（`bool` 値）を、関数の返り値とする

```cpp:isNunchukConnectedで通信状態を管理
// ヌンチャクの通信判定
bool isNunchukConnected = false;

void setup() {
  if (ENABLE_DEBUG_MODE) Serial.begin(9600);
  Wire.begin();
  Mouse.begin();
  Keyboard.begin();
  if(ENABLE_LED) pinMode(LEDPIN, OUTPUT);

  isNunchukConnected = initializeNunchuk();
}
```

これで、通信が上手くできているかを `isNunchukConnected` で管理できるようになりました。

この変数は他にも、コントローラーからの情報取得が上手くいかなかった時など、通信全般で流用していくことになります。

#### 通信失敗時の処理

`isNunchukConnected` で通信状態を管理できるようになったので、次は これが `false`（≒ 通信失敗）になった時の処理を考えていきます。

この時は コントローラーが反応しないはずですので、「正常に動作していませんよ」とユーザーに通知する必要があります。またコントローラーとの通信を復旧する必要もあります。

通信失敗時の処理をまとめると、下記になります。

- LED を点滅させ、エラーを通知する
- コントローラーとの初期通信（`initializeNunchuk()`）をやり直す
  - 通信が復旧できた場合は、LED の点滅を止める

```cpp:isNunchukConnectedを使った処理
// ヌンチャクの通信判定
bool isNunchukConnected = false;

void setup() {
  if (ENABLE_DEBUG_MODE) Serial.begin(9600);
  Wire.begin();
  Mouse.begin();
  Keyboard.begin();
  if(ENABLE_LED) pinMode(LEDPIN, OUTPUT);

  isNunchukConnected = initializeNunchuk();
}

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

:::message

`loop()` の最後に、`delay(MOUSE_DELAY)` をつけています。これは 1 回の処理にかける時間を上下させることで、マウスの反応速度を調整するためです。

`MOUSE_DELAY` は、設定項目で定義しています。

|定数名|型|内容|
|---|---|:---|
|`MOUSE_DELAY`|`int`|マウス操作のディレイ（ミリ秒）|

:::

### コントローラーとの初期通信

ここでは、コントローラーとデータのやり取りを行えるよう通信準備を行います。両者を繋ぐ最初の段階で、応答確認や設定値の共有を行いますが、こういった**通信の初期化であり事前準備**のことを**ハンドシェイク**（*handshake*）といいます。

#### なぜハンドシェイクが必要か

コントローラーと Arduino とを繋ぐ際、ハンドシェイクが必要なのは、**同じデバイスアドレス**上に異なる機器が繋がりうるためです。

コントローラーは生産時期により「**シロ**」「**クロ**」の２種類があります。この２つはそれぞれ、内部データの扱いが若干異なるのですが、一方で I2C デバイスとしてのアドレスは同じ `0x52` を使用しています。

そのため、ハンドシェイクの際に適切な信号を送ることで、デバイスの種別（シロ/クロ）を確認しておく必要があるわけです。

#### デバイス別ハンドシェイク

本リポジトリでは、コントローラー側のレジスタ[^1]に特定のデータを送信することでハンドシェイクを行います。

[^1]: レジスタ：I2C デバイスに搭載されている、データを格納するための保存領域のこと。アドレスとデータで 1 つのセットとなる。**デバイス**のアドレスと**レジスタ**のアドレスは意味が異なるので、混同しないよう注意。

デバイス別に、ここでのレジスタアドレスとデータが異なります。「**シロ**に対して**クロ**用のハンドシェイク信号を送る」といった誤った処理をした場合、適正なデータを受け取れなくなるため注意が必要です。

コントローラーが**シロ**の場合は、レジスタアドレス `0x40` に対し、`0x00` を送信することで、通信を初期化します。

コントローラーが**クロ**の場合は、最初にレジスタアドレス `0xF0` に `0x55`、次にレジスタアドレス `0xFB` に `0x00` を送信し、通信を初期化します。

:::message

Arduino では、`Wire.write()` を使い I2C デバイスにデータを書き込みます。

所定のレジスタに値を書き込みたい場合は、「レジスタのアドレス」「データ」の順で `Wire.write()` で送り込むことができます。

例えば、シロ（デバイスアドレス `0x52`）に対して初期通信する場合は、下記のようになります。

```cpp:サンプル：ハンドシェイク（シロ）
Wire.beginTransmission(0x52);  // 指定したデバイスに送信処理を開始
Wire.write(0x40);              // レジスタアドレスを送信
Wire.write(0x00);              // データを送信
Wire.endTransmission();        // 送信処理を終了
```

:::

ハンドシェイクが適正に行われると、通信が行える状態になります。通信周波数は 100KHz です。

#### `initializeNunchuk()`

必要な部分は解説できたので、ここからは実際に初期通信を処理する関数 `initializeNunchuk()` を作成していきます。

「[通信状態の監視](#通信状態の監視)」で説明したように、通信結果を返すため関数の返り値は `bool` です。

また、固定となるデバイスアドレスは、定数として切り出しておきます。

```cpp:アドレス定数化と関数定義
// ヌンチャクのアドレス
const int NUNCHUK_ADDRESS = 0x52;

bool initializeNunchuk() {
  // デバイス別に初期通信し、通信結果を返す

  // 通信の調整
  delay(10);
}
```

次に、デバイス種別で異なるハンドシェイクを処理します。デバイス種別に関しては、設定項目から判断させます。

|定数名|型|内容|
|---|---|:---|
|`IS_NUNCHUK_SHIRO`|`bool`|使用するヌンチャクが「シロ」か|

```cpp:デバイス別にハンドシェイク
// ヌンチャクのアドレス
const int NUNCHUK_ADDRESS = 0x52;

bool initializeNunchuk() {
  // 種類別に通信
  if(IS_NUNCHUK_SHIRO){
    // ヌンチャク・シロの初期化
    Wire.beginTransmission(NUNCHUK_ADDRESS);
    Wire.write(0x40);
    Wire.write(0x00);
    Wire.endTransmission();
  } else {
    // ヌンチャク・クロの初期化
    Wire.beginTransmission(NUNCHUK_ADDRESS);
    Wire.write(0xF0);
    Wire.write(0x55);
    Wire.endTransmission();
    delay(10);

    Wire.beginTransmission(NUNCHUK_ADDRESS);
    Wire.write(0xFB);
    Wire.write(0x00);
    Wire.endTransmission();
  }
  delay(10);
}
```

大まかには完成しましたが、**通信結果を返す**ようにはなっていません。

なので通信結果を管理する変数 `result` を設け、これに通信結果を含めるよう修正します。通信結果自体は `Wire.endTransmission()` の返り値が 0 かそれ以外かで判定可能です。

クロの場合は 2 回通信していますが、その両方で通信成功していなければなりません（≒ 1 回でも失敗すれば、結果は `false`）

上記内容に沿って修正したのが、下記になります。

```cpp:コントローラーとの通信確立
// ヌンチャクのアドレス
const int NUNCHUK_ADDRESS = 0x52;

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
```

これで初期通信は完成しました。

## まとめ

本項では、コントローラーとArduino間の通信を確立するまでの一連のプロセスについて説明しました。

次項では 実際にコントローラーと通信し、入力情報を引き出していきます。

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
// グローバル変数・インスタンス (Global variables & instances)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// ヌンチャクのアドレス
const int NUNCHUK_ADDRESS = 0x52;

// ヌンチャクの通信判定
bool isNunchukConnected = false;





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 関数プロトタイプ宣言 (Function prototypes)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

bool initializeNunchuk();





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
```

:::
