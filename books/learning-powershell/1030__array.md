---
title: "🐣 配列と連想配列"
---

## 配列（基礎）

https://learn.microsoft.com/ja-jp/powershell/scripting/learn/deep-dives/everything-about-arrays

PowerShell では、複数の値をまとめて扱うために **配列** を使用します。

### 配列の作成

要素をコンマ（`,`）で区切る、`@()` で囲むなどで 配列を作成できます。

```ps1
# コンマで区切る方法
$array1 = "Apple", "Banana", "Cherry"

# `@()` で囲む方法（空配列の生成にも使用）
$array2 = @(10, 20, 30)
$emptyArray = @()
```

:::message

ネストさせることで、配列を格納した配列（ジャグ配列）を作ることもできます。

```ps1
# 二次元配列
$array = @(@(1, 2, 3), @(4, 5, 6))
<#
$array
├─ [0] -> @(1, 2, 3)
│   ├─ [0] -> 1
│   ├─ [1] -> 2
│   └─ [2] -> 3
└─ [1] -> @(4, 5, 6)
    ├─ [0] -> 4
    ├─ [1] -> 5
    └─ [2] -> 6
#>

$array  # -> 1 2 3 4 5 6
# ※フラット化ではなく、単に各子配列の内容を列挙

$array[0] # -> 1 2 3
$array[0][0]  # -> 1

# それぞれの要素数は `Count` プロパティで確認可能
$array.Count  # -> 2
$array[0].Count # -> 3
```

:::

:::message

連番の配列を作りたい場合は、`..` 演算子が便利です。

```ps1
$numbers = 1..5
$numbers
<#
1
2
3
4
5
#>
```

ちなみに、.NET での多次元配列も作成可能です。

```ps1
$array = [int[,]]::new(2,3)
```

:::

### 要素へのアクセスと操作

配列の要素には、インデックス（`0` 開始）を指定してアクセスします。

```ps1
$fruits = @("Apple", "Banana", "Cherry")

# 最初の要素を取得
$fruits[0] # -> Apple
```

最後の要素を取り出す場合は、いくつかの手段があります。

```ps1
$fruits = @("Apple", "Banana", "Cherry")

# 配列の要素数は、`Count` プロパティから取得が可能
$fruits.Count # -> 3

# 上記を使って、最後の要素を取り出す（※ゼロ開始なので 1 つずらす）
$fruits[$fruits.Count - 1] # -> Cherry

# ※ インデックス -1 を指定すると、末尾を表します
$fruits[-1] # -> Cherry
```

要素を追加する場合は `+=` 演算子を使用します。

```ps1
$fruits += "Orange"
$fruits
<#
Apple
Banana
Cherry
Orange
#>
```

## ハッシュテーブル（連想配列）

https://learn.microsoft.com/ja-jp/powershell/scripting/learn/deep-dives/everything-about-hashtable

配列が「順番」でデータを管理するのに対し、**ハッシュテーブル（連想配列）** は「キー（名前）」と「値」のペアでデータを管理します。

PowerShell において役立つのは、例えば下記のような場面です。

- 設定情報やキー・値の対応表の管理
- オブジェクトのプロパティをカスタマイズ

### 連想配列の作成

連想配列は `キー = 値` のセットを `@{ }` で囲むことで作成します。

```ps1
# ユーザー情報を連想配列で作成
$user = @{
  Name = "Taro Yamada"
  Department = "IT"
  Age = 30
}
```

### 要素へのアクセスと追加・変更

キーを指定することで、対応する値を取得または更新できます。

指定方法は `$変数名["キー"]` または `$変数名.キー` の形です。

```ps1
# 値の取得
$user["Name"] # -> Taro Yamada
$user.Department # -> IT

# 値の更新
$user.Age = 31

# 新しいキーと値の追加
$user.Role = "Admin"

$user
<#
Name                           Value
----                           -----
Name                           Taro Yamada
Department                     IT
Role                           Admin
Age                            31
#>
```

### 注意：キー名の扱いについて

ハッシュテーブルのキー名に、ハイフンが入っているとエラーになります。これはハイフンが評価されてしまっているためです。

```ps1
$user = @{
  Givenname = "Taro"
  Surname = "Tanaka"
  # ↓ ここで構文エラーになる
  Display-Name = "Tanaka Taro"
}
<#
発生場所 行:5 文字:10
+   Display-Name = "Tanaka Taro"
+          ~
ハッシュ リテラルのキーの後に '=' 演算子が存在しません。
発生場所 行:5 文字:10
+   Display-Name = "Tanaka Taro"
+          ~
ハッシュ リテラルが不完全です。
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : MissingEqualsInHashLiteral
#>
```

