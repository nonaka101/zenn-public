---
title: "🐥 指定範囲内の乱数を生成する"
---

## 本処理の目的

PowerShell を扱う際、乱数が欲しくなる場面というのが出てきます。下記はその一例です。

- テストデータの生成（ID など）
- **パスワードやトークン**の生成
- 配列要素をシャッフルする（抽選など）

PowerShell で乱数を扱う方法は いくつかありますが、本項では暗号学的に十分な強度を持つ乱数 `System.Security.Cryptography.RandomNumberGenerator` を使い、様々なケースに対応できるような関数を目指します。

:::message

[Utility モジュール](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/get-random)には 簡易的に乱数を扱える `Get-Random` コマンドレットがあります。

しかし `Get-Random` は一般的な用途には便利ですが、疑似乱数のため暗号学的に安全な乱数であることは保証されません。そのため本書では パスワードやトークンにも利用できるだけの強度を持つ `RandomNumberGenerator` を扱います。

なお、PowerShell 7.4 以降であればこの問題を解決した `Get-SecureRandom` を使えるようになっています。しかしこれも互換性の問題から、本書では取り上げません。

:::

### `System.Security.Cryptography.RandomNumberGenerator`

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.cryptography.randomnumbergenerator

乱数生成器を準備するには、`RandomNumberGenerator` クラスの静的メソッド `Create()` を使います。生成されたインスタンスにバイト情報（バイト数、もしくはバイト配列）を渡すことで、`Byte[]` での乱数を生成してくれます。

本書では わかりやすい `GetBytes(Byte[])` の方で扱っていきます。

```ps1
# 乱数生成器（Random Number Generator）の作成
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()

# 1バイトの配列を生成し それを生成機に渡すと、中身を書き換えてくれる
# 注：ここで扱う `$OneByte` はバイト配列なので、値を取り出す場合は添字 `[0]` をつける
$OneByte = New-Object byte[] 1
$rng.GetBytes($OneByte) # <- 生成した乱数は、$OneByte に格納される
$OneByte[0]  # -> 3
$rng.GetBytes($OneByte)
$OneByte[0]  # -> 249
```

ここで生成される乱数は、**1バイト情報であることから `0` 〜 `255` の範囲を取ります**。

:::message

なお 複数要素のバイト配列を渡すと、配列の各要素が乱数バイトで埋められます。複数の乱数を扱う場合や、256 以上の範囲で乱数を扱いたい場合などに使うと良いかもしれません。  
（ただ 複数バイトを1つの整数として扱う場合は、バイト順や整数への変換方法を別途考慮する必要が別途出てきます）

```ps1
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()

# 2バイトの配列の場合を扱ってみる
$TwoByte = New-Object byte[] 2
$rng.GetBytes($TwoByte)
$TwoByte  # -> 190, 2

# 添字を使えば個別に取り出しが可能
$rng.GetBytes($TwoByte)
$TwoByte[0]  # -> 7
$TwoByte[1]  # -> 12
```

:::

### 指定範囲内の乱数を取得したい場合

`RandomNumberGenerator.GetBytes(Byte[])` で埋められるバイト配列の各要素は **`0` 〜 `255`** の値を取ります。

仮に「`0` 〜 `99` までの範囲で乱数を作りたい」という場面を想定しましょう。以降 2 つの範囲について説明をすることになるので、下記のように呼称します。

- **生成範囲**：乱数生成器から生成される乱数範囲（`0` 〜 `255`）
- **必要範囲**：必要とされる、最終的な乱数範囲（`0` 〜 `99`）

`GetBytes` で得られる*生成範囲*（`0` 〜 `255`）を*必要範囲*（`0` 〜 `99`）に当てはめる場合、**100 で割った余り**を使うと良さそうです。

| *生成範囲* の乱数値 | *100* で割った商 | **余り**（ *必要範囲* ） |
| --- | --- | --- |
| `0` 〜 `99` | `0` | `0` 〜 `99` |
| `100` 〜 `199` | `1` | `0` 〜 `99` |
| `200` 〜 `255` | `2` | `0` 〜 `55` |

これをコードに置き換えると、下記のようになります。

