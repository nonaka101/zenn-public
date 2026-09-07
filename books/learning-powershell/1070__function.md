---
title: "🐣 関数"
---

## 関数とは

PowerShell では、よく使う一連の処理をまとめて **関数（Function）** として定義することができます。

他言語同様、関数化することで 同じコードを何度も書く必要がなくなり、スクリプト全体の可読性や保守性が向上します。

### 基本形

関数を定義するには、`function` キーワードに続けて関数名を書き、処理内容を `{}`（スクリプトブロック）の中に記述します。

```ps1
# 関数の定義
function Write-Greeting {
  Write-Host "Hello, world."
}

# 関数の呼び出し
Write-Greeting  # -> Hello, world.
```

:::message

PowerShell の関数名は、コマンドレットでの形式同様 **`動詞-名詞`**（例：`Get-UserInfo`、`New-CustomReport`）の形にすることが推奨されています。

[Microsoft Learn](https://learn.microsoft.com/ja-jp/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands)によると、`Get` や `New` といった（承認された）動詞と、単数形の名詞を組み合わせたものにするよう案内が出されています。

:::

### 関数の構造

関数は、本体となるコードの前に様々な要素が入る場合があります。基本的な構造は、下記のようになっています。

1. 関数レベルの属性リスト
2. `param` ステートメント
   - 各種パラメータの属性リスト
3. 関数本体
4. 必要に応じて、（`return` などで）処理結果を出力

:::message

実際には、関数によって `return` が ないなどの差異はあります。

:::

```ps1
function Test-Sample {

  # ① 関数レベルの属性リスト
  [CmdletBinding()]
  [OutputType([string])]

  # ② param ステートメント
  param(
    # パラメーターの属性リスト
    [Parameter(Mandatory)]
    [string]$Name
  )

  # ③ 関数本体
  $text = "Hello, $Name"

  # ④ 関数の返り値
  return $text
}
```

#### 関数レベルの属性リスト

##### `CmdletBinding`：コマンドレットとしての動作

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_functions_cmdletbindingattribute

`param` ブロックの前に `[CmdletBinding()]` を置くことで、その関数をコマンドレットのように動作させることができるようになります。

例えば、次のようなコマンドレットに近い機能を利用できるようになります。

- 共通パラメーター
- コマンドレットと同様のパラメーターバインド
- `$PSCmdlet` を介した高度な処理
- `SupportsShouldProcess` による `-WhatIf` や `-Confirm` への対応

:::message

共通パラメーターとは、`-Verbose` や `-ErrorAction` など、多くのコマンドレットや高度な関数で共通して使用できるパラメーターです。コマンドアドオンの画面では、画面下部に表示されます。

![コマンドアドオン画面](/images/books/learning-powershell/command-addon-01.png)

なお パイプラインから入力を受け取る場合は、パラメータ側にも `ValueFromPileline` などの設定が必要になってきます。ただ `CmdletBinding` をつければコマンドレットのように動作するわけではないことは、注意してください。

:::

##### `OutputType`：関数が返すオブジェクトの種類を報告

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_functions_outputtypeattribute

`OutputType` を使用すると、その関数が出力する（予定の）オブジェクトの型を、メタデータとして指定できます。

:::message

扱いとしては「この型を返す**予定です**」という宣言メタデータであり、実際に異なる型を返してもエラーにはなりません。

:::

```ps1
function Send-Greeting
{
  [OutputType([string])]
  param ($Name)

  return "Hello, $Name"
}

# 関数の返り値の型を調べると、このメタデータの情報が返ってくる
(Get-Command Send-Greeting).OutputType
<#
Name               Type
----               ----
System.String      System.String
#>
```

#### `param` ステートメント

関数は外部からデータを受け取り、そのデータに基づいて処理を行うことができます。この受け取るデータを **パラメータ** と呼びます。

:::message

厳密には、パラメータと引数は同義ではありません。

関数側で定義するデータの受け取り口を**パラメータ**と呼び、関数を呼び出すときに渡す具体的な値を**引数**と呼びます。

:::

パラメータを定義するには、関数本体の先頭に `param` ブロックを記述します。  
（もし関数レベルの属性を指定する場合は、それらの属性に続けて記述します）

パラメータは、`[型情報]$パラメータ名` の形で定義できます。  
（型指定せず `$パラメータ名` と省略することもできます）

```ps1
function Write-GreetingToUser {
  param(
    [string]$Name
  )
  
  Write-Host "Hello, $Name !"
}

Write-GreetingToUser -Name "Hoge" # -> Hello, Hoge !


# 複数のパラメータを定義する場合は、コンマ（`,`）で区切ります
function Get-TotalPrice {
  param(
    [int]$Price,
    [int]$Amount
  )
  
  Write-Host "Total price is $($Price * $Amount) yen."
}

Get-TotalPrice -Price 150 -Amount 3 # -> Total price is 450 yen.
```

なお、このブロック内で定義するパラメータ自体にも様々な属性リストをつけて詳細を決めることが可能です。

```ps1
# パラメータ指定が必須で、$null または空文字列を許可しない文字列型の Name パラメータ
param(
  [Parameter(Mandatory)]
  [ValidateNotNullOrEmpty()]
  [string]$Name
)
```

:::message

`param` ブロックを用いず、関数名の後ろに引数を定義する方法もあります。

```ps1
# どちらの形式でもパラメーターを定義できる

function Test-Sample1 {
  param($Name)
}

function Test-Sample2($Name) {
}
```

:::

##### `Mandatory`：パラメータ指定の必須化

パラメータの前に `[Parameter(Mandatory = $True)]` を置くことで、そのパラメータの *指定* を（原則）必須にすることができます。

```ps1
function Write-GreetingToUser {
  param(
    # `[Parameter(Mandatory)]` でも可
    [Parameter(Mandatory = $True)]
    [string]$Name
  )
  
  Write-Host "Hello, $Name !"
}

# パラメータを指定しなかった場合、入力を要求される
Write-GreetingToUser
<#
コマンド パイプライン位置 1 のコマンドレット Write-GreetingToUser
次のパラメーターに値を指定してください:
Name: hoge
Hello, hoge !
#>
```

##### `ValidateNotNullOrEmpty`：空値等を拒否

パラメータの前に `[ValidateNotNullOrEmpty()]` を置くことで、そのパラメータの *入力* を必須にすることができます。

:::message

厳密に言えば、`$null` や空文字、（配列などのコレクションを指定した場合は）空のコレクションでないことを検証しています。

逆に、空文字列を有効な値として許容する場合は `[AllowEmptyString()]` を使います。

:::

```ps1
function Write-GreetingToUser {
  param(
    [ValidateNotNullOrEmpty()]
    [string]$Name
  )
  
  Write-Host "Hello, $Name !"
}

# 下記のような入力は、拒否される
Write-GreetingToUser -Name ''   # -> エラー
Write-GreetingToUser -Name $null  # -> エラー
```

:::message

`Mandatory`, `ValidateNotNullOrEmpty` の両方を組み合わせることで、**パラメータの指定を必須にした上で、さらに `$null` や空文字列を拒否する**ことができます。

```ps1
function Write-GreetingToUser {
  param(
    [Parameter(Mandatory = $True)]
    [ValidateNotNullOrEmpty()]
    [string]$Name
  )
  
  Write-Host "Hello, $Name !"
}
```

:::

### コード本体

関数本体の属性リストや `param` ステートメントの後に、関数本体となるコードが記述されます。

:::message

応用分野として、[入力処理方法と呼ばれる応用メソッド](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_functions_advanced_methods)が存在します。しかし本書は初心者向けのものであり、知らなくても基本 問題はないので、説明は省略いたします。

```ps1
function Test-ScriptCmdlet {
  [CmdletBinding()]
  param($Parameter1)

  # 必要に応じて、名前付きブロックを使用できる
  #（利用例：パイプラインから受け取った複数の入力を処理する場合）
  begin {}
  process {}
  end {}
}
```

:::

#### `return`：返り値（出力）を流す

関数の中で計算した結果などを、呼び出し元に返したい場合は `return` を使用するか、単に結果を出力ストリームに流します。

初心者の内は、「返り値には `return` を指定するもの」という認識でいいでしょう。  
（`return` には他にも、現在のスコープの処理を終了するという役割もあったりします）

実務のスクリプトでは、特定の情報を取得・加工する処理を関数化しておき、それをメインの処理で呼び出すという構成をとることが多くなります。

```ps1
function Calculate-Tax {
  param(
    [int]$Price
  )
  
  $taxIncluded = [math]::Floor($Price * 1.1)
  return $taxIncluded
}

# 関数の実行結果を変数に格納する
$result = Calculate-Tax -Price 1000
Write-Host "税込み価格は $result 円です。"  # -> 税込み価格は 1100 円です。
```

:::message

`return` を設けない場合 例えば下記のように、計算の結果は関数の内部に留まってしまう場合があります。

```ps1
Function Check-Folder([String]$path){
  $exists = Test-Path $path
}
Check-Folder 'C:\temp'  # -> 
# $exists を出力していないため、呼び出し元には何も出力されない
```

ただ PowerShellでは、`return` に指定した値だけでなく、関数内で成功出力ストリームに出力されたほかのオブジェクトも、関数の実行結果に含むことができます。

:::