これを回避するには、（ハイフン等使う場合は）文字列として識別できるよう**クォート文字で括る**ようにします。

:::message

この対処法は、今から説明する **スプラッティング** を使う際に 覚えておく必要があります。

自分で作成し運用するだけのハッシュテーブルであれば、キー名にハイフンを使わなければ済むだけです。

:::

```ps1
$user = @{
  Givenname = "Taro"
  Surname = "Tanaka"
  'Display-Name' = "Tanaka Taro"
}
$user
<#
Name                           Value
----                           -----
Display-Name                   Tanaka Taro
Givenname                      Taro
Surname                        Tanaka
#>
```

### スプラッティング

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_splatting

連想配列の非常に便利な使い方として、**スプラッティング** というテクニックがあります。

これは、コマンドの引数（パラメータ）が長くなってしまう場合に、連想配列を使ってスッキリと記述する方法です。

#### 例：可読性が低いコード

例えば下記はサンプル用のコードになりますが、非常に長く読みづらい状態です。こうした状況は実務上でも起こり得ます。

```ps1
# 可読性が低い（≒ 人が読みづらい）
Test-Sample -ParamA 0 -ParamB "Hello World" -ParamC $object -ParamD $false -ParamE (ConvertTo-SecureString -String $plainPassword -AsPlainText -Force) -ParamF @("Apple", "Orange", "Grape")
```

:::message

実務では、扱うパラメータが多くなる ActiveDirectory や Exchange 関係で見られます。

下記は ActiveDirectory 管理センターを使って新規ユーザーを作った場合のコードを想定したものです。

```ps1
New-ADUser -Name hoge001 -SamAccountName hoge001 -UserPrincipalName 'hoge@fuga.jp' -Server 'dc01.fuga.jp' -Path 'OU=Users,DC=fuga,DC=jp' -DisplayName '田中　太郎' -Surname '田中' -GivenName '太郎' -EmailAddress 'hoge@fuga.jp' -Title Member -AccountType User -Enabled $True
```

:::

これを解決する方法は いくつか存在します。1つは、改行するための制御文字（バッククォート）を使って行を分割する方法です。

```ps1
# 改行することで見やすく
Test-Sample `
  -ParamA 0 `
  -ParamB "Hello World" `
  -ParamC $object `
  -ParamD $false `
  -ParamE (ConvertTo-SecureString -String $plainPassword -AsPlainText -Force) `
  -ParamF @(
    "Apple",
    "Orange",
    "Grape"
  )
```

:::message

ただ、上記の方法だとバッククォートの後ろは**必ず改行文字でなければなりません**。

そのため空白スペースが入っていたり、インラインコメントを挿入できなかったりと、扱いには若干の注意が必要となります。

:::

#### スプラッティングを使い、コードを見やすくする

前述した問題を解決する別の方法として、**スプラッティング**（ *Splatting* ）があります。

これはハッシュテーブルとして生成した変数をコマンドレットのパラメータとして渡すことができる記述方法です。

パラメータを渡す際は、**変数の `$` 記号を `@` に変えて**コマンドに渡します。

```ps1
# 関数：渡された引数を出力するだけ
function Test-Sample {
  param(
    $ParamA,
    $ParamB,
    $ParamC,
    $ParamD,
    $ParamE,
    $ParamF
  )

  $PSBoundParameters
}

# ハッシュテーブルの形式で引数を整理する
$HashArguments = @{
  ParamA = 0  # 先ほどと違い、インラインコメントが使用できる
  ParamB = "Hello World"
  ParamC = [PSCustomObject]@{Name = "Hoge"}
  ParamD = $false
  # 行単位でもコメントは可能
  ParamE = (ConvertTo-SecureString -String "Passw0rd" -AsPlainText -Force)
  ParamF = @(
    "Apple",
    "Orange",
    "Grape"
  )
}

# スプラッティングを使ってコマンド文をスッキリさせる
#（パラメータとして渡す場合は、`$` でなく `@`）
Test-Sample @HashArguments
<#
Key    Value
---    -----
ParamF {Apple, Orange, Grape}
ParamA 0
ParamE System.Security.SecureString
ParamC @{Name=Hoge}
ParamD False
ParamB Hello World
#>
```

スプラッティングで渡す場合には、バッククォートで改行した場合と異なり 前後のスペース等を気にする必要はありません。

可読性を考えた際のテクニックとして、覚えておいても損はないでしょう。
