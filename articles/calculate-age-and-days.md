---
title: "生年月日から、年齢と経過日数を算出" # 記事のタイトル
emoji: "🎂" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## はじめに

本記事では「**生年月日から、年齢と経過日数を算出する**」ことを目的とし、その実装を考えていきます。

なお 本記事で扱った内容は、下記場所にて確認することができます。

https://nonaka101.github.io/jig-a/

## 事前準備

ここでは設計を始める前段階として、目的や条件等を考えていきます。

### 環境の定義

本記事では JavaScript を使って処理していきます。

:::message

本記事では `Date` 型を使いますが、**下記の機能を実装できれば、他言語でも同じことができます**。

- `年`, `月`, `日`, `時間` の情報を持ち、取得・設定できる
- 月の末日（例えば 8月は **31**日で、9月は **30**日）や、閏日などが考慮されている
  - 実在しない日付は、インクリメント・デクリメント処理しれくれると良い（例：8月0日は7月31日に、8月32日は9月1日に）
- 日付データを数値化できる（例：UNIXエポックからの経過秒数）
  - 数値からの日数換算ができる
  - 日付同士の大小比較ができる

:::

### 目的と算出時の条件

本記事の目的は「**生年月日から、年齢と経過日数を算出する**」です。これをもう少し具体化し、下記のように定義します。

- 生年月日を、**`yyyy-MM-dd` 形式の文字列** で受け取る
- 今日の日付から、その人の**満年齢**と**そこからの経過日数**を計算する

そして算出の際は、下記の条件を満たさなければならないものとします。

#### 閏年に対応する

例えば、閏日である 2000年2月29日生まれの人は 4 年に 1 回しか歳を取らない、という状況は避けなければなりません。

#### 加齢のタイミングは、誕生日前日の 24 時とする

