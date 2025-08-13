---
title: "📄コード：向きの計算"
---

## はじめに

本項では、加速度センサーのデータを利用してデバイスの向きを計算し、モード切替に利用するための理論と実装について解説します。

## 理論上の部分

ここでは実際の計算を行う前段階として、理論的な部分を説明します。

最初に加速度に関する説明をし、そこから各軸の成分をどうやって抽出するかを考えます。最後に空間ベクトルの為す角を使って、2つのベクトルデータの角度を算出できるようにします。

### 加速度と加速度センサーについて

加速度とは、物体が動き始めたり停止する際にかかる力を指します。加速度センサーはこの力（エネルギー）を検知・計測するものです。

また 地球上では重力が存在し、中央に向かって常にかかり続ける この力を重力加速度と言います。この重力加速度を加速度センサーで計測することで、センサーがどの程度傾いているかを検知することが可能です。

加速度センサーは 1 軸のものもありますが、主流となっているのは 3 軸のセンサーです。3軸（X, Y, Z）あれば、加速度の大小だけでなく、どの方向に加速しているか、重力方向の算出等が可能になります。

### 各軸の成分抽出

ここでは加速度センサーの各軸の成分を計算する方法について説明します。

:::message

このあたりの話は、直接コードに関わるようなものではありません。

加速度情報から利用可能なデータを計算するサンプルとして掲載しています。

なお、詳しい話は "*Three-Axis Accelerometer for Tilt Sensing*" 等で調べるといくつか出てくると思いますので、そちらを参照願います。

:::

加速度センサーには常に重力方向への力が加わっている状態です。それを利用すれば、重力方向の算出が可能となります。

算出については、X, Y, Z それぞれの軸に対し、どれくらいの重力（エネルギー）がかかっているかを使います。

角度 `θ` から辺の比 `b/a` を求めるには、下記のように正接関数を使って表現できます。

$$
\tan\theta = \frac{b}{a} \quad
$$

一方、辺の比 `b/a` から角度 `θ` を求めるためには、下記のように逆正接関数を使います。

$$
\quad \theta = \tan^{-1}\left(\frac{b}{a}\right)
$$

そして、X, Y, Z の各軸の傾き角の計算は下記になります。

$$
\theta_x = \tan^{-1}\times\frac{x}{\sqrt{y^2 + z^2}}
$$

$$
\theta_y = \tan^{-1}\times\frac{y}{\sqrt{x^2 + z^2}}
$$

$$
\theta_z = \tan^{-1}\times\frac{z}{\sqrt{x^2 + y^2}}
$$

コードにすると、下記のようになります。

```cpp
// sqrt は平方根、atan2 は逆正接関数の計算
// atan2 での結果はラジアン単位なので、最後に度数法に変換している
xAngle = (atan2(x, sqrt(y*y + z*z))) * 180.0 / PI;
yAngle = (atan2(y, sqrt(x*x + z*z))) * 180.0 / PI;
zAngle = (atan2(z, sqrt(x*x + y*y))) * 180.0 / PI;
```

このように、加速度情報と重力を組み合わせれば、デバイスの傾き角を算出することができます。

### 空間ベクトルの為す角

先程の例は、重力に対し各軸の成分を抽出するものでした。ここでは**重力方向**と**計測したい向き**の 2 つを線分で表し、その角度を計算する方法について考えていきます。

「重力方向」と「計測したい向き」の角度については、両者をベクトル情報と見立て、**空間ベクトルの為す角**として計算します。

「空間ベクトルの為す角」とは、2 つの 3 次元ベクトルを 1 つの平面上に とらえた際に計測できる角度を指します。

これには公式があり、例えば $\overrightarrow{a} = (a_1, a_2, a_3)$, $\overrightarrow{b} = (b_1, b_2, b_3)$ のベクトルデータがある時、両者の為す角 $\theta (0\degree \leqq \theta \leqq 180\degree)$ とすると、下記の式が成り立ちます。

$$
\cos \theta = \frac{\overrightarrow{a} \cdot \overrightarrow{b}}{|\overrightarrow{a}| \, |\overrightarrow{b}|} = \frac{a_1 b_1 + a_2 b_2 + a_3 b_3}{\sqrt{a_1^2 + a_2^2 + a_3^2} \sqrt{b_1^2 + b_2^2 + b_3^2}}
$$

