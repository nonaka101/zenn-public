---
title: "📄コード：マウス操作の反映"
---

## はじめに

本項では、ここまで集めたデータを実際のマウス操作に反映する関数 `sendMouseEvents()` を作成していきます。

## マウス操作の反映

`MouseControl` の中身を実際の操作へと反映させる関数 `sendMouseEvents()` を作成していきます。

なお 本項で関係してくる設定項目は、下表のとおりです。

|定数名|型|内容|
|---|---|:---|
|`IS_MACINTOSH`|`bool`|接続先が Macintosh か|
|`ENABLE_SHORTCUT_FUNCTIONS`|`bool`|ホイールモード時にショートカットを有効にするか|
|`SHORTCUT_WAIT_TIME`|`int`|ショートカット入力の待機時間|

### 関数の定義

関数名は `sendMouseEvents`、引数にはマウス操作を管理する構造体 `MouseControl` を受け取るようにします。

```cpp:関数の定義
void sendMouseEvents(MouseControl& mouseControl) {
  // 1. マウスボタン系の操作
  // 2. 各種移動系の操作
  //    - カーソル
  //    - ホイール
  //    - ショートカット（戻る/進む）
}
```

この関数は、[前項](./380__print-debug-info)で用意した `loop` 関数内の条件分岐（デバッグモードか、マウス操作を反映か）に組み込みます。

