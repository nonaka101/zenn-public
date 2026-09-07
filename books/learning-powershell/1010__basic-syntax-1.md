---
title: "🐣 基本構文 1"
---

## 基本的な構文

コマンド（PowerShell では *コマンドレット* とも言ったりします）については後の項で詳しく説明しますが、ここでは説明の都合上 基本的な構文のみ説明します。

下記が、基本的な形の命令文です。

```ps1
# 基本的なコマンドレットの形式
# （コマンドレットや使い方によって、パラメータ等は省略する場合もある）
# CmdletName -Parameter Value -SwitchParameter

# 例：現在日時を取得
Get-Date  # -> 2026年7月4日 14:54:32

# 例：文字列をコンソールに出力
Write-Output "Hello, world."    #-> Hello, world.

# 例：フォルダの作成
New-Item -Path "C:\User\Temp\TestFolder" -ItemType Directory

# 例：フォルダの削除(再帰的に削除)
Remove-Item -Path "C:\User\Temp\TestFolder" -Recurse
```

## コメント

PowerShell では、`#` をコメント記号として解釈します。

```ps1
# シャープ記号以降は、コメントとして解釈されます。

Get-Date    # インラインでコメントを入れることもできます
```

複数行のコメントでは、`<# ... #>` を使うこともできます。

```ps1
<#
このカッコ内に書かれたものは、
全てコメントとして解釈されます。
#>
```

## 変数

下記のように、`$変数名` の形で 変数を作成することができます。

```ps1
$hoge = 100
$hoge   # -> 100

$hogehoge = "Hello, world"
$hogehoge   # -> Hello, world


# コマンドレットも変数に格納可能
$now = Get-Date
$now  # -> 2026年7月4日 14:55:18
```

PowerShell は上記のような *ユーザーが自由に設定できる変数* の他に、様々な変数が用意されています。

### 自動変数

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_automatic_variables

PowerShell ではコンソール上で特別な意味を持つ変数が定義されています。

この中で、PowerShell の状態情報を格納する変数を**自動変数**と言います。

| 自動変数名 | 説明 |
| :--- | :--- |
| `$_` | パイプラインオブジェクト |
| `$$` | 直前の行の最後のトークン |
| `$?` | 直前のコマンドの処理判定（`boolean`） |
| `$args` | 関数の引数 |

:::message

*状態を格納している* という性質上、その殆どは**読み取り専用**となっています。  
（例えば、`$$` や `$?` の中身を *直接* 設定する、といったことはできません）

:::

#### `$_`：パイプラインオブジェクト

`$_` は パイプラインで渡されたオブジェクトを格納しています。配列など複数の要素があるものに対し、ループ処理やフィルタリングで取り出す際に よく用いられます。

:::message

イメージとしては、他言語でいう `foreach` 内での *ループ変数* や、*ラムダ引数* が近いでしょうか。

:::

```ps1
$array = @(1, 2, 3, 4, 5) # 配列で 1 〜 5 までを用意

# 配列要素をループで取り出し、`$_` を使って文字列を出力
$array | ForEach-Object {
  "Now Number is $_"
}
<#
Now Number is 1
Now Number is 2
Now Number is 3
Now Number is 4
Now Number is 5
#>
```

#### `$$`：直前に入力した行の最後のトークン

`$$` には、直前に入力した行の最後のトークンが格納されます。

:::message

コマンドの実行結果と誤認しがちですが、下記コードでわかるように厳密には異なります。

:::

```ps1
# 行の最後のトークンは "Hello"
Write-Output "Hello"

$$  # -> Hello

# 行の最後のトークンは "World"
Write-Output "Hello" "World"  # -> Hello World

$$  # -> World

# コマンドの出力は 3
1 + 2 # -> 3

# 直前の行の最後のトークンは 2
$$  # -> 2
```

#### `$?`：直前のコマンド処理の判定

`$?` は 直前のコマンド処理の判定を格納しています。要は「適正に処理が終わったか？」を *真偽値* の形で管理しています。

```ps1
Write-Output "Hello, world."    #-> Hello, world.
Write-Output $? # -> True

# エラーを発生させる
Write-Error "This is an error message."
<#
Write-Error "This is an error message." : This is an error message.
  + CategoryInfo          : NotSpecified: (:) [Write-Error], WriteErrorException
  + FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException
#>
Write-Output $? # -> False
```

:::message

下記のような エラーが発生した場合にも、`$?` は `False` を返します。

```ps1
throw "This is error message."   # <- 意図的にエラーを発生
<#
This is error message.
発生場所 行:10 文字:1
+ throw "This is error message."   # <- 意図的にエラーを発生
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  + CategoryInfo          : OperationStopped: (This is an error.:String) [], RuntimeException
  + FullyQualifiedErrorId : This is an error.
#>
```

