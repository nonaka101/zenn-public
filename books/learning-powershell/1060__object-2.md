---
title: "🐣 オブジェクトを扱ってみる"
---

前項で、PowerShell に関する下記の事項について確認しました。

- オブジェクトを中心とした設計
- オブジェクトの中身を知る方法について

この項では、下記の内容について解説していきます。

- オブジェクトの生成、加工
- パイプラインとは何か
- 代表的なコマンドレットはどういったものがあるか

## オブジェクトの生成

### `PSCustomObject`：新規に生成

https://learn.microsoft.com/ja-jp/powershell/scripting/learn/deep-dives/everything-about-pscustomobject

新規にオブジェクトを生成する場合、ハッシュテーブル（`@{ }`）から `[PSCustomObject]` を使って構築するのが基本となります。

```ps1
$userObject = [PSCustomObject]@{
  Name = "Taro"
  Age = 20
  Address = "hoge@fuga.jp"
}
```

#### 補足：ハッシュテーブル（連想配列）との違い

配列のところで、オブジェクトと似たような *ハッシュテーブル*（連想配列）について説明しました。

両者は、型や所有しているメンバーが異なります。

```ps1
# ハッシュテーブルで作成
$userHashTable = @{
  Name = "Taro"
  Age = 20
  Address = "hoge@fuga.jp"
}

# オブジェクトとして生成
$userObject = [PSCustomObject]@{
  Name = "Taro"
  Age = 20
  Address = "hoge@fuga.jp"
}

# ハッシュテーブルの型及びメンバー
$userHashTable | Get-Member
<#
   TypeName: System.Collections.Hashtable

Name              MemberType            Definition
----              ----------            ----------
Add               Method                void Add(System.Object key, System.Object value), void IDictionary.Add(System.Object key, System.Object value)
Clear             Method                void Clear(), void IDictionary.Clear()
Clone             Method                System.Object Clone(), System.Object ICloneable.Clone()
Contains          Method                bool Contains(System.Object key), bool IDictionary.Contains(System.Object key)
ContainsKey       Method                bool ContainsKey(System.Object key)
ContainsValue     Method                bool ContainsValue(System.Object value)
CopyTo            Method                void CopyTo(array array, int arrayIndex), void ICollection.CopyTo(array array, int index)
Equals            Method                bool Equals(System.Object obj)
GetEnumerator     Method                System.Collections.IDictionaryEnumerator GetEnumerator(), System.Collections.IDictionaryEnumerator IDictionary.GetEnumerator(), System.Collections.IEnumerator IEnumerable.GetEnumera...
GetHashCode       Method                int GetHashCode()
GetObjectData     Method                void GetObjectData(System.Runtime.Serialization.SerializationInfo info, System.Runtime.Serialization.StreamingContext context), void ISerializable.GetObjectData(System.Runtime.Seria...
GetType           Method                type GetType()
OnDeserialization Method                void OnDeserialization(System.Object sender), void IDeserializationCallback.OnDeserialization(System.Object sender)
Remove            Method                void Remove(System.Object key), void IDictionary.Remove(System.Object key)
ToString          Method                string ToString()
Item              ParameterizedProperty System.Object Item(System.Object key) {get;set;}
Count             Property              int Count {get;}
IsFixedSize       Property              bool IsFixedSize {get;}
IsReadOnly        Property              bool IsReadOnly {get;}
IsSynchronized    Property              bool IsSynchronized {get;}
Keys              Property              System.Collections.ICollection Keys {get;}
SyncRoot          Property              System.Object SyncRoot {get;}
Values            Property              System.Collections.ICollection Values {get;}
#>


# オブジェクトの型及びメンバー
$userObject | Get-Member
<#
   TypeName: System.Management.Automation.PSCustomObject

Name        MemberType   Definition
----        ----------   ----------
Equals      Method       bool Equals(System.Object obj)
GetHashCode Method       int GetHashCode()
GetType     Method       type GetType()
ToString    Method       string ToString()
Address     NoteProperty string Address=hoge@fuga.jp
Age         NoteProperty int Age=20
Name        NoteProperty string Name=Taro
#>
```

### `New-Object`：既存クラスを流用

完全に新規に作るのでなく、`New-Object` を使い .NET の既存クラス等を流用する方法もあります。

```ps1
# 例: System.Collections.ArrayList の作成
$arrayList = New-Object -TypeName "System.Collections.ArrayList"
[void]$arrayList.Add("item1")
[void]$arrayList.Add("item2")
# ↑ Add メソッドはそのままだとインデックスを返すため、void を用いて消している

$arrayList  # -> item1 item2
```

### 備考：過去に使われた、空文字からカスタムオブジェクトを生成するハック

:::message

ここに記載している内容は 利用を推奨するものではありません。

記載の意図としては、過去のコードに対応するためとなります。`PSCustomObject` が主流になる以前はここに記載したやり方が主だった時があるため、それに出くわしても混乱しないよう ここで解説しております。

:::

`PSCustomObject` によるカスタムオブジェクトが主流でなかった過去では、空文字からカスタムオブジェクトを作るというハックが行われていました。

