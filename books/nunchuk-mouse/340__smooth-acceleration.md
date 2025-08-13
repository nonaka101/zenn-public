---
title: "📄コード：加速度の平滑化"
---

## はじめに

本項では下記の事項について見ていきます。

- コントローラーから取得した加速度情報を確認
- 上記情報の問題点を把握
- 対応策としての平滑化がどんなものかを確認
- 実際のコードに落とし込む

## 加速度の平滑化

ここでは加速度に関する問題から、その対応策までの一連を扱っていきます。

### コントローラーから取得したデータの問題点

#### コントローラーからのデータを確認

まずはコントローラーから取得したデータを確認してみます。下図の要素 `A` が、コントローラーから直接取得した **生データ** になります。

![デバッグモードで各種情報がシリアルモニタに出力されている](/images/books/nunchuk-mouse/ide-debug-00.gif)
*各種情報の中で 加速度(`A`) に注目*

#### 生データの問題点

このデータの問題点は **1 回あたりの計測誤差が大きい**ということです。上図は実機をゆっくりと動かしたものですが、それでも結構な割合で 10 以上の変動を確認することができます。

この計測誤差の何が問題かと言うと、**モード切替の境界線上にデバイスがある時**です。下図はモードが切り替わるギリギリの位置で、モード切替の判定に 生データを そのまま使った場合のデバッグ出力です。

![サンプル数1でシリアルモニタに表示、モードが1秒の間に頻繁に切り替わっている](/images/books/nunchuk-mouse/acceleration-01.gif)
*各種情報の中で モード(`Mode`) に注目*

上図では、1 秒間で頻繁にモードが切り替わっているのが確認できます。計測誤差によるブレ幅が、そのままモード切替に現れているということです。

この切り替わりは**ユーザーの感覚では把握できない**ものなので、意図せぬ動作に繋がってしまいます。

### 移動平均による、データの平滑化

前述の内容から、計測誤差を少なくする必要が確認できました。計測誤差が少ないということはつまり、**滑らかな移り変わりであること**が求められているわけです。

この「データを滑らかにするための処理」を**平滑化**（*smoothing*）と呼ぶことにします。今回は**移動平均**による平滑化処理を使います。

#### 移動平均とは

**単純移動平均（*Simple Moving Average* : SMA）** では、過去の**一定期間内での平均値を計算**することに特徴があります。

計算方法は非常にシンプルで、指定した期間（例えば 5 回）の合計を、その期間の回数で割るだけです。つまり、指定期間内の平均値を算出するわけです。

ある期間 `n` における単純移動平均（*SMA*）を計算する数式は以下の通りです。`P` は各時点での値を指し、`n` は計測する期間（数）を指します。

$$
SMA = \frac{P_1 + P_2 + \dots + P_n}{n}
$$

例えば、**5 回分のデータを使った移動平均**を計算する場合、直近 5 回分の値を合計し、5で割ります。その次の回では、**一番古いデータを除外し、代わりに最新のデータを加えて**再度計算します。これを繰り返しグラフにプロットすれば、平滑化された移動平均線が描画されるわけです。

##### 移動平均のメリット・デメリット

移動平均のメリットは、平滑化する計算の仕組みがシンプルで、非常にわかりやすいことです。

一方でデメリットとしては、反応の鈍化です、平滑化する以上 当然の話ですが、1 回で計測される急激な変化というのはなくなりますし、値の変化する速度はゆっくりになります。特にこれは、移動平均で使う期間が長くなるほど顕著です。

## 平滑化する一連の処理を作成

ここからはコードとして、先程説明した平滑化処理を実装していきます。なお、ここで使用する設定項目は下記になります。

|定数名|型|内容|
|---|---|:---|
|`ACCEL_CENTER_X`|`int`|加速度 X 軸の中心値|
|`ACCEL_CENTER_Y`|`int`|加速度 Y 軸の中心値|
|`ACCEL_CENTER_Z`|`int`|加速度 Z 軸の中心値|
|`ACCEL_SMOOTHING_SAMPLES`|`int`|加速度を平滑化するためのサンプリング数|

### 平滑化済みの加速度を管理する構造体を作成

まずは平滑化した加速度情報を管理するための構造体を設けることにします。

:::message

ポータビリティ性確保の観点から、前項で作成したコントローラーからの入力情報を管理する構造体 `NunchukInput` とは独立した形にします。

平滑化はあくまで本リポジトリの目的（マウスとして動作させる）に対して、センサーの生データが扱いづらいという事情から行う処理となります。そのため `NunchukInput` は**他のケースでも扱えるように、生のデータをそのまま管理**する形式にしたいという意図があります。

`NunchukInput` で加速度やスティック入力情報が 0 中心でない符号なしデータにしたままというのも、そうした観点からの判断です。

:::

#### 構造体の定義

まずは構造体を定義します。構造体名は `SmoothedAccelermeter` としておきます。

