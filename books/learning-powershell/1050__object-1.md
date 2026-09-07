---
title: "🐣 PowerShell 上のオブジェクトを知る"
---

PowerShell が他のシェルと異なるのが、**オブジェクトを中心とした設計**であることです。

`bash` などの一般的なシェルでは テキストの入出力を扱うアプローチなのに対し、PowerShell では構造化されたオブジェクトの形でデータを受け渡しすることができます。これは使いこなせればかなり強力なものになります。

一方で PowerShell を学び始めた際、おそらく最初にぶつかるであろう壁も、この「オブジェクト」という考え方になるかもしれません。他のシェルにない この「オブジェクト中心に回す」という考え方を、シェル初心者が扱うには難しい部分があるからです。

本項では、PowerShell に触れる際 **まず何を確認すればよいか** がわかる状態を目指すために、次の点を整理します。

- PowerShell でいう「オブジェクト」とは何か
- オブジェクトにはどのような情報や機能があるのか
- わからない時に、どのように調べればよいか
- 画面で見づらい時に、どうやってファイルへ出力するか

:::message

オブジェクトについての厳密な話は、ここではしません。私も詳しく語れるレベルではないですし、PowerShell に触れるという趣旨を考えれば、そこまで必要がないためです。

:::

## オブジェクトとは何か

PowerShell では ファイル、サービス、プロセス、フォルダなど、さまざまな情報を扱うことができます。

そして PowerShell の特徴として、こうした対象を**単なる文字列**ではなく、**意味を持ったデータのまとまり**（≒ オブジェクト）として扱う点があります。

例えば PowerShell では、多くのコマンドの実行結果が **オブジェクト** として返されます。ここでは一例として、実行中のプロセスを取得するコマンドレット `Get-Process` を取り扱ってみます。

### 例：`Get-Process` コマンドレット

```ps1
Get-Process
<#
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
（... 中略 ...）
    184      18    30068     105012       0.19   7100   3 Code
   3951      99   108460     286096      14.22  21480   3 explorer
    164      10    11324      22236       0.05   6896   3 msedge
   1243      67    65284     154668       4.09  25584   3 OneDrive
    797      60    62536      92720       4.70  19264   3 powershell
    867      79   398288     433868       5.42  15632   3 powershell_ise
    886      41    32288      88672       0.59  14576   3 PowerToys
（... 中略 ...）
#>
```

この結果は、単に `explorer`, `msedge` という文字だけを返しているわけではありません。実際には、各プロセスについて次のような情報を **構造化された形**で持っています。

- プロセス名
- プロセス ID
- CPU 使用時間
- メモリ使用量
- 実行ファイルのパス

このように、**1つの対象について、複数の情報をひとまとめにして扱えるもの**が、ここでいうオブジェクトです。

### イメージで考える

オブジェクトは、下表のような「情報カード」に近いものだと考えると理解しやすいです。

PowerShell は、この「情報カード」のようなものをパイプラインで次のコマンドへ渡して処理します。

| 項目 | 値 |
| :---: | :--- |
| 対象 | `explorer` |
| 名前 | `explorer` |
| ID | `1234` |
| CPU | `10.5` |
| パス | `C:\Windows\explorer.exe` |

## オブジェクトには何があるのか

PowerShell のオブジェクトを見る時、まず押さえたいのは次の 2 つです。

| 項目 | 説明 |
| :--- | :--- |
| **プロパティ** | そのオブジェクトが持っている**情報** |
| **メソッド** | そのオブジェクトに対して行える**操作** |

### プロパティ

プロパティは、「その対象についての項目（情報）」です。

たとえば、`Get-Process` の結果には次のようなプロパティがあります。

たとえば「プロセス名を知りたい」「ID を見たい」といった時は、プロパティを確認します。

- `Name`
- `Id`
- `CPU`
- `Path`
- `WorkingSet`

### メソッド

メソッドは、「その対象に対して行える操作」です。オブジェクトにもよりますが、よくあるメソッドとして下記を取り上げてみます。

- `Add()` メソッドを使うと、オブジェクトに新しい要素を加えることができる
- `Remove()` メソッドがあると、特定の要素を削除できる
- `ToString()`, `ToJson()` などで、特定の形式に変換することができる