```cpp:デバッグモードが無効の場合にマウス操作を反映させる
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
        sendMouseEvents(mouseControl);
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

なお この段階において、`MouseControl` には下記のデータが格納されている状態です。

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

### マウスボタン系の操作

まずはマウスボタン関係を処理します。

これは `actions[4]` の中に格納された指示セット（`MouseAction`）を順々に取り出し、処理していきます。

コード的には、下記のような形です。

```cpp:配列要素をループで処理する
for (int i = 0; i < mouseControl.actionCount; i++) {
  // 1. `actions[4]` にある `MouseAction` を取り出す
  // 2. `MouseAction.type` の内容を確認する
  //   - `ACTION_PRESS` の場合は、`MouseAction.button` に割り振られたボタンを押下状態に
  //   - `ACTION_RELEASE` ならば、`MouseAction.button` に割り振られたボタンを解放状態に
}
```

`MouseAction` の取り出しは `i` を使い、`mouseControl.actions[i]` で OK です。なので関数内に組み込むと、下記のようになります。

```cpp:マウスボタン系の処理
void sendMouseEvents(MouseControl& mouseControl) {
  // マウスボタン操作
  for (int i = 0; i < mouseControl.actionCount; i++) {
    const MouseAction& action = mouseControl.actions[i];

    if (action.type == ACTION_PRESS) {
      Mouse.press(action.button);
    } else if (action.type == ACTION_RELEASE) {
      Mouse.release(action.button);
    }
  }

  // 移動操作
}
```

これでマウスボタン系の操作は完了です。

### 各種移動系の操作

次に、各種移動関係の操作を行なっていきます。これには下表の操作が含まれてます。

|操作|操作が反映されるモード|
|:---|:---|
|カーソルを移動|カーソルモード|
|ホイールスクロール|ホイールモード|
|ショートカットボタン関係|ホイールモード|

そのために、現在のモードによる条件分岐を設けておきます。下記がその大枠になります。

```cpp:モードによる処理の分岐
void sendMouseEvents(MouseControl& mouseControl) {
  // 省略：マウスボタン系の操作

  // 各種移動系の操作（カーソル、ホイール、ショートカット操作）

  // モードに応じた操作反映を実行
  if (mouseControl.currentMode == MODE_CURSOR) {
    // カーソルモード時の処理
    // 1. カーソルを移動 
  } else {
    // ホイールモード時の処理
    // 2. ホイールスクロール
    // 3. ショートカットボタン関係
  }
}
```

#### カーソル移動

まずは、カーソル移動です。これは `Mouse.move()` 関数を使います。

https://docs.arduino.cc/language-reference/en/functions/usb/Mouse/mouseMove/

第一、第二引数に `moveX`, `moveY` を渡すことで、カーソル移動ができます。第三引数はホイール移動なので、ここでは 0 固定です。

```cpp:カーソルの移動
void sendMouseEvents(MouseControl& mouseControl) {
  // 省略：マウスボタン系の操作

  // 各種移動系の操作（カーソル、ホイール、ショートカット操作）

  // モードに応じた操作反映を実行
  if (mouseControl.currentMode == MODE_CURSOR) {
    // カーソルモード時の処理
    Mouse.move(mouseControl.moveX, mouseControl.moveY, 0);
  } else {
    // ホイールモード時の処理
    // 2. ホイールスクロール
    // 3. ショートカットボタン関係
  }
}
```

#### ホイール移動

次はホイール移動です。これはカーソル移動で使った `Mouse.move()` 関数の第三引数を使うことで実装できます。

```cpp:ホイール移動の処理
Mouse.move(0, 0, mouseControl.moveW);
```

ただしここで問題になってくるのは、ホイールモードは**上下がホイール移動**で**左右がショートカット機能**に割り振っていることです。

なので、**縦方向と横方向のどちらに強く傾けているかで、どちらの処理を行うかを分岐処理させなければなりません**。  
（※ 斜めに傾けて どちらの処理も行なってしまうのは、操作的には望ましくなく、避けなければなりません。）

そこでカーソル移動用の `moveX` と `moveY` を使い、どちらに強く傾けているかを判定させます。

```cpp:どちらに強く傾けているかで分岐
void sendMouseEvents(MouseControl& mouseControl) {
  // 省略：マウスボタン系の操作

  // 各種移動系の操作（カーソル、ホイール、ショートカット操作）

  // モードに応じた操作反映を実行
  if (mouseControl.currentMode == MODE_CURSOR) {
    Mouse.move(mouseControl.moveX, mouseControl.moveY, 0);
  } else {
    // ホイールモード
    // XY どちらの方向に強く傾けているかで、スクロール・ボタン操作に分岐
    if (abs(mouseControl.moveY) > (abs(mouseControl.moveX) + 1)){

      // "縦"方向に強く傾けている -> ホイールスクロール処理

    } else if(abs(mouseControl.moveX) > (abs(mouseControl.moveY) + 1)) {

      // "横"方向に強く傾けている -> ショートカット操作

    }
  }
}
```

:::message

強めに傾けているかについては、下記のように判断しています。

- 「**縦方向**の判断」では 横方向に `+1` して、それより 縦方向の値 が強いか
- 「**横方向**の判断」では 縦方向に `+1` して、それより 横方向の値 が強いか

:::

これで分岐ができたので、ホイール操作を適切な位置に組み込みます。

```cpp
void sendMouseEvents(MouseControl& mouseControl) {
  // 省略：マウスボタン系の操作

  // 各種移動系の操作（カーソル、ホイール、ショートカット操作）

  // モードに応じた操作反映を実行
  if (mouseControl.currentMode == MODE_CURSOR) {
    Mouse.move(mouseControl.moveX, mouseControl.moveY, 0);
  } else {
    // ホイールモード
    // XY どちらの方向に強く傾けているかで、スクロール・ボタン操作に分岐
    if (abs(mouseControl.moveY) > (abs(mouseControl.moveX) + 1)){
      // ホイールスクロール
      Mouse.move(0, 0, mouseControl.moveW);
    } else if(abs(mouseControl.moveX) > (abs(mouseControl.moveY) + 1)) {

      // "横"方向に強く傾けている -> ショートカット操作

    }
  }
}
```

#### ショートカット操作

最後に残ったのは、ショートカット操作（戻る/進む）です。

##### ショートカットの発動条件

まずはショートカットの操作が発動できる状況についてです。これは設定項目 `ENABLE_SHORTCUT_FUNCTION` が `true` で、かつ `mouseControl.shortcutThrottling` の値が `0` である場合です。

```cpp:条件を満たしていればショートカットを発動
void sendMouseEvents(MouseControl& mouseControl) {
  // 省略：マウスボタン系の操作

  // 各種移動系の操作（カーソル、ホイール、ショートカット操作）

  // モードに応じた操作反映を実行
  if (mouseControl.currentMode == MODE_CURSOR) {
    Mouse.move(mouseControl.moveX, mouseControl.moveY, 0);
  } else {
    // ホイールモード
    // XY どちらの方向に強く傾けているかで、スクロール・ボタン操作に分岐
    if (abs(mouseControl.moveY) > (abs(mouseControl.moveX) + 1)){
      // ホイールスクロール
      Mouse.move(0, 0, mouseControl.moveW);
    } else if(abs(mouseControl.moveX) > (abs(mouseControl.moveY) + 1)) {
      // ショートカットボタン（設定で有効にしている場合に機能）
      // 待機状態終了時、左なら「戻る」、右なら「進む」を実行
      if (
        (ENABLE_SHORTCUT_FUNCTIONS) &&
        (mouseControl.shortcutThrottling == 0)
      ){

        // ショートカットの発動

      }
    }
  }
}
```

##### ショートカットの処理

ショートカット機能の処理は、`Keyboard` ライブラリを使い、ショートカットキーの入力を行うことで「戻る」「進む」機能を実行させます。

各ショートカットキーは OS によって微妙に違い、Mac と それ以外（Win など）に分かれます。

|OS|ショートカットキー「戻る」|ショートカットキー「進む」|
|---|---|---|
|Mac|`Command` + `[`|`Command` + `]`|
|Win など|`Alt` + `←`|`Alt` + `→`|

これはつまり、「OS」「方向（戻る/進む）」「キー」の 3 つを管理する必要があるということです。そこで 3 次元関数で管理することにします。

```cpp
// キーマップ（OS と アクション内容別で、キー 2 つを管理）
const int KEY_MAP[2][2][2] = {
  {    // Win, Linux 系
    {KEY_LEFT_ALT, KEY_LEFT_ARROW},  // <-「戻る」
    {KEY_LEFT_ALT, KEY_RIGHT_ARROW}  // <-「進む」
  },{  // Mac
    {KEY_LEFT_GUI, '['},  // <-「戻る」
    {KEY_LEFT_GUI, ']'}   // <-「進む」
  }
};
```

最初の添字で OS、2 番目の添字で 方向、3番目の添字で キー を管理しています。

OS に関しては、設定項目 `IS_MACINTOSH` によって決めることができます。

```cpp:OSに応じて1番目の添字を決定
int osIndex = IS_MACINTOSH ? 1 : 0;
```

2 番目の方向に関しては、スティックをどちらに傾けたかで判定します。ここでは `moveS` の値に応じて判定をします。

```cpp:スティックの傾けに応じて2番目の添字を決定
int actionIndex = (mouseControl.moveS < 0) ? 0 : 1;
```

これらの添字を使うことで、キーを取り出すことができます。取り出したキーは、`Keyboard.press(キー)` で押下状態にすることができます。

ショートカットに必要なキーを押下状態にすれば、ショートカットを発動させることができます。

押下したキーの解放は `Keyboard.releaseAll()` で可能です。

ここまでの内容を詰めたものが、下記になります。

```cpp
// キーマップ（OS と アクション内容別で、キー 2 つを管理）
const int KEY_MAP[2][2][2] = {
  {    // Win, Linux 系
    {KEY_LEFT_ALT, KEY_LEFT_ARROW},  // <-「戻る」
    {KEY_LEFT_ALT, KEY_RIGHT_ARROW}  // <-「進む」
  },{  // Mac
    {KEY_LEFT_GUI, '['},  // <-「戻る」
    {KEY_LEFT_GUI, ']'}   // <-「進む」
  }
};