上記で $\cos \theta$ が算出できれば、後は角度に変換するだけです。$\arccos(\cos \theta)$ にかけてラジアンを取得し、度数法に変換（≒ $\frac{180}{\pi}$ を掛ける）すれば、角度を取得できます。

## 空間ベクトルの為す角をArduino関数として作成

[空間ベクトルの為す角](#空間ベクトルの為す角)を使うことで、2 つのベクトル情報の角度を算出できることがわかりました。ここではそれをコード化し、Arduino 上で扱えるようにしていきます。

### 関数の定義

まずは、土台となる関数定義から行います。

- 関数名は、`angleBetweenTwoVectors()`
- 返り値は、角度を小数（`float`）で
- 引数は、2 つのベクトル情報

```cpp:関数の定義
float angleBetweenTwoVectors(int v1x, int v1y, int v1z, int v2x, int v2y, int v2z) {
  // 空間ベクトルの為す角から cosθ を算出し、度数法に変換して返す
}
```

### 異常値を弾く

「空間ベクトルの為す角（$\theta (0\degree \leqq \theta \leqq 180\degree)$）」の計算式は、2 つのベクトルデータを $\overrightarrow{a} = (a_1, a_2, a_3)$, $\overrightarrow{b} = (b_1, b_2, b_3)$ とした場合、下記になります。

$$
\cos \theta = \frac{a_1 b_1 + a_2 b_2 + a_3 b_3}{\sqrt{a_1^2 + a_2^2 + a_3^2} \sqrt{b_1^2 + b_2^2 + b_3^2}}
$$

逆正接関数でラジアンを取得し度数法に変換する一連の流れも含め、これを単純にコード化したものが下記になります。

```cpp:公式をコード化
float angleBetweenTwoVectors(int vecX1, int vecY1, int vecZ1, int vecX2, int vecY2, int vecZ2) {
  float cosTheta = ((vecX1 * vecX2) + (vecY1 * vecY2) + (vecZ1 * vecZ2)) / (sqrt(pow(vecX1, 2) + pow(vecY1, 2) + pow(vecZ1, 2)) * (sqrt(pow(vecX2, 2) + pow(vecY2, 2) + pow(vecZ2, 2))));
  return (acos(cosTheta) * 180) / PI;
}
```

これでも動くことには動くのですが、下記の問題が考慮されていない状態です。

- いずれかがゼロベクトルの場合【角度算出が不能】
- 内積や各ベクトルの大きさの内、いずれかが 0 になった場合【角度算出が不能】
- θが 0°から 180°までという制限【$\arccos(\cos \theta)$ に影響】

それらを踏まえた上で、異常値を弾いたり調整したりする必要があります。まずは、ゼロベクトルを除外する処理を挟みます。

```cpp:ゼロベクトルの場合は NAN を返す
float angleBetweenTwoVectors(int v1x, int v1y, int v1z, int v2x, int v2y, int v2z) {
  // ゼロベクトルの場合は NAN
  if ((v1x == 0 && v1y == 0 && v1z == 0) || (v2x == 0 && v2y == 0 && v2z == 0)) {
    return NAN;
  }
}
```

### 公式の各要素を計算

次に公式の各要素を、わかりやすいよう別個に計算していきます。

```cpp:公式を要素ごとに分け算出
float angleBetweenTwoVectors(int v1x, int v1y, int v1z, int v2x, int v2y, int v2z) {
  // ゼロベクトルの場合は NAN
  if ((v1x == 0 && v1y == 0 && v1z == 0) || (v2x == 0 && v2y == 0 && v2z == 0)) {
    return NAN;
  }

  // ベクトルの大きさと内積を計算する
  float dotProduct = (float)v1x * v2x + (float)v1y * v2y + (float)v1z * v2z;
  float magV1 = sqrt(pow((float)v1x, 2) + pow((float)v1y, 2) + pow((float)v1z, 2));
  float magV2 = sqrt(pow((float)v2x, 2) + pow((float)v2y, 2) + pow((float)v2z, 2));
}
```

一応、ゼロベクトルを弾いているので問題はないと思うのですが、ここでもベクトルの大きさを確認し今後の計算に異常が生じるケースを弾いておきます。

```cpp:ベクトルの大きさが 0 になる場合は NAN を返す
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
}
```

### 公式に当てはめ、度数法に変換した形で返す

ここまで来たら、後はメインとなる処理を残すのみです。

1. 公式に当てはめ $\cos \theta$ を算出
2. $\cos \theta$ は `-1` から `+1` の範囲内である必要があるので調整
3. 逆正接を使ってラジアンを度数法に変換して、返り値とする

```cpp:メインとなる処理
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
```

## デバイス向きの判定

先程の「[空間ベクトルの為す角をArduino関数として作成](#空間ベクトルの為す角をarduino関数として作成)」では、**2 つのベクトルの為す角**を関数として整備しました。

これにより「重力加速度のデータ」と「計測したい向き」をベクトルデータとして見立てることで、角度を計算できるようになりました。言い換えれば、**デバイスが指定した向きにどれくらい近いか**を数値で持って計測することができるようになったわけです。

ここでは 指定した向き と 加速度情報 からデバイスの向きを判定し、モード切替を行うための仕組みを実装していきます。

本段落で扱う設定項目は、下記になります。

|定数名|型|内容|
|---|---|:---|
|`ENABLE_DEBUG_MODE`|`bool`|デバッグモードの制御|
|`ENABLE_LED`|`bool`|状態通知に LED を利用するか|
|`TARGET_AXIS`|`TargetAxis`[^1]|モード切替の判定に使用する軸|
|`MODE_SWITCH_ANGLE_MARGIN`|`float`|モード切替の判定角度の許容範囲|

[^1]: 3軸加速度の向き（計 6 つ）は、列挙型として独自に定義します。

### 実装に入る前に

実際の動作をコーディングする前に まず、デバイス向き判定の流れを一度 例として見てみます。ここでは下図で言う *Pitch Up(Lean Backward)* かを判定に使うものとします。

![デバイスの向き6種](/images/books/nunchuk-mouse/axis-01.png)

再度おさらいになりますが、「空間ベクトルの為す角（$\theta (0\degree \leqq \theta \leqq 180\degree)$）」の計算式は、2 つのベクトルデータを $\overrightarrow{a} = (a_1, a_2, a_3)$, $\overrightarrow{b} = (b_1, b_2, b_3)$ とした場合、下記になります。

$$
\cos \theta = \frac{a_1 b_1 + a_2 b_2 + a_3 b_3}{\sqrt{a_1^2 + a_2^2 + a_3^2} \sqrt{b_1^2 + b_2^2 + b_3^2}}
$$

この計算式において、$\overrightarrow{a}$ は**加速度情報**をベクトル化したものが入ります。$(a_1, a_2, a_3)$ には、各軸の加速度情報（0 中心の符号ありデータ）が入ります。

一方 $\overrightarrow{b}$ は**計測したい向き**が入ります。今回ですと *Pitch Up(Lean Backward)* で、$(b_1, b_2, b_3)$ は `(0, -1, 0)` で固定です[^2]。この固定値は、**Y 軸上のマイナス方向**を意味しています。

[^2]: ベクトルの大きさは結果に影響を与えません。`(0, -1, 0)` でも `(0, -100, 0)` でも、最終的に求めるのは 2 つのベクトルの**角度**だからです。

となると 先程の数式はもっと簡略化することができ、最終的には下記のようになります。

$$
\cos \theta = \frac{a_1 0 + a_2 (-1) + a_3 0}{\sqrt{a_1^2 + a_2^2 + a_3^2} \sqrt{0^2 + (-1)^2 + 0^2}}
$$

$$
\cos \theta = \frac{-a_2}{\sqrt{a_1^2 + a_2^2 + a_3^2}}
$$

:::details この数式の意味

この数式は、「**ベクトルの大きさに対して y の割合がどの程度あるか**」を意味しています。つまり、ベクトルの大きさに対して y の割合が 100% であれば 1 を算出し、無関係（0%）であれば 0 を算出します。

下図は高校数学等で見る、単位円上の $\cos \theta$ 値です。角度 $\theta$ が 0°から180°へ変化するに伴って、$\cos \theta$ は 1 から -1 へと変化します。

![コサインθのグラフ、0から180まで](/images/books/nunchuk-mouse/graph-of-cosine-theta.png)

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

最終的には逆正接からラジアンを算出し、度数法に変換するために $\arccos(\cos \theta) \times \frac{180}{\pi}$ という計算式が加わるので、*Pitch Up(Lean Backward)* 方向に対するデバイス向きの角度 `θ` は、下記の式で計算することができます。

$$
\theta = \arccos(\frac{-a_2}{\sqrt{a_1^2 + a_2^2 + a_3^2}}) \times \frac{180}{\pi}
$$

この式を使えば、加速度情報である $\overrightarrow{a}$ の値に応じた角度を算出できます。例えば $(0, -10, 0)$ なら 0°、$(100, -173, 0)$ なら約 30°といった具合です。

#### Arduino上で検証

上記の式を Arduino で実装したのが下記です。異常値を弾く箇所等は解説済みですので、説明は省略します。

```cpp:1つの空間ベクトルから縦方向との角度を算出
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

```cpp:検証
angleToPortrait(0, -10, 0);    // -> 0
angleToPortrait(100, -173, 0); // -> 30.029401761514674
angleToPortrait(10, 0, 0);     // -> 90
```

ここまでで、Arduino 上で重力加速度を使い、特定方向との角度を算出することが可能であることがわかりました。

### コードへの実装

今回リポジトリで扱うのは、*Pitch Up(Lean Backward)* 方向だけでなく、下図 6 方向となります。

![コントローラー6方向の図と対応する定数名](/images/books/nunchuk-mouse/axis-02.png)

そのために、これまでの段階で設定項目として 6 方向を定義し設定可能な状態にしておきました。

```cpp:向きを表す列挙型と設定項目
// 向き判定に使う 6 方向：XYZ軸(3) ✕ 正負(2)
enum TargetAxis {
  AXIS_X_NEG, AXIS_X_POS,  // X 軸方向（-/+）: 左傾斜 / 右傾斜
  AXIS_Y_NEG, AXIS_Y_POS,  // Y 軸方向（-/+）: 後傾 / 前傾
  AXIS_Z_NEG, AXIS_Z_POS  // Z 軸方向（-/+）: 表向き / 裏向き
};

// モード切替で判定する向き（デフォルト：AXIS_Y_NEG）
// - AXIS_X_NEG : X 軸方向マイナス、左傾斜
// - AXIS_X_POS : X 軸方向プラス、右傾斜
// - AXIS_Y_NEG : Y 軸方向マイナス、後傾
// - AXIS_Y_POS : Y 軸方向プラス、前傾
// - AXIS_Z_NEG : Z 軸方向マイナス、裏向き
// - AXIS_Z_POS : Z 軸方向プラス、表向き
const TargetAxis TARGET_AXIS = AXIS_Y_NEG;
```

ここからはこれらを使って、特定の向きを判定できる仕組みを実装していきます。

#### モードの定義

まずは、扱うモードを `define` で定義しておきます（※ `const` でも恐らく問題はないです）

```cpp:モード定義
#define MODE_CURSOR 0
#define MODE_WHEEL 1
```

:::message

`0` と `1` は 後ほどの処理で意味を持つため、列挙型にはしていません。

この数値は、ボタンマップ等の配列インデックスに使用しており、モードに応じたデータを引き出せるようにする意味があります。

:::

#### モード切替する関数を定義

関数に必要な諸々が説明できたので、ここからはモード切替用の関数を作っていきます。

モード切替とはつまり、下記の一連の処理を指します。

1. 指定する向きと加速度情報から角度を算出
2. 角度が閾値内かを判定し、条件を満たしたかに応じてモードを切り替える

関数名は `determineOperationMode` とし、引数には平滑化済みの加速度情報 `SmoothedAccelerometer` を用います。

```cpp
int determineOperationMode(const SmoothedAccelerometer& smoothedAccel) {
  // 指定する向きのベクトルデータを生成
  // 加速度情報をベクトルデータに変換
  // 上記 2 つを使って、角度を算出
  // 角度が閾値内かで判定、閾値内ならホイールモード、そうでなければカーソルモードに
}
```

:::message

なお `SmoothedAccelerometer` は下表のデータを扱ってますが、本項で使うのは `accelX`, `accelY`, `accelZ` のみです。

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

:::

関数の返り値としては、`MODE_WHEEL`, `MODE_WHEEL` を返す想定です。

#### 指定する向きのベクトルデータを生成

まずは指定向きのベクトルデータを作ります。これは設定項目 `TARGET_AXIS` で設定した向きです。

```cpp:設定項目
// モード切替で判定する向き（デフォルト：AXIS_Y_NEG）
// - AXIS_X_NEG : X 軸方向マイナス、左傾斜
// - AXIS_X_POS : X 軸方向プラス、右傾斜
// - AXIS_Y_NEG : Y 軸方向マイナス、後傾
// - AXIS_Y_POS : Y 軸方向プラス、前傾
// - AXIS_Z_NEG : Z 軸方向マイナス、裏向き
// - AXIS_Z_POS : Z 軸方向プラス、表向き
const TargetAxis TARGET_AXIS = AXIS_Y_NEG;
```

例えば `AXIS_X_NEG` の場合は ベクトルデータは `(-1, 0, 0)` になり、`AXIS_Y_POS` の場合は ベクトルデータは `(0, 1, 0)` です。

コードではあらかじめ `(0, 0, 0)` としておき、`switch` 文で適正な値に調整しています。

```cpp:指定する向きのベクトルデータを生成
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
}
```

:::message

ベクトルデータは大きくなくて問題ないです。

最終的に必要なのは重力加速度ベクトルとの角度なので、ベクトルの大きさは計算結果に影響しません。

:::

#### 加速度情報をベクトルデータにして角度算出

次に加速度情報ですが、問題となるのは**コントローラーから取得した加速度は 符号なし 10 bit 情報**であることです。そのため 0 中心の符号ありデータに置き換えることで、ベクトル情報に見立てる必要があります。

各軸の中心値は 設定項目 `ACCEL_CENTER_X`, `ACCEL_CENTER_Y`, `ACCEL_CENTER_Z` で定義済みですので、計測値からこれらを引けば OK です。

- `ベクトルX`：`smoothedAccel.accelX - ACCEL_CENTER_X`
- `ベクトルY`：`smoothedAccel.accelY - ACCEL_CENTER_Y`
- `ベクトルZ`：`smoothedAccel.accelZ - ACCEL_CENTER_Z`

これで 2 つのベクトル情報が揃いましたので、角度を算出することができます。下記のコードでは引数内で加速度情報のベクトル変換を行なってます。

```cpp
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
}
```

これで `float angle` に、指定した方向との角度が格納されました。

#### 角度が閾値内かでモード切替

最後に `angle` で得られた角度に応じて、モードを切り替える処理を実装します。ただし `angleBetweenTwoVectors()` はゼロベクトルなど内容によっては `NAN` を返すことがあるため、そのあたりも対処する必要があります。

`angle` が `NAN` でなく、設定項目 `MODE_SWITCH_ANGLE_MARGIN` の閾値内であれば、`MODE_WHEEL` を返すようにします。条件に満たない場合は、`MODE_CURSOR` になります。

```cpp
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
```

:::message

モード切替を LED で通知するため、最後の判定部で LED の点灯消灯の切替を行なっています。

:::

### 補足：デバイス向きの出力

「[コードへの実装](#コードへの実装)」では、**デバイスが特定向きの閾値内にあるか**を判定し、それをモード切替に利用していました。

ここでは少し違った使い方として、デバイス向き自体を算出してみます。言い換えると、デバイス向きが 下図 6 方向の内どれに一番近いかを計算で求めるということです。

![デバイスの向き6種](/images/books/nunchuk-mouse/axis-02.png)

:::message

この内容は実際の処理には使われておらず、デバッグモードで利用しているものになります。

:::

まずは向き一覧を配列として用意します。

```cpp
// ここで出力する、デバイス向き一覧
String names[6] = {"X_Neg", "X_Pos", "Y_Neg", "Y_Pos", "Z_Neg", "Z_Pos"};
```

そして、平滑化した加速度情報をベクトルデータに変換し、それぞれの向きとの角度を算出します。角度情報は要素数 6 の配列で定義し、向き一覧配列と対応する形にしておきます。

```cpp
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
```

ここで**算出した角度の内 最も小さいものが、デバイスの向きとして最も近しい**ものとなります。

下記は、最小値のインデックスを格納する変数 `minIndex` を用意し、`for` ループを使って最小値のインデックスを算出しています。

```cpp
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
int minIndex = 0;
for (int i = 1; i < 6; i++) {
  if (angles[i] < angles[minIndex]) minIndex = i;
}
```

`angles[]` と `names[]` は対応関係にあるので、`names[minIndex]` でデバイス向きを出力することができるというわけです。

## まとめ

本項では、加速度センサーのデータを利用したデバイスの向きを計算する方法について説明しました。

またそれをモード切替の判定に使えるように実装しました。

次項では、最終的な反映先であるマウス操作について考え、それを管理するための構造体を作成します。

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

#define MODE_CURSOR 0
#define MODE_WHEEL 1





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
int determineOperationMode(const SmoothedAccelerometer& smoothedAccel);






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
```

:::
