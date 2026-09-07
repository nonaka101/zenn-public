---
title: "☕ Coffee Break : 2"
---

## `ToLower()` と `ToLowerInvariant()`

https://learn.microsoft.com/ja-jp/dotnet/api/system.string.tolower

https://learn.microsoft.com/ja-jp/dotnet/api/system.string.tolowerinvariant

文字列を扱う際、「小文字（あるいは大文字）に統一したい」といった場面が出てきます。その際に利用できるメソッドには、`ToLower()` 及び `ToLowerInvariant()` の 2 種があります。

この両者の違いは、**現在のカルチャ（文化圏・言語ルール）に依存するか**という部分になります。

結論から先に述べますと、**本書の事情（日本文化圏）であれば どちらを使っても差異が生じることはありません**。本書でもこのメソッドを使う場面はありますが、そこでは言語に依存しない `ToLowerInvariant()` で統一しています。

### `ToLower()` を使って差異が生じるケースについて

例として `FILE` という文字列を小文字にする際、英語圏であれば通常 `file` と出力されます。

一方で例えばトルコ語（`tr-TR`）の場合、`i` にはドットのあるなしで 2 種類存在しています。

```txt
I → ı   （ドットなしの i ）
İ → i   （ドット付きの i ）
```

ユーザーに向けた表示用である場合には、こうした言語ごとのルールに応じたものが望ましいです。

しかし一方で システム内部用（例：ID比較、ファイル名の正規化 など）においては、こうした言語間でのルールは想定外の動きをおこす危険性が生じてきます。

## 可変長のオブジェクトを返す場合の注意事項

PowerShell ではコマンドレットや関数を使用することが多く、これらの中で処理した内容を返り値として受け取り、のちの処理に使うといったことはよく行われています。

この返り値が可変長になり、**配列を想定している場合**には注意が必要です。

PowerShell において可変長のオブジェクトを帰す場合、**0 個の場合には `$null` を、1 個の場合はオブジェクトを、そして 2 個以上の場合に配列を返します**。

```ps1
function Get-UserName {
  param (
    [int]$Count
  )

  $users = foreach ($i in 1..$Count) {
    "User$i"
  }

  return $users
}

# 0件
$result = Get-UserName -Count 0
$result.GetType().Name  # エラー ($null)

# 1件
$result = Get-UserName -Count 1
$result.GetType().Name  # String

# 2件
$result = Get-UserName -Count 2
$result.GetType().Name  # Object[]
```

この仕様は、**配列（あるいはオブジェクト）を想定してコードを作成していると、思わぬ挙動をすることがあります**。

特に `Set-StrictMode` を使って厳格に動作させようとした際に問題になりやすかったりします。

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/set-strictmode

### 挙動への対策

対策としては、（配列を想定する場合は）配列化をコードの中に含めることです。

まず、関数内の返り値を配列化しておく方法が考えられます。

```ps1
function Get-UserName {
  param (
    [int]$Count
  )

  $users = foreach ($i in 1..$Count) {
    "User$i"
  }

  return @($users)  # 配列化した上で返す
}
```

それが難しければ、変数として受け取る際に配列化する方法でも有効です。

```ps1
$result = @(Get-UserName -Count 1)  # 配列化した上で変数に格納

$result.GetType().Name   # Object[]
$result.Count            # 1
$result[0].Id            # 1
```

## 返り値を捨てたい場合

メソッドには**返り値を持つ**ものもあれば、**何も返さない**メソッド（`void` メソッド）も存在します。

コードの中で メソッドを使う場面は数多くありますが、中には「**返り値が不要なことは明確で、パイプラインに流したくない**」といった場面も出てくるかもしてません。

そこで使える手法は、大きく下記の 3 つがあります。

- `Out-Null` を使う
- `[void]` を使う
- `$null` を使う

### `[void]`

まずは `[void]` を使った方法です。これは、メソッド呼び出し結果を `[void]` 型で変換することで捨てています。

```ps1
# 意図としては「メソッドの中の処理で完結していて、そこから返る結果は使わないよ」
[void]$obj.Method()
```

特徴としては、下記となります。

- パイプラインを経由しない
- オーバーヘッドが小さい
- 「返り値は不要」という意図が明確