int osIndex = IS_MACINTOSH ? 1 : 0;



void sendMouseEvents(MouseControl& mouseControl) {
  // 省略：マウスボタン系の操作

  // 各種移動系の操作（カーソル、ホイール、ショートカット操作）

  // モードに応じた操作反映を実行
  if (mouseControl.currentMode == MODE_CURSOR) {
    Mouse.move(mouseControl.moveX, mouseControl.moveY, 0);
  } else {
    // ホイールモード
    // XY どちらの方向に強く傾けているかで、スクロール・ボタン操作に分岐
    if (abs(mouseControl.moveY) > (abs(mouseControl.moveX) + 1)){
      // ホイールスクロール
      Mouse.move(0, 0, mouseControl.moveW);
    } else if(abs(mouseControl.moveX) > (abs(mouseControl.moveY) + 1)) {
      // ショートカットボタン（設定で有効にしている場合に機能）
      // 待機状態終了時、左なら「戻る」、右なら「進む」を実行
      if (
        (ENABLE_SHORTCUT_FUNCTIONS) &&
        (mouseControl.shortcutThrottling == 0)
      ){
        // `mouseControl.moveS` の値に応じたアクションのインデックス（≒スティック左右）
        int actionIndex = (mouseControl.moveS < 0) ? 0 : 1;

        // OS及びマウス操作に基づいたキー処理
        Keyboard.press(KEY_MAP[osIndex][actionIndex][0]);
        Keyboard.press(KEY_MAP[osIndex][actionIndex][1]);
        Keyboard.releaseAll();
      }
    }
  }
}
```

##### 待機状態の管理

ショートカットを発動することができましたが、今の状態では 1 つ問題があります。`loop` 処理は高速で処理されるので、ショートカットを発動しようと傾けたら、恐らく十数回分の発動が生じてしまいます。

これを防ぐためには、一度発動したら**一定期間の待機時間を設ける**必要があります。

`Keyboard.releaseAll();` の時点で発動が完了しているので、この後ろに `mouseControl.shortcutThrottling = SHORTCUT_WAIT_TIME;` を追加し、待機時間を 0 から設定項目分 追加することにします。

以降は `sendMouseEvents()` の実行の度に 値を 1 ずつ減らしていくようしておきます。これが 0 になったら、再度ショートカット機能が発動可能となるわけです。

なので関数の最初の方に、`shortcutThrottling` を漸減させる処理も加えておいたのが、下記のコードとなります。

```cpp
void sendMouseEvents(MouseControl& mouseControl) {
  // ショートカット待機状態の更新（0 へと漸減し、0 になったら待機解除）
  if (mouseControl.shortcutThrottling != 0){
    mouseControl.shortcutThrottling--;
  }

  // 省略：マウスボタン系の操作

  // 各種移動系の操作（カーソル、ホイール、ショートカット操作）

  // モードに応じた操作反映を実行
  if (mouseControl.currentMode == MODE_CURSOR) {
    Mouse.move(mouseControl.moveX, mouseControl.moveY, 0);
  } else {
    // ホイールモード
    // XY どちらの方向に強く傾けているかで、スクロール・ボタン操作に分岐
    if (abs(mouseControl.moveY) > (abs(mouseControl.moveX) + 1)){
      // ホイールスクロール
      Mouse.move(0, 0, mouseControl.moveW);
    } else if(abs(mouseControl.moveX) > (abs(mouseControl.moveY) + 1)) {
      // ショートカットボタン（設定で有効にしている場合に機能）
      // 待機状態終了時、左なら「戻る」、右なら「進む」を実行
      if (
        (ENABLE_SHORTCUT_FUNCTIONS) &&
        (mouseControl.shortcutThrottling == 0)
      ){
        // `mouseControl.moveS` の値に応じたアクションのインデックス（≒スティック左右）
        int actionIndex = (mouseControl.moveS < 0) ? 0 : 1;

        // OS及びマウス操作に基づいたキー処理
        Keyboard.press(KEY_MAP[osIndex][actionIndex][0]);
        Keyboard.press(KEY_MAP[osIndex][actionIndex][1]);
        Keyboard.releaseAll();

        // ボタン発動したので、（一定時間）待機状態に移行
        mouseControl.shortcutThrottling = SHORTCUT_WAIT_TIME;
      }
    }
  }
}
```

これで各種移動系の操作は完了しました。

## まとめ

本項では、実際にマウス操作に反映する関数 `sendMouseEvents()` を作成しました。これで本リポジトリの一連のコードの説明が終わりとなります。

:::message

コードに関してはこれで完成のため、「[⭐附録：コード全文](./900__code.md)」を参照ください。

:::
