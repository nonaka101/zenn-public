---
title: "言語練習：生年月日から年齢と経過日数を算出" # 記事のタイトル
emoji: "🚌" # アイキャッチとして使われる絵文字（1文字だけ）
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["vba", "java", "python", "php", "c"] # タグ。["javascript", "excel"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## はじめに

本記事は、下記記事で行った内容を**他の言語等で実装**することを目的にしております。

https://zenn.dev/nonaka101/articles/calculate-age-and-days

本記事では JavaScript に加え、下記の言語等で実装を行っています。

- [Excel 関数](#excel)
- [VBA](#vba)
- [PHP](#php)
- [Python](#python)
- [Java](#java)
- [C 言語](#c言語)

:::details タイトル「言語練習」について

タイトルの「言語練習」は、フランスの小説家 レーモン・クノーの著書『文体練習』に倣ったものです。

この本は、端的に言ってしまえば「バスで口論している男を見かけ、後で駅前に行った際に その男が助言を受けているのを見かけた」という話を、99通りのパターンで表現した書籍です。

ある言語で作成したコードを別言語で表現した場合、という本記事の主旨に似ていたので、参考にしました。

:::

### 目標について

本記事で取り扱っている内容についての概要を、ここで説明します。年齢の計算方法等の詳細については、[前記事](https://zenn.dev/nonaka101/articles/calculate-age-and-days)を参照してください。

#### 基本的な条件の設定

本記事では様々な言語等での実装を行っていきますが、その土台となるものは下記になります。

- 引数として、必須要素 1 つ 任意要素 1 つを受け取る
  1. 必須：生年月日を表すテキスト（`yyyy-MM-dd` 形式）
  2. 任意：計算基準日となる日付データ（デフォルト値は 今日）
- 上述のデータを基に、満年齢と経過日数を算出する

#### 検証について

日付に関する計算であるため、閏年を考慮する必要があります。本記事では、下記のテストデータを用いた場合に同じ結果になることを目指します。

:::details テストデータ

| 生年月日 | 計算基準日 | 満年齢 | 経過日数 |
| --- | --- | ---: | ---: |
| 1999-02-28 | 2023-02-28 | 24  | 0  |
| 1999-03-01 | 2023-02-28 | 23  | 364  |
| 2000-02-28 | 2023-02-28 | 23  | 0  |
| 2000-02-29 | 2023-02-28 | 22  | 364  |
| 2000-03-01 | 2023-02-28 | 22  | 364  |
| 1999-02-28 | 2023-03-01 | 24  | 1  |
| 1999-03-01 | 2023-03-01 | 24  | 0  |
| 2000-02-28 | 2023-03-01 | 23  | 1  |
| 2000-02-29 | 2023-03-01 | 23  | 0  |
| 2000-03-01 | 2023-03-01 | 23  | 0  |
| 1999-02-28 | 2024-02-28 | 25  | 0  |
| 1999-03-01 | 2024-02-28 | 24  | 364  |
| 2000-02-28 | 2024-02-28 | 24  | 0  |
| 2000-02-29 | 2024-02-28 | 23  | 364  |
| 2000-03-01 | 2024-02-28 | 23  | 364  |
| 1999-02-28 | 2024-02-29 | 25  | 1  |
| 1999-03-01 | 2024-02-29 | 24  | 365  |
| 2000-02-28 | 2024-02-29 | 24  | 1  |
| 2000-02-29 | 2024-02-29 | 24  | 0  |
| 2000-03-01 | 2024-02-29 | 23  | 365  |
| 1999-02-28 | 2024-03-01 | 25  | 2  |
| 1999-03-01 | 2024-03-01 | 25  | 0  |
| 2000-02-28 | 2024-03-01 | 24  | 2  |
| 2000-02-29 | 2024-03-01 | 24  | 1  |
| 2000-03-01 | 2024-03-01 | 24  | 0 |

:::

:::message

記事作成時点で、C言語以外は確認が取れています。

:::

## JavaScript

まずは、[前記事](https://zenn.dev/nonaka101/articles/calculate-age-and-days)でも扱っている JavaScript でのコードです。仕組みについては、[前記事](https://zenn.dev/nonaka101/articles/calculate-age-and-days)の方を参照ください。

### 満年齢の計算（JavaScript）

```javascript:満年齢を計算する関数（JavaScript）
/**
 * 生年月日から、満年齢を計算する（時間基準）
 *
 * @param {date} dateBirth - 生年月日
 * @param {date} dateBase - 計算基準日（省略時は現在の日付）
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
```

### 期間内の日数計算（JavaScript）

```javascript:期間の日数を計算する関数（JavaScript）
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
```

### 日付文字列の判定（JavaScript）

```javascript:日付文字列を判定する関数（JavaScript）
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
    !(month >= 1 && month <= 12) ||
    !(day >= 1 && day <= 31)
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

### 満年齢と経過日数の算出（JavaScript）

[前記事](https://zenn.dev/nonaka101/articles/calculate-age-and-days)では、関数化していたのはここまででした。本記事ではこれにプラスし、**満年齢と経過日数を出力する処理**を関数として独立させます。

```javascript:生年月日から満年齢と経過日数を算出する関数（JavaScript）
/**
 * 生年月日から年齢と経過日数を計算
 *
 * @param {string} textBirth - 生年月日（yyyy-MM-dd形式）
 * @param {date} dateBase - 計算基準日（省略時は現在の日付）
 */
function calcAgeAndDays(textBitrh, dateBase = new Date()){
  if(isDateFormat(textBitrh)) {
    const [year, month, day] = textBitrh.split('-').map(Number);
    const dateBirth = new Date(year, month - 1, day);

    const age = calcAge(dateBirth, dateBase);

    // 現年齢の加齢タイミング（≒誕生日前日24時）
    const timeAging = new Date(
      dateBirth.getFullYear() + age,
      dateBirth.getMonth(),
      dateBirth.getDate() - 1,
      24
    );

    const days = dateDiff(timeAging, dateBase);

    console.log(`年齢は ${age} 歳、経過日数は ${days} 日です`);
  }
}
```

これで JavaScript による実装は完成となります。

## Excel

ここでは、（プログラミングではありませんが）Excel の関数を使った実装を考えていきます。最終的な形は下図のようになり、`C`列 及び `D`列 に関数を仕込んでいます。

![最初に掲載していたテストデータをExcelで表現したもの](/images/articles/exercise_in_language__calculate_age/calc_age-01.png)

| セル | 式 |
| --- | --- |
| `C2` | `=LET(Age, YEAR(B2)-YEAR(A2), IF(B2>=DATE(YEAR(B2), MONTH(A2), DAY(A2)-1)+1, Age, Age-1))` |
| `D2` | `=DATEDIF(DATE(YEAR(A2)+C2, MONTH(A2), DAY(A2)-1)+1, B2, "D")` |

:::message

上記数式は、処理速度より**手続きの説明性**を重視したものです。

例えば `C2` セルの式ですが、処理速度を重視する場合 例えば簡略化するために `DATEDIF()` 関数を使うなどの方法が考えられます。

:::

### 計算式について

`C`列, `D`列にある関数は、複数の計算をまとめた形になっております。そのため、それぞれを分解しつつ説明をしていきます。

#### C2セル：満年齢計算について

`=LET(Age, YEAR(B2)-YEAR(A2), IF(B2>=DATE(YEAR(B2), MONTH(A2), DAY(A2)-1)+1, Age, Age-1))`

この計算式は、下記の要素を組み合わせたものです。

1. 生年月日から、計算基準年の誕生日を生成
2. 計算基準日との大小比較をし、処理を分岐
3. 分岐内容に従って、満年齢の計算

これらをステップごとに見ていきます。

##### 計算基準年の誕生日を生成

`DATE(YEAR(B2), MONTH(A2), DAY(A2)-1)+1` の部分が、計算基準年の誕生日を表しています。日付データの生成には `DATE(年, 月, 日)` 関数を使っています。

- 「年」の引数部には、`B2`セル（≒ 計算基準日）から `YEAR(日付)` 関数で年数だけを取り出しています
- 「月」の引数部には、`A2`セル（≒ 生年月日）から `MONTH(日付)` 関数で月数だけを取り出しています
- 「日」の引数部には、`A2`セルから `DAY(日付)` 関数で日数を取り出しています。この時 `-1` を加えることで、前日となるように調整しています

`DATE()` 関数により、計算基準年の誕生日**前日**のデータが生成できました。式では最後、このデータに `+1` を加えるようになっています。これは**誕生日前日の24時**を表すための処理です。

::::message

Excel上で日付データは `2000/01/01` といったように表示されていますが、実際のデータは数値（シリアル値）で管理されています。

内部的には 1900 年 1 月 1 日を「1」としており、例えば先程の `2000/01/01` の場合は `36526` という値になります。整数部が日付、小数部が時間を表すため、`2000/01/02` は `36527`、`2000/01/02 12:00` は `36527.5` となるわけです。

:::details VBAでのシリアル値確認

この Excel の日付データの扱いは、VBA側にも適用されています。下記はそれを確認するためのコードです。

```vba:VBAでのシリアル値確認
Dim d1 As Date
d1 = "2000-01-01"
Debug.Print CDbl(d1)  ' -> 36526

Dim d2 As Date
d2 = "2000-01-02"
Debug.Print CDbl(d2)  ' -> 36527

Dim d3 As Date
d3 = "2000-01-02 12:00"
Debug.Print CDbl(d3)  ' -> 36527.5
```

:::

::::

##### 大小判定による分岐

`IF({条件式}, {TRUE時の挙動}, {FALSE時の挙動})` の部分が、大小判定による分岐を表しています。条件式の部分で、計算基準日である `B2` セルと[前述した計算基準年の誕生日](#計算基準年の誕生日を生成)を比較し、その結果により処理を分岐させています。Excel の日付データは実際には数値（シリアル値）であるため、このような大小比較が行えるわけです。

- `B2` セルが大きい（≒ 計算基準日が誕生日を越えている）場合は、両者で年数を引いた値が満年齢
- そうでない場合は、両者で年数を引き `-1` をした値が満年齢

##### 満年齢の計算

`LET(Age, YEAR(B2)-YEAR(A2), {計算式})` の部分が、満年齢の計算を簡略化した部分です。ここでは `LET(名前, 計算内容, 「名前」を使った計算式)` 関数を使うことで、式の簡略化を行っています。

`LET()` 関数は、プログラミングで言う変数のような役割を果たします。同じ計算内容を使う場合に都度手打ちで入力してしまうと、分かりづらいですし、変更時のミスにも繋がります。そのため式に繰り返し利用する計算内容は、事前に共有できるようにしています。

ここでは満年齢の計算（`YEAR(B2)-YEAR(A2)`）を `Age` という名前で登録し、`IF()` 内での分岐処理（そのまま使うか、デクリメントした形で使うか）の部分で利用しています。

#### D2セル：経過日数について

`=DATEDIF(DATE(YEAR(A2)+C2, MONTH(A2), DAY(A2)-1)+1, B2, "D")`

この計算式は、下記の内容を組み合わせたものです。

1. 満年齢になった時点の日付データを生成
2. 計算基準日との差分を、日数換算で計算

これらをステップごとに見ていきます。

##### 満年齢になった時点の日付データを生成

`DATE(YEAR(A2)+C2, MONTH(A2), DAY(A2)-1)+1` の部分が、満年齢になった日付データを表します。これは[計算基準年の誕生日を生成](#計算基準年の誕生日を生成)と同様、`DATE()` 関数を使って作っています。

「年」の引数については、満年齢を計算した `C2` セルがあるので これを `A2` セルの年数と組み合わせることで算出できます。後は同じように、前日の日付データを生成し それに `+1` を加えることで「誕生日前日の24時」を表します。

##### 両日付の差分を日数換算で算出

`DATEDIF({満年齢になった日付}, B2, "D")` の部分が、日数換算での差分計算を表します。これは今年の誕生日と計算基準日との日数を、`DATEDIF(日付A, 日付B, 計算単位)` 関数によって算出しています。計算単位は 今回は日数を表す `D` を使っています（他にも年数を表す `Y` や、月数を表す `M` 、年数を無視した形での日数を表す `YD` なども指定可能です）

### 補足：DATEDIF関数で計算した場合

ちなみに、「上述した手続きを踏まずとも、そのまま `DATEDIF` 関数1つで済むのでは？」と思われるかも知れません。下表は、先程の表データに加える形で `DATEDIF` 関数での計算結果を載せたものになります。先程と同じように、2行目の数式も載せております。

![上図のデータにDATEDIF関数を使ったものを付け加えたもの、いくつか計算結果が違っている](/images/articles/exercise_in_language__calculate_age/calc_age-02.png)

| セル | 式 |
| --- | --- |
| `E2` | `=DATEDIF(A2,B2,"Y")` |
| `F2` | `=DATEDIF(A2,B2,"YD")` |

満年齢計算に関しては `DATEDIF()` は同じ計算結果を出しています。なので処理効率を優先する場合はこちらを使ったほうがいいかもしれません。

一方で経過日数については、5 行目や 15 行目にあるように違う挙動をしていることが確認できます。これは `DATEDIF()` 関数で算出する際の起算日がこちらの想定と違っており、閏日に対応できていないことが原因のようです。

`DATEDIF()` は元々[Lotusとの互換性のために用意された関数](https://support.microsoft.com/ja-jp/office/datedif-%E9%96%A2%E6%95%B0-25dba1a4-2812-480b-84dd-8b32a451b35c)であり、非公式的な位置づけとなっています（「関数」タブにある一覧には表示されない）  
そのため、`DATEDIF()` の使用は極力避けるべきなのかもしれません。

## VBA

次に VBA 上での実装を行っていきます。

:::message

なお、VBA では `DateDiff()` [関数がビルトインで用意されている](https://learn.microsoft.com/ja-jp/office/vba/language/reference/user-interface-help/datediff-function)ため、JavaScript にあった `dateDiff()` 関数を自作する必要はありません。

:::

### 満年齢の計算（VBA）

満年齢を計算する関数 `CalcAge()` のコードは下記になります。

```vba:満年齢を計算する関数（VBA）
' 生年月日から、満年齢を計算する（時間基準）
'
' 引数 dateBirth As Date - 生年月日
' 引数 (Optional) dateBase As Date - 計算基準日（省略時は現在の日付）
' 返り値 Integer - 満年齢
'
Function CalcAge(dateBirth As Date, Optional dateBase As Date = 0) As Integer
  ' 引数チェック
  If IsDate(dateBirth) = False Then
    Err.Raise vbObjectError + 1, , "引数はDate型でなければなりません"
  End If

  Dim dateCalc As Date
  If (IsDate(dateBase) = False) Or (dateBase = 0) Then
    dateCalc = Now
  Else
    dateCalc = dateBase
  End If

  Dim age As Integer
  age = Year(dateCalc) - Year(dateBirth)

  ' 加齢タイミング（計算基準年）は、誕生日の前日24時
  Dim timeBoundaryForAging As Date
  timeBoundaryForAging = DateSerial(Year(dateCalc), Month(dateBirth), Day(dateBirth) - 1) + TimeSerial(24, 0, 0)

  ' 満年齢は、加齢タイミングを まだ越えていない場合、デクリメントして返す
  If timeBoundaryForAging > dateCalc Then
    CalcAge = age - 1
  Else
    CalcAge = age
  End If
End Function
```

このコードで特徴的なのは、下記の部分です。

- `Function` プロシージャでの初期値に指定できるのは**任意の定数**（または定数式）、なので今日の日付を指定するにはワンクッション必要
- `Date` 型のデータは、実際には Excel 同様シリアル値での管理
- `Err.Raise` や `Now` など、一部の関数では他と違った書き方をする

日付生成の関数 `DateSerial(年, 月, 日)` で誕生日前日を作り、時間生成の関数 `TimeSerial(時, 分, 秒)` で 「24 時間」を生成、両者を組み合わせることで「誕生日前日の 24 時」を生成しています。

#### Functionプロシージャ

VBA での関数は、`Function 関数名(引数) As データ型` といった形で定義します。この時 引数でデフォルト値を設定する場合は、キーワード `Optional` を使う必要があります。

JavaScript では、引数のデフォルト値として `new Date()` で新規生成したインスタンスを割り当てることができました。一方で VBA で似たようなことをすると、エラーになってしまいます。下図は 今日の日付データを返す `Date` 関数をデフォルト値に設定したもので、コンパイルエラーになっている状態です。

![引数にある Date関数にマーカーが引かれ、「定数式が必要です」とポップアップが表示されている](/images/articles/exercise_in_language__calculate_age/calc_age-04.png)

そのため初期値は 0 としておき、関数内で `If` 分岐を設けて中身を確認、適正な形になるように処理しています。

#### Date型の変数について

[Excelでの日付データの扱い](#計算基準年の誕生日を生成)で説明しましたが、Excel上での日付は、実際にはシリアル値による管理となっています。

そのため、存在しない日付（閏年でない閏日など）を文字列形式から `Date` 型に直接代入しようとすると、エラーが発生します。

![日付型のエラー、DateSerial関数では調整されるが、直接日付文字列からの代入だとエラーになる](/images/articles/exercise_in_language__calculate_age/calc_age-06.png)
*`DateSerial` による日付生成は問題なく、日付文字列からの生成でエラーとなっている*

[MS Learn](https://learn.microsoft.com/ja-jp/office/vba/language/reference/user-interface-help/dateserial-function)によると、`DateSerial()` による日付生成では、存在しないケースはインクリメント（あるいはデクリメント）処理されるようになっています。この挙動は、後述の日付文字列の判定に利用することになります。

```vba:存在しない日付でもDateSerial関数を使えば調整される
Debug.Print DateSerial(2023, 2, 29) ' -> 2023/03/01
Debug.Print DateSerial(2023, 13, 40) ' -> 2024/02/09
```

`isDate()` 関数でのチェックは、文字列の形が厳格に決まっていない場合は あまり推奨されません。というのも、この関数は日付文字列を結構広い形で解釈しているようだからです。`isDate("2/29")`, `isDate("2/30")` は、どちらも `true` を返します。これは調べてみると `2029-02-01`, `2030-02-01` と解釈しているようです。

https://learn.microsoft.com/ja-jp/office/vba/language/reference/user-interface-help/isdate-function

### 日付文字列の判定（VBA）

日付文字列であるかを判定する関数 `IsDateFormat()` のコードは下記になります。

```vba:日付文字列を判定する関数（VBA）
' 年月日を表す文字列が、適正かを判断
'
' 引数 dateString As String - 'yyyy-MM-dd' 形式の文字列
' 返り値 Boolean - 存在しうる日付かの判定
'
Function IsDateFormat(dateString As String) As Boolean

  ' 正規表現による形式チェック（yyyy-MM-dd）
  Dim dateFormatRegex As Object
  Set dateFormatRegex = CreateObject("VBScript.RegExp")
  With dateFormatRegex
    .IgnoreCase = True
    .Global = False
    .Pattern = "^\d{4}-\d{2}-\d{2}$"
  End With

  If Not dateFormatRegex.test(dateString) Then
    IsDateFormat = False
    Exit Function
  End If

  ' 簡易的な月日の妥当性チェック（月の範囲は1～12か、日の範囲は1～31か）
  Dim numYear As Integer
  Dim numMonth As Integer
  Dim numDay As Integer
  numYear = CInt(Split(dateString, "-")(0))
  numMonth = CInt(Split(dateString, "-")(1))
  numDay = CInt(Split(dateString, "-")(2))

  If (numMonth < 1) Or (numMonth > 12) Or (numDay < 1) Or (numDay > 31) Then
    IsDateFormat = False
    Exit Function
  End If

' date型による妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
  Dim dateCheck As Date
  dateCheck = DateSerial(numYear, numMonth, numDay)
  If (Year(dateCheck) <> numYear) Or (Month(dateCheck) <> numMonth) Or (Day(dateCheck) <> numDay) Then
    IsDateFormat = False
    Exit Function
  End If

  IsDateFormat = True
End Function
```

:::details 別案：Errorを利用した妥当性チェック

最後の妥当性チェックについては、下記の**エラー処理**による方法も使えます。

```vba:Errorを利用した妥当性チェック
' date型による妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
On Error Resume Next
Dim dateCheck As Date
dateCheck = dateString
If (Err.Number <> 0) Or _
  ((numYear <> Year(dateCheck)) Or (numMonth <> Month(dateCheck)) Or (numDay <> Day(dateCheck))) Then
  IsDateFormat = False
  Exit Function
End If
On Error GoTo 0
```

ここでは引数で受け取った日付文字列（`dateString`）を直接 `Date` 型に格納しています。存在しない日付文字列を `Date` 型に格納するとエラーが発生するので、それを利用して妥当性を判断しているわけです。

なお、JavaScript とは違って VBA では、エラーが発生すると処理は中断されてしまいます。なので `On Error Resume Next` 〜 `On Error GoTo 0` 間でエラーが発生しそうな処理を挟む必要があります。  
（他の言語で言う、`try` 〜 `catch` と似たような感じでしょうか？）

:::

このコードで特徴的なのは、下記の部分です。

- 正規表現を利用するためのオブジェクト `RegExp` を生成

#### 正規表現用のオブジェクト生成

VBA で正規表現を利用するには、外部ライブラリ `Microsoft VBScript Regular Expressions` から、`RegExp` オブジェクトを呼び出す方法があります。

![参照設定から、外部ライブラリの VBScript Regular Expressions を選択した状態](/images/articles/exercise_in_language__calculate_age/calc_age-05.png)

![VBScriptを外部参照から取り込んだ状態で、VBEのオブジェクトライブラリで「RegExp」を検索した状態](/images/articles/exercise_in_language__calculate_age/calc_age-03.png)

今回は `CreateObject()` を使って参照設定無しで呼び込むようにします。

ここで呼び出した `RegExp` オブジェクトには、上図にあるように プロパティやメソッドが用意されています。これを使って、正規表現による日付文字列のチェックを行っています。

### 満年齢と経過日数の算出（VBA）

満年齢と経過日数を算出し出力する関数 `CalcAgeAndDays()` のコードは下記になります。

```vba:生年月日から満年齢と経過日数を算出する関数（VBA）
' 生年月日から年齢と経過日数を計算
'
' 引数 textBirth As String - 生年月日（yyyy-MM-dd形式）
' 引数 (Optional) dateBase As Date - 計算基準日（省略時は現在の日付）
'
Function CalcAgeAndDays(textBirth As String, Optional dateBase As Date = 0)
  Dim dateCalc As Date
  If (IsDate(dateBase) = False) Or (dateBase = 0) Then
    dateCalc = Now
  Else
    dateCalc = dateBase
  End If

  If (IsDateFormat(textBirth) = True) Then
    Dim dateBirth As Date
    dateBirth = DateSerial( _
      CInt(Split(textBirth, "-")(0)), _
      CInt(Split(textBirth, "-")(1)), _
      CInt(Split(textBirth, "-")(2)) _
    )
    
    Dim age As Integer
    age = CalcAge(dateBirth, dateCalc)
    
    ' 現年齢の加齢タイミング（≒誕生日前日24時）
    Dim timeAging As Date
    timeAging = DateSerial( _
      Year(dateBirth) + age, _
      Month(dateBirth), _
      Day(dateBirth) - 1 _
    ) + TimeSerial(24, 0, 0)

    Dim days As Integer
    days = DateDiff("d", timeAging, dateCalc)  ' DateDiffは標準で用意されている
    
    Debug.Print "年齢は " & age & " 歳、経過日数は " & days & " 日です"
  End If

End Function
```

## PHP

PHP で日付を扱う際は、`DateTime` クラスを使うのが主となります。

```php:DateTimeによる日付生成
// コンストラクタ（日付文字列）
$date1 = new DateTime('2023-02-29');
echo $date1->format('Y-m-d'), PHP_EOL; // 2023-03-01

// setDateメソッド（各種数値）
$date2 = new DateTime();
$date2->setDate(2023,2,29);
echo $date2->format('Y-m-d'), PHP_EOL; // 2023-03-01

$date2->setDate(2023,0,29);
echo $date2->format('Y-m-d'), PHP_EOL; // 2022-12-29
```

https://www.php.net/manual/ja/class.datetime.php

https://www.php.net/manual/ja/datetime.formats.php

:::message

なお、PHP では [`DateTime`クラス](https://www.php.net/manual/ja/class.datetime.php)にあるメソッド [`diff()`](https://www.php.net/manual/ja/datetime.diff.php)で 2 つの日付の差分を取ることができ、生成された [`DateInterval`クラス](https://www.php.net/manual/ja/class.dateinterval.php)のプロパティ `days` で日数換算ができるため、JavaScript にあった `dateDiff()` 関数を自作する必要はありません。

:::

### 満年齢の計算（PHP）

```php:満年齢を計算する関数（PHP）
/**
 * 生年月日から、満年齢を計算する（時間基準）
 *
 * @param DateTime $dateBirth 生年月日
 * @param DateTime|null $dateBase 計算基準日（省略時は現在の日付）
 * @return int 満年齢
 * @throws InvalidArgumentException 引数が DateTime 型でない場合
 */
function calcAge(DateTime $dateBirth, DateTime $dateBase = null): int {
  // 基準日を現在の日付に設定（引数が省略された場合）
  $dateBase = $dateBase ?? new DateTime();

  // 型チェック
  if (!($dateBirth instanceof DateTime) || !($dateBase instanceof DateTime)) {
    throw new InvalidArgumentException('引数は DateTime 型でなければなりません');
  }

  // 年齢計算
  $age = $dateBase->format('Y') - $dateBirth->format('Y');

  // 加齢タイミングを計算（誕生日の前日24時）
  $timeBoundaryForAging =  new DateTime();
  $timeBoundaryForAging -> setDate($dateBirth->format('Y') + $age, $dateBirth->format('m'), $dateBirth->format('d')) 
                        -> modify('-1 day')
                        -> setTime(24, 0, 0, 0);
  
  // 満年齢の調整
  if ($timeBoundaryForAging > $dateBase) {
    return $age - 1;
  } else {
    return $age;
  }
}
```

### 日付文字列の判定（PHP）

```php:日付文字列を判定する関数（PHP）
/**
 * 年月日を表す文字列が、適正かを判断
 *
 * @param string $date 'yyyy-MM-dd' 形式の文字列
 * @return bool 存在しうる日付かの判定
 */
function isDateFormat(string $date): bool {
  if(!preg_match('/\A\d{4}-\d{2}-\d{2}\z/', $date)) return false;
  
  // 妥当性チェック（月日の簡易的チェックを含む）
  list($year, $month, $day) = explode("-", $date);
  return checkdate((int)$month, (int)$day, (int)$year);
}
```

:::details テスト

```php:検証コード
echo var_export(isDateFormat("2024-02-29")), PHP_EOL;  // -> true
echo var_export(isDateFormat("2023-02-29")), PHP_EOL;  // -> false
```

:::

日付データの確認に関しては、PHP4 以降から利用できる `checkdate()` 関数を使います。これは `checkdate(int $month, int $day, int $year): bool` の形となっており、閏年を考慮した日付の妥当性をチェックし、真偽値で返してくれる関数です。

https://www.php.net/manual/ja/function.checkdate.php

`\A\d{4}-\d{2}-\d{2}\z/` の正規表現については、始端終端に余計なものがない形での「4桁-2桁-2桁」を指定しています。

https://www.php.net/manual/ja/regexp.reference.escape.php

### 満年齢と経過日数の算出（PHP）

```php:生年月日から満年齢と経過日数を算出する関数（PHP）
/**
 * 生年月日から年齢と経過日数を計算
 *
 * @param string $textBirth 生年月日（yyyy-MM-dd形式）
 * @param DateTime|null $dateBase 計算基準日（省略時は現在の日付）
 * @return void
 */
function calcAgeAndDays(string $textBirth, DateTime $dateBase = null){
  // 基準日を現在の日付に設定（引数が省略された場合）
  $dateBase = $dateBase ?? new DateTime();
  
  if(isDateFormat($textBirth)){
    $dateBirth =  new DateTime($textBirth);
    
    $age = calcAge($dateBirth, $dateBase);
    
    // 現年齢の加齢タイミング（≒誕生日前日24時）
    $timeAging = new DateTime();
    $timeAging -> setDate($dateBirth->format('Y') + $age, $dateBirth->format('m'), $dateBirth->format('d')) 
               -> modify('-1 day')
               -> setTime(24, 0, 0, 0);

    $days = $timeAging -> diff($dateBase) -> days;
    echo "年齢は $age 歳、経過日数は $days 日です";
  }
}
```

:::details テスト（一部）

```php:検証
calcAgeAndDays('2000-02-28', new DateTime('2023-02-28'));  // -> OK：年齢は 23 歳、経過日数は 0 日です
calcAgeAndDays('2000-02-29', new DateTime('2023-02-28'));  // -> OK：年齢は 22 歳、経過日数は 364 日です
calcAgeAndDays('2000-02-29', new DateTime('2024-02-28'));  // -> OK：年齢は 23 歳、経過日数は 364 日です
calcAgeAndDays('2000-03-01', new DateTime('2024-02-29'));  // -> OK：年齢は 23 歳、経過日数は 365 日です
```

:::

## Python

Python 全体で使う `import` 文に関しては、下記になります。

```python:全体で使用するimport文
import re
from datetime import date, datetime, time, timedelta
```

Python での日付データには、`date`, `datetime` を使うのが主です。しかし `date`, `datetime` は、生成時に **2001/02/29** や **2000/03/00** といった、存在しない日付への調整が行われないため、日付調整用の関数を拵える必要があります。

```python:数値からの日付生成
def generate_date(year: int, month: int, day: int):
  """
  存在しない日付に対応する形で date を作成する

  Args:
    year (int): 年
    month (int): 月
    day (int): 日

  Returns:
    date: 日付（閏日などを調整した形）
  """
  try:
    return date(year, month, day)
  except ValueError:
    # 無効な日付の場合、調整する
    if day == 0:
      adjusted_date = date(year, month, 1) - timedelta(days=1)
    else:
      adjusted_date = date(year, month, 1) + timedelta(days=day - 1)
    return adjusted_date
```

:::message

なお、Python では [`datetime`クラス](https://docs.python.org/ja/3.13/library/datetime.html)同士で引き算すると `datetime.timedelta` となり、`timedelta.days` で日数を取り出せるため、JavaScript にあった `dateDiff()` 関数を自作する必要はありません。

:::

### 満年齢の計算（Python）

```python:満年齢を計算する関数（Python）
def calc_age(date_birth: date, date_base: date = date.today()) -> int:
  """
  生年月日から、満年齢を計算する（時間基準）

  Args:
    date_birth (date): 生年月日
    date_base (date, optional): 計算基準日（省略時は現在の日付）

  Returns:
    int: 満年齢

  Raises:
    ValueError: 引数が datetime 型でない場合
  """

  # 型チェック
  if (not isinstance(date_birth, date)) or (not isinstance(date_base, date)):
    raise ValueError("引数は date 型で指定してください")

  # 年齢計算
  age = date_base.year - date_birth.year

  # 加齢タイミング（計算基準年）は、誕生日の前日24時
  time_boundary_for_aging = datetime.combine(generate_date(date_base.year, date_birth.month, date_birth.day -1), time(0, 0, 0)) + timedelta(hours = 24)

  # 満年齢は、加齢タイミングをまだ越えていない場合、デクリメントして返す
  if time_boundary_for_aging > datetime.combine(date_base, time(0, 0, 0)):
    return age - 1
  else:
    return age
```

### 日付文字列の判定（Python）

```python:日付文字列を判定する関数（Python）
def is_date_format(date_string: str) -> bool:
  """
  年月日を表す文字列が、適正かを判断

  Args:
    date_string (str): 'yyyy-MM-dd' 形式の文字列

  Returns:
    bool: 存在しうる日付かの判定
  """
  # 正規表現による形式チェック（yyyy-MM-dd）
  date_format_regex = r"^\d{4}-\d{2}-\d{2}$"
  if not re.match(date_format_regex, date_string):
    return False

  # 簡易的な月日の妥当性チェック（月の範囲は1〜12か、日の範囲は1〜31か）
  try:
    year, month, day = map(int, date_string.split('-'))
  except ValueError:
    return False

  if not ((1 <= month <= 12) and (1 <= day <= 31)):
    return False

  # 日付の妥当性チェック
  date_check = generate_date(year, month, day)
  if(date_check.year != year or date_check.month != month or date_check.day != day):
    return False
  else:
    return True
```

:::details テスト

```python:テスト文
print(is_date_format("2021-04-30"))  # True
print(is_date_format("2025-01-18"))  # True
print(is_date_format("2024-02-29"))  # True
print(is_date_format("2023-02-29"))  # False
print(is_date_format("2021-04-31"))  # False
print(is_date_format("2025-02-30"))  # False
print(is_date_format("abcd-01-01"))  # False
print(is_date_format("2025-13-01"))  # False
print(is_date_format("2025-01-32"))  # False
```

:::

:::details 別案：コンパイルする方法

正規表現によるチェックの部分については、コンパイルする方法もあります。

```python:compileする場合
# 正規表現による形式チェック（yyyy-MM-dd）
date_format_regex = re.compile(r'^\d{4}-\d{2}-\d{2}$')
if not date_format_regex.match(date_string):
  return False
```

ただし本ケースにおいては、利用場面は限られており また再利用を考慮しないため、公式ドキュメントに則り採用はしませんでした。

> `re.compile()` を使い、結果の正規表現オブジェクトを保存して再利用するほうが、一つのプログラムでその表現を何回も使うときに効率的です。
>
> **注釈** `re.compile()` やモジュールレベルのマッチング関数に渡された最新のパターンはコンパイル済みのものがキャッシュされるので、一度に正規表現を少ししか使わないプログラムでは正規表現をコンパイルする必要はありません。
>
> 『[公式ドキュメント](https://docs.python.org/ja/3/library/re.html#re.compile)』より一部抜粋

:::

:::details 別案：ValueErrorを利用した実在テスト

また、`datetime`, `date` は実在しない日付（閏年の考えを含む）で生成しようとすると `ValueError: day is out of range for month` と言ったようにエラーを吐き出します。これを利用する形で実在テストをすることも可能です。

```python:実在テストの別解
try:
  # パターン1：datetime.strptime() による実在テスト
  datetime.strptime(date_string, '%Y-%m-%d')

  # パターン2：date による実在テスト
  date(year, month, day)
  return True
except ValueError:
  return False
```

:::

### 満年齢と経過日数の算出（Python）

```python:生年月日から満年齢と経過日数を算出する関数（Python）
def calc_age_and_days(text_birth: str, date_base: date = date.today()):
  """
  生年月日から、満年齢と経過日数を算出

  Args:
    text_birth (str): 生年月日（'yyyy-MM-dd' 形式）
    date_base (date, optional): 計算基準日（省略時は現在の日付）
  """
  if(is_date_format(text_birth)):
    year, month, day = map(int, text_birth.split('-'))
    date_birth = generate_date(year, month, day)

    age = calc_age(date_birth, date_base)

    # 現年齢の加齢タイミング（≒ 誕生日前日24時）
    time_aging = datetime.combine(generate_date(date_birth.year + age, date_birth.month, date_birth.day -1), time(0, 0, 0)) + timedelta(hours = 24)

    days = datetime.combine(date_base, time(0, 0, 0)) - time_aging
    print(f"{age}歳、{days.days}日経過")
```

## Java

Java で使う `import` 文は、全体としては下記のとおりです。

```java:全体で使用するライブラリ集
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.time.format.DateTimeFormatter;
import java.time.format.ResolverStyle;

// 検証用に使用
import java.util.Arrays;
import java.util.List;
```

また、Java においては日付データの扱いは `LocalDate`（または `LocalDateTime`）が主となりますが、存在しない日付を使った生成が難しいです。そのため、生成用の関数を設けるようにしています。

```java:寛容にLocalDateを生成する関数
/**
 * 存在しない日付値からも寛大な形で LocalDate を生成する（例：1999-02-29 → 1999-03-01）
 * 
 * @param year 年
 * @param month 月
 * @param day 日
 * @return LocalDate 調整された日付データ
 */
public static LocalDate generateLocalDateWithLenient(int year, int month, int day) {
  DateTimeFormatter dtf = DateTimeFormatter.ISO_LOCAL_DATE.withResolverStyle(ResolverStyle.LENIENT);
  return LocalDate.parse(String.format("%04d-%02d-%02d", year, month, day), dtf);
}
```

`LocalDate`(`LocalDateTime`) は閏日等も考慮してくれるので、それ自身に対しての日数や時間の加算に対しては問題はありません（例：`{LocalDateTime}.minusDays(1).plusHours(24)`）

しかし `LocalDate.of(int year, int month, int day)` で生成する場合、存在しない日付値だと `DateTimeParseException` が生じてしまいます。

下記のコードは 後ほど登場する関数の一部で、作成当初の状態です。生年月日と計算基準日から新たに日付データを生成しようとしてますが、両方で失敗する可能性を持っています。

```java:エラー例１：誕生日が閏日の場合に問題
// 満年齢になったタイミングを生成（≒ 誕生日前日の 24 時）
LocalDateTime timeBoundaryForAging = LocalDateTime.of(
 birthDate.getYear() + age, birthDate.getMonthValue(), birthDate.getDayOfMonth(), 0, 0
).minusDays(1).plusHours(24);
// この場合 `dateBirth` が 2000/02/29 だと、`age` の値によっては閏年でない閏日を生成しかねない
```

```java:エラー例２：誕生日が１日の場合に問題
LocalDateTime timeBoundaryForAging = LocalDateTime.of(
 birthDate.getYear() + age, birthDate.getMonthValue(), birthDate.getDayOfMonth() - 1, 0, 0
).plusHours(24);
// この場合 `dateBirth` の日が 1 だと、引数となる日数が 0 となる
```

こうしたケースに対応するため、`DateTimeFormatter` が持つ `ResolverStyle`（日付文字列を解析する際の動作スタイル）を使い、`LENIENT`(寛容)で生成できるように関数を作成しました。

:::message

なお Java では日付間の計算は `Period` や `Duration` クラス、あるいは `ChronoUnit.DAYS.between()` を使って容易に行えるので、JavaScript にあった `dateDiff()` 関数を自作する必要はありません。

:::

### 満年齢の計算（Java）

```java:満年齢を計算する関数（Java）
/**
 * 生年月日から、満年齢を計算する（時間基準）
 *
 * @param birthDate 生年月日
 * @param baseDate  計算基準日（省略時は現在の日付）
 * @return int 満年齢
 */
public static int calcAge(LocalDate birthDate, LocalDate baseDate){
  if (baseDate == null) baseDate = LocalDate.now();
  int age = baseDate.getYear() - birthDate.getYear();

  // 加齢タイミング（計算基準年）は、誕生日の前日24時
  LocalDateTime timeBoundaryForAging = generateLocalDateWithLenient(
    baseDate.getYear(), birthDate.getMonthValue(), birthDate.getDayOfMonth() - 1
  ).atTime(0, 0).plusHours(24);
  LocalDateTime baseDateTime = baseDate.atTime(0, 0);

  // 満年齢は、加齢タイミングを まだ越えていない場合、デクリメントして返す
  if (timeBoundaryForAging.isAfter(baseDateTime)) {
    return age - 1;
  } else {
    return age;
  }
}
```

:::details 別案：期間を扱うクラスの利用

Java では２つの日付データの感覚を計算するために、`Period`（年月日）や `Duration`（時間）があるので、そちらを使うこともできます。

```java:ageの取得方法の別案
// Periodクラスを利用して期間を計算し、年数を取得
Period period = Period.between(birthDate, baseDate);
int age = period.getYears();
```

:::

### 日付文字列の判定（Java）

```java:日付文字列を判定する関数（Java）
  /**
   * 年月日を表す文字列が、適正かを判断
   *
   * @param dateString 'yyyy-MM-dd' 形式の文字列
   * @return boolean 存在しうる日付かの判定
   */
  public static boolean isDateFormat(String dateString){
    // 正規表現による形式チェック（yyyy-MM-dd）
    String dateFormatRegex = "^\\d{4}-\\d{2}-\\d{2}$";
    if (!dateString.matches(dateFormatRegex)) return false;

    // 簡易的な月日の妥当性チェック（月の範囲は1〜12か、日の範囲は1〜31か）
    String[] dateValues = dateString.split("-");
    int year = Integer.parseInt(dateValues[0]);
    int month = Integer.parseInt(dateValues[1]);
    int day = Integer.parseInt(dateValues[2]);
    if(month < 1 || month > 12 || day < 1 || day > 31) return false;

    // 妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
    LocalDate dateCheck = generateLocalDateWithLenient(year, month, day);
    if (dateCheck.getYear() != year || dateCheck.getMonthValue() != month || dateCheck.getDayOfMonth() != day) return false;

    return true;
  }
```

正規表現の部分は、`import java.util.regex.Pattern;` で呼び出した `Pattern.matches()` で行うこともできます。

:::details 別案：SimpleDateFormatを使った方法

最後の妥当性チェックの部分では、`generateLocalDateWithLenient()` を使って `LocalDate` を作ってズレがないかを確かめています。この関数がない場合は、`try` 文を設けて中で検証することになります。当初は `SimpleDateFormat` の `setLenient` を使ってました（デフォルトでは `true` になっているので、明示する意図で一文設けています）

```java:SimpleDateFormatによる別案
// 妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
try {
  SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
  sdf.setLenient(true);
  Date date = sdf.parse(dateString);
  LocalDate ld = LocalDate.parse(sdf.format(date), DateTimeFormatter.ISO_LOCAL_DATE);
  if(ld.getYear() != year || ld.getMonthValue() != month || ld.getDayOfMonth() != day) return false;
} catch (ParseException | DateTimeParseException e) {
  return false;
}
```

:::

### 満年齢と経過日数の算出（Java）

```java:生年月日から満年齢と経過日数を算出する関数（Java）
/**
 * 生年月日から、満年齢と経過日数を計算する（時間基準）
 *
 * @param birthDate 生年月日
 * @param baseDate  計算基準日（省略時は現在の日付）
 * @return void
 */
public static void calcAgeAndDays(String textBirth, LocalDate baseDate){
  if (baseDate == null) baseDate = LocalDate.now();

  if (isDateFormat(textBirth)) {
    LocalDate birthDate = LocalDate.parse(textBirth, DateTimeFormatter.ISO_LOCAL_DATE);

    int age = calcAge(birthDate, baseDate);

    // 現年齢の加齢タイミング（≒誕生日前日24時）
    LocalDateTime timeAging = generateLocalDateWithLenient(
      birthDate.getYear() + age, birthDate.getMonthValue(), birthDate.getDayOfMonth() -1
    ).atTime(0,0).plusHours(24);
    LocalDateTime baseDateTime = baseDate.atTime(0, 0);

    // 計算基準日と現年齢の加齢タイミング間の日数を算出
    long days = ChronoUnit.DAYS.between(timeAging, baseDateTime);

    System.out.println("満年齢: " + age + "経過日数: " + days);
  }
}
```

経過日数の算出には、`ChronoUnit.DAYS.between()` を使ってますが、`Duration.between().toDays()` でもいいです。

## C言語

全体で必要となるものは、下記のとおりです。

```c:全体で必要なライブラリ
#include <stdio.h>
#include <time.h>
#include <string.h>
#include <regex.h>
#include <stdlib.h>
```

C 言語で日時を扱う際には、`tm` 構造体や `time_t` 型を使うのが主となります。存在しない日付については自動で調整はしてくれないので、適正な日付値に調整する関数を作成しておきます。

```c:適正な日付データに調整する関数
void adjustDate(int *year, int *month, int *day){
  struct tm date = {0};
  date.tm_year = *year - 1900;
  date.tm_mon = *month - 1;
  date.tm_mday = *day;

  // tm 構造体を time_t に変換（適正でない値は mktime　関数内で調節）
  mktime(&date);

  *year = date.tm_year + 1900;
  *month = date.tm_mon + 1;
  *day = date.tm_mday;
}
```

これは引数となる `year`, `month`, `day` をポインタで受け取っています。参照の形で受け取っておき `time_t` に変換、その際に調整された各種値をポインタを使って入れ直すことで 存在する日付値になるよう処理しています。

### 満年齢の計算（C言語）

```c:満年齢を計算する関数（C言語）
int calcAge(struct tm dateBirth, struct tm *dateBase){
  // 現在の日付を取得（dateBaseがNullの場合は現在を使用）
  time_t now = time(NULL);
  struct tm dateCalc = *localtime(&now);
  if(dateBase != NULL) dateCalc = *dateBase;

  // 年齢を計算
  int age = dateCalc.tm_year - dateBirth.tm_year;

  // 満年齢時の前日24時を生成
  struct tm timeBoundaryForAging = {0};
  timeBoundaryForAging.tm_year = dateCalc.tm_year;
  timeBoundaryForAging.tm_mon = dateBirth.tm_mon;
  timeBoundaryForAging.tm_mday = dateBirth.tm_mday -1;
  timeBoundaryForAging.tm_hour = 24;

  time_t boundaryTime = mktime(&timeBoundaryForAging);

  // 満年齢を調整
  if (difftime(boundaryTime, mktime(&dateCalc)) > 0) {
    return age - 1;
  } else {
    return age;
  }
};
```

:::details テスト

```c:手動テスト用
void testCalcAge(void){
  // 生年月日の入力を受け付け、適正な日付として調整
  char textBirth[11];
  int birthYear, birthMonth, birthDay;
  printf("生年月日を 10 文字で入力してください（例：2000-01-23）: ");
  scanf("%s", textBirth);
  sscanf(textBirth, "%d-%d-%d", &birthYear, &birthMonth, &birthDay);

  struct tm dateBirth = {0};
  dateBirth.tm_year = birthYear - 1900;
  dateBirth.tm_mon = birthMonth - 1;
  dateBirth.tm_mday = birthDay;

  // 計算基準日の入力を受け付け、適正な日付として調整
  char textBase[11];
  struct tm dateBase = {0};
  printf("計算日を 10 文字で入力してください（0 入力で今日にします）: ");
  scanf("%s", textBase);
  if(strcmp(textBase, "0") == 0){
    // 現在の日付を取得
    time_t now = time(NULL);
    dateBase = *localtime(&now);
  }else{
    // 計算基準日を適正な日付として調整
    int baseYear, baseMonth, baseDay;
    sscanf(textBase, "%d-%d-%d", &baseYear, &baseMonth, &baseDay);

    dateBase.tm_year = baseYear - 1900;
    dateBase.tm_mon = baseMonth - 1;
    dateBase.tm_mday = baseDay;
  }

  // 満年齢を計算
  int age = calcAge(dateBirth, &dateBase);
  printf("満年齢: %d\n", age);
}



int main(){
  while(1){
    static int isContinue = 1;

    // テスト実行
    testCalcAge();

    printf("続けますか？ 0 で終了します。: ");
    scanf("%d", &isContinue);
    if(isContinue == 0) break;
  }
  return 0;
}
```

:::

### 期間内の日数計算（C言語）

```c:期間の日数を計算する関数（C言語）
int absDiffTime(time_t time1, time_t time2){
  return abs((int)(difftime(time1, time2) / (60 * 60 * 24)));
}
```

### 日付文字列の判定（C言語）

```c:日付文字列を判定する関数（C言語）
int isDateFormat(char *text){
  int result = 1;

  // 正規表現による判定
  regex_t ptnBuf;
  const char regex[] = "^[0-9]{4}-[0-9]{2}-[0-9]{2}$";
  if(regcomp(&ptnBuf, regex, REG_EXTENDED | REG_NOSUB) == 0){
    int year, month, day;
    sscanf(text, "%d-%d-%d", &year, &month, &day);

    // 各値が適正かどうかを判定（月数、日数が現実の範囲内か）
    if((month >= 1 && month <= 12) && (day >= 1 && day <= 31)){
      // 各値のコピーを adjustDate 関数に通し、変化しないかで判定
      int yearCopy = year, monthCopy = month, dayCopy = day;
      adjustDate(&yearCopy, &monthCopy, &dayCopy);
      if(year == yearCopy && month == monthCopy && day == dayCopy){
        result = 0;
      }
    }
  }

  // 領域を解放し、結果を返す
  regfree(&ptnBuf);
  return result;
}
```

:::details テスト

```c:手動テスト用
void testIsDateFormat(void){
  char textTest[11];
  printf("テストする日付を 10 文字で入力してください（例：2000-01-23）: ");
  scanf("%s", textTest);
  char textResult[4];
  if(isDateFormat(textTest) == 0){
    strcpy(textResult, "OK");
  }else{
    strcpy(textResult, "NG");
  }
  // 入力値と結果の表示
  printf("入力値: %s, 結果: %s\n", textTest, textResult);
}



int main(){
  while(1){
    static int isContinue = 1;

    // テスト実行
    testIsDateFormat();

    printf("続けますか？ 0 で終了します。: ");
    scanf("%d", &isContinue);
    if(isContinue == 0) break;
  }
  return 0;
}
```

:::

他言語と違い、ここでは挙動の流れを変えています。

他の言語では、条件を満たしていない時点で `return false;` といった形で関数を抜けるようにしていま。一方の C 言語では どのケースでもブロック最後まで行くようになっています。これは正規表現用に確保したメモリ領域を `regfree()` で解放する必要があるためです。

### 満年齢と経過日数の算出（C言語）

```c:生年月日から満年齢と経過日数を算出する関数（C言語）
void calcAgeAndDays(void){
  // 生年月日の入力
  char textBirth[11];
  printf("生年月日を 10 文字で入力してください（例：2000-01-23）: ");
  scanf("%s", textBirth);

  // 生年月日の形式チェック
  if(isDateFormat(textBirth) != 0){
    printf("生年月日の形式が正しくありません\n");
    return;
  } else {
    // 入力値から tm 構造体を生成
    int birthYear, birthMonth, birthDay;
    sscanf(textBirth, "%d-%d-%d", &birthYear, &birthMonth, &birthDay);

    struct tm dateBirth = {0};
    dateBirth.tm_year = birthYear - 1900;
    dateBirth.tm_mon = birthMonth - 1;
    dateBirth.tm_mday = birthDay;

    // 計算基準日の入力
    char textBase[11];
    printf("計算日を 10 文字で入力してください（0 入力で今日にします）: ");
    scanf("%s", textBase);

    // 入力値に応じた処理分岐
    struct tm dateBase = {0};
    if(strcmp(textBase, "0") == 0){
      // 現在の日付情報を dateBase に代入
      time_t now = time(NULL);
      dateBase = *localtime(&now);
    } else if(isDateFormat(textBase) != 0){
      // 適正な日付形式でないためやり直し
      printf("計算日の形式が正しくありません\n");
      return;
    } else {
      // 入力値から tm 構造体を生成
      int baseYear, baseMonth, baseDay;
      sscanf(textBase, "%d-%d-%d", &baseYear, &baseMonth, &baseDay);

      dateBase.tm_year = baseYear - 1900;
      dateBase.tm_mon = baseMonth - 1;
      dateBase.tm_mday = baseDay;
    }

    // 満年齢を計算
    int age = calcAge(dateBirth, &dateBase);

    // 現年齢の加齢タイミング（≒誕生日前日24時）
    struct tm timeAging = dateBirth;
    timeAging.tm_year += age;
    timeAging.tm_mday -= 1;
    timeAging.tm_hour = 24;

    // 計算基準日との日数差分（≒経過日数）を算出
    time_t date1 = mktime(&timeAging);
    time_t date2 = mktime(&dateBase);
    int days = absDiffTime(date1, date2);

    printf("年齢は %d 歳、経過日数は %d 日です\n", age, days);
  }
}



int main(){
  while(1){
    static int isContinue = 1;

    calcAgeAndDays();

    printf("続けますか？ 0 で終了します。: ");
    scanf("%d", &isContinue);
    if(isContinue == 0) break;
  }
  return 0;
}
```

:::message alert

暫定的な状態です。後ほど、関数と検証用に切り分けする予定です。

:::

## まとめ

:::details JavaScriptコード

```javascript
/**
 * 生年月日から、満年齢を計算する（時間基準）
 *
 * @param {date} dateBirth - 生年月日
 * @param {date} dateBase - 計算基準日（省略時は現在の日付）
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
    !(month >= 1 && month <= 12) ||
    !(day >= 1 && day <= 31)
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



/**
 * 生年月日から年齢と経過日数を計算
 *
 * @param {string} textBirth - 生年月日（yyyy-MM-dd形式）
 * @param {date} dateBase - 計算基準日（省略時は現在の日付）
 */
function calcAgeAndDays(textBitrh, dateBase = new Date()){
  if(isDateFormat(textBitrh)) {
    const [year, month, day] = textBitrh.split('-').map(Number);
    const dateBirth = new Date(year, month - 1, day);

    const age = calcAge(dateBirth, dateBase);

    // 現年齢の加齢タイミング（≒誕生日前日24時）
    const timeAging = new Date(
      dateBirth.getFullYear() + age,
      dateBirth.getMonth(),
      dateBirth.getDate() - 1,
      24
    );

    const days = dateDiff(timeAging, dateBase);

    console.log(`年齢は ${age} 歳、経過日数は ${days} 日です`);
  }
}
```

:::

:::details VBAコード

```vba
' 生年月日から、満年齢を計算する（時間基準）
'
' 引数 dateBirth As Date - 生年月日
' 引数 (Optional) dateBase As Date - 計算基準日（省略時は現在の日付）
' 返り値 Integer - 満年齢
'
Function CalcAge(dateBirth As Date, Optional dateBase As Date = 0) As Integer
  ' 引数チェック
  If IsDate(dateBirth) = False Then
    Err.Raise vbObjectError + 1, , "引数はDate型でなければなりません"
  End If

  Dim dateCalc As Date
  If (IsDate(dateBase) = False) Or (dateBase = 0) Then
    dateCalc = Now
  Else
    dateCalc = dateBase
  End If

  Dim age As Integer
  age = Year(dateCalc) - Year(dateBirth)

  ' 加齢タイミング（計算基準年）は、誕生日の前日24時
  Dim timeBoundaryForAging As Date
  timeBoundaryForAging = DateSerial(Year(dateCalc), Month(dateBirth), Day(dateBirth) - 1) + TimeSerial(24, 0, 0)

  ' 満年齢は、加齢タイミングを まだ越えていない場合、デクリメントして返す
  If timeBoundaryForAging > dateCalc Then
    CalcAge = age - 1
  Else
    CalcAge = age
  End If
End Function



' 年月日を表す文字列が、適正かを判断
'
' 引数 dateString As String - 'yyyy-MM-dd' 形式の文字列
' 返り値 Boolean - 存在しうる日付かの判定
'
Function IsDateFormat(dateString As String) As Boolean

  ' 正規表現による形式チェック（yyyy-MM-dd）
  Dim dateFormatRegex As Object
  Set dateFormatRegex = CreateObject("VBScript.RegExp")
  With dateFormatRegex
    .IgnoreCase = True
    .Global = False
    .Pattern = "^\d{4}-\d{2}-\d{2}$"
  End With

  If Not dateFormatRegex.test(dateString) Then
    IsDateFormat = False
    Exit Function
  End If

  ' 簡易的な月日の妥当性チェック（月の範囲は1～12か、日の範囲は1～31か）
  Dim numYear As Integer
  Dim numMonth As Integer
  Dim numDay As Integer
  numYear = CInt(Split(dateString, "-")(0))
  numMonth = CInt(Split(dateString, "-")(1))
  numDay = CInt(Split(dateString, "-")(2))

  If (numMonth < 1) Or (numMonth > 12) Or (numDay < 1) Or (numDay > 31) Then
    IsDateFormat = False
    Exit Function
  End If

' date型による妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
  Dim dateCheck As Date
  dateCheck = DateSerial(numYear, numMonth, numDay)
  If (Year(dateCheck) <> numYear) Or (Month(dateCheck) <> numMonth) Or (Day(dateCheck) <> numDay) Then
    IsDateFormat = False
    Exit Function
  End If

  IsDateFormat = True
End Function



' 生年月日から年齢と経過日数を計算
'
' 引数 textBirth As String - 生年月日（yyyy-MM-dd形式）
' 引数 (Optional) dateBase As Date - 計算基準日（省略時は現在の日付）
'
Function CalcAgeAndDays(textBirth As String, Optional dateBase As Date = 0)
  Dim dateCalc As Date
  If (IsDate(dateBase) = False) Or (dateBase = 0) Then
    dateCalc = Now
  Else
    dateCalc = dateBase
  End If

  If (IsDateFormat(textBirth) = True) Then
    Dim dateBirth As Date
    dateBirth = DateSerial( _
      CInt(Split(textBirth, "-")(0)), _
      CInt(Split(textBirth, "-")(1)), _
      CInt(Split(textBirth, "-")(2)) _
    )
    
    Dim age As Integer
    age = CalcAge(dateBirth, dateCalc)
    
    ' 現年齢の加齢タイミング（≒誕生日前日24時）
    Dim timeAging As Date
    timeAging = DateSerial( _
      Year(dateBirth) + age, _
      Month(dateBirth), _
      Day(dateBirth) - 1 _
    ) + TimeSerial(24, 0, 0)

    Dim days As Integer
    days = DateDiff("d", timeAging, dateCalc)  ' DateDiffは標準で用意されている
    
    Debug.Print "年齢は " & age & " 歳、経過日数は " & days & " 日です"
  End If
End Function
```

:::

:::details PHPコード

```php
/**
 * 生年月日から、満年齢を計算する（時間基準）
 *
 * @param DateTime $dateBirth 生年月日
 * @param DateTime|null $dateBase 計算基準日（省略時は現在の日付）
 * @return int 満年齢
 * @throws InvalidArgumentException 引数が DateTime 型でない場合
 */
function calcAge(DateTime $dateBirth, DateTime $dateBase = null): int {
  // 基準日を現在の日付に設定（引数が省略された場合）
  $dateBase = $dateBase ?? new DateTime();

  // 型チェック
  if (!($dateBirth instanceof DateTime) || !($dateBase instanceof DateTime)) {
    throw new InvalidArgumentException('引数は DateTime 型でなければなりません');
  }

  // 年齢計算
  $age = $dateBase->format('Y') - $dateBirth->format('Y');

  // 加齢タイミングを計算（誕生日の前日24時）
  $timeBoundaryForAging =  new DateTime();
  $timeBoundaryForAging -> setDate($dateBirth->format('Y') + $age, $dateBirth->format('m'), $dateBirth->format('d')) 
                        -> modify('-1 day')
                        -> setTime(24, 0, 0, 0);
  
  // 満年齢の調整
  if ($timeBoundaryForAging > $dateBase) {
    return $age - 1;
  } else {
    return $age;
  }
}


/**
 * 年月日を表す文字列が、適正かを判断
 *
 * @param string $date 'yyyy-MM-dd' 形式の文字列
 * @return bool 存在しうる日付かの判定
 */
function isDateFormat(string $date): bool {
  if(!preg_match('/\A\d{4}-\d{2}-\d{2}\z/', $date)) return false;
  
  // 妥当性チェック（月日の簡易的チェックを含む）
  list($year, $month, $day) = explode("-", $date);
  return checkdate((int)$month, (int)$day, (int)$year);
}


/**
 * 生年月日から年齢と経過日数を計算
 *
 * @param string $textBirth 生年月日（yyyy-MM-dd形式）
 * @param DateTime|null $dateBase 計算基準日（省略時は現在の日付）
 * @return void
 */
function calcAgeAndDays(string $textBirth, DateTime $dateBase = null){
  // 基準日を現在の日付に設定（引数が省略された場合）
  $dateBase = $dateBase ?? new DateTime();
  
  if(isDateFormat($textBirth)){
    $dateBirth =  new DateTime($textBirth);
    
    $age = calcAge($dateBirth, $dateBase);
    
    // 現年齢の加齢タイミング（≒誕生日前日24時）
    $timeAging = new DateTime();
    $timeAging -> setDate($dateBirth->format('Y') + $age, $dateBirth->format('m'), $dateBirth->format('d')) 
               -> modify('-1 day')
               -> setTime(24, 0, 0, 0);

    $days = $timeAging -> diff($dateBase) -> days;
    echo "年齢は $age 歳、経過日数は $days 日です";
  }
}
```

:::

:::details Pythonコード

```python
import re
from datetime import date, datetime, time, timedelta


def generate_date(year: int, month: int, day: int):
  """
  存在しない日付に対応する形で date を作成する

  Args:
    year (int): 年
    month (int): 月
    day (int): 日

  Returns:
    date: 日付（閏日などを調整した形）
  """
  try:
    return date(year, month, day)
  except ValueError:
    # 無効な日付の場合、調整する
    if day == 0:
      adjusted_date = date(year, month, 1) - timedelta(days=1)
    else:
      adjusted_date = date(year, month, 1) + timedelta(days=day - 1)
    return adjusted_date



def calc_age(date_birth: date, date_base: date = date.today()) -> int:
  """
  生年月日から、満年齢を計算する（時間基準）

  Args:
    date_birth (date): 生年月日
    date_base (date, optional): 計算基準日（省略時は現在の日付）

  Returns:
    int: 満年齢

  Raises:
    ValueError: 引数が datetime 型でない場合
  """
  # 型チェック
  if (not isinstance(date_birth, date)) or (not isinstance(date_base, date)):
    raise ValueError("引数は date 型で指定してください")

  # 年齢計算
  age = date_base.year - date_birth.year

  # 加齢タイミング（計算基準年）は、誕生日の前日24時
  time_boundary_for_aging = datetime.combine(generate_date(date_base.year, date_birth.month, date_birth.day -1), time(0, 0, 0)) + timedelta(hours = 24)

  # 満年齢は、加齢タイミングをまだ越えていない場合、デクリメントして返す
  if time_boundary_for_aging > datetime.combine(date_base, time(0, 0, 0)):
    return age - 1
  else:
    return age



def is_date_format(date_string: str) -> bool:
  """
  年月日を表す文字列が、適正かを判断

  Args:
    date_string (str): 'yyyy-MM-dd' 形式の文字列

  Returns:
    bool: 存在しうる日付かの判定
  """
  # 正規表現による形式チェック（yyyy-MM-dd）
  date_format_regex = r"^\d{4}-\d{2}-\d{2}$"
  if not re.match(date_format_regex, date_string):
    return False

  # 簡易的な月日の妥当性チェック（月の範囲は1〜12か、日の範囲は1〜31か）
  try:
    year, month, day = map(int, date_string.split('-'))
  except ValueError:
    return False

  if not ((1 <= month <= 12) and (1 <= day <= 31)):
    return False

  # 日付の妥当性チェック
  date_check = generate_date(year, month, day)
  if(date_check.year != year or date_check.month != month or date_check.day != day):
    return False
  else:
    return True



def calc_age_and_days(text_birth: str, date_base: date = date.today()):
  """
  生年月日から、満年齢と経過日数を算出

  Args:
    text_birth (str): 生年月日（'yyyy-MM-dd' 形式）
    date_base (date, optional): 計算基準日（省略時は現在の日付）
  """
  if(is_date_format(text_birth)):
    year, month, day = map(int, text_birth.split('-'))
    date_birth = generate_date(year, month, day)

    age = calc_age(date_birth, date_base)

    # 現年齢の加齢タイミング（≒ 誕生日前日24時）
    time_aging = datetime.combine(generate_date(date_birth.year + age, date_birth.month, date_birth.day -1), time(0, 0, 0)) + timedelta(hours = 24)

    days = datetime.combine(date_base, time(0, 0, 0)) - time_aging
    print(f"{age}歳、{days.days}日経過")
```

:::

:::details Javaコード

```java:AgeCalculator.java
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.time.format.DateTimeFormatter;
import java.time.format.ResolverStyle;
import java.util.Arrays;
import java.util.List;



public class AgeCalculator {

  /**
   * 存在しない日付値からも寛大な形で LocalDate を生成する（例：1999-02-29 → 1999-03-01）
   * 
   * @param year 年
   * @param month 月
   * @param day 日
   * @return LocalDate 調整された日付データ
   */
  public static LocalDate generateLocalDateWithLenient(int year, int month, int day) {
    DateTimeFormatter dtf = DateTimeFormatter.ISO_LOCAL_DATE.withResolverStyle(ResolverStyle.LENIENT);
    return LocalDate.parse(String.format("%04d-%02d-%02d", year, month, day), dtf);
  }



  /**
   * 生年月日から、満年齢を計算する（時間基準）
   *
   * @param birthDate 生年月日
   * @param baseDate  計算基準日（省略時は現在の日付）
   * @return int 満年齢
   */
  public static int calcAge(LocalDate birthDate, LocalDate baseDate){
    if (baseDate == null) baseDate = LocalDate.now();
    int age = baseDate.getYear() - birthDate.getYear();

    // 加齢タイミング（計算基準年）は、誕生日の前日24時
    LocalDateTime timeBoundaryForAging = generateLocalDateWithLenient(
      baseDate.getYear(), birthDate.getMonthValue(), birthDate.getDayOfMonth() - 1
    ).atTime(0, 0).plusHours(24);
    LocalDateTime baseDateTime = baseDate.atTime(0, 0);

    // 満年齢は、加齢タイミングを まだ越えていない場合、デクリメントして返す
    if (timeBoundaryForAging.isAfter(baseDateTime)) {
      return age - 1;
    } else {
      return age;
    }
  }



  /**
   * 年月日を表す文字列が、適正かを判断
   *
   * @param dateString 'yyyy-MM-dd' 形式の文字列
   * @return boolean 存在しうる日付かの判定
   */
  public static boolean isDateFormat(String dateString){
    // 正規表現による形式チェック（yyyy-MM-dd）
    String dateFormatRegex = "^\\d{4}-\\d{2}-\\d{2}$";
    if (!dateString.matches(dateFormatRegex)) return false;

    // 簡易的な月日の妥当性チェック（月の範囲は1〜12か、日の範囲は1〜31か）
    String[] dateValues = dateString.split("-");
    int year = Integer.parseInt(dateValues[0]);
    int month = Integer.parseInt(dateValues[1]);
    int day = Integer.parseInt(dateValues[2]);
    if(month < 1 || month > 12 || day < 1 || day > 31) return false;

    // 妥当性チェック（入力値が `2/31` といった存在しない日付だったり、閏日により実際の日付とズレてないか）
    LocalDate dateCheck = generateLocalDateWithLenient(year, month, day);
    if (dateCheck.getYear() != year || dateCheck.getMonthValue() != month || dateCheck.getDayOfMonth() != day) return false;

    return true;
  }



  /**
   * 生年月日から、満年齢と経過日数を計算する（時間基準）
   *
   * @param birthDate 生年月日
   * @param baseDate  計算基準日（省略時は現在の日付）
   * @return void
   */
  public static void calcAgeAndDays(String textBirth, LocalDate baseDate){
    if (baseDate == null) baseDate = LocalDate.now();

    if (isDateFormat(textBirth)) {
      LocalDate birthDate = LocalDate.parse(textBirth, DateTimeFormatter.ISO_LOCAL_DATE);

      int age = calcAge(birthDate, baseDate);

      // 現年齢の加齢タイミング（≒誕生日前日24時）
      LocalDateTime timeAging = generateLocalDateWithLenient(
        birthDate.getYear() + age, birthDate.getMonthValue(), birthDate.getDayOfMonth() -1
      ).atTime(0,0).plusHours(24);
      LocalDateTime baseDateTime = baseDate.atTime(0, 0);

      // 計算基準日と現年齢の加齢タイミング間の日数を算出
      long days = ChronoUnit.DAYS.between(timeAging, baseDateTime);

      System.out.println("満年齢: " + age + "経過日数: " + days);
    }
  }



  public static void main(String[] args) {
        // テストデータの準備
        List<TestData> testCases = Arrays.asList(
            new TestData("1999-02-28", LocalDate.of(2023, 2, 28)),
            new TestData("1999-03-01", LocalDate.of(2023, 2, 28)),
            new TestData("2000-02-28", LocalDate.of(2023, 2, 28)),
            new TestData("2000-02-29", LocalDate.of(2023, 2, 28)),
            new TestData("2000-03-01", LocalDate.of(2023, 2, 28)),
            new TestData("1999-02-28", LocalDate.of(2023, 3, 1)),
            new TestData("1999-03-01", LocalDate.of(2023, 3, 1)),
            new TestData("2000-02-28", LocalDate.of(2023, 3, 1)),
            new TestData("2000-02-29", LocalDate.of(2023, 3, 1)),
            new TestData("2000-03-01", LocalDate.of(2023, 3, 1)),
            new TestData("1999-02-28", LocalDate.of(2024, 2, 28)),
            new TestData("1999-03-01", LocalDate.of(2024, 2, 28)),
            new TestData("2000-02-28", LocalDate.of(2024, 2, 28)),
            new TestData("2000-02-29", LocalDate.of(2024, 2, 28)),
            new TestData("2000-03-01", LocalDate.of(2024, 2, 28)),
            new TestData("1999-02-28", LocalDate.of(2024, 2, 29)),
            new TestData("1999-03-01", LocalDate.of(2024, 2, 29)),
            new TestData("2000-02-28", LocalDate.of(2024, 2, 29)),
            new TestData("2000-02-29", LocalDate.of(2024, 2, 29)),
            new TestData("2000-03-01", LocalDate.of(2024, 2, 29)),
            new TestData("1999-02-28", LocalDate.of(2024, 3, 1)),
            new TestData("1999-03-01", LocalDate.of(2024, 3, 1)),
            new TestData("2000-02-28", LocalDate.of(2024, 3, 1)),
            new TestData("2000-02-29", LocalDate.of(2024, 3, 1)),
            new TestData("2000-03-01", LocalDate.of(2024, 3, 1))
        );

        // テストの実行
        for (TestData testCase : testCases) {
            String birthDate = testCase.birthDate;
            LocalDate referenceDate = testCase.referenceDate;

            // calcAgeAndDays関数を呼び出し
            System.out.println("生年月日: " + birthDate + ", 計算基準日: " + referenceDate);
            calcAgeAndDays(birthDate, referenceDate);
        }
  }
  // テストデータの保持用クラス
  static class TestData {
    String birthDate;
    LocalDate referenceDate;

    TestData(String birthDate, LocalDate referenceDate) {
      this.birthDate = birthDate;
      this.referenceDate = referenceDate;
    }
  }
}
```

:::

:::details Cコード

```c
#include <stdio.h>
#include <time.h>
#include <string.h>
#include <regex.h>
#include <stdlib.h>

// プロトタイプ宣言
void adjustDate(int *, int *, int *);
int calcAge(struct tm, struct tm *);
int absDiffTime(time_t, time_t);
int isDateFormat(char *);
void calcAgeAndDays(struct tm, struct tm *);
void testCalcAge(void);
void testIsDateFormat(void);
void testCalcAgeAndDays(void)

int main(){
  while(1){
    static int isContinue = 1;

    calcAgeAndDays();

    printf("続けますか？ 0 で終了します。: ");
    scanf("%d", &isContinue);
    if(isContinue == 0) break;
  }
  return 0;
}



// 満年齢を計算する関数
int calcAge(struct tm dateBirth, struct tm *dateBase){
  // 現在の日付を取得（dateBaseがNullの場合は現在を使用）
  time_t now = time(NULL);
  struct tm dateCalc = *localtime(&now);
  if(dateBase != NULL) dateCalc = *dateBase;

  // 年齢を計算
  int age = dateCalc.tm_year - dateBirth.tm_year;

  // 満年齢時の前日24時を生成
  struct tm timeBoundaryForAging = {0};
  timeBoundaryForAging.tm_year = dateCalc.tm_year;
  timeBoundaryForAging.tm_mon = dateBirth.tm_mon;
  timeBoundaryForAging.tm_mday = dateBirth.tm_mday -1;
  timeBoundaryForAging.tm_hour = 24;

  time_t boundaryTime = mktime(&timeBoundaryForAging);

  // 満年齢を調整
  if (difftime(boundaryTime, mktime(&dateCalc)) > 0) {
    return age - 1;
  } else {
    return age;
  }
};



void testCalcAge(void){
  // 生年月日の入力を受け付け、適正な日付として調整
  char textBirth[11];
  int birthYear, birthMonth, birthDay;
  printf("生年月日を 10 文字で入力してください（例：2000-01-23）: ");
  scanf("%s", textBirth);
  sscanf(textBirth, "%d-%d-%d", &birthYear, &birthMonth, &birthDay);

  struct tm dateBirth = {0};
  dateBirth.tm_year = birthYear - 1900;
  dateBirth.tm_mon = birthMonth - 1;
  dateBirth.tm_mday = birthDay;

  // 計算基準日の入力を受け付け、適正な日付として調整
  char textBase[11];
  struct tm dateBase = {0};
  printf("計算日を 10 文字で入力してください（0 入力で今日にします）: ");
  scanf("%s", textBase);
  if(strcmp(textBase, "0") == 0){
    // 現在の日付を取得
    time_t now = time(NULL);
    dateBase = *localtime(&now);
  }else{
    // 計算基準日を適正な日付として調整
    int baseYear, baseMonth, baseDay;
    sscanf(textBase, "%d-%d-%d", &baseYear, &baseMonth, &baseDay);

    dateBase.tm_year = baseYear - 1900;
    dateBase.tm_mon = baseMonth - 1;
    dateBase.tm_mday = baseDay;
  }

  // 満年齢を計算
  int age = calcAge(dateBirth, &dateBase);
  printf("満年齢: %d\n", age);
}



// 適正な日付データに調整する関数（例：2001/02/29 -> 2001/03/01）
void adjustDate(int *year, int *month, int *day){
  struct tm date = {0};
  date.tm_year = *year - 1900;
  date.tm_mon = *month - 1;
  date.tm_mday = *day;

  // tm 構造体を time_t に変換（適正でない値は mktime　関数内で調節）
  mktime(&date);

  *year = date.tm_year + 1900;
  *month = date.tm_mon + 1;
  *day = date.tm_mday;
}



// `yyyy-MM-dd` 形式の文字列かどうかを判定する関数
int isDateFormat(char *text){
  int result = 1;

  // 正規表現による判定
  regex_t ptnBuf;
  const char regex[] = "^[0-9]{4}-[0-9]{2}-[0-9]{2}$";
  if(regcomp(&ptnBuf, regex, REG_EXTENDED | REG_NOSUB) == 0){
    int year, month, day;
    sscanf(text, "%d-%d-%d", &year, &month, &day);

    // 各値が適正かどうかを判定（月数、日数が現実の範囲内か）
    if((month >= 1 && month <= 12) && (day >= 1 && day <= 31)){
      // 各値のコピーを adjustDate 関数に通し、変化しないかで判定
      int yearCopy = year, monthCopy = month, dayCopy = day;
      adjustDate(&yearCopy, &monthCopy, &dayCopy);
      if(year == yearCopy && month == monthCopy && day == dayCopy){
        result = 0;
      }
    }
  }

  // 領域を解放し、結果を返す
  regfree(&ptnBuf);
  return result;
}



void testIsDateFormat(void){
  char textTest[11];
  printf("テストする日付を 10 文字で入力してください（例：2000-01-23）: ");
  scanf("%s", textTest);
  char textResult[4];
  if(isDateFormat(textTest) == 0){
    strcpy(textResult, "OK");
  }else{
    strcpy(textResult, "NG");
  }
  // 入力値と結果の表示
  printf("入力値: %s, 結果: %s\n", textTest, textResult);
}



// ２つの日付間にある日数（絶対値）を算出
int absDiffTime(time_t time1, time_t time2){
  return abs((int)(difftime(time1, time2) / (60 * 60 * 24)));
}



void calcAgeAndDays(void){
  // 生年月日の入力
  char textBirth[11];
  printf("生年月日を 10 文字で入力してください（例：2000-01-23）: ");
  scanf("%s", textBirth);

  // 生年月日の形式チェック
  if(isDateFormat(textBirth) != 0){
    printf("生年月日の形式が正しくありません\n");
    return;
  } else {
    // 入力値から tm 構造体を生成
    int birthYear, birthMonth, birthDay;
    sscanf(textBirth, "%d-%d-%d", &birthYear, &birthMonth, &birthDay);

    struct tm dateBirth = {0};
    dateBirth.tm_year = birthYear - 1900;
    dateBirth.tm_mon = birthMonth - 1;
    dateBirth.tm_mday = birthDay;

    // 計算基準日の入力
    char textBase[11];
    printf("計算日を 10 文字で入力してください（0 入力で今日にします）: ");
    scanf("%s", textBase);

    // 入力値に応じた処理分岐
    struct tm dateBase = {0};
    if(strcmp(textBase, "0") == 0){
      // 現在の日付情報を dateBase に代入
      time_t now = time(NULL);
      dateBase = *localtime(&now);
    } else if(isDateFormat(textBase) != 0){
      // 適正な日付形式でないためやり直し
      printf("計算日の形式が正しくありません\n");
      return;
    } else {
      // 入力値から tm 構造体を生成
      int baseYear, baseMonth, baseDay;
      sscanf(textBase, "%d-%d-%d", &baseYear, &baseMonth, &baseDay);

      dateBase.tm_year = baseYear - 1900;
      dateBase.tm_mon = baseMonth - 1;
      dateBase.tm_mday = baseDay;
    }

    // 満年齢を計算
    int age = calcAge(dateBirth, &dateBase);

    // 現年齢の加齢タイミング（≒誕生日前日24時）
    struct tm timeAging = dateBirth;
    timeAging.tm_year += age;
    timeAging.tm_mday -= 1;
    timeAging.tm_hour = 24;

    // 計算基準日との日数差分（≒経過日数）を算出
    time_t date1 = mktime(&timeAging);
    time_t date2 = mktime(&dateBase);
    int days = absDiffTime(date1, date2);

    printf("年齢は %d 歳、経過日数は %d 日です\n", age, days);
  }
}
```

:::