```cpp:構造体の定義
// 平滑化された加速度情報
struct SmoothedAccelerometer {
  // 各軸（X, Y, Z）に対し、下記データを管理
  // - 指定した期間分の計測データ
  // - 平滑化した加速度情報（計測データを平均化することで算出）
};

// インスタンス生成
SmoothedAccelerometer smoothedAccel;
```

#### 管理する各種データの定義

次に構造体内で管理する各種データを定義していきます。必要となるデータは下記になります。

- 平滑化済み加速度 `accelX`, `accelY`, `accelZ`
- 指定期間分のデータ `accelBufX[]`, `accelBufY[]`, `accelBufZ`
  - データの合計 `accelSumX`, `accelSumY`, `accelSumZ`
  - 配列のインデックス管理 `accelBufIndex`

指定した期間の計測データは、要素数 `ACCEL_SMOOTHING_SAMPLES` の配列として定義することになります。期間を定義するサンプル数を、設定項目である `ACCEL_SMOOTHING_SAMPLES` で管理している形です。

そして配列のインデックス管理用に `accelBufIndex` という数値型の変数も用意しておきます。

`accelSumX` のように 合計値を変数で管理する理由は、都度配列から合計値を算出する手間を除くためです。

ここまでをコードに起こすと、下記のようになります。

```cpp:データ定義
struct SmoothedAccelerometer {
  int accelX, accelY, accelZ;
  uint32_t accelSumX, accelSumY, accelSumZ;
  int accelBufX[ACCEL_SMOOTHING_SAMPLES];
  int accelBufY[ACCEL_SMOOTHING_SAMPLES];
  int accelBufZ[ACCEL_SMOOTHING_SAMPLES];
  int accelBufIndex;
};
```

加速度データに関しては、`0` 〜 `1024` の範囲の 10bit データなので `int` で定義します。一方 合計値に関しては `ACCEL_SMOOTHING_SAMPLES` 次第で値が `int` からオーバーフローする可能性があるため、`uint32_t` で定義しています。

データ構造をまとめると、下表のようになります。

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

#### 初期化で仮の値を埋める

ここまでで構造体の大枠が決まりましたが、現段階では定義を済ませただけで中身がない状態です。

この状況では本処理の中で「期間データが埋まっているか」を管理しなければならず、使いづらいことが想定されます。

そこで初期化子リスト `:` を使い、期間データ等を仮データで埋め、即時使える状態にしておきます。

- 平滑化済み加速度（`accel*`）は、中央値 `ACCEL_CENTER_*` で仮埋めしておく
- 期間データ（`accelBuf*[]`）は、中央値 `ACCEL_CENTER_*` で要素全てを埋める
- 合計値 `accelSum*` は、上記から `ACCEL_CENTER_*` にサンプル数 `ACCEL_SMOOTHING_SAMPLES` を掛けた値で設定
- インデックス管理（`accelBufIndex`）は、`0` に設定

```cpp
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
```

これでインスタンス生成時に各種データが仮埋めされた状態になり、扱いやすい状態になっているという寸法です。

:::details 補足：uint32_t でキャストしている理由

初期化の処理で、当初 `accelSum*` の部分は `(uint32_t)` でキャストしていない状態でしたが、検証で動作がおかしくなる問題が発生しました。具体的には `ACCEL_SMOOTHING_SAMPLES` を 63 以上に設定すると、加速度データが固まってしまいました。

```cpp:構造体 SmoothedAccelerometer のコンストラクタ
SmoothedAccelerometer() :
  accelX(ACCEL_CENTER_X), accelY(ACCEL_CENTER_Y), accelZ(ACCEL_CENTER_Z),
  accelSumX(ACCEL_CENTER_X * ACCEL_SMOOTHING_SAMPLES), // ← 問題の箇所
  accelSumY(ACCEL_CENTER_Y * ACCEL_SMOOTHING_SAMPLES), // ← 問題の箇所
  accelSumZ(ACCEL_CENTER_Z * ACCEL_SMOOTHING_SAMPLES), // ← 問題の箇所
  accelBufIndex(0)
  {
    // ... (省略) ...
  }
```

調べた結果、合計値を管理する `accelSum*`（`uint32_t` 型）がオーバーフローを起こしていることが原因とわかりました。

確かに `accelSum*` 自体は `uint32_t` で定義していますが、一方でそこに格納する演算式自体は `int * int` です。`int * int` の演算結果は `int` であり、この段階でオーバーフローが発生してしまいます。その結果、このオーバーフローを起こしたデータを `accelSum*` への格納しているために、検証がおかしくなったと考えられます。

対策としては、計算結果が `uint32_t` となるよう あらかじめキャストを施しています。

```cpp:キャストして演算結果が uint32_t になるよう修正
  accelSumX((uint32_t)ACCEL_CENTER_X * ACCEL_SMOOTHING_SAMPLES),
  accelSumY((uint32_t)ACCEL_CENTER_Y * ACCEL_SMOOTHING_SAMPLES),
  accelSumZ((uint32_t)ACCEL_CENTER_Z * ACCEL_SMOOTHING_SAMPLES),
```