```ps1
# 乱数（0 〜 255）の生成
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$OneByte = New-Object byte[] 1
$rng.GetBytes($OneByte) # 20

# 欲しい範囲は 100 までなので、それで割った余りを出すと、必ず範囲内となる
$OneByte[0] % 100  # -> 20   ※ 答えは 0 余り 20 で、その余りが出力されている

# 乱数が 100 を超えるような場合も、上記の計算式で算出が可能
$rng.GetBytes($OneByte) # 199
$OneByte[0] % 100  # -> 99   ※ 答えは 1 余り 99 で、その余りが出力されている
```

さて、この方法は一見良さそうにも見えますが、**致命的な欠点**を抱えています。

それは、**生成範囲での乱数 `200` 〜 `255` までを加味すると、必要範囲での乱数の出方に偏りが生じること**です。

生成範囲での乱数 `200` 〜 `255` を `100` で割った時の剰余は、`0` 〜 `55` までとなります。つまり**生成範囲での乱数 `0` 〜 `255`までの中で、必要範囲での乱数 `0` 〜 `99` を作る**場合、下記のようになります。

- `0` 〜 `55` : 登場する機会が**3 回も**あり、確率として**高い**
- `56` 〜 `99` : 登場する機会が**2 回しか**なく、確率として**低い**

この偏りを除去するためには、**均等に登場する範囲を算出し、それ以上の値は切り捨てる**（≒ もう一度生成し直す）といった処理を設ける必要があります。

```ps1
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$OneByte = New-Object byte[] 1

# 0 〜 99 までの場合、200以上の乱数が生成された場合は、範囲内の乱数になるまでやり直す
do {
  $rng.GetBytes($OneByte)
} while ($OneByte[0] -ge 200)

$OneByte[0] % 100 # -> 72  ※ 確率的に均等な形で、乱数が生成されている
```

:::message

均等になる範囲の算出については、**生成範囲の個数 `256` を必要範囲の個数 `$Maximum` で割った商に `$Maximum` を掛ける**ことで算出できます。

```ps1
# 例：100（0 〜 99）で作成したい場合
$Maximum = 100
$acceptedUpperBound = [Math]::Floor(256 / $Maximum) * $Maximum
# Floor(256/100)*100 
# -> 2*100
# -> 200
```

:::

## サンプル：乱数生成器

:::message

今回作成した関数は、その性質上 256 までに限定されます。Maximum は排他的上限であり、指定できる最大範囲は 0 ～ 255 です。

それ以上の数で乱数を作りたい場合は、別の手段か この関数を用いて新たな仕組みを作る必要があります。

:::

```ps1
function Get-CryptoRandomNumber {
  <#
  .SYNOPSIS
    暗号学的に安全な乱数を、指定された範囲内で生成します。

  .DESCRIPTION
    System.Security.Cryptography.RandomNumberGenerator を使用して、0 以上 Maximum 未満の
    整数を生成します。

    1 バイトの乱数に単純な剰余演算を適用した際の確率の偏りを避けるため、256 を
    Maximum で分割できる範囲の上限を算出し、その範囲外の値を再生成する棄却法を
    使用します。

  .PARAMETER Maximum
    生成する乱数の排他的上限を指定します。指定できる値は 1 から 256 までです。
    戻り値は 0 以上 Maximum 未満になります。

  .OUTPUTS
    System.Int32
    0 以上 Maximum *未満* の整数を返します。

  .EXAMPLE
    $RandomNumber = Get-CryptoRandomNumber -Maximum 10

    0 から 9 までの範囲で乱数を生成し、$RandomNumber に格納します。

  .NOTES
    乱数生成に使用した RandomNumberGenerator オブジェクトは、処理の完了時に
    確実に破棄されます。
  #>
  param (
    [Parameter(Mandatory = $true)]
    [ValidateRange(1, 256)]
    [int] $Maximum
  )

  # 0 - 255 までの範囲で必要な乱数範囲を生成する際、確率に偏りが生じない範囲を算出
  $acceptedUpperBound = [Math]::Floor(256 / $Maximum) * $Maximum

  # 乱数を生成する準備
  $rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
  $buffer = New-Object byte[] 1

  try {
    # 偏りが生じない範囲内の値が出るまで、乱数を生成する
    do {
      $rng.GetBytes($buffer)
    } while ($buffer[0] -ge $acceptedUpperBound)

    # 引数として渡された範囲値の剰余が、必要となる乱数
    return $buffer[0] % $Maximum
  }
  finally {
    # リソースを解放
    $rng.Dispose()
  }
}
```