```ps1
$obj = "" | Select-Object Name, Age
# String にはこれらのプロパティは存在しないので、これらの空プロパティを持つ新規オブジェクトを生成できる。

$obj.Name = "Taro"
$obj.Age  = 20
$obj | Format-List *
<#
Name : Taro
Age  : 20
#>
```

これは `Select-Object` には、入力オブジェクトから指定したプロパティだけを持つ新しいオブジェクトを生成する機能があることを利用したものでした。

ただし現在は、カスタムオブジェクトを作成する目的には `PSCustomObject` を使用する方が、意図が明確で簡潔です。新しくコードを書く場合は、この方法ではなく `PSCustomObject` の使用を推奨します。

## カスタムプロパティ

既存のオブジェクトにカスタムプロパティを追加したり、必要なプロパティを持つ新しいオブジェクトを作成したりできます。

### `Add-Member` を使用して既存のオブジェクトにプロパティを追加

```ps1
# NoteProperty を使い、（値を保持する）プロパティを追加
$object = [PSCustomObject]@{}
$object | Add-Member -MemberType NoteProperty -Name "Name" -Value "Yoshida"
$object | Add-Member -MemberType NoteProperty -Name "Role" -Value "Administrator"
$object | Format-List *
<#
Name : Yoshida
Role : Administrator
#>
```

### `Select-Object` を使って、カスタムプロパティを含む新しいオブジェクトを作成

```ps1
# 例: プロセス情報にカスタムプロパティを追加
Get-Process | Select-Object Name, @{
  Name="Memory(MB)";                # カスタムプロパティの名前
  Expression={$_.WorkingSet / 1MB}  # カスタムプロパティの値を計算
}
<#
Name                                    Memory(MB)
----                                    ----------
backgroundTaskHost                      16.9765625
Code                                   253.6484375
explorer                              274.51953125
M365Copilot                            156.8515625
msedge                                   26.515625
msedgewebview2                        114.86328125
Notepad                               147.00390625
...
#>
```

## パイプライン

PowerShell を扱う上で最も重要かつ強力な機能が **パイプライン** です。

パイプラインは `|`（縦棒、パイプ記号）を使って、あるコマンドの結果（≒ **オブジェクト**）を **次のコマンドにそのまま引き渡す** 仕組みのことです。

```ps1
# コマンドA の結果を コマンドB に渡す
コマンドA | コマンドB
```

### なぜパイプラインが強力なのか？

コマンドプロンプト（`cmd.exe`）や Linux のシェル（bash など）のパイプラインでは、一般に標準出力のデータを次のコマンドの標準入力へ渡します。多くの場合、そのデータは**テキスト**として解釈・処理されます。

一方、PowerShell のコマンドレット間では、パイプラインを通して **オブジェクト（データの塊）** を受け渡すことができます。

これにより、受け取った側のコマンドは文字列の分割（パース）などを自力で行う必要がなくなり、オブジェクトの「プロパティ」を直接参照して処理することができます。

```ps1
# 例：実行中のプロセスを取得し、CPU使用時間が多い順に並び替え、上位5つを取得する
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
<#
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    539      57   560372     500036   1,052.89   5316   1 Code
    553      84   882900     562900     553.55  17232   1 Code
    488      25   272288     295920     228.89   5964   1 Code
   2174     146   116428     263508     196.03   3116   1 msedge
   1181      58   118004     156308     151.41   7044   1 Code
#>
```

上記のコマンドでは、以下の流れでオブジェクトが渡されています。

1. `Get-Process` が **プロセスオブジェクトのまとまり** を出力し、パイプライン（`|`）に流す。
2. `Sort-Object` がそのオブジェクトを受け取り、各オブジェクトが持つ `CPU` プロパティの値を基準に 降順 で並び替え、再びパイプラインに流す。
3. `Select-Object` がそれを受け取り、最初の 5 つ（`-First 5`）だけを抽出して最終的な出力とする。

### 自動変数 `$_` の活用

パイプラインで渡されてきた「現在処理中の1つのオブジェクト」を参照するためには、自動変数の `$_`（または `$PSItem`）を使用します。

これは `Where-Object` による条件の絞り込みや、`ForEach-Object` による各要素への処理で頻繁に登場します。

```ps1
# サービスの一覧から、Status プロパティが 'Running'（実行中）のものだけを抽出する
Get-Service | Where-Object { $_.Status -eq 'Running' }

# 1から3までの数字を順番に受け取り、それぞれ2倍にして出力する
1..3 | ForEach-Object { $_ * 2 }
<#
2
4
6
#>
```

## `Where-Object`：オブジェクトの絞り込み

取得したオブジェクトの中から、特定の条件を満たすものだけを抽出したい場合は `Where-Object` を使用します。