### `Out-Null`

2つ目の方法は、`Out-Null` を使ったものです。これはメソッドの結果をパイプラインに流し、その上で `Out-Null` で捨てています。

```ps1
# 意図としては「通常の処理ではパイプラインに渡すけど、ココではしないよ」
$obj.Method() | Out-Null
```

一旦パイプラインを経由する以上、やや余計な処理が入ることにはなります。

一方で、コマンドレットの出力抑止という意味では自然な使い方であり、「パイプライン出力を捨てる」といった意図で行うことが多いです。

### `$null`

最後は `$null` を使った方法です。

これは上記 `[void]` とほぼ同じですが、向こうが「返り値はいらない」という意図に対し、こちらは「結果を捨てる」という意図で用いることが多いです。

```ps1
$null = $obj.Method()
```

## `[]` を使った文について

PowerShell コードを見ていると、何度も `[]` を使った文というのが登場します。これは基本的に .NET クラスの型を参照している場合がほとんどです。

ここでは `[]` を使った文について、基本的な 4 つのパターンを見ていきます。

### 型変換を行う

`[SomeType]$value` の形で、`$value` を特定の型に変換します。

下記コードでは、文字列 `"123"` を `Int32` に変換しています。

```ps1
# 文字列として結合される
"123" + 4 # -> 1234

# 数値に変換され、加算される
[int]"123" + 4  # -> 127
```

### 型自体を、メソッドの引数として渡す

メソッドの引数に、型自体を `[SomeType]` の形で引き渡します。

ここでは、`[DateTime]` という型オブジェクトを引数として渡しています。  
（結果は `DateTime` の既定値 `0001/01/01 00:00:00` です）

```ps1
# ここで見るのは、CreateInstance メソッドに `[DateTime]` という型を渡している所
[Activator]::CreateInstance([DateTime])  # -> 0001年1月1日 0:00:00
```

### Enum の値を参照する

`[SomeEnum]::Member` の形で、列挙体（Enum）のメンバーを参照します。

下記コードでは、`FileAttributes` 列挙型の `ReadOnly` メンバーを参照しています。

```ps1
[System.IO.FileAttributes]::ReadOnly
```

### Static メソッドを呼び出す

`[SomeType]::Method()`の形で、（インスタンス化はせず）.NET クラスの静的メソッドを呼び出します。

下記コードでは、`Guid` クラスの静的メソッド `NewGuid()` を呼び出しています。

```ps1
[Guid]::NewGuid()
<#
Guid
----
1856e355-8a25-47f9-974b-3280b5826291
#>
```

## エイリアスについて

エイリアス（ *Alias* ）とは「別名」「通称」などを意味する言葉であり、IT 用語としては **とある機能やデータに対し 元の名前とは別の名前を割り当てる**ことを意味します。こうすることによって、識別しやすくしたり、操作の利便性を上げたりするわけです。

```ps1
# コマンドプロンプトで使用してきた `dir` は、PowerShellコマンドレット Get-ChildItem のエイリアス
Get-Alias dir
<#
CommandType     Name
-----------     ----
Alias           dir -> Get-ChildItem
#>
```

:::message

エイリアスに対し 新規にエイリアスを設定した場合、そのエイリアスはベースとなる方に置き換わります。

```ps1
Get-Alias dir
<#
CommandType     Name
-----------     ----
Alias           dir -> Get-ChildItem
#>

# 新たに blar というエイリアスを、dir（エイリアス）に設定
Set-Alias blar dir
Get-Alias blar  # 結果としては、dir でなく元である Get-ChildItem を参照
<#
CommandType     Name
-----------     ----
Alias           blar -> Get-ChildItem
#>
```

:::

例えば `bash` などでは、下記のコマンドを使ったりするのですが、これらは実は PowerShell 上でも動作したりします。

| `bash` コマンド | 説明 |
| :--- | :--- |
| `ls` | 特定ディレクトリ上のファイル一覧を表示 |
| `pwd` | カレントディレクトリを出力 |

### エイリアスの取得

現在のセッションで有効なエイリアスは、`Get-Alias` で取得できます。

