---
title: "📄コード：デバッグ用の出力"
---

## はじめに

本項では、デバッグ用として ここまで取得したデータ内容を確認できるようモニタに出力する関数を作成します。

## 各構造体が持つデータ

まずは ここまでに作成した各構造体について、それぞれ何を管理しているかを見てみます。

### `NunchukInput`

`NunchukInput` では主に、コントローラーからの入力情報を管理しています。

| 名前 | データ型 | 備考 |
| --- | --- | --- |
| `stickX` | `uint8_t` | スティック軸、符号なし 8bit |
| `stickY` | `uint8_t` | スティック軸、符号なし 8bit |
| `buttonC` | `bool` | true: 押下 / false: 解放 |
| `buttonZ` | `bool` | true: 押下 / false: 解放 |
| `accelX` | `uint16_t` | 加速度、符号なし 10bit |
| `accelY` | `uint16_t` | 加速度、符号なし 10bit |
| `accelZ` | `uint16_t` | 加速度、符号なし 10bit |

この内 加速度情報に関しては、扱いづらさの観点から 次の `SmoothedAccelerometer` にデータを送って平滑化処理を行なっています。

### `SmoothedAccelerometer`

`SmoothedAccelerometer` では、ブレが大きい加速度情報を平滑化するためのデータを扱っています。

| 名前 | データ型 | 備考 |
| --- | --- | --- |
| `accelX` | `int` | 平滑化済みの加速度、符号なし 10bit |
| `accelY` | `int` | 平滑化済みの加速度、符号なし 10bit |
| `accelZ` | `int` | 平滑化済みの加速度、符号なし 10bit |
| `accelSumX` | `uint32_t` | 期間データの合計値、平滑化処理用 |
| `accelSumY` | `uint32_t` | 期間データの合計値、平滑化処理用 |
| `accelSumZ` | `uint32_t` | 期間データの合計値、平滑化処理用 |
| `accelBufX` | `int[*]` | 期間データを配列で管理 |
| `accelBufY` | `int[*]` | 期間データを配列で管理 |
| `accelBufZ` | `int[*]` | 期間データを配列で管理 |
| `accelBufIndex` | `int` | 上記配列のインデックス管理用 |

:::message

期間データについては、設定項目 `ACCEL_SMOOTHING_SAMPLES` にてサンプリング数を管理しています。

:::

実際にマウス操作に使うのは `accelX`, `accelY`, `accelZ` になります。

`NunchukInput` と `SmoothedAccelerometer` を組み合わせ、次の `MouseControl` でマウス操作用の情報に変換することになります。

### `MouseControl`

`MouseControl` では、コントローラーから取得した各種データをマウス操作用のデータに変換し管理しています。

| 名前 | データ型 | 備考 |
| --- | --- | --- |
| `currentMode` | `int` | `MODE_CURSOR`, `MODE_WHEEL` の 2 択 |
| `moveX` | `int` | カーソル移動量（横） |
| `moveY` | `int` | カーソル移動量（縦） |
| `moveW` | `int` | ホイール移動量 |
| `moveS` | `int` | ショートカットボタン用 |
| `shortcutThrottling` | `int` | ショートカット操作の待機状態を管理 |
| `buttonStatesCurrent` | `bool[2]` | 物理ボタン2種（C,Z）の現在の状態 |
| `buttonStatesLast` | `bool[2]` | 物理ボタン2種（C,Z）の直近の状態 |
| `MAX_ACTIONS` | `int` | 1回の `loop` で処理する指示セット最大数（≒ 4） |
| `actions` | `MouseAction[4]` | 指示セットを配列管理 |
| `actionCount` | `int` | 指示セット操作用のインデックス |

## デバッグ用の出力

