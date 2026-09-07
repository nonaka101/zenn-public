---
title: "🐣 コマンド関係"
---

## コマンドレット

PowerShell では、PowerShell 固有のコマンドに加え、実行可能ファイルなどのネイティブコマンド（例：コマンドプロンプト）も実行できます。

そして PowerShell でいう **コマンドレット** とは、PowerShell 上で扱える様々な *コマンド* の内、PowerShell 用に実装された（コンパイル済み）コマンドを指します。

:::message

コマンドレットは、実体としては `.NET` クラスとして実装されています。

:::

コマンドレットの特徴は、下記の通りです。

- PowerShell 独自のコマンド（コマンドプロンプトでは使えない）
- `{Verb}-{Noun}` といった命名規則（例：`Get-Process`）

https://learn.microsoft.com/ja-jp/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands

### 基本的な使い方

基本的なコマンドレットの使い方は、下記の通りです。

```ps1
Get-Date    # -> 2026年7月4日 15:13:15
```

コマンドレットには、パラメータを指定して動作を変更できるものが存在します。

```ps1
# 日時データを特定のフォーマットの形で取得する
Get-Date -Format "yyyy-MM-dd HH:mm" # -> 2026-07-04 15:14
```

### コマンドレットの一覧

PowerShell で利用可能なコマンドは、`Get-Command` で確認できます。

```ps1
Get-Command
<# 例
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Add-AppPackage                                     2.0.1.0    Appx
Alias           Add-AppPackageVolume                               2.0.1.0    Appx
Alias           Add-AppProvisionedPackage                          3.0        Dism
...
Function        A:
Function        Add-BitLockerKeyProtector                          1.0.0.0    BitLocker
Function        Add-DnsClientDohServerAddress                      1.0.0.0    DnsClient
Function        Add-DnsClientNrptRule                              1.0.0.0    DnsClient
Function        Add-DtcClusterTMMapping                            1.0.0.0    MsDtc
...
Filter          more
Cmdlet          Add-AppProvisionedSharedPackageContainer           3.0        Dism
Cmdlet          Add-AppSharedPackageContainer                      2.0.1.0    Appx
Cmdlet          Add-AppxPackage                                    2.0.1.0    Appx
Cmdlet          Add-AppxProvisionedPackage                         3.0        Dism
...
#>
```

:::message

コマンド種別によるフィルタリングも可能です。

```ps1
# コマンドレットに限定して取得
Get-Command -CommandType Cmdlet
```

:::

### コマンドレットのモジュール

PowerShell ではモジュールをインポートすることで、使用できるコマンドレットを追加することができます。

現在インポートされているモジュールを確認するには、`Get-Module` を使用します。

```ps1
Get-Module
<# 例
ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     1.0.0.0    ISE                                 {Get-IseSnippet, Import-IseSnippet, New-IseSnippet}
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Cl...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Obje...
#>
```

### コマンドレットのエイリアス

PowerShell のコマンドレットでは、慣れ親しんだエイリアスが用意されています。

例えばディレクトリ配下の子要素を取得するコマンドレットとして `Get-ChildItem` というのがありますが、下記がエイリアスとして設定されており同じように使うことが可能です。

- `dir`
- `gci`
- `ls`

:::message

ただし、パラメータの指定が異なる場合があるので注意が必要です。

:::

### コマンド履歴

過去に実行したコマンド履歴は、`Get-History` で確認できます。また履歴上のコマンドライン番号を使えば、`Invoke-History` で実行できます。

```ps1
Get-History
<#
  Id CommandLine
  -- -----------
   1 pwd
   2 ls
   3 cd "C:\Users\myAcc\OneDrive\デスクトップ\"
   4 Get-Command | ConvertTo-Html | Out-File .\command.html
   5 Get-Date
   6 Get-Command
   7 cd
   8 cd ~
   9 Get-Module
  10 Get-Module
  11 cd "C:\Users\myAcc\OneDrive\デスクトップ\"
  12 ls
  13 Get-ChildItem .\新しいフォルダ
  14 Get-ChildItem .\SG
  15 Get-ChildItem .\cpt
  16 Get-Date
#>

# 履歴 16 番にある Get-Date を呼び出す
Invoke-History 16   # -> 2026年7月4日 15:13:15
```