```ps1
Get-Alias
<#
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           % -> ForEach-Object
Alias           ? -> Where-Object
Alias           ac -> Add-Content
Alias           asnp -> Add-PSSnapin
Alias           cat -> Get-Content
Alias           cd -> Set-Location
Alias           CFS -> ConvertFrom-String                          3.1.0.0    Microsoft.PowerShell.Utility
Alias           chdir -> Set-Location
Alias           clc -> Clear-Content
Alias           clear -> Clear-Host
Alias           clhy -> Clear-History
Alias           cli -> Clear-Item
Alias           clp -> Clear-ItemProperty
Alias           cls -> Clear-Host
Alias           clv -> Clear-Variable
Alias           cnsn -> Connect-PSSession
Alias           compare -> Compare-Object
Alias           copy -> Copy-Item
Alias           cp -> Copy-Item
Alias           cpi -> Copy-Item
Alias           cpp -> Copy-ItemProperty
Alias           curl -> Invoke-WebRequest
Alias           cvpa -> Convert-Path
Alias           dbp -> Disable-PSBreakpoint
Alias           del -> Remove-Item
Alias           diff -> Compare-Object
Alias           dir -> Get-ChildItem
Alias           dnsn -> Disconnect-PSSession
Alias           ebp -> Enable-PSBreakpoint
Alias           echo -> Write-Output
Alias           epal -> Export-Alias
Alias           epcsv -> Export-Csv
Alias           epsn -> Export-PSSession
Alias           erase -> Remove-Item
Alias           etsn -> Enter-PSSession
Alias           exsn -> Exit-PSSession
Alias           fc -> Format-Custom
Alias           fhx -> Format-Hex                                  3.1.0.0    Microsoft.PowerShell.Utility
Alias           fl -> Format-List
Alias           foreach -> ForEach-Object
Alias           ft -> Format-Table
Alias           fw -> Format-Wide
Alias           gal -> Get-Alias
Alias           gbp -> Get-PSBreakpoint
Alias           gc -> Get-Content
Alias           gcb -> Get-Clipboard                               3.1.0.0    Microsoft.PowerShell.Management
Alias           gci -> Get-ChildItem
Alias           gcm -> Get-Command
Alias           gcs -> Get-PSCallStack
Alias           gdr -> Get-PSDrive
Alias           ghy -> Get-History
Alias           gi -> Get-Item
Alias           gin -> Get-ComputerInfo                            3.1.0.0    Microsoft.PowerShell.Management
Alias           gjb -> Get-Job
Alias           gl -> Get-Location
Alias           gm -> Get-Member
Alias           gmo -> Get-Module
Alias           gp -> Get-ItemProperty
Alias           gps -> Get-Process
Alias           gpv -> Get-ItemPropertyValue
Alias           group -> Group-Object
Alias           gsn -> Get-PSSession
Alias           gsnp -> Get-PSSnapin
Alias           gsv -> Get-Service
Alias           gtz -> Get-TimeZone                                3.1.0.0    Microsoft.PowerShell.Management
Alias           gu -> Get-Unique
Alias           gv -> Get-Variable
Alias           gwmi -> Get-WmiObject
Alias           h -> Get-History
Alias           history -> Get-History
Alias           icm -> Invoke-Command
Alias           iex -> Invoke-Expression
Alias           ihy -> Invoke-History
Alias           ii -> Invoke-Item
Alias           ipal -> Import-Alias
Alias           ipcsv -> Import-Csv
Alias           ipmo -> Import-Module
Alias           ipsn -> Import-PSSession
Alias           irm -> Invoke-RestMethod
Alias           ise -> powershell_ise.exe
Alias           iwmi -> Invoke-WmiMethod
Alias           iwr -> Invoke-WebRequest
Alias           kill -> Stop-Process
Alias           lp -> Out-Printer
Alias           ls -> Get-ChildItem
Alias           man -> help
Alias           md -> mkdir
Alias           measure -> Measure-Object
Alias           mi -> Move-Item
Alias           mount -> New-PSDrive
Alias           move -> Move-Item
Alias           mp -> Move-ItemProperty
Alias           mv -> Move-Item
Alias           nal -> New-Alias
Alias           ndr -> New-PSDrive
Alias           ni -> New-Item
Alias           nmo -> New-Module
Alias           npssc -> New-PSSessionConfigurationFile
Alias           nsn -> New-PSSession
Alias           nv -> New-Variable
Alias           ogv -> Out-GridView
Alias           oh -> Out-Host
Alias           popd -> Pop-Location
Alias           ps -> Get-Process
Alias           pushd -> Push-Location
Alias           pwd -> Get-Location
Alias           r -> Invoke-History
Alias           rbp -> Remove-PSBreakpoint
Alias           rcjb -> Receive-Job
Alias           rcsn -> Receive-PSSession
Alias           rd -> Remove-Item
Alias           rdr -> Remove-PSDrive
Alias           ren -> Rename-Item
Alias           ri -> Remove-Item
Alias           rjb -> Remove-Job
Alias           rm -> Remove-Item
Alias           rmdir -> Remove-Item
Alias           rmo -> Remove-Module
Alias           rni -> Rename-Item
Alias           rnp -> Rename-ItemProperty
Alias           rp -> Remove-ItemProperty
Alias           rsn -> Remove-PSSession
Alias           rsnp -> Remove-PSSnapin
Alias           rujb -> Resume-Job
Alias           rv -> Remove-Variable
Alias           rvpa -> Resolve-Path
Alias           rwmi -> Remove-WmiObject
Alias           sajb -> Start-Job
Alias           sal -> Set-Alias
Alias           saps -> Start-Process
Alias           sasv -> Start-Service
Alias           sbp -> Set-PSBreakpoint
Alias           sc -> Set-Content
Alias           scb -> Set-Clipboard                               3.1.0.0    Microsoft.PowerShell.Management
Alias           select -> Select-Object
Alias           set -> Set-Variable
Alias           shcm -> Show-Command
Alias           si -> Set-Item
Alias           sl -> Set-Location
Alias           sleep -> Start-Sleep
Alias           sls -> Select-String
Alias           sort -> Sort-Object
Alias           sp -> Set-ItemProperty
Alias           spjb -> Stop-Job
Alias           spps -> Stop-Process
Alias           spsv -> Stop-Service
Alias           start -> Start-Process
Alias           stz -> Set-TimeZone                                3.1.0.0    Microsoft.PowerShell.Management
Alias           sujb -> Suspend-Job
Alias           sv -> Set-Variable
Alias           swmi -> Set-WmiInstance
Alias           tee -> Tee-Object
Alias           trcm -> Trace-Command
Alias           type -> Get-Content
Alias           wget -> Invoke-WebRequest
Alias           where -> Where-Object
Alias           wjb -> Wait-Job
Alias           write -> Write-Output
#>
```