ここでは[前述](#各構造体が持つデータ)の内容を、デバッグ用に出力する方法について考えていきます。

### デバッグ出力の概要

Arduino では、シリアル通信用の機能がデフォルトで搭載されており、`Serial` を使うことで利用できます。

https://docs.arduino.cc/language-reference/en/functions/communication/serial/

シリアル通信を使えば、Arduino IDE のシリアルモニタ上にデータを出力することができます。

デバッグ処理では これを使って、コード内で処理したデータを確認できるようモニタ上に出力するようにします。

ここまでのコードでは、デバッグモードを設定項目 `ENABLE_DEBUG_MODE` で管理しており、モード有効時はシリアル通信を開始するよう処理しています。

```cpp
// デバッグモードを有効化する（デフォルト：false）
// ※ デバッグ時は、IDE シリアルモニター上に状態を出力します
const bool ENABLE_DEBUG_MODE = false;

void setup() {
  if (ENABLE_DEBUG_MODE) Serial.begin(9600);

  // 省略：各種 初期化処理
}
```

### データをモニタ上に出力する

シリアル通信自体は既に完了しているので、後は `Serial.print()` 等でデータをモニタ上に出力できます。ただし、モニタ上でデータの移り変わりをわかりやすく表示させるには、もうひと手間必要となります。

本リポジトリのデバッグでは、各種データを 1 秒に複数回出力し、それぞれの中身がどう変化するかを見ていく必要があります。そうするためには下記の条件が必要となります。

1. 1 回の出力を、1 行でまとめること（≒ 1 行の中に、各種データが収まっている）
2. 各種データの位置が、1 行の中で常に同じカラム位置に出力されること

上記に則り、各種データを 1 行にまとめたのが下記です。

```txt
S(128,186) -> Mv(-1, 5, 1, 0) | A( 532, 320, 724) -> As( 524, 323, 723) -> X_POS | Mode:0 | C:1 Z:0
```

各要素は下記のようになっています。

|要素名|備考|
|---|:---|
|`S`(*Stick*)|スティック情報（2軸）|
|`Mv`(*Move*)|カーソル移動量（2軸）|
|`A`(*Accel*)|加速度（3軸）|
|`As`(*AccelSmoothed*)|平滑化済みの加速度（3軸）|
|`X_POS`|デバイス向き（`軸` と`±` 表記）|
|`Mode`|モード|
|`C`|C ボタン|
|`Z`|Z ボタン|

実際にシリアルモニタ上に出力しているのが下図です。各種データが整列された状態なので、データの移り変わりが視覚的にわかりやすくなっています。

![デバッグモードで各種情報がシリアルモニタに出力されている](/images/books/nunchuk-mouse/ide-debug-00.gif)

これを実現するためには、標準入出力を扱うライブラリ `stdio.h` にある、`sptintf()` が必要です。

### `sprintf()`

`sprintf()` は、様々なデータ型を一定のフォーマットに従い文字列として出力するための関数です。この関数を使うことで、様々なデータ型を文字列として扱うことができ、また数値に関しては 出力する桁数調整も可能となります。

https://cplusplus.com/reference/cstdio/printf/

引数は、**出力内容分の容量を確保した文字列ポインタ**と**フォーマット用の文字列**、それからフォーマットに当てはめるための各種データとなります。

文字列ポインタは、今回 200 と少し多めに確保しておきます。

```cpp
// デバッグ出力用のバッファ
char debugBuffer[200];
```

### 構造体のデータを出力する

引数の内容に従ってフォーマット文字列とデータを当てはめたものが下記です。

```cpp:構造体の各種データを出力
void printDebugInfo(const NunchukInput& nunchukInput, const SmoothedAccelerometer& smoothedAccel, const MouseControl& mouseControl) {
  sprintf(debugBuffer, "S(%3d,%3d) -> Mv(%2d,%2d,%2d,%2d) | A(%4d,%4d,%4d) -> As(%4d,%4d,%4d) | Mode:%d | C:%d Z:%d",
          nunchukInput.stickX, nunchukInput.stickY,
          mouseControl.moveX, mouseControl.moveY, mouseControl.moveW, mouseControl.moveS,
          nunchukInput.accelX, nunchukInput.accelY, nunchukInput.accelZ,
          smoothedAccel.accelX, smoothedAccel.accelY, smoothedAccel.accelZ,
          mouseControl.currentMode,
          nunchukInput.buttonC, nunchukInput.buttonZ
          );
  Serial.println(debugBuffer);
}
```

#### デバイス向きを含めた形に改良

改良案として、「[📄コード：向きの計算](./350__calculate-angle)」の補足にて述べた**デバイス向き**を出力するようにします。処理の仕組みについては向こうで解説していますので、詳細はそちらを参照ください。

出力の際には `String` 型をそのまま `sprintf()` に突っ込むことはできないので、一度 `c_str()` で変換をかけてあげる必要があります。

```cpp:デバイス向き情報を加えて出力
void printDebugInfo(const NunchukInput& nunchukInput, const SmoothedAccelerometer& smoothedAccel, const MouseControl& mouseControl) {

  // ここで出力する、デバイス向き一覧
  String names[6] = {"X_Neg", "X_Pos", "Y_Neg", "Y_Pos", "Z_Neg", "Z_Pos"};

  // 平滑化加速度をベクトル情報にし、6 方向（XYZ の各軸±）との角度算出
  int vecX = smoothedAccel.accelX - ACCEL_CENTER_X;
  int vecY = smoothedAccel.accelY - ACCEL_CENTER_Y;
  int vecZ = smoothedAccel.accelZ - ACCEL_CENTER_Z;
  float angles[6] = {
    angleBetweenTwoVectors(vecX, vecY, vecZ, -1, 0, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 1, 0, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, -1, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, 1, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, 0, -1),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, 0, 1)
  };

  // 6要素の内、最小な値（≒ デバイス向き）を探す
  //（ここではインデックスを用意するまで、使うのは `sprintf` 側）
  int minIndex = 0;
  for (int i = 1; i < 6; i++) {
    if (angles[i] < angles[minIndex]) minIndex = i;
  }

  sprintf(debugBuffer, "S(%3d,%3d) -> Mv(%2d,%2d,%2d,%2d) | A(%4d,%4d,%4d) -> As(%4d,%4d,%4d) -> %4s | Mode:%d | C:%d Z:%d",
          nunchukInput.stickX, nunchukInput.stickY,
          mouseControl.moveX, mouseControl.moveY, mouseControl.moveW, mouseControl.moveS,
          nunchukInput.accelX, nunchukInput.accelY, nunchukInput.accelZ,
          smoothedAccel.accelX, smoothedAccel.accelY, smoothedAccel.accelZ,
          names[minIndex].c_str(),
          mouseControl.currentMode,
          nunchukInput.buttonC, nunchukInput.buttonZ
          );
  Serial.println(debugBuffer);
}
```

これでデバッグ用の関数が完成しました。

### `loop` 処理に組み込む

最後に、デバッグ関数を `loop` 関数の中に組み込みます。

各種構造体に適正なデータが格納されるのは、`updateMouse()` の後になります。なので `printDebugInfo()` はその後に書き込みます。

ここではデバッグモードは設定項目 `ENABLE_DEBUG_MODE` にて管理していますので、それが有効な場合に処理するよう条件分岐を設けておきます。

```cpp
void loop() {
  if(isNunchukConnected){
    // コントローラーから入力を読み取る（正常に読み取れたかで分岐）
    if(readNunchuk(nunchukInput)){
      // 加速度を平滑化処理する
      smoothAccelerometer(smoothedAccel, nunchukInput.accelX, nunchukInput.accelY, nunchukInput.accelZ);

      // マウス操作情報を更新 (マウスボタン指示に関するイベント関係も、ここで管理)
      updateMouse(nunchukInput, smoothedAccel, mouseControl);

      // マウス操作情報を反映、またはデバッグ情報を出力
      if (ENABLE_DEBUG_MODE) {
        printDebugInfo(nunchukInput, smoothedAccel, mouseControl);
      } else {
        // マウス操作情報を実際に反映
      }
    } else {
      // 省略：読み取り失敗時の処理
    }
  } else {
    // 省略：エラー時（未接続・通信断）の処理
  }
  delay(MOUSE_DELAY);
}
```

:::message

上記コードでは、**デバッグモード有効時にはマウス操作を反映しない**ようなっていることに注目してください。

検証段階でマウスのコントロールを取ってしまうのは、PC 操作ができなくなる恐れがあり危険です。そのため、「デバッグでの検証」と「マウス操作への反映」は どちらか片方しか行えないようしておくべきと考えます。

:::

## まとめ

本項では、デバッグ用として ここまで取得したデータ内容を確認できるようモニタに出力する関数を作成しました。

次項は本章最後として、実際にマウス操作として反映する処理を作成します。

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

// マウス操作での処理情報（`ACTION_NONE`, `ACTION_PRESS`, `ACTION_RELEASE`）
enum MouseActionType { ACTION_NONE, ACTION_PRESS, ACTION_RELEASE };

#define MODE_CURSOR 0
#define MODE_WHEEL 1

const int BUTTON_C = 0;
const int BUTTON_Z = 1;
const int NUM_BUTTONS = 2;





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



// 平滑化された加速度情報
struct SmoothedAccelerometer {
  int accelX, accelY, accelZ;
  uint32_t accelSumX, accelSumY, accelSumZ;
  int accelBufX[ACCEL_SMOOTHING_SAMPLES];
  int accelBufY[ACCEL_SMOOTHING_SAMPLES];
  int accelBufZ[ACCEL_SMOOTHING_SAMPLES];
  int accelBufIndex;
  SmoothedAccelerometer() :
    accelX(ACCEL_CENTER_X), accelY(ACCEL_CENTER_Y), accelZ(ACCEL_CENTER_Z),
    accelSumX((uint32_t)ACCEL_CENTER_X * ACCEL_SMOOTHING_SAMPLES),  // `int * int = int` 回避のキャスト
    accelSumY((uint32_t)ACCEL_CENTER_Y * ACCEL_SMOOTHING_SAMPLES),
    accelSumZ((uint32_t)ACCEL_CENTER_Z * ACCEL_SMOOTHING_SAMPLES),
    accelBufIndex(0)
    {
      for(int i = 0; i < ACCEL_SMOOTHING_SAMPLES; ++i) {
        accelBufX[i] = ACCEL_CENTER_X;
        accelBufY[i] = ACCEL_CENTER_Y;
        accelBufZ[i] = ACCEL_CENTER_Z;
      }
    }
};



// マウスボタン操作の指示セット
// ※ マウスボタンと処理（押下/解放）で 1 セット
struct MouseAction {
  int button = 0; // MOUSE_LEFT, MOUSE_RIGHT など
  MouseActionType type = ACTION_NONE;
};

// マウスの状態を管理する構造体
struct MouseControl {
  int currentMode = MODE_CURSOR;  // 現在のモード

  // カーソルモード時のスティック関係
  int moveX = 0, moveY = 0;  // カーソル移動量（スティックの値を利用）

  // ホイールモード時のスティック関係
  int moveW = 0;  // ホイール移動量（※ スティックの Y 値を利用）
  int moveS = 0;  // ショートカットボタン用（※ スティック X 値を利用）
  int shortcutThrottling = 0;  // ショートカットボタン（進む、戻る）のスロットル用

  // ボタン状態
  bool buttonStatesCurrent[NUM_BUTTONS] = {false, false};  // 現在のボタン状態（押下/解放）
  bool buttonStatesLast[NUM_BUTTONS] = {false, false};  // 直近のボタン状態（押下/解放）

  // マウスボタン操作指示セットのキュー
  static const int MAX_ACTIONS = 4; // （1 回のループ処理での）最大指示数
  MouseAction actions[MAX_ACTIONS];
  int actionCount = 0;
};





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// グローバル変数・インスタンス (Global variables & instances)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// 各種構造体のインスタンス
NunchukInput nunchukInput;
SmoothedAccelerometer smoothedAccel;
MouseControl mouseControl;

// ヌンチャクのアドレス
const int NUNCHUK_ADDRESS = 0x52;

// ヌンチャクの通信判定
bool isNunchukConnected = false;

// デバッグ出力用のバッファ
char debugBuffer[200];




// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 関数プロトタイプ宣言 (Function prototypes)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

bool initializeNunchuk();
bool readNunchuk(NunchukInput& nunchukInput);
void smoothAccelerometer(SmoothedAccelerometer& smoothedAccel, int newAccelX, int newAccelY, int newAccelZ);
void updateMouse(const NunchukInput& nunchukInput, const SmoothedAccelerometer& smoothedAccel, MouseControl& mouseControl);
void printDebugInfo(const NunchukInput& nunchukInput, const SmoothedAccelerometer& smoothedAccel, const MouseControl& mouseControl);
int determineOperationMode(const SmoothedAccelerometer& smoothedAccel);
void addMouseAction(MouseControl& mouseControl, MouseActionType type, int button);
void onSwitchMode(MouseControl& mouseControl, int newMode);






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
      // 加速度を平滑化処理する
      smoothAccelerometer(smoothedAccel, nunchukInput.accelX, nunchukInput.accelY, nunchukInput.accelZ);

      // マウス操作情報を更新 (マウスボタン指示に関するイベント関係も、ここで管理)
      updateMouse(nunchukInput, smoothedAccel, mouseControl);

      // マウス操作情報を反映、またはデバッグ情報を出力
      if (ENABLE_DEBUG_MODE) {
        printDebugInfo(nunchukInput, smoothedAccel, mouseControl);
      } else {
        // マウス操作情報を実際に反映
      }
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

// === データ処理 ==================================================================

/**
 *  加速度データを平滑化するための関数
 *
 *  @param smoothedAccel SmoothedAccelerometer 構造体への参照
 *  @param newAccelX 新しい加速度 X 値
 *  @param newAccelY 新しい加速度 Y 値
 *  @param newAccelZ 新しい加速度 Z 値
 *  @note この関数は、加速度データを平滑化するために、過去のデータを使用して新しい値を計算します。
 *        平滑化のために、過去のサンプル数（`ACCEL_SMOOTHING_SAMPLES`）を使用します。
 */
void smoothAccelerometer(SmoothedAccelerometer& smoothedAccel, int newAccelX, int newAccelY, int newAccelZ) {
  // 平滑値を算出（最も古いデータを新規データに入れ替えて計算）
  smoothedAccel.accelSumX -= smoothedAccel.accelBufX[smoothedAccel.accelBufIndex];
  smoothedAccel.accelSumY -= smoothedAccel.accelBufY[smoothedAccel.accelBufIndex];
  smoothedAccel.accelSumZ -= smoothedAccel.accelBufZ[smoothedAccel.accelBufIndex];
  smoothedAccel.accelSumX += newAccelX;
  smoothedAccel.accelSumY += newAccelY;
  smoothedAccel.accelSumZ += newAccelZ;
  smoothedAccel.accelX = smoothedAccel.accelSumX / ACCEL_SMOOTHING_SAMPLES;
  smoothedAccel.accelY = smoothedAccel.accelSumY / ACCEL_SMOOTHING_SAMPLES;
  smoothedAccel.accelZ = smoothedAccel.accelSumZ / ACCEL_SMOOTHING_SAMPLES;

  // データ更新（最も古いデータを新規のデータに差し替え）
  smoothedAccel.accelBufX[smoothedAccel.accelBufIndex] = newAccelX;
  smoothedAccel.accelBufY[smoothedAccel.accelBufIndex] = newAccelY;
  smoothedAccel.accelBufZ[smoothedAccel.accelBufIndex] = newAccelZ;

  // インデックスの更新（サンプル数を超えると 0 に戻る）
  smoothedAccel.accelBufIndex = (smoothedAccel.accelBufIndex + 1) % ACCEL_SMOOTHING_SAMPLES;
}



// `updateMouse()` 用の計算済み定数（スティックに関する各種補正：遊び移動量、各軸の最小最大値）

// 遊び範囲となる移動量（入力された移動量が下回った場合、カーソル移動しない）
const int STICK_DEADZONE_SQ = pow(STICK_DEADZONE_X, 2) + pow(STICK_DEADZONE_Y, 2);

const int STICK_MIN_X = STICK_CENTER_X - STICK_RANGE_X;  // スティック X の最小値
const int STICK_MAX_X = STICK_CENTER_X + STICK_RANGE_X;  // スティック X の最大値
const int STICK_MIN_Y = STICK_CENTER_Y - STICK_RANGE_Y;  // スティック Y の最小値
const int STICK_MAX_Y = STICK_CENTER_Y + STICK_RANGE_Y;  // スティック Y の最大値

/**
 * マウスの状態を更新する関数
 *
 * 1. モードの決定と切り替え
 * 2. 物理ボタンの状態更新とマウスボタン処理
 * 3. カーソル及びホイール移動量の算出
 *
 * @param nunchukInput コントローラーの状態（`NunchukInput` 構造体）
 * @param smoothedAccel 平滑化された加速度情報（`SmoothedAccelerometer` 構造体）
 * @param mouseControl マウスの状態（`MouseControl` 構造体）
 */
void updateMouse(const NunchukInput& nunchukInput, const SmoothedAccelerometer& smoothedAccel, MouseControl& mouseControl) {
  // 指示セットのキューをリセット
  mouseControl.actionCount = 0;

  // 1. モード決定と切り替え
  //    現在の状態を算出し、変更があれば切り替え処理
  if (mouseControl.currentMode != determineOperationMode(smoothedAccel)) {
    onSwitchMode(mouseControl);
  }

  // 2. 物理ボタンの状態更新とマウスボタン処理
  //    現在の物理ボタン状態を更新し、直近の状態から変化があれば、マウスボタン処理を行う
  mouseControl.buttonStatesCurrent[BUTTON_C] = nunchukInput.buttonC;
  mouseControl.buttonStatesCurrent[BUTTON_Z] = nunchukInput.buttonZ;
  for (int i = 0; i < NUM_BUTTONS; i++) {
    // 処理対象となるマウスボタン（現モード時の物理ボタン i に割り振られたもの）
    int mouseButton = MOUSE_BUTTON_MAPS[mouseControl.currentMode][i];

    // 状態が変化（解放から押下、またはその逆）している場合は、対応するマウスボタン処理を指示リストに追加
    if ((mouseControl.buttonStatesCurrent[i] == true) && (mouseControl.buttonStatesLast[i] == false)){
      addMouseAction(mouseControl, ACTION_PRESS, mouseButton);
    } else if ((mouseControl.buttonStatesCurrent[i] == false) && (mouseControl.buttonStatesLast[i] == true)){
      addMouseAction(mouseControl, ACTION_RELEASE, mouseButton);
    }
    // 更新した物理ボタン状態を反映
    mouseControl.buttonStatesLast[i] = mouseControl.buttonStatesCurrent[i];
  }

  // 3. カーソル及びホイール移動量の算出
  //    スティックの傾きを移動量に変換し、設定項目の内容（遊び範囲, 最大移動量）から各種計算
  int stickDisplacementSq = pow(nunchukInput.stickX - STICK_CENTER_X, 2) + pow(nunchukInput.stickY - STICK_CENTER_Y, 2);
  if (stickDisplacementSq > STICK_DEADZONE_SQ) {
    // --- カーソルモード関係 ---

    // スティック情報は中心値 128 のため、0 中心に変換 ＋ 最大移動量に合わせた形で整形
    mouseControl.moveX = map(
      constrain(nunchukInput.stickX, STICK_MIN_X, STICK_MAX_X),
      STICK_MIN_X, STICK_MAX_X,
      -MOVEMENT_CURSOR, MOVEMENT_CURSOR
    );
    mouseControl.moveY = map(
      constrain(nunchukInput.stickY, STICK_MIN_Y, STICK_MAX_Y),
      STICK_MIN_Y, STICK_MAX_Y,
      -MOVEMENT_CURSOR, MOVEMENT_CURSOR
    ) * (-1); // `Mouse.move()` 用に反転処理

    // --- ホイールモード関係 ---

    // 入力に対するホイール方向
    int direction = ENABLE_SCROLL_REVERSE ? -1 : 1;

    // ホイール移動量は、スティック Y 値を利用
    mouseControl.moveW = map(
      constrain(nunchukInput.stickY, STICK_MIN_Y, STICK_MAX_Y),
      STICK_MIN_Y, STICK_MAX_Y,
      -MOVEMENT_WHEEL, MOVEMENT_WHEEL
    ) * direction;  // 反転モード時はマイナスを掛ける

    // ショートカットボタンの判定はスティック X 値を利用（ホイールモード時）
    mouseControl.moveS = map(
      constrain(nunchukInput.stickX, STICK_MIN_X, STICK_MAX_X),
      STICK_MIN_X, STICK_MAX_X,
      -1, 1    // 調整の必要はないので、±1 で固定
    );
  } else {
    // スティック移動量が遊び範囲内のため、移動量 0
    mouseControl.moveX = 0;
    mouseControl.moveY = 0;
    mouseControl.moveW = 0;
    mouseControl.moveS = 0;
  }
}



// === マウスイベント処理 ============================================================

/**
 * マウスの指示リストに、指示を追加する
 *
 *  @param mouseControl `MouseControl` 構造体への参照
 *  @param type マウス操作の指示情報（`ACTION_NONE`, `ACTION_PRESS`, `ACTION_RELEASE`）
 *  @param button 対応するマウスボタン（`MOUSE_LEFT`, `MOUSE_RIGHT`, `MOUSE_MIDDLE`）
 *  @note ここで設定した `mouseControl.actions[]` の内容が、最終的に `sendMouseEvents()` で反映される
 */
void addMouseAction(MouseControl& mouseControl, MouseActionType type, int button) {
  if (mouseControl.actionCount < MouseControl::MAX_ACTIONS) {
    mouseControl.actions[mouseControl.actionCount].type = type;
    mouseControl.actions[mouseControl.actionCount].button = button;
    mouseControl.actionCount++;
  }
}



/**
 * モード切替の処理を行う
 *
 * 1. モード切替時に 違うマウスボタンが割り当てられている場合は、適切に処理（押下と解放）
 * 2. `currentMode` の更新
 *
 * @param mouseControl `MouseControl` 構造体への参照
 */
void onSwitchMode(MouseControl& mouseControl) {
  int oldMode = mouseControl.currentMode;
  int newMode = (oldMode == MODE_CURSOR) ? MODE_WHEEL : MODE_CURSOR;

  // 各物理ボタンについて、ループ処理
  for (int i = 0; i < NUM_BUTTONS; i++) {
    // 押下し続けた状態でモードを切り替えた際、
    // 違うマウスボタンが割り振られている場合は、下記 2 つの指示を追加
    //   - 切替前のマウスボタンを "解放"
    //   - 切替後のマウスボタンを "押下"
    // （モード間で同じマウスボタンが割り振られている場合は、何もしない）
    if (mouseControl.buttonStatesCurrent[i] == true) {
      int oldMouseButton = MOUSE_BUTTON_MAPS[oldMode][i];
      int newMouseButton = MOUSE_BUTTON_MAPS[newMode][i];
      if (oldMouseButton != newMouseButton) {
        addMouseAction(mouseControl, ACTION_RELEASE, oldMouseButton);
        addMouseAction(mouseControl, ACTION_PRESS, newMouseButton);
      }
    }
  }
  // 現在のモードを更新
  mouseControl.currentMode = newMode;
}

// === ヘルパー関数 ===============================================================

/**
 * 2つの空間ベクトルのなす角を計算します。
 *
 * @param v1x, v1y, v1z ベクトル1の成分
 * @param v2x, v2y, v2z ベクトル2の成分
 * @return 角度（度単位）または `NAN`（ゼロベクトルの場合等）
 */
float angleBetweenTwoVectors(int v1x, int v1y, int v1z, int v2x, int v2y, int v2z) {
  // ゼロベクトルの場合は NAN
  if ((v1x == 0 && v1y == 0 && v1z == 0) || (v2x == 0 && v2y == 0 && v2z == 0)) {
    return NAN;
  }

  // ベクトルの大きさと内積を計算する
  float dotProduct = (float)v1x * v2x + (float)v1y * v2y + (float)v1z * v2z;
  float magV1 = sqrt(pow((float)v1x, 2) + pow((float)v1y, 2) + pow((float)v1z, 2));
  float magV2 = sqrt(pow((float)v2x, 2) + pow((float)v2y, 2) + pow((float)v2z, 2));

  // ベクトルの大きさが 0 の場合は NAN
  if (magV1 == 0 || magV2 == 0) return NAN;

  // 空間ベクトルの為す角から cosθ を算出し、角度に変換して返す
  float cosTheta = dotProduct / (magV1 * magV2);
  cosTheta = constrain(cosTheta, -1.0, 1.0);
  return acos(cosTheta) * 180.0 / PI;
}



/**
 *  モード切替のための関数
 *
 * コントローラーの向きと指定方向との角度を算出し、その角度が閾値以下であれば `MODE_WHEEL`、そうでなければ `MODE_CURSOR` を返す
 *
 *  @param smoothedAccel `SmoothedAccelerometer` 構造体への参照
 *  @return モード番号（`MODE_CURSOR` または `MODE_WHEEL`）
 */
int determineOperationMode(const SmoothedAccelerometer& smoothedAccel) {
  int targetVecX = 0, targetVecY = 0, targetVecZ = 0;  // 指定する向きの各ベクトル情報（`TARGET_AXIS` によって変化）
  switch(TARGET_AXIS) {
    case AXIS_X_NEG: targetVecX = -1; break;
    case AXIS_X_POS: targetVecX =  1; break;
    case AXIS_Y_NEG: targetVecY = -1; break;
    case AXIS_Y_POS: targetVecY =  1; break;
    case AXIS_Z_NEG: targetVecZ = -1; break;
    case AXIS_Z_POS: targetVecZ =  1; break;
  }

  // コントローラーの向きと指定方向との角度を算出（加速度情報は、中心値 0 にすることでベクトルとして扱う）
  float angle = angleBetweenTwoVectors(
    smoothedAccel.accelX - ACCEL_CENTER_X, smoothedAccel.accelY - ACCEL_CENTER_Y, smoothedAccel.accelZ - ACCEL_CENTER_Z,
    targetVecX, targetVecY, targetVecZ
  );

  // 算出した角度がマージン域内であれば、ホイールモードに切り替える
  if (!isnan(angle) && angle <= MODE_SWITCH_ANGLE_MARGIN) {
    if(ENABLE_LED) digitalWrite(LEDPIN, HIGH);
    return MODE_WHEEL;
  } else {
    if(ENABLE_LED) digitalWrite(LEDPIN, LOW);
    return MODE_CURSOR;
  }
}

// === デバッグ ==================================================================

/**
 * デバッグ用の、シリアルモニターに状態を出力する処理
 * 出力例：`S(128,186) -> Mv(-1, 5, 1, 0) | A( 532, 320, 724) -> As( 524, 323, 723) -> X_POS | Mode:0 | C:1 Z:0`
 *
 * @param nunchukInput コントローラーの状態（`NunchukInput` 構造体）
 * @param smoothedAccel 平滑化された加速度情報（`SmoothedAccelerometer` 構造体）
 * @param mouseControl マウスの状態（`MouseControl` 構造体）
 * @note この関数は、デバッグモードが有効な場合にのみ呼び出されます。各出力の形式は下記のとおりです。
 *     - `S(スティックX, スティックY)` : スティックの位置
 *     - `Mv(マウス移動X, マウス移動Y, マウス移動W, マウス移動S)` : マウスの移動量（W,S はホイールモード時の挙動）
 *     - `A(加速度X, 加速度Y, 加速度Z)` : 生の加速度情報
 *     - `As(平滑化加速度X, 平滑化加速度Y, 平滑化加速度Z)` : 平滑化された加速度情報
 *    - `X_POS` : 現在の向き（3 軸 + 正負 を4文字で表現）
 *     - `Mode:モード番号` : 現在のモード
 *     - `C:ボタンC状態` : 物理ボタンCの状態（0:解放, 1:押下）
 *     - `Z:ボタンZ状態` : 物理ボタンZの状態（0:解放, 1:押下）
 */
void printDebugInfo(const NunchukInput& nunchukInput, const SmoothedAccelerometer& smoothedAccel, const MouseControl& mouseControl) {

  // ここで出力する、デバイス向き一覧
  String names[6] = {"X_Neg", "X_Pos", "Y_Neg", "Y_Pos", "Z_Neg", "Z_Pos"};

  // 平滑化加速度をベクトル情報にし、6 方向（XYZ の各軸±）との角度算出
  int vecX = smoothedAccel.accelX - ACCEL_CENTER_X;
  int vecY = smoothedAccel.accelY - ACCEL_CENTER_Y;
  int vecZ = smoothedAccel.accelZ - ACCEL_CENTER_Z;
  float angles[6] = {
    angleBetweenTwoVectors(vecX, vecY, vecZ, -1, 0, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 1, 0, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, -1, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, 1, 0),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, 0, -1),
    angleBetweenTwoVectors(vecX, vecY, vecZ, 0, 0, 1)
  };

  // 6要素の内、最小な値（≒ デバイス向き）を探す
  //（ここではインデックスを用意するまで、使うのは `sprintf` 側）
  int minIndex = 0;
  for (int i = 1; i < 6; i++) {
    if (angles[i] < angles[minIndex]) minIndex = i;
  }

  sprintf(debugBuffer, "S(%3d,%3d) -> Mv(%2d,%2d,%2d,%2d) | A(%4d,%4d,%4d) -> As(%4d,%4d,%4d) -> %4s | Mode:%d | C:%d Z:%d",
          nunchukInput.stickX, nunchukInput.stickY,
          mouseControl.moveX, mouseControl.moveY, mouseControl.moveW, mouseControl.moveS,
          nunchukInput.accelX, nunchukInput.accelY, nunchukInput.accelZ,
          smoothedAccel.accelX, smoothedAccel.accelY, smoothedAccel.accelZ,
          names[minIndex].c_str(),
          mouseControl.currentMode,
          nunchukInput.buttonC, nunchukInput.buttonZ
          );
  Serial.println(debugBuffer);
}
```

:::