計算は [『年齢計算ニ関スル法律 | e-Gov 法令検索』](https://laws.e-gov.go.jp/law/135AC1000000050) に基づき、時刻を単位とする方式で行うものとします。

#### 1日に達しない経過日数は切り捨てる

経過日数は、現年齢となる加齢タイミングから経過した日数を意味します。1日分に達しない時間は切り捨てるものとします。  
なお、経過日数の最小値は `0`（誕生日当日の場合）、最大値は `365`（誕生日前日で、その年が閏年の場合）となります。

:::details 経過「月」数について

一部の書類には「◯歳◯ヶ月」と書く欄があったりと、月数算出には需要があります。

本記事においては、**実装の難易度**（月に応じて末日がバラバラな中で どう計算すべきか、など）や**得られるメリットとの比較衡量**（日数なら、概算でも月数を容易に出せる）から、難度が低く明確な「日数」での算出にしました。

:::

## 設計

ここからは[事前準備](#事前準備)の内容から、仕組みを考えていきます。

### 加齢タイミングについて

![2000年8月1日を生年月日とする、2024年8月25日までのタイムラインチャート。各年の7月31日24時が加齢タイミングとなっている](/images/articles/calculate-age-and-days/calcdate-00.png)

まずは、**どのタイミングで歳を取るか**を考えます。「誕生日当日の 0 時」だと、閏日生まれは 4 年に 1 度しか歳を取らないことになりかねません。

また（概算値ならともかく）「生年月日から365日ずつ回して算出する」という手法も適切でないでしょう。閏年により1年の日数は変わりますし、今回は経過日数も必要となるので（ある程度の）精度を持つ値が必要となります。

そこで、[『年齢計算ニ関スル法律 | e-Gov 法令検索』](https://laws.e-gov.go.jp/law/135AC1000000050)にあるように、加齢タイミングは**誕生日前日の24時**という考え方を使います。

この場合、閏日前後の生まれは下表のようになります。

|誕生日|加齢時点：閏年でない場合|加齢時点：閏年の場合|
|:---:|:---|:---|
| 2月29日 | 2月28日 24:00（3月1日 0:00） | 2月28日 24:00（2月29日 0:00） |
| 3月1日  | 2月28日 24:00（3月1日 0:00） | 2月29日 24:00（3月1日 0:00） |

### 満年齢について

![上図タイムラインチャートの、加齢タイミングに着目した図](/images/articles/calculate-age-and-days/calcdate-01.png)

年齢は、単純に「**今の年数** - **生まれた年数**」から出します。ただし 今年まだ誕生日を迎えていない（≒ 加齢タイミングを越えていない）場合、**そこから 1 を引く**ことで調整します。

### 経過日数について

![上図タイムラインチャートの、今年の加齢タイミングから今日までの期間に着目した図](/images/articles/calculate-age-and-days/calcdate-02.png)

経過日数は、「現年齢となる加齢タイミングから経過した日数」のことです。そのためには、**現年齢となる加齢タイミング**が必要です。  
これは、**月数**と**日数**は生年月日の情報をそのまま使い、**年数**については 生まれた年数と[先程の満年齢](#満年齢について)を組み合わせることで作成できます。

これを基に 今日との期間を数値化できれば、経過日数を算出することができます。

## 作成

ここからは、[設計](#設計)で考えた内容から、JavaScript でコード化していきます。

流れとしては、下記の3つを関数として作り、それらを組み合わせて[目的の処理](#生年月日から年齢と経過日数を算出)を実装します。

1. [満年齢を計算する関数](#満年齢を計算する関数)
2. [期間の日数を計算する関数](#期間の日数を計算する関数)
3. [日付文字列を判定する関数](#日付文字列を判定する関数)

### 満年齢を計算する関数

[満年齢について](#満年齢について)で考えた内容を、コードに起こします。

関数名は `calcAge` とし、必要な引数として2つを設定します。

1. 生年月日（`Date` 型）
2. 計算の基準日（`Date` 型、デフォルト値として今日の日付）

```javascript:関数名と引数を設定
function calcAge(dateBirth, dateBase = new Date()){
  // 日付から満年齢を計算し、返す
}
```

#### 年齢計算と事前チェック

まずは、引数となる `dateBirth`, `dateBase` が `Date` 型かを確かめます。

そしてその後、それぞれの年数を抜き出し、年齢を算出します。

```javascript:事前チェックをして、年齢を計算
function calcAge(dateBirth, dateBase = new Date()){
  // 型チェック
  if(!(dateBirth instanceof Date && dateBase instanceof Date)) throw new Error('引数は Date 型でなければなりません');

  const age = dateBase.getFullYear() - dateBirth.getFullYear();
}
```

#### 加齢タイミングを判定し、満年齢を返す

先ほど算出した年齢は、今年の誕生日を既に終えている場合です。まだの場合は 1 つ引いた値が年齢となります。

そのためには、今年の加齢タイミングを算出し、それを越えているかを判定する必要があります。

```javascript:加齢タイミングから、年齢を調整して返す
function calcAge(dateBirth, dateBase = new Date()){
  // 型チェック
  if(!(dateBirth instanceof Date && dateBase instanceof Date)) throw new Error('引数は Date 型でなければなりません');

  const age = dateBase.getFullYear() - dateBirth.getFullYear();

  // 加齢タイミング（計算基準年）は、誕生日の前日24時
  const timeBoundaryForAging = new Date(
    dateBase.getFullYear(),
    dateBirth.getMonth(),
    dateBirth.getDate() - 1,
    24
  );

  // 満年齢は、加齢タイミングを まだ越えていない場合、デクリメントして返す
  if(timeBoundaryForAging > dateBase){
    return age - 1;
  } else {
    return age;
  }
}
```

### 期間の日数を計算する関数

[経過日数について](#経過日数について)で考えた内容を、コードに起こします。ここでは、2点の日付データの間にある期間（日数単位）を算出する関数を作っていきます。

関数名は `dateDiff` とし、必要な引数として2つを設定します。

1. 日付A（`Date` 型）
2. 日付B（`Date` 型）

```javascript:関数名と引数を設定
function dateDiff(date1, date2) {
  // 2つの期間（ミリ秒単位）を求め、日数単位にして返す
}
```

#### 期間を数値化し、日数単位にして返す

まずは `calcAge()` 同様、受け取る引数 `date1`, `date2` の型チェックを行います。

```javascript:引数がDate型かチェック
function dateDiff(date1, date2) {
  // 型チェック
  if(!(date1 instanceof Date && date2 instanceof Date)) throw new Error('引数は Date 型でなければなりません');
}
```

その後 各日付を数値化（JavaScript の場合は ECMAScript 元期からの経過ミリ秒数）し、差分から期間を求めます。  
この時、`date1`, `date2` の順番によってはマイナスになりかねませんので、絶対値化しておきます。

```javascript:期間を数値化する
function dateDiff(date1, date2) {
  // 型チェック
  if(!(date1 instanceof Date && date2 instanceof Date)) throw new Error('引数は Date 型でなければなりません');

  // ミリ秒単位での期間を求め、日数単位にして返す
  const ms = Math.abs(date1.getTime() - date2.getTime());
}
```

最後に、JavaScript では先程の期間はミリ秒単位となっているので、それを日数換算（※）にして返します。  
（※ `Math.floor()` を使い、1日に満たない部分は切り捨てる形で整数化します）

```javascript:数値化した期間から、日数単位にして返す
function dateDiff(date1, date2) {
  // 型チェック
  if(!(date1 instanceof Date && date2 instanceof Date)) throw new Error('引数は Date 型でなければなりません');

  // ミリ秒単位での期間を求め、日数単位にして返す
  const ms = Math.abs(date1.getTime() - date2.getTime());
  return Math.floor(ms / (1000 * 60 * 60 * 24));
}
```

### 日付文字列を判定する関数

ここでは[目的と算出時の条件](#目的と算出時の条件)で決めた、 `yyyy-MM-dd` 形式で入力される文字列が、日付として適正なのかを検証する関数を作成します。

関数名は `isDateFormat` とし、必要な引数は `yyyy-MM-dd` 形式の文字列 1 つとします。

```javascript:関数名と引数を設定
function isDateFormat(dateString) {
  // 文字列をチェックし、適正な日付文字列かを判定する
}
```

検証は下記の 3 段階に分けて行います。満たさなければその時点で `false` で弾き、最後まで満たせば `true` を返します。

1. 形式チェック（数値で `0000-00-00` のような形式か）
2. 月日の妥当性チェック（月は1〜12の範囲内か、日は1〜31の範囲内か）
3. `Date` 型による妥当性チェック（暦上存在し得ない日付や、閏年の関係でズレが生じてないか）

#### 形式チェック

最初は `0000-00-00` のような形式かを検証します。ここでは**正規表現**によるチェックを行います。

```javascript:形式に一致しない場合を弾く
function isDateFormat(dateString) {
  // 正規表現による形式チェック（yyyy-MM-dd）
  const dateFormatRegex = /^\d{4}-\d{2}-\d{2}$/
  if (!dateString.match(dateFormatRegex)) return false;

  // チェックをすべて満たせば、true を返す
  return true;
}
```

#### 月日の妥当性チェック

次に、月日の妥当性チェックです。「0月0日」や「13月40日」といった日付は存在しないので、そうしたケースを弾きます。

```javascript:月日として成立し得ない場合を弾く
function isDateFormat(dateString) {
  // 正規表現による形式チェック（yyyy-MM-dd）
  const dateFormatRegex = /^\d{4}-\d{2}-\d{2}$/
  if (!dateString.match(dateFormatRegex)) return false;

  // 簡易的な月日の妥当性チェック（月の範囲は1〜12か、日の範囲は1〜31か）
  const [year, month, day] = dateString.split('-').map(Number);
  if(
    (month < 1 || month > 12) ||
    (day < 1 && day > 31)
  ) return false;

  // チェックをすべて満たせば、true を返す
  return true;
}
```

#### `Date` による妥当性チェック

月日の妥当性チェックでは、月や日の範囲について検証しました。しかしそれでも、「2月31日」や「1999年2月29日（閏年でない）」といった、暦上 存在し得ない日付の可能性が残っています。

そこで最後に、`Date` 型を使って実在する日付かを検証します。

`Date` 型は存在し得ない日付の場合、インクリメント・デクリメントされる性質を持ちます。

```javascript:閏日からDateを作成
const dateInLeapYear = new Date(2000, 1, 29);     // -> 2000年2月29日 00:00
const dateInNotLeapYear = new Date(1999, 1, 29);  // -> 1999年3月1日 00:00
```

それを使い、引数の文字列にある年月日との比較を行い、ズレが生じていないかを確かめます。

```javascript:Date化しズレた場合を弾く、全ての検証が通れば正しいと判断
function isDateFormat(dateString) {
  // 正規表現による形式チェック（yyyy-MM-dd）
  const dateFormatRegex = /^\d{4}-\d{2}-\d{2}$/
  if (!dateString.match(dateFormatRegex)) return false;

  // 簡易的な月日の妥当性チェック（月の範囲は1〜12か、日の範囲は1〜31か）
  const [year, month, day] = dateString.split('-').map(Number);
  if(
    (month < 1 || month > 12) ||
    (day < 1 && day > 31)
  ) return false;

  // date 型による妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
  const date = new Date(year, month - 1, day);
  if (
    date.getFullYear() !== year ||
    date.getMonth() + 1 !== month ||
    date.getDate() !== day
  ) return false;

  // チェックをすべて満たせば、true を返す
  return true;
}
```

### 生年月日から年齢と経過日数を算出

ここまでで、下記の関数を作成しました。

1. [満年齢を計算する関数 `calcAge()`](#満年齢を計算する関数)
2. [期間の日数を計算する関数 `dateDiff()`](#期間の日数を計算する関数)
3. [日付文字列を判定する関数 `isDateFormat()`](#日付文字列を判定する関数)

これらを使い、生年月日から年齢と経過日数を算出する処理を書いていきます。

#### 生年月日データを用意する

まずは、生年月日の文字列を用意します。

```javascript:生年月日の用意
const birthday = '2000-08-23';
```

次に 用意した関数に入れるために、これを `Date` 型に変換します。`isDateFormat()` で検証した後、年月日を数値として取り出し、`Date` コンストラクタに送って生成します。  
また このタイミングで今日の日付も `Date` で作成しておきます。

```javascript:処理前準備
const birthday = '2000-08-23';

if(isDateFormat(birthday)) {
  const [year, month, day] = birthday.split('-').map(Number);
  const dateBirth = new Date(year, month - 1, day);
  const today = new Date();
}
```

#### 年齢と経過日数を算出して出力

満年齢を計算するのは、`calcAge()` を使えば出せます。ここでは後の処理用に、`age` という名前で格納しておきます。

```javascript:年齢の算出
const birthday = '2000-08-23';

if(isDateFormat(birthday)) {
  const [year, month, day] = birthday.split('-').map(Number);
  const dateBirth = new Date(year, month - 1, day);
  const today = new Date();

  const age = calcAge(dateBirth, today);
}
```

次に**現年齢になった加齢タイミング**を求めます。月日については生年月日のデータをそのまま使いますが、年に関しては**今年の誕生日を越えているか**に応じて場合分け（※）になります。  
（※：越えている場合は**その年**、そうでなければ**前年**）

今回は その情報から得られた満年齢 `age` を既に持っているので、**誕生年に満年齢を足す**ことで算出できます。

```javascript:現年齢の加齢タイミングを算出
const birthday = '2000-08-23';

if(isDateFormat(birthday)) {
  const [year, month, day] = birthday.split('-').map(Number);
  const dateBirth = new Date(year, month - 1, day);
  const today = new Date();

  const age = calcAge(dateBirth, today);

  // 現年齢の加齢タイミング（≒誕生日前日24時）
  const timeAging = new Date(
    dateBirth.getFullYear() + age,
    dateBirth.getMonth(),
    dateBirth.getDate() - 1,
    24
  );
}
```

最後に、加齢タイミングと今日との期間日数を求めて経過日数を算出します。これらを `console.log()` で出力すれば、諸々の処理の完成です。

```javascript:経過日数を算出し、諸々を出力
const birthday = '2000-08-23';

if(isDateFormat(birthday)) {
  const [year, month, day] = birthday.split('-').map(Number);
  const dateBirth = new Date(year, month - 1, day);
  const today = new Date();

  const age = calcAge(dateBirth, today);

  // 現年齢の加齢タイミング（≒誕生日前日24時）
  const timeAging = new Date(
    dateBirth.getFullYear() + age,
    dateBirth.getMonth(),
    dateBirth.getDate() - 1,
    24
  );

  const days = dateDiff(timeAging, today);

  console.log(`年齢は ${age} 歳、経過日数は ${days} 日です`);
}
```

上記コードでは、例として生年月日を **2000年8月23日**としています。そして記事作成時は2024年（閏年）です。その状況下で、誕生日前後を今日とした場合の計算結果は、下記のとおりとなります。

|今日の日付|出力|
|:---:|:---|
| 2024年8月22日 | 年齢は 23 歳、経過日数は 365 日です |
| 2024年8月24日  | 年齢は 24 歳、経過日数は 1 日です |

## 補足

### コード全文

:::details コード全文（長いので格納しています）

```javascript:関数
/**
 * 生年月日から、年齢を計算する（時間基準）
 *
 * @param {date} dateBirth - 生年月日
 * @param {date} dateBase - 計算基準日（今日）
 * @returns {int} - 満年齢
 */
function calcAge(dateBirth, dateBase = new Date()){
  // 型チェック
  if(!(dateBirth instanceof Date && dateBase instanceof Date)) throw new Error('引数は Date 型でなければなりません');

  const age = dateBase.getFullYear() - dateBirth.getFullYear();

  // 加齢タイミング（計算基準年）は、誕生日の前日24時
  const timeBoundaryForAging = new Date(
    dateBase.getFullYear(),
    dateBirth.getMonth(),
    dateBirth.getDate() - 1,
    24
  );

  // 満年齢は、加齢タイミングを まだ越えていない場合、デクリメントして返す
  if(timeBoundaryForAging > dateBase){
    return age - 1;
  } else {
    return age;
  }
}



/**
 * ２つの日付間にある日数（絶対値）を算出
 *
 * @param {date} date1 - 日付１
 * @param {date} date2 - 日付２
 * @returns {int} 期間内の日数（余剰の時間は切り捨て）
 */
function dateDiff(date1, date2) {
  // 型チェック
  if(!(date1 instanceof Date && date2 instanceof Date)) throw new Error('引数は Date 型でなければなりません');

  // ミリ秒単位での期間を求め、日数単位にして返す
  const ms = Math.abs(date1.getTime() - date2.getTime());
  return Math.floor(ms / (1000 * 60 * 60 * 24));
}



/**
 * 年月日を表す文字列が、適正かを判断
 *
 * @param {string} dateString - 'yyyy-MM-dd' 形式の文字列
 * @returns {boolean} 存在しうる日付かの判定
 */
function isDateFormat(dateString) {
  // 正規表現による形式チェック（yyyy-MM-dd）
  const dateFormatRegex = /^\d{4}-\d{2}-\d{2}$/
  if (!dateString.match(dateFormatRegex)) return false;

  // 簡易的な月日の妥当性チェック（月の範囲は1〜12か、日の範囲は1〜31か）
  const [year, month, day] = dateString.split('-').map(Number);
  if(
    (month < 1 || month > 12) ||
    (day < 1 && day > 31)
  ) return false;

  // date 型による妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
  const date = new Date(year, month - 1, day);
  if (
    date.getFullYear() !== year ||
    date.getMonth() + 1 !== month ||
    date.getDate() !== day
  ) return false;

  return true;
}
```

```javascript:処理本文
const birthday = '2000-08-23';

if(isDateFormat(birthday)) {
  const [year, month, day] = birthday.split('-').map(Number);
  const dateBirth = new Date(year, month - 1, day);
  const today = new Date();

  const age = calcAge(dateBirth, today);

  // 現年齢の加齢タイミング（≒誕生日前日24時）
  const timeAging = new Date(
    dateBirth.getFullYear() + age,
    dateBirth.getMonth(),
    dateBirth.getDate() - 1,
    24
  );

  const days = dateDiff(timeAging, today);

  console.log(`年齢は ${age} 歳、経過日数は ${days} 日です`);
}
```

:::