:::

#### `$args`：関数の引数

`$args` は主に関数で用いられ、関数に渡される（宣言されていない）パラメータの値を格納しています。

:::message

スクリプトブロックでも使え、関数やスクリプトブロックに**可変長の引数を渡せる**という特徴があります。

:::

```ps1
# 引数をそのまま出力する関数を用意
function Print-Args {
  "全ての引数 : $args"
}

# 複数の引数を関数に渡してみる
Print-Args "Apple" "Orange" "Grape" # -> 全ての引数 : Apple Orange Grape
```

パラメータ付きの場合や配列やオブジェクトを渡しても、未宣言であるものは `$ars` に格納されます。

```ps1
function Test-Sample {$args}

Test-Sample `
  -ParamA 0 `
  -ParamB "Hello World" `
  -ParamC [PSCustomObject]@{Name = "Hoge"} `
  -ParamD $false `
  -ParamE (ConvertTo-SecureString `
    -String "Passw0rd" `
    -AsPlainText `
    -Force `
  ) `
  -ParamF @(
    "Apple",
    "Orange",
    "Grape"
  )
# 関数でパラメーターを宣言していないため、-ParamA なども含め、入力した各トークンが $args に格納
<#
-ParamA
0
-ParamB
Hello World
-ParamC
[PSCustomObject]@
Name = "Hoge"
-ParamD
False
-ParamE
System.Security.SecureString
-ParamF
Apple
Orange
Grape
#>
```

### ユーザー設定変数（基本設定変数）

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_preference_variables

