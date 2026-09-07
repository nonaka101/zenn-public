---
title: "🐥 スクリプトの親フォルダパスを取得する"
---

## 本処理の目的

PowerShell スクリプトと同じフォルダにある設定ファイルを読み込んだり、ログを出力したりする場合があります。また、現在の作業フォルダ（カレントディレクトリ）と **スクリプト自身が保存されているフォルダ** が異なる場合も考えられます。

本項では、スクリプト自身のパスを取得することを目標とし、関数化を目指します。

### 判定の流れ

本項の内容を簡単に表すと、下記のようになります。流れとしては、数値の若い順から試していく形です。

1. スクリプトファイル自身のパスから算出：`$PSScriptRoot`
2. スクリプトファイル自身のパスから算出：`$MyInvocation.MyCommand.Path`
3. 実行中プロセスにある実行ファイルのパスから算出：`Process.MainModule`

```ps1
if ($PSScriptRoot) {
  $BaseDir = $PSScriptRoot
} elseif ($MyInvocation.MyCommand.Path) {
  $BaseDir = Split-Path -Parent $MyInvocation.MyCommand.Path
} else {
  $BaseDir = Split-Path -Parent (
    [System.Diagnostics.Process]::GetCurrentProcess().MainModule.FileName
  )
}
```

#### 1. `$PSScriptRoot` を使用する

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_automatic_variables#psscriptroot

自動変数である `$PSScriptRoot` には、実行中のスクリプトファイル (`.ps1`) が保存されているフォルダの絶対パスが格納されています。

```ps1
# C:\Scripts\Test.ps1 を実行している場合
$PSScriptRoot # -> C:\Scripts
```

通常は、この変数を使用する方法が最も簡潔です。

#### 2. `$MyInvocation.MyCommand.Path` を使用する

`$PSScriptRoot` を利用できない環境では、スクリプト自身のフルパス（`$MyInvocation.MyCommand.Path`）から親フォルダを取得します。

```ps1
# C:\Scripts\Test.ps1 を実行している場合
Split-Path -Parent $MyInvocation.MyCommand.Path # -> C:\Scripts
```

:::message

関数内で `$MyInvocation` を使う場合は注意が必要です。

関数内での `$MyInvocation` は　関数自身の呼び出し情報を表すため、呼び出し元スコープの `$MyInvocation` も確認する必要があります。

:::

#### 3. 実行中プロセスの場所を使用する

上記のどちらからもスクリプトのパスを取得できない場合は、現在実行中のプロセスの実行ファイルから親フォルダを取得します。

```ps1
Split-Path -Parent (
  [System.Diagnostics.Process]::GetCurrentProcess().MainModule.FileName
)
```

この値は**スクリプトの保存フォルダではありません**。趣旨としては、実行ファイル（`.exe`）化した場合などを想定した措置です。

そのため、たとえば Windows PowerShell では `powershell.exe`（PowerShell では `pwsh.exe`）が置かれているフォルダが返ってくる場合があります。

そのため、これは**最後のフォールバック**として扱います。

## サンプル：スクリプトの親フォルダパスを取得する

```ps1
function Get-ScriptBaseDirectory {
  <#
  .SYNOPSIS
    スクリプトの実行基準となるフォルダのパスを取得します。

  .DESCRIPTION
    スクリプトや PowerShell プロセスの情報をもとに、実行基準として使用できる
    フォルダのパスを文字列で返します。

    最初に $PSScriptRoot を確認し、値が取得できる場合は、スクリプトが保存されている
    フォルダを返します。取得できない場合は、呼び出し元スコープの MyInvocation から
    スクリプトのフルパスを取得し、その親フォルダを返します。

    それらの方法で取得できない場合は、実行中の PowerShell プロセスの実行ファイルが
    保存されているフォルダを返します。いずれの方法でも取得できない場合は、終了エラーを
    発生させます。

  .OUTPUTS
    System.String
    スクリプトまたは PowerShell プロセスを基準として取得したフォルダのパスを返します。

  .EXAMPLE
    $BaseDirectory = Get-ScriptBaseDirectory

    実行基準となるフォルダのパスを取得し、$BaseDirectory に格納します。
  #>
  [CmdletBinding()]
  [OutputType([string])]
  param()

  # 第1候補: スクリプトが保存されているフォルダ
  if (-not [string]::IsNullOrWhiteSpace($PSScriptRoot)) {
    return $PSScriptRoot
  }

  # 第2候補: 呼び出し元スコープにあるスクリプトのフルパス
  $callerInvocation = (
    Get-Variable -Name MyInvocation -Scope 1 -ErrorAction SilentlyContinue
  ).Value

  if ($callerInvocation -and $callerInvocation.MyCommand.Path) {
    return Split-Path -Parent $callerInvocation.MyCommand.Path
  }

  # 第3候補: 実行中の PowerShell プロセスのフォルダ
  try {
    $processPath = [System.Diagnostics.Process]::GetCurrentProcess().MainModule.FileName

    if (-not [string]::IsNullOrWhiteSpace($processPath)) {
      return Split-Path -Parent $processPath
    }
  } catch {
    throw "基準フォルダを取得できませんでした。$($_.Exception.Message)"
  }

  throw '基準フォルダを取得できませんでした。'
}
```

使用例としては、例えば下記のようにログファイルの出力に利用したりがあります。

:::message

パスを組み立てる場合は `Join-Path` を使用すると、区切り文字を適切に処理できます。

:::

```ps1
# ログファイルをスクリプトファイルの同階層に出力する
$BaseDir = Get-ScriptBaseDirectory
$LogPath = Join-Path -Path $BaseDir -ChildPath 'log.txt'
Start-Transcript -Path $LogPath
```

他にも、スクリプトファイルと同階層にあるファイル（例：`list.csv`）を拾ってくるなどにも利用できます。

```ps1
# 同階層にある CSV ファイルを指定する
$BaseDir   = Get-ScriptBaseDirectory
$csvPath = Join-Path -Path $BaseDir -ChildPath 'list.csv'

# パスの存在を確認し、データを読み込む
if (Test-Path -LiteralPath $csvPath) {
    $csv = Import-Csv -Path $csvPath -Encoding UTF8
} else {
    throw "設定ファイルが見つかりません: $ConfigPath"
}
```