:::message

それぞれのクラスに応じて、持っているメソッドは異なります。

:::

## わからない時の基本方針

PowerShell でコマンドを実行した際、結果の見方がわからない時は 次の順で考えると整理しやすいです。

1. **どんな種類のオブジェクト**が返ってきているかを確認する
2. **どんなプロパティ・メソッドを持っているか**を確認する
3. 画面表示を変えて、中身を見やすくする
4. 必要な項目だけを抜き出す
5. グリッドビューワーで確認する
6. （必要なら）ファイルに出力して確認する

つまり、まずは最初に**オブジェクトの中身を把握すること**が重要というわけです。

### `GetType()` で型を確認する

オブジェクトには `GetType()` メソッドが搭載されており、そこから型情報を確認することができます。

```ps1
# コマンドの実行結果を直接オブジェクトとして扱う場合は、括弧で括る
(Get-Process | Select-Object -First 1).GetType()
<#
IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     False    Process                                  System.ComponentModel.Component
#>


"あいうえお".GetType()
<#
IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     String                                   System.Object
#>

"あいうえお".GetType().FullName # -> System.String


# `-as` で型をキャストできるが、下記の場合は 基底型として参照しても実際のランタイム型が DirectoryInfo のままとなる
$fsroot = (Get-Item /) -as [System.IO.FileSystemInfo]
$fsroot.GetType().FullName  # -> System.IO.DirectoryInfo
```

`GetType()` で**型が分かれば、Microsoft Learn を使ってクラス情報が調べられる**ため、どのようなオブジェクトなのかを掴む第一歩となります。

:::message