### ヘルプ機能

コマンドレットの使い方がわからない場合は、ヘルプ機能を使って調べることができます。

ヘルプを呼び出す方法として、まずコマンドレットに `-?` をつけるというのがあります。

```ps1
Get-Date -? # ヘルプを表示
<#
名前
    Get-Date
    
構文
    Get-Date [[-Date] <datetime>]  [<CommonParameters>]
    
    Get-Date [[-Date] <datetime>]  [<CommonParameters>]
    

エイリアス
    なし
    

注釈
    Get-Help を実行しましたが、このコンピューターにこのコマンドレットのヘルプ ファイルは見つかりませんでした。ヘルプの一部だけが表示されています。
        -- このコマンドレットを含むモジュールのヘルプ ファイルをダウンロードしてインストールするには、Update-Help を使用してください。
        -- このコマンドレットのヘルプ トピックをオンラインで確認するには、「Get-Help Get-Date -Online」と入力するか、
           https://go.microsoft.com/fwlink/?LinkID=113313 を参照してください。
#>
```

より詳細なヘルプ情報を閲覧したい場合は、`Get-Help` コマンドレットや、`Help` 関数を使用します。

```ps1
Get-Help Get-Date   # Get-Help では、ヘルプ情報を一挙に表示
Help Get-Date       # Help 関数では、画面単位で表示
```

また `Get-Help` では調べたい内容に応じてパラメータを設定することが可能です。

例えば `Get-Help Get-Date -Full` で、パラメーターの詳細などを含む完全な形式でヘルプを表示します。

```ps1
# `-Online` を指定すると、既定のWebブラウザを使いオンライン版ヘルプを開く
Get-Help Get-Date -Online
<#
下記 URL の、Microsoft のサイト上にあるドキュメントを表示
https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/get-date?view=powershell-5.1&WT.mc_id=ps-gethelp
#>


Get-Help Get-Date -Detailed # 詳細を表示
<#
名前
    Get-Date
    
概要
    Gets the current date and time.
    
    
構文
    Get-Date [[-Date] <System.DateTime>] [-Day <System.Int32>] [-DisplayHint <Microsoft.PowerShell.Commands.DisplayHintType>] [-Format <System.String>] [-Hour <System.Int32>] [-Mil
    lisecond <System.Int32>] [-Minute <System.Int32>] [-Month <System.Int32>] [-Second <System.Int32>] [-Year <System.Int32>] [<CommonParameters>]
    
    Get-Date [[-Date] <System.DateTime>] [-Day <System.Int32>] [-DisplayHint <Microsoft.PowerShell.Commands.DisplayHintType>] [-Hour <System.Int32>] [-Millisecond <System.Int32>] [
    -Minute <System.Int32>] [-Month <System.Int32>] [-Second <System.Int32>] [-UFormat <System.String>] [-Year <System.Int32>] [<CommonParameters>]
    
    
説明
    The `Get-Date` cmdlet gets a **DateTime** object that represents the current date or a date that you specify. `Get-Date` can format the date and time in several .NET and Unix f
    ormats. You can use `Get-Date` to generate a date or time character string, and then send the string to other cmdlets or programs.
    
    `Get-Date` uses the current culture settings of the operating system to determine how the output is formatted. To view your computer's settings, use `(Get-Culture).DateTimeForm
    at`.
    

パラメーター
    -Date [<System.DateTime>]
        Specifies a date and time. Time is optional and if not specified, returns 00:00:00. Enter the date
        and time in a format that is standard for the currently selected locale. You can change the
        current locale using the `Set-Culture` cmdlet.
        
        For example, in US English:
        
        `Get-Date -Date "6/25/2019 12:30:22"` returns **Tuesday, June 25, 2019 12:30:22**
        
    -Day [<System.Int32>]
        Specifies the day of the month that is displayed. Enter a value from 1 to 31.
        
        If the specified value is greater than the number of days in a month, PowerShell adds the number of
        days to the month. For example, `Get-Date -Month 4 -Day 31` displays **May 1**, not **April 31**.
...
#>
```