[自動変数](#自動変数)は PowerShell 上で事前に定義された特別な意味を持つ変数の内、*PowerShell の状態情報を格納しているもの（≒ ユーザー側で意図的に設定を変えることができないもの）と説明* しました。

ここで紹介する **ユーザー設定変数**（基本設定変数）も、PowerShell で事前定義された特別な変数です。自動変数と異なるのは、**設定値はユーザー側で変更できる**点です。

ユーザー設定変数の例として、下表があります。

| ユーザー設定変数名 | 説明 |
| :--- | :--- |
| `$OutputEncoding` | 出力の文字エンコード（例：`UTF-8`） |
| `$ErrorActionPreference` | （重大でない）エラーが生じた際の対応手段（例：処理を継続、中断） |
| `$DebugPreference` | デバッグ系の対応手段 |
| `$ConfirmPreference` | 確認が必要とされるようなコマンドでの挙動（例：高い重要度の場合に確認する、確認不要） |

## 標準入力

例えばターミナル上でユーザーに入力を求めたい場合、例えば `Read-Host` コマンドレットを用いて入力を促すことができます。

```ps1
$userInput = Read-Host "何か入力してください"   # <- hoge
Write-Host "入力文字は $userInput です"  # -> 入力文字は hoge です
```

## 文字列を扱うクォート記号

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_quoting_rules

主に文字列を扱うための記号として、シングルクォート（`'`）とダブルクォート（`"`）があります。

この使い分けですが、まず 1 つには（他言語でも見られるように）文字列中にクォート文字を使いたい場合に使用します。

```ps1
# 文字列中にクォート文字を使いたい場合、別のクォート文字で括る
$hoge = "He said 'hello' to me."
```

そして ダブルクォートの場合、中の値が（変数や制御文字などの）評価対象として扱われるという特徴があります。

```ps1
$name = "Hoge"

# シングルクォート：文字列をそのまま扱う
'$name'  # -> $name

# ダブルクォート：中身が評価され、変数が展開される
"$name"  # -> Hoge
```

改行を表す文字列（バッククォート + `n`）も、評価対象になるかで異なってきます。

```ps1
# シングルクォート：文字列をそのまま扱う
Write-Host 'ここで→`n←改行'
<#
ここで→`n←改行
#>

# ダブルクォート：改行文字として評価される
Write-Host "ここで→`n←改行"
<#
ここで→
←改行
#>
```

### Subexpression 演算子 `$(...)`

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_operators#subexpression-operator--

例えば下記のように変数名と文字列が繋がってしまうと、PowerShell は繋がったものを変数名として解釈してしまいます。こうした場合に、サブ式（ *SubExpression* ）である `$( )` を使うことで、**正確な変数展開を明示すること**ができます。

```ps1
$name = 'Hoge'

Write-Host "$name_is_user"  # -> （なし）
Write-Host "$($name)_is_user" # -> Hoge_is_user
```

:::message

[公式によれば](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_quoting_rules#double-quoted-strings)、単純な変数展開であれば `${}` を使うほうが適切とされています。

> 変数名を文字列内の後続の文字から分離するには、中かっこ（`{}`）で囲みます。 これは、変数名の後にコロン（`:`）が続く場合に特に重要です。
>
> たとえば、`"$HOME: where the heart is."` はエラーをスローしますが、`"${HOME}: where the heart is."` は意図したとおりに動作します。

:::

他にもサブ式を使うメリットとして、変数名だけでなく**式そのもの**を書き、その計算結果を埋め込むこともできます。

```ps1
# 例：文字列を大文字に
$name = 'Hoge'
Write-Host "User name is $($name.ToUpper())"  # -> User name is HOGE

# 例：指定フォルダ内のファイル群の数を算出
$path = "C:\temp\"
$files = Get-ChildItem $path
Write-Host "Folder '$($path)' has $($files.Count) files." # -> Folder 'C:\temp\' has 6 files.
```

## スクリプトブロック

スクリプトブロックは 複数（一連）の命令を扱うためのもので、中括弧（`{ }`）を使うことで表現できます。

```ps1
# スクリプトブロック：実行しても、中身を出力するだけ
{
  Get-Date
  Write-Host "Hello World"
}
<#
  Get-Date
  Write-Host "Hello World"
#>
```

ブロック前に `&` を置くことで、実行することができます。

```ps1
& {
  Get-Date
  Write-Host "Hello World"
}
<#
2026年7月4日 15:03:03
Hello World
#>
```

## 改行について

### 複数の文の場合

基本的に PowerShell では、改行文字で文の区切り（終わり）を識別できます。

```ps1
# 改行で別の文と識別できる
$hoge = 100
$fuga = "Hello, world."
```

:::message

C# や JavaScript などで見られるセミコロン（`;`）は、PowerShell では不要となります。

```ps1
# セミコロンを使って明示はできる（ただし行末では通常不要）
$hoge = 100;
$fuga = "Hello, world.";
```

ただし、**1 行に複数の文を含めたい場合**には、セミコロンを使うことでそれが行なえます。

```ps1
# 使用ケースとしては、例えば下記が挙げられる
# - 1 行にまとめた形で記述したほうが わかりやすい
# - 1 行で管理しておきたい
$TAX_A = 1.08; $TAX_B = 1.10;
```

:::

### 1文での場合

PowerShell では基本的に、1 つの文を 1 行で書いていくことになります。

一方で、PowerShell では下記のように非常に長い文になるケースもあります。

```ps1
# 例：様々なパラメータを含めたコマンド
Test-Sample -ParamA 0 -ParamB "Hello World" -ParamC  [PSCustomObject]@{Name = "Hoge"} -ParamD $false -ParamE (ConvertTo-SecureString -String "Passw0rd" -AsPlainText -Force) -ParamF @("Apple", "Orange", "Grape")
```

こうした場合、行末にバッククォートを使うことで 1 文を複数の行に分けることができます。

```ps1
# 改行することで見やすく
Test-Sample `
  -ParamA 0 `
  -ParamB "Hello World" `
  -ParamC [PSCustomObject]@{Name = "Hoge"} `
  -ParamD $false `
  -ParamE (
    ConvertTo-SecureString `
      -String "Passw0rd" `
      -AsPlainText `
      -Force `
  ) `
  -ParamF @(
    "Apple",
    "Orange",
    "Grape"
  )
```

:::message

コンソール上で入力する場合は `Enter` キーで改行してしまうと 文の終了と見なされてしまいます。

バッククォートの後においては、`Enter` キーを押すことでコンソール上でも改行することができます。

また、`PSReadLine` では、`Shift` + `Enter` によって任意の位置で入力行を追加することもできます。

:::

ただし *バッククォートの後ろに空白があるとエラーになる* という問題があるので注意が必要です。

### オブジェクトの場合

後述で出てくるオブジェクトに関係する話で、パイプラインで渡す場合はそこで改行を挟む事ができます。

```ps1
# 複数の処理を、パイプラインで繋いで一連の処理としている
Get-Process | Where-Object CPU -gt 100 | Sort-Object CPU

# 上記の処理は、下記のように改行することができる
Get-Process |
  Where-Object CPU -gt 100 |
  Sort-Object CPU
<#
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   2783     153   212808     393272     183.09   7064  34 msedge
   7459     208   515152     618628     242.13  18952  34 explorer
   1339      59   485024     544784   1,036.58   5088  34 msedge
   1189      82   777440     869692   1,471.41  11584  34 msedge
#>
```

### その他

その他 配列や開始記号の直後などでも改行することができます。

```ps1
# 例：配列（両者は同じ意味となる）
$numbers1 = 1, 2, 3, 4
$numbers2 = 1,
  2,
  3,
  4

# 例：条件分岐（開始記号の後は、改行しても継続とみなされる）
if (
  $hoge -gt 0
) {
  Write-Host "hoge is greater than 0"
} else {
  Write-Host "hoge is not greater than 0"
}
```