例えば先程の `System.IO.DirectoryInfo` を Web検索にかければ、[DirectoryInfo クラス](https://learn.microsoft.com/ja-jp/dotnet/api/system.io.directoryinfo)の情報を拾うことができます。

ここからプロパティにはどんなものがあるのか、メソッドには何を持っているのか、といったものが調べられます。

:::

### `Get-Member` で「何を持っているか」を調べる

オブジェクトの調査で、型の確認の次に行うことは `Get-Member` コマンドレットによるメンバーの確認です。

このメソッドを使うと、**プロパティ**, **メソッド** を含む オブジェクトのメンバーを確認することができます。

#### 基本的な使い方

`Get-Member` の結果では、最初は次の点に注目すれば十分です。

- `TypeName` : どんな**種類**のオブジェクトか
- `Property` : どんな**情報**を持っているか
- `Method` : どんな**操作**ができるか

```ps1
# Get-Process が返すオブジェクトに対し、メンバーを確認する
Get-Process | Get-Member
<#
   TypeName: System.Diagnostics.Process

Name                       MemberType     Definition
----                       ----------     ----------
Handles                    AliasProperty  Handles = Handlecount
Name                       AliasProperty  Name = ProcessName
NPM                        AliasProperty  NPM = NonpagedSystemMemorySize64
PM                         AliasProperty  PM = PagedMemorySize64
SI                         AliasProperty  SI = SessionId
VM                         AliasProperty  VM = VirtualMemorySize64
WS                         AliasProperty  WS = WorkingSet64
Disposed                   Event          System.EventHandler Disposed(System.Object, System.EventArgs)
ErrorDataReceived          Event          System.Diagnostics.DataReceivedEventHandler ErrorDataReceived(System.Object, System.Diagnostics.DataReceivedEventArgs)
Exited                     Event          System.EventHandler Exited(System.Object, System.EventArgs)
OutputDataReceived         Event          System.Diagnostics.DataReceivedEventHandler OutputDataReceived(System.Object, System.Diagnostics.DataReceivedEventArgs)
BeginErrorReadLine         Method         void BeginErrorReadLine()
BeginOutputReadLine        Method         void BeginOutputReadLine()
CancelErrorRead            Method         void CancelErrorRead()
CancelOutputRead           Method         void CancelOutputRead()
Close                      Method         void Close()
CloseMainWindow            Method         bool CloseMainWindow()
CreateObjRef               Method         System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                    Method         void Dispose(), void IDisposable.Dispose()
Equals                     Method         bool Equals(System.Object obj)
GetHashCode                Method         int GetHashCode()
GetLifetimeService         Method         System.Object GetLifetimeService()
GetType                    Method         type GetType()
InitializeLifetimeService  Method         System.Object InitializeLifetimeService()
Kill                       Method         void Kill()
Refresh                    Method         void Refresh()
Start                      Method         bool Start()
ToString                   Method         string ToString()
WaitForExit                Method         bool WaitForExit(int milliseconds), void WaitForExit()
WaitForInputIdle           Method         bool WaitForInputIdle(int milliseconds), bool WaitForInputIdle()
__NounName                 NoteProperty   string __NounName=Process
BasePriority               Property       int BasePriority {get;}
Container                  Property       System.ComponentModel.IContainer Container {get;}
EnableRaisingEvents        Property       bool EnableRaisingEvents {get;set;}
ExitCode                   Property       int ExitCode {get;}
ExitTime                   Property       datetime ExitTime {get;}
Handle                     Property       System.IntPtr Handle {get;}
HandleCount                Property       int HandleCount {get;}
HasExited                  Property       bool HasExited {get;}
Id                         Property       int Id {get;}
MachineName                Property       string MachineName {get;}
MainModule                 Property       System.Diagnostics.ProcessModule MainModule {get;}
MainWindowHandle           Property       System.IntPtr MainWindowHandle {get;}
MainWindowTitle            Property       string MainWindowTitle {get;}
MaxWorkingSet              Property       System.IntPtr MaxWorkingSet {get;set;}
MinWorkingSet              Property       System.IntPtr MinWorkingSet {get;set;}
Modules                    Property       System.Diagnostics.ProcessModuleCollection Modules {get;}
NonpagedSystemMemorySize   Property       int NonpagedSystemMemorySize {get;}
NonpagedSystemMemorySize64 Property       long NonpagedSystemMemorySize64 {get;}
PagedMemorySize            Property       int PagedMemorySize {get;}
PagedMemorySize64          Property       long PagedMemorySize64 {get;}
PagedSystemMemorySize      Property       int PagedSystemMemorySize {get;}
PagedSystemMemorySize64    Property       long PagedSystemMemorySize64 {get;}
PeakPagedMemorySize        Property       int PeakPagedMemorySize {get;}
PeakPagedMemorySize64      Property       long PeakPagedMemorySize64 {get;}
PeakVirtualMemorySize      Property       int PeakVirtualMemorySize {get;}
PeakVirtualMemorySize64    Property       long PeakVirtualMemorySize64 {get;}
PeakWorkingSet             Property       int PeakWorkingSet {get;}
PeakWorkingSet64           Property       long PeakWorkingSet64 {get;}
PriorityBoostEnabled       Property       bool PriorityBoostEnabled {get;set;}
PriorityClass              Property       System.Diagnostics.ProcessPriorityClass PriorityClass {get;set;}
PrivateMemorySize          Property       int PrivateMemorySize {get;}
PrivateMemorySize64        Property       long PrivateMemorySize64 {get;}
PrivilegedProcessorTime    Property       timespan PrivilegedProcessorTime {get;}
ProcessName                Property       string ProcessName {get;}
ProcessorAffinity          Property       System.IntPtr ProcessorAffinity {get;set;}
Responding                 Property       bool Responding {get;}
SafeHandle                 Property       Microsoft.Win32.SafeHandles.SafeProcessHandle SafeHandle {get;}
SessionId                  Property       int SessionId {get;}
Site                       Property       System.ComponentModel.ISite Site {get;set;}
StandardError              Property       System.IO.StreamReader StandardError {get;}
StandardInput              Property       System.IO.StreamWriter StandardInput {get;}
StandardOutput             Property       System.IO.StreamReader StandardOutput {get;}
StartInfo                  Property       System.Diagnostics.ProcessStartInfo StartInfo {get;set;}
StartTime                  Property       datetime StartTime {get;}
SynchronizingObject        Property       System.ComponentModel.ISynchronizeInvoke SynchronizingObject {get;set;}
Threads                    Property       System.Diagnostics.ProcessThreadCollection Threads {get;}
TotalProcessorTime         Property       timespan TotalProcessorTime {get;}
UserProcessorTime          Property       timespan UserProcessorTime {get;}
VirtualMemorySize          Property       int VirtualMemorySize {get;}
VirtualMemorySize64        Property       long VirtualMemorySize64 {get;}
WorkingSet                 Property       int WorkingSet {get;}
WorkingSet64               Property       long WorkingSet64 {get;}
PSConfiguration            PropertySet    PSConfiguration {Name, Id, PriorityClass, FileVersion}
PSResources                PropertySet    PSResources {Name, Id, Handlecount, WorkingSet, NonPagedMemorySize, PagedMemorySize, PrivateMemorySize, VirtualMemorySize, Threads.Coun...
Company                    ScriptProperty System.Object Company {get=$this.Mainmodule.FileVersionInfo.CompanyName;}
CPU                        ScriptProperty System.Object CPU {get=$this.TotalProcessorTime.TotalSeconds;}
Description                ScriptProperty System.Object Description {get=$this.Mainmodule.FileVersionInfo.FileDescription;}
FileVersion                ScriptProperty System.Object FileVersion {get=$this.Mainmodule.FileVersionInfo.FileVersion;}
Path                       ScriptProperty System.Object Path {get=$this.Mainmodule.FileName;}
Product                    ScriptProperty System.Object Product {get=$this.Mainmodule.FileVersionInfo.ProductName;}
ProductVersion             ScriptProperty System.Object ProductVersion {get=$this.Mainmodule.FileVersionInfo.ProductVersion;}
#>
```

:::message

オブジェクトによっては `GetType()` メソッドを使い、型の詳細を確認することもできます。

:::

#### オブジェクトの詳細が知りたい場面で活用

様々な場面で、オブジェクトの詳細を確認することは有用です。`Get-Member` は、次のような場面で特に役立ちます。

- そのコマンドの結果から、何が取れるのかわからない時
- 欲しい項目名がわからない時
- プロパティ名の正確な綴りを確認したい時

```ps1
Get-Service | Get-Member
<#
   TypeName: System.ServiceProcess.ServiceController

Name                      MemberType    Definition
----                      ----------    ----------
Name                      AliasProperty Name = ServiceName
RequiredServices          AliasProperty RequiredServices = ServicesDependedOn
Disposed                  Event         System.EventHandler Disposed(System.Object, System.EventArgs)
Close                     Method        void Close()
Continue                  Method        void Continue()
CreateObjRef              Method        System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                   Method        void Dispose(), void IDisposable.Dispose()
Equals                    Method        bool Equals(System.Object obj)
ExecuteCommand            Method        void ExecuteCommand(int command)
GetHashCode               Method        int GetHashCode()
GetLifetimeService        Method        System.Object GetLifetimeService()
GetType                   Method        type GetType()
InitializeLifetimeService Method        System.Object InitializeLifetimeService()
Pause                     Method        void Pause()
Refresh                   Method        void Refresh()
Start                     Method        void Start(), void Start(string[] args)
Stop                      Method        void Stop()
WaitForStatus             Method        void WaitForStatus(System.ServiceProcess.ServiceControllerStatus desiredStatus), void WaitForStatus(System.ServiceProcess.ServiceControll...
CanPauseAndContinue       Property      bool CanPauseAndContinue {get;}
CanShutdown               Property      bool CanShutdown {get;}
CanStop                   Property      bool CanStop {get;}
Container                 Property      System.ComponentModel.IContainer Container {get;}
DependentServices         Property      System.ServiceProcess.ServiceController[] DependentServices {get;}
DisplayName               Property      string DisplayName {get;set;}
MachineName               Property      string MachineName {get;set;}
ServiceHandle             Property      System.Runtime.InteropServices.SafeHandle ServiceHandle {get;}
ServiceName               Property      string ServiceName {get;set;}
ServicesDependedOn        Property      System.ServiceProcess.ServiceController[] ServicesDependedOn {get;}
ServiceType               Property      System.ServiceProcess.ServiceType ServiceType {get;}
Site                      Property      System.ComponentModel.ISite Site {get;set;}
StartType                 Property      System.ServiceProcess.ServiceStartMode StartType {get;}
Status                    Property      System.ServiceProcess.ServiceControllerStatus Status {get;}
ToString                  ScriptMethod  System.Object ToString();
#>
```

この結果を見ると、サービスに対して次のような情報があることがわかります。

- サービス名
- 表示名
- 状態
- 開始種別に関する情報

### `Format-*` で「どう見えるか」を確認する

`Get-Member` が「何を持っているか」を調べるためのコマンドだとすると、`Format-*` は **どんな情報があるかを見やすく確認するためのコマンドレット** です。

:::message

より正確に言えば、オブジェクトを画面上でどのように表示するかを指定するコマンドレットとなります。

:::

代表的なものは、次の3つです。

| コマンドレット | 利用場面 |
| --- | --- |
| `Format-Wide` | **単一の項目**を、ざっと一覧で出したい時 |
| `Format-List` | **1件ごとの詳細**を、縦に並べて確認したい時 |
| `Format-Table` | **複数のオブジェクト**を、表形式で比較したい時 |

これは書式設定用のオブジェクトとなるため、**パイプラインの末尾で使用する**ことが基本となります。

#### `Format-Wide`

`Format-Wide` は、**単一の項目をざっと一覧で出したい時**に向いています。

- 向いている場面
  - 名前だけをざっと見たい時
  - 画面を省スペースで使いたい時
  - まず全体像をつかみたい時
- 特徴
  - 横方向に並べて表示する
  - 情報量は少ないが、一覧性が高い
  - 詳細確認には向かない

```ps1
Get-Process | Format-Wide Name
<#
（... 中略 ...）
Code                                 explorer
msedge                               OneDrive
powershell_ise                       PowerToys
（... 中略 ...）
#>
```

#### `Format-List`

`Format-List` は、**1 件ごとの詳細を、縦に並べて確認したい時**に向いています。

- 向いている場面
  - 1件のオブジェクトを詳しく見たい時
  - どんなプロパティが入っているかを確認したい時
  - 行列使った表形式では横幅が足りない時
- 特徴
  - 1 件ずつ、縦方向に表示する
  - 詳細確認に適している
  - 件数が多いと見づらくなる

```ps1
Get-Process -Name explorer | Format-List *
<#
Name                       : explorer
Id                         : 21480
PriorityClass              : Normal
FileVersion                : 10.0.26100.8457 (WinBuild.160101.0800)
HandleCount                : 3385
WorkingSet                 : 280444928
PagedMemorySize            : 96493568
PrivateMemorySize          : 96493568
VirtualMemorySize          : 1130418176
TotalProcessorTime         : 00:00:22.7343750
SI                         : 3
Handles                    : 3385
VM                         : 2204448641024
WS                         : 280444928
PM                         : 96493568
NPM                        : 91944
Path                       : C:\Windows\Explorer.EXE
Company                    : Microsoft Corporation
CPU                        : 22.734375
ProductVersion             : 10.0.26100.8457
Description                : エクスプローラー
Product                    : Microsoft® Windows® Operating System
__NounName                 : Process
BasePriority               : 8
ExitCode                   :
HasExited                  : False
ExitTime                   :
Handle                     : 3988
SafeHandle                 : Microsoft.Win32.SafeHandles.SafeProcessHandle
MachineName                : .
MainWindowHandle           : 132592
MainWindowTitle            :
MainModule                 : System.Diagnostics.ProcessModule (Explorer.EXE)
MaxWorkingSet              : 1413120
MinWorkingSet              : 204800
Modules                    : {System.Diagnostics.ProcessModule (Explorer.EXE), System.Diagnostics.ProcessModule (ntdll.dll), System.Diagnostics.ProcessModule (KERNEL32.DLL), System
                             .Diagnostics.ProcessModule (KERNELBASE.dll)...}
NonpagedSystemMemorySize   : 91944
NonpagedSystemMemorySize64 : 91944
PagedMemorySize64          : 96493568
PagedSystemMemorySize      : 2564800
PagedSystemMemorySize64    : 2564800
PeakPagedMemorySize        : 132833280
PeakPagedMemorySize64      : 132833280
PeakWorkingSet             : 302280704
PeakWorkingSet64           : 302280704
PeakVirtualMemorySize      : 1381314560
PeakVirtualMemorySize64    : 2204699537408
PriorityBoostEnabled       : True
PrivateMemorySize64        : 96493568
PrivilegedProcessorTime    : 00:00:12.0156250
ProcessName                : explorer
ProcessorAffinity          : 255
Responding                 : True
SessionId                  : 3
StartInfo                  : System.Diagnostics.ProcessStartInfo
StartTime                  : 2026/06/14 21:15:12
SynchronizingObject        :
Threads                    : {16184, 21512, 21516, 10236...}
UserProcessorTime          : 00:00:10.7187500
VirtualMemorySize64        : 2204448641024
EnableRaisingEvents        : False
StandardInput              :
StandardOutput             :
StandardError              :
WorkingSet64               : 280444928
Site                       :
Container                  :
#>
```

:::message

`Format-*` 系は出力する項目を絞ることができますが、ワイルドカード（`*`）を使うことで広く確認できます。

:::

#### `Format-Table`

`Format-Table` は、**複数のオブジェクトを、表形式で比較したい時**に向いています。

- 向いている場面
  - 複数件を横並びで比較したい時
  - 必要な列だけに絞って確認したい時
  - 一覧として整然と見たい時
- 特徴
  - 表形式で見やすい
  - 比較しやすい
  - 列が多すぎると見切れやすい

```ps1
Get-Process | Format-Table Name, Id, CPU
<#

Name                                               Id         CPU
----                                               --         ---
（... 中略 ...）
Code                                             2316   10.140625
explorer                                        21480    24.15625
msedge                                           6848      0.1875
OneDrive                                        25584    7.734375
powershell_ise                                  15632   14.234375
（... 中略 ...）
#>
```

:::message

既定では、オブジェクトはこの形式で出力しようとします。

```ps1
Get-Process
<#
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
（... 中略 ...）
    184      18    30068     105012       0.19   7100   3 Code
   3951      99   108460     286096      14.22  21480   3 explorer
    164      10    11324      22236       0.05   6896   3 msedge
   1243      67    65284     154668       4.09  25584   3 OneDrive
    797      60    62536      92720       4.70  19264   3 powershell
    867      79   398288     433868       5.42  15632   3 powershell_ise
    886      41    32288      88672       0.59  14576   3 PowerToys
（... 中略 ...）
#>
```

:::

### `Select-Object` で必要な項目だけを抜き出す

オブジェクトには多くのプロパティが含まれることがあります。しかし、実際の業務では「全部」ではなく、必要な項目だけを見たい場面がよくあります。

その時に便利なのが `Select-Object` です。

```ps1
# Name, Id, CPU だけを取り出して表示
Get-Process | Select-Object Name, Id, CPU
<#
Name                                               Id        CPU
----                                               --        ---
Code                                             2316   10.96875
explorer                                        21480   26.53125
msedge                                           6848   0.203125
OneDrive                                        25584   8.640625
powershell_ise                                  15632  15.890625
#>
```

#### `Format-Table` との違い

`Format-*` 系と `Select-Object` は、おおよその場面で処理結果が似通うケースが多いですが、考え方は異なります。

| コマンドレット | 役割 |
| :--- | :--- |
| `Format-*` | 画面で見やすく確認する |
| `Select-Object` | 必要な項目だけ抜き出す |

`Select-Object` は例えば、次のような使われ方をします。

```ps1
# Get-Service の結果から、必要な項目だけを選んで CSV に保存
Get-Service |
  Select-Object Status, Name, DisplayName |
  Export-Csv -Path .\services.csv -NoTypeInformation -Encoding UTF8
```

出力された CSV は、下記のようなデータが大量に並んでいるはずです。

```csv
"Status","Name","DisplayName"
"Running","BDESVC","BitLocker Drive Encryption Service"
"Running","Dhcp","DHCP Client"
"Running","EventLog","Windows Event Log"
"Stopped","OneDrive Updater Service","OneDrive Updater Service"
"Running","Spooler","Print Spooler"
"Running","W32Time","Windows Time"
"Stopped","WManSvc","Windows 管理サービス"
"Running","wscsvc","セキュリティ センター"
"Running","WSearch","Windows Search"
"Running","wuauserv","Windows Update"
"Stopped","XblAuthManager","Xbox Live Auth Manager"
```

:::message

`Format-*` 系は**画面に見やすく表示するためのコマンド**であり、何かしらの処理に用いることには向いていません。一方で `Select-Object` は必要なプロパティを抜き出す動作なので、**後続の処理に繋げる**用途に向いています。

それぞれの役割を掴んでおくと、実際の業務にも役立てられるかと思います。

:::

### `Out-GridView` を使いビューワー上で確認する

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/out-gridview

簡易的に確認したい場合は、`Out-Gridview` を使いオブジェクトを**グリッドビューワー**に渡す方法が便利です。

```ps1
Get-Process | Out-GridView
```

![グリッドビュー](/images/books/learning-powershell/out-gridview-01.png)

このグリッドビューの便利なところは、フィルターを追加し複雑な条件での抽出ができるところです。一方で、グリッドで表示される内容は *文字列* として扱われているので、**数値を昇順に並び替えるみたいなことはできない**点には注意が必要です。

![フィルターを使って抽出](/images/books/learning-powershell/out-gridview-02.png)

:::message

`Out-GridView` のデメリットは、**ビューワーに渡したものを別の処理に使うことができないこと**です。言い換えると、`Out-GridView` は *デフォルトでは* 返り値を持たないコマンドレットということです。

ただ、`-PassThru` パラメータを使うことで自身の選択したオブジェクトを返し、処理を繋げることができたりします。  
（例：`GridView` に項目を表示し、ユーザーに選択してもらう）

```ps1
# グリッド上で選択したプロセスだけをCSVに保存する
Get-Process |
  Out-GridView -PassThru |
  Export-Csv -Path .\ProcessLog.csv -NoTypeInformation
```

:::

### ターミナルだけで見づらい時は、ファイルへ出力する

PowerShell の結果は、画面上で見るだけでは確認しにくいことがあります。特に次のような場合は、ファイル出力を考えると効率的です。

- 件数が多い
- 列が多くて画面に収まらない
- 後で見返したい
- Excel で並べ替えやフィルターをしたい
- 他の人に共有したい
- 構造化されたデータ（CSV や HTML 形式など）として出力したい

ここでは代表的な方法を紹介します。

#### リダイレクト(`>`)でテキストとして保存する

もっとも手軽なのが、**リダイレクト**によるテキスト出力です。

- 簡単に結果を残したい時
- まずテキストとして保存したい時
- 一時的な記録を取りたい時

```ps1
# この例では、結果を `process.txtに保存します

# 上書き保存（`>` を使うと、既存ファイルを上書きする）
Get-Process > .\process.txt

# 追記保存(`>>` を使うと、既存ファイルの末尾に追記する)
Get-Date >> .\process.txt
```

:::message

リダイレクトで保存した場合、基本的には *画面表示に近い形* の **文字列** として保存されます。

そのため、後から項目ごとに再利用したい場合には、後述する別の方法（CSV 形式での出力 など）の方が扱いやすいかもしれません。

:::

#### `Out-File` で文字コードや幅を意識して保存する

`Out-File`は、テキストファイルへの出力をより明示的に行いたい時に使います。

- 文字コードを指定したい時
- スクリプト内で出力処理を明確に書きたい時
- リダイレクトよりも設定を意識して扱いたい時

```ps1
# 基本例
Get-Service | Out-File -FilePath .\services.txt

# 文字コードを指定する例
Get-Service | Out-File -FilePath .\services.txt -Encoding UTF8
```

:::message

使用できる文字コードについては、下記ように調べることができます。

```ps1
Get-Help Out-File -Parameter Encoding
<#
-Encoding [<System.String>]
    Specifies the type of encoding for the target file. The default value is `unicode`.

    The acceptable values for this parameter are as follows:

    - `ascii` Uses ASCII (7-bit) character set.
    - `bigendianunicode` Uses UTF-16 with the big-endian byte order.
    - `default` Uses the encoding that corresponds to the system's active code page (usually ANSI).
    - `oem` Uses the encoding that corresponds to the system's current OEM code page.
    - `string` Same as `unicode`.
    - `unicode` Uses UTF-16 with the little-endian byte order.
    - `unknown` Same as `unicode`.
    - `utf7` Uses UTF-7.
    - `utf8` Uses UTF-8.
    - `utf32` Uses UTF-32 with the little-endian byte order.

    必須                         false
    位置                         1
    既定値
    パイプライン入力を許可する   false
    ワイルドカード文字を許可する false
#>
```

:::

#### 特定のデータ構造に変換

表示幅によっては、長い内容が省略されたように見えることがあります。

画面表示をそのまま保存するより、必要な項目を `Select-Object` で選んでから **何かしらの構造化データ（例： CSV ）に変換**した方が便利な場面が多いです。

- 後の確認に使用（例：ログとして証跡保管）
- 生成 AI への活用（構造化しておくことで、回答精度の向上が見込まれます）

##### `Export-Csv` で一覧を整理して保存する

構造化データとしての出力として、まずは`Export-Csv` が挙げられます。CSV 形式で保存すると、Excel で開いて確認しやすくなったりと有用です。

- 列ごとに整理される
- Excel で確認しやすい
- フィルターや並び替えがしやすい
- 他の人へ共有しやすい

:::message

`-NoTypeInformation` は、CSV の先頭に型情報の行を入れたくない場合に付けます。

:::

```ps1
# プロセス一覧を出力
Get-Process |
  Select-Object Name, Id, CPU |
  Export-Csv -Path .\process.csv -NoTypeInformation -Encoding UTF8

# サービス一覧を出力
Get-Service |
  Select-Object Status, Name, DisplayName |
  Export-Csv -Path .\services.csv -NoTypeInformation -Encoding UTF8
```

## 基本的な確認の流れ

コマンドの結果がよくわからない時は、次の流れで確認すると失敗しにくいです。

```ps1
# 手順 1: まず実行して全体を見る
# まずは何が返ってくるかを見ます。
Get-Process

# 手順 2: `Get-Member` で調べる
# 持っているプロパティやメソッドを確認します。
Get-Process | Get-Member

# 手順 3: `Format-List *` で詳細を見る
# 各項目を詳しく見ると、どんな情報が含まれているか把握しやすくなります。
Get-Process -Name explorer | Format-List *

# 手順 4 : `Select-Object で必要な項目に絞る
# 必要な項目だけに絞ることで、目的が明確になります。
Get-Process | Select-Object Name, Id, CPU

# 手順 5: 必要ならファイルへ出力する
# Excel で確認したい時や、結果を共有したい時に有効です。
Get-Process |
  Select-Object Name, Id, CPU |
  Export-Csv -Path .\process.csv -NoTypeInformation -Encoding UTF8
```

例えば、`Get-ChildItem` を使ってフォルダ内の情報を確認する流れを見てみます。

```ps1
# まず実行する
#これで、現在のフォルダにあるファイルやフォルダが表示されます。
Get-ChildItem

# 何を持っているか調べる
# ここでファイルについての 名前、サイズ、更新日時 などを確認できます
Get-ChildItem | Get-Member

# 詳細を見る
Get-ChildItem | Format-List Name, Length, LastWriteTime

#一覧で比較する
Get-ChildItem | Format-Table Name, Length, LastWriteTime

# 必要な項目だけ CSV に出力する
Get-ChildItem |
  Select-Object Name, Length, LastWriteTime |
  Export-Csv -Path .\files.csv -NoTypeInformation -Encoding UTF8
```

この流れを理解しておくと、ファイルだけでなく、プロセスやサービスでも同じように応用できます。

## 本項のまとめ

PowerShell の 「オブジェクト」は、初心者にとって難しく感じやすい概念です。しかし、実務で最初に必要なのは厳密な理論ではなく、**調べながら扱う方法**を身につけることです。

本項の内容を整理したものが、下記となります。

- PowerShell では、対象を単なる文字列ではなく、オブジェクト（情報のまとまり）として扱う
- オブジェクトには、プロパティとメソッドがある
- `Get-Member` で、何を持っているかを調べられる
- `Format-Wide`, `Format-List`, `Format-Table` で見やすく確認できる
- `Select-Object` で必要な項目だけを抜き出せる
- `Out-GridView` から、グリッドビューで簡易的な確認が可能
- 画面で見づらい場合は、リダイレクト、`Out-File`、`Export-Csv` でファイルへ出力できる
