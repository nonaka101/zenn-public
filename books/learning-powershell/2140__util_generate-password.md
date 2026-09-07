---
title: "🐥 英数字のみのパスワード生成"
---

## 本処理の目的

何かしらのサービスにユーザーを登録する際、パスワードを必要とする場合が出てきます。  
（例：ActiveDirectory　にユーザーを登録する）

本項では、以前に紹介した暗号強度のある乱数生成関数を組み合わせ、パスワード生成関数を作成していきます。

### 本項の代替案

https://learn.microsoft.com/ja-jp/dotnet/api/system.web.security.membership.generatepassword

ちなみに、記号を含んでいい場合は `Menbership` クラスの静的メソッド `GeneratePassword` が使えます。

```ps1
# 記号の使用は最小 1 文字以上で、12文字のパスワード生成
[System.Web.Security.Membership]::GeneratePassword(12,1)  # -> qK%6g$Le>oq6
```

本項では、**記号を含まない形での生成もできるようしたい**ので、別の手法での実装を目指します。

## サンプル：英数字のみのパスワード生成

:::message

この関数では、以前紹介した乱数生成関数を組み合わせて使います。

```ps1
# 指定範囲での乱数を生成する関数
function Get-CryptoRandomNumber {
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

:::

```ps1
function New-Password {
  <#
  .SYNOPSIS
    指定された長さと文字種の条件に基づいて、ランダムなパスワードを生成します。

  .DESCRIPTION
    英大文字、英小文字、数字を使用して、指定された長さのパスワードを生成します。
    IncludeSymbol スイッチを指定した場合は、使用可能な ASCII 記号の一部も文字種に
    追加します。

    有効な各文字種から最低 1 文字ずつ選出した後、残りの文字をすべての有効な
    文字種から選出します。生成した文字配列は Fisher-Yates 法でランダムに
    並べ替えられます。

    文字の選出と並べ替えには Get-CryptoRandomNumber を使用します。Length が
    有効な文字種の数より小さい場合は、終了エラーを発生させます。

  .PARAMETER Length
    生成するパスワードの文字数を指定します。1 以上の整数を指定できますが、
    実際には有効な文字種の数以上である必要があります。

    IncludeSymbol を指定しない場合は 3 以上、指定する場合は 4 以上の値が必要です。

  .PARAMETER IncludeSymbol
    パスワードに記号を含めることを指定します。このスイッチを指定すると、
    `!#$%&*+-=?@_` のいずれかが最低 1 文字含まれます。

  .OUTPUTS
    System.String
    指定された長さと文字種の条件を満たすパスワード文字列を返します。

  .EXAMPLE
    $Password = New-Password -Length 12

    英大文字、英小文字、数字をそれぞれ最低 1 文字含む、12 文字のパスワードを
    生成して $Password に格納します。

  .EXAMPLE
    $Password = New-Password -Length 16 -IncludeSymbol

    英大文字、英小文字、数字、記号をそれぞれ最低 1 文字含む、16 文字の
    パスワードを生成して $Password に格納します。

  .NOTES
    この関数を使用するには、Get-CryptoRandomNumber 関数があらかじめ定義されて
    いる必要があります。
  #>
  [CmdletBinding()]
  [OutputType([string])]
  param (
    [Parameter(Mandatory = $true, Position = 0)]
    [ValidateRange(1, [int]::MaxValue)]
    [int] $Length,

    [Parameter()]
    [switch] $IncludeSymbol
  )

  # 使用する文字種
  $upperCase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
  $lowerCase = 'abcdefghijklmnopqrstuvwxyz'
  $digits    = '0123456789'
  $symbols   = '!#$%&*+-=?@_' # 一般的なパスワード要件を鑑み、ASCII 記号の一部のみに制限

  # 使用する文字種の選別（記号を含めるか 含めないか）
  $requiredPools = @($upperCase, $lowerCase, $digits)
  if ($IncludeSymbol) {
    $requiredPools += $symbols
  }

  # 不適正な要件をはじく：使用文字種以下の長さ
  if ($Length -lt $requiredPools.Count) {
    throw "Length は、有効な文字種の数 ($($requiredPools.Count)) 以上にしてください。"
  }

  # 生成されたパスワード文字列を格納する Char 配列
  $passwordCharacters = New-Object char[] $Length

  try {
    # Char 配列のインデックス管理
    $position = 0

    # 有効な各文字種から、最低1文字を選出
    foreach ($pool in $requiredPools) {
      $index = Get-CryptoRandomNumber -Maximum $pool.Length
      $passwordCharacters[$position] = $pool[$index]
      $position++
    }

    # 使用する文字種要件は満たしたので、残りを有効な全文字種から選出して埋める
    $allCharacters = -join $requiredPools
    while ($position -lt $Length) {
      $index = Get-CryptoRandomNumber -Maximum $allCharacters.Length
      $passwordCharacters[$position] = $allCharacters[$index]
      $position++
    }

    # 最初のほうが必須文字種で固定されているので、Fisher-Yates 法でランダムに並べ替える
    for ($i = $passwordCharacters.Length - 1; $i -gt 0; $i--) {
      $j = Get-CryptoRandomNumber -Maximum ($i + 1)
      $temporary = $passwordCharacters[$i]
      $passwordCharacters[$i] = $passwordCharacters[$j]
      $passwordCharacters[$j] = $temporary
    }

    # 配列を文字列として繋ぎ合わせて返す
    return -join $passwordCharacters
  }
  finally {
    # リソースの解放
    if ($null -ne $rng) {
      $rng.Dispose()
    }
  }
}
```