### エイリアスの設定

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/new-alias

セッション中に新たにエイリアスを設けたい場合は、`New-Alias`（または `Set-Alias`）を使います。

```ps1
# New-Alias -Name {エイリアス名} -Value {エイリアス元のコマンドレットなど}
New-Alias Now Get-Date

Now # -> 2026年7月4日 15:25:18

# エイリアス情報は、Get-Alias から取得が可能
Get-Alias -Name Now
<#
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Now -> Get-Date
#>
```

なお、`Set-Alias` というものもあり、こちらは既存のエイリアスを変更することもできます。  
（既存のエイリアスがなければ、`New-Alias` と同様に動作します）

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/set-alias

```ps1
# パラメータを設けなければ、対話形式でエイリアスを設定していく
Set-Alias
<#
コマンド パイプライン位置 1 のコマンドレット Set-Alias
次のパラメーターに値を指定してください:
Name: hoge
Value: Get-Date
#>

hoge  # -> 2026年7月26日 11:49:49
```

#### エイリアスの注意事項：有効期限

コンソール上でエイリアスを `New-Alias`（`Set-Alias`）コマンドレッドで設定した場合、エイリアスはセッション中、要は**そのコンソール上でのみ**有効となります。

再起動したり別のコンソールを起動した場合、そちらにはエイリアスは反映されません。

もし自身の環境下で永続的にエイリアスを有効化したい場合は、PowerShell のプロファイル上で定義を挟んで上げる必要があります。

:::message

`bash` でいう、`.bashrc` や `.bash_profile` のようなものという認識で良いです。詳しい説明は省略しますが、Microsoft Learn に情報があるのでそちらを参照ください。

:::