:::message

個人的にオススメなのは、**コマンド例を出力する** `-Examples` です。

Web 上で掲載されているようなサンプルコマンドは汎用的すぎる場合があるので、PowerShell に慣れてきたらまずはココで大まかな使い方を閲覧すると良いかもしれません。  
（説明は英語にはなりますが・・・）

```ps1
Get-Help Get-Date -Examples
# （※ markdown の表現上、一部の出力内容を弄っています）
<#
名前
    Get-Date
    
概要
    Gets the current date and time.
    
    
    --------- Example 1: Get the current date and time ---------
    
    In this example, `Get-Date` displays the current system date and time. The output is in the
    long-date and long-time formats.
    
    '''powershell
    Get-Date
    '''
    
    '''Output
    Tuesday, June 25, 2019 14:53:32
    '''
    
     --------- Example 2: Get elements of the current date and time ---------
    
    This example shows how to use `Get-Date` to get either the date or time element. The parameter uses
    the arguments **Date**, **Time**, or **DateTime**.
    
    '''powershell
    Get-Date -DisplayHint Date
    '''
    
    '''Output
    Tuesday, June 25, 2019
    '''
    
    `Get-Date` uses the **DisplayHint** parameter with the **Date** argument to get only the date.
    
     --------- Example 3: Get the date and time with a .NET format specifier ---------
    
    In this example, a .NET format specifier is used to customize the output's format. The output is a
    **String** object.
    
    '''powershell
    Get-Date -Format "dddd MM/dd/yyyy HH:mm K"
    '''
    
    '''Output
    Tuesday 06/25/2019 16:17 -07:00
    '''
    
    `Get-Date` uses the **Format** parameter to specify several format specifiers.
    
    The .NET format specifiers used in this example are defined as follows:
    
    | Specifier |                      Definition                       |
    | --------- | ----------------------------------------------------- |
    | `dddd`    | Day of the week - full name                           |
    | `MM`      | Month number                                          |
    | `dd`      | Day of the month - 2 digits                           |
    | `yyyy`    | Year in 4-digit format                                |
    | `HH:mm`   | Time in 24-hour format - no seconds                   |
    | `K`       | Time zone offset from Universal Time Coordinate (UTC) |
    
    For more information about .NET format specifiers, see
    [Custom date and time format strings](/dotnet/standard/base-types/custom-date-and-time-format-strings).
...
#>
```

:::

#### Help 関数

`Get-Help` では出力を一気に表示するため、文書が長いとスクロールを戻して確認していく必要があります。

`Help` 関数は、ページ単位で表示するためのユーティリティ関数です。

```ps1
help Get-ChildItem
help get-childitem -online
```

:::message

中身を知りたい場合は、`(Get-Command help).Definition` から詳細を見ることができます。

:::

## 基本的なコマンドレット

コマンドレットの練習として、下記の要素を使って挙動を見てみるのをおすすめしております。

:::message

紙幅の関係上 それぞれの項目の詳細説明は省略いたします、ご了承ください。

:::

- [New-Item](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/new-item)
- [Copy-Item](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/copy-item)
- [Rename-Item](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/rename-item)
- [Remove-Item](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/remove-item)
- [Move-Item](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/move-item)
- [Get-Item](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/get-item)
- [Get-Member](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/get-member)
- [Set-Content](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/set-content)
- [Add-Content](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/add-content)
- [Clear-Content](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/clear-content)
- [Test-Path](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/test-path)
- [Get-Date](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/get-date)
- [Write-Host](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/write-host)
- [Get-ChildItem](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/get-childitem)