:::

### 加速度情報から平滑化する関数を作成

ここでは前述の構造体 `SmoothedAccelerometer` を使って、加速度情報を平滑化処理する関数を作成します。

#### 関数の定義

関数名を `smoothAccelerometer` とし、引数に計算に使う加速度情報（≒コントローラーで計測された値）と、先程作成した構造体 `SmoothedAccelerometer` を受け取るようにします。構造体はポインタで受け取り、関数内でデータの読み書きを行えるようにしておきます。

```cpp:関数の定義
void smoothAccelerometer(SmoothedAccelerometer& smoothedAccel, int newAccelX, int newAccelY, int newAccelZ) {
  // 平滑値を算出（最も古いデータを新規データに入れ替えて計算）
  // データ更新（最も古いデータを新規のデータに差し替え）
  // インデックスの更新（サンプル数を超えると 0 に戻る）
}
```

#### 平滑値の算出

最初に行うのは、平滑値 `accelX`, `accelY`, `accelZ` の算出です。これは合計値 `accelSum*` を**最新の状態に更新**し、それをサンプル数 `ACCEL_SMOOTHING_SAMPLES` で割ることで算出します。

「最新の状態に更新」とは、**古いデータ**を捨て**新しいデータ**を加える処理を指します。

古いデータというのは、期間データの中で最も古い情報を指します。ここでは、配列のインデックス `accelBufIndex` にある要素が対象です。

新しいデータは、コントローラーからの入力情報を指します。

```cpp:古いデータを捨て新しいデータを加えた合計値を平均することで、平滑値を算出
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
  // インデックスの更新（サンプル数を超えると 0 に戻る）
}
```

#### データ更新

次に期間データ（配列）の更新処理を行います。配列の中で最も古いデータというのは、インデックス番号 `accelBufIndex` で焦点が当たっている状態です。

その要素を、新しいデータ（≒ コントローラーからの入力情報）に置き換えることで、更新処理は完了です。

```cpp:配列内のデータを最新のものに更新
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
}
```

#### インデックス更新

配列データのインデックス番号 `accelBufIndex` は、最新の情報に更新されました。この段階で最も古いデータというのは、現在のインデックスに **+1** されたものに置き換わります。

そこでインデックスを管理している `accelBufIndex` を 1 つずらす処理を行い更新します。

ただし `smoothedAccel.accelBufIndex = smoothedAccel.accelBufIndex + 1` ではダメです。仮に `accelBufX[]` が要素数 10 の場合、インデックス `accelBufIndex` は **9 の次は 0 にならなければならない**ためです。

ここではサンプル数 `ACCEL_SMOOTHING_SAMPLES` での剰余を使うことで、インデックスを最大値から 0 に戻す処理を実現します。

```cpp:インデックスの更新
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
```

これで加速度情報の平滑化処理する関数は完成です。

### メイン処理の中に組み込む

構造体と平滑化処理関数が完成したので、最後にこれらをメイン関数の中に組み込みます。

組み込む場所は、`readNunchuk()` が正常に処理できた状況下です。この段階ではインスタンス `nunchukInput` に適切なデータが収められている状態ですので、そこから計測値を引っ張ってくることができます。

```cpp
// セットアップ後、常時処理される関数
void loop() {
  if(isNunchukConnected){
    // コントローラーから入力を読み取る（正常に読み取れたかで分岐）
    if(readNunchuk(nunchukInput)){
      // 加速度を平滑化処理する
      smoothAccelerometer(smoothedAccel, nunchukInput.accelX, nunchukInput.accelY, nunchukInput.accelZ);

    } else {
      // 省略：読み取り失敗時の処理
    }
  } else {
    // 省略：通信エラー時（未接続・通信断）の処理
  }
  delay(MOUSE_DELAY);
}
```

## まとめ

本項では、加速度情報の詳細を見ていきました。そして扱いづらいという問題に対処するための平滑化処理について説明し、コードの中に落とし込んでいきました。

次項では ここで用意した加速度情報を使い、デバイス向きを算出する仕組みを実装していきます。

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





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// グローバル変数・インスタンス (Global variables & instances)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

// 各種構造体のインスタンス
NunchukInput nunchukInput;
SmoothedAccelerometer smoothedAccel;

// ヌンチャクのアドレス
const int NUNCHUK_ADDRESS = 0x52;

// ヌンチャクの通信判定
bool isNunchukConnected = false;





// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢
// 関数プロトタイプ宣言 (Function prototypes)
// ◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢◤◢

bool initializeNunchuk();
bool readNunchuk(NunchukInput& nunchukInput);
void smoothAccelerometer(SmoothedAccelerometer& smoothedAccel, int newAccelX, int newAccelY, int newAccelZ);





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
```

:::