```ps1
# 実行中のサービス一覧を取得し、Status が "Running" のものだけを抽出する
Get-Service | Where-Object { $_.Status -eq 'Running' }
<#
Status   Name               DisplayName
------   ----               -----------
Running  Appinfo            Application Information
Running  AudioEndpointBu... Windows Audio Endpoint Builder
Running  Audiosrv           Windows Audio
...
#>


# メモリ使用量が 100MB を超えるプロセスのみを取得
Get-Process | Where-Object {$_.WorkingSet -gt 100MB}
<#
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1273      68   148192     197792       7.00   3024  34 Code
   7449     208   516340     620812     247.78  18952  34 explorer
   1357      60    98696     150272       2.53  26480  34 M365Copilot
    548      40   124344     190484      19.45   9252  34 msedge
   1272      50   162516     205004      20.52  15880  34 Notepad
   1024      58   128072     212700      20.16   6968  34 OneDrive
   1724     115   449784     483956      96.67   2068  34 powershell_ise
   1721      68   603316     610520      13.92  16372  34 SnippingTool
   1155      39    67284     114384       4.70  14520  34 TextInputHost
#>

```

上記の `$_` は、パイプラインから渡されてきた「1つ1つのオブジェクト」を表す自動変数です。

:::message

よりシンプルに書く構文も用意されています。

```ps1
# 下記の簡略構文で、上記と同じ意味になります
Get-Service | Where-Object Status -eq 'Running'
```

:::

## `Select-Object`：プロパティの抽出と作成

オブジェクトが持つ多くのプロパティのうち、「名前と状態だけを取り出したい」といった場面が出てきます。

`Select-Object` を使うと、不要なプロパティを除き、**必要なプロパティだけを持つ新しいオブジェクト**を作成することができます。

```ps1
# サービス名と状態だけを抽出する
Get-Service | Select-Object Name, Status
<#
Name                                     Status
----                                     ------
AarSvc_d1872                            Stopped
Appinfo                                 Running
AppMgmt                                 Stopped
...
#>
```

### カスタムプロパティ（計算プロパティ）の作成

`Select-Object` では、既存のプロパティだけでなく、ハッシュテーブル（連想配列）を使ってその場で新しいプロパティを作り出すことも可能です。これは実務上で非常に役立ちます。（例：レポート出力）

ハッシュテーブルのキーには `Name`（または `Label`, `N`, `L`）と `Expression`（または `E`）を指定します。

```ps1
# ファイル一覧を取得し、ファイルのサイズを KB (キロバイト) 単位に計算して表示する例
Get-ChildItem -File |
  Select-Object Name, @{
    Name="Size(KB)"
    Expression={ [math]::Round($_.Length / 1KB, 2) }
  }
<#
Name                     Size(KB)
----                     --------
sample_document.docx        15.24
test_script.ps1              2.10
report_data.csv            105.88
#>
```

## `Sort-Object`：オブジェクトの並び替え

オブジェクトを特定のプロパティで昇順・降順に並び替えたい場合は `Sort-Object` を使用します。

```ps1
# CPU使用時間が多い順（降順：Descending）にプロセスを並び替えて、上位5件を取得する
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
<#
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   2451       0      192        852   1,524.31      4   0 System
   1340      59   485024     544784   1,036.58   5088  34 msedge
    842      48   152340     193488     412.18   7216  34 msedge
   7459     208   515152     618628     242.13  18952  34 explorer
   2783     153   212808     393272     183.09   7064  34 msedge
#>



# カレントディレクトリ内の項目（ディレクトリ含む）を、最終更新日時で降順にソート
Get-ChildItem | Sort-Object -Property LastWriteTime -Descending
<#
    ディレクトリ: C:\WINDOWS\system32\WindowsPowerShell\v1.0

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        2026/05/15      7:00                ja-JP
d-----        2026/05/15      7:00                en
d-----        2026/05/15      7:00                en-US
d-----        2026/05/15      7:00                ja
-a----        2025/09/10     12:42         454656 powershell.exe
-a----        2025/06/12     17:29          65536 PSEvents.dll
d-----        2025/06/12     17:24                Modules
-a----        2025/06/12     17:04         212992 powershell_ise.exe
d-----        2024/04/01     16:26                Examples
d-----        2024/04/01     16:26                Schemas
d-----        2024/04/01     16:26                SessionConfig
-a----        2024/04/01     16:22         210376 types.ps1xml
-a----        2024/04/01     16:22           4994 Diagnostics.Format.ps1xml
-a----        2024/04/01     16:22           8458 Registry.format.ps1xml
-a----        2024/04/01     16:22           4097 PowerShellTrace.format.ps1xml
-a----        2024/04/01     16:22         206468 PowerShellCore.format.ps1xml
-a----        2024/04/01     16:22            395 powershell.exe.config
-a----        2024/03/28     21:51           1030 powershell_ise.exe.config
#>
```

## `Group-Object`：オブジェクトのグループ化

特定のプロパティの値ごとにオブジェクトをまとめたい場合は `Group-Object` を使用します。

```ps1
# サービスを Status（実行中か停止中か）ごとにグループ化し、それぞれの数をカウントする
Get-Service | Group-Object Status
<#
Count Name                      Group
----- ----                      -----
  112 Stopped                   {AarSvc_d1872, AppMgmt, AppReadiness, AppVClient...}
   76 Running                   {Appinfo, AudioEndpointBuilder, Audiosrv, BFE...}
#>
```
