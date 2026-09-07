---
title: "🐥 コンソール上のメッセージ装飾"
---

## 本処理の目的

本処理では、作業確認や処理結果の報告の際、作業者への注意喚起や可読性を向上することを目的とします。

この目的を達成するために、本項では**コンソール上の内容（テキスト）に対し装飾を施す**方法について解説します。

## コンソール上での装飾について

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/write-host#-backgroundcolor

PowerShell のコンソールは、文字色だけでなく背景色などの装飾を行うことができます。

```ps1
Write-Host 'Red' -ForegroundColor Red # -> `Red` と赤文字で出力
Write-Host 'Red' -BackgroundColor Red # -> `Red` と赤背景で出力
```

こうした色による表示に加えて 様々な表現を組み合わせることで、コンソール上の可読性を向上させることができます。

- **色による装飾**を施す
- 前後の**文脈間のマージン**を取る
- **見出しとなる文字列の上下に装飾文字**を挿入する（例：`===`）

```ps1
function Write-Section {
  param(
    [Parameter(Mandatory = $true)]
    [string]$Title
  )
  Write-Host ''
  Write-Host '===============================' -ForegroundColor Cyan
  Write-Host " $Title" -ForegroundColor Cyan
  Write-Host '===============================' -ForegroundColor Cyan
}

Write-Host '〜〜 何らかの処理 〜〜'
Write-Section '作業 A'
Write-Host 'ここでは、◯◯に関する処理を行います。'
<#
〜〜 何らかの処理 〜〜

===============================
 作業 A
===============================
ここでは、◯◯に関する処理を行います。
#>
```

## サンプル：コンソール上のメッセージ装飾用の関数群

```ps1
function Write-Section {
  <#
  .SYNOPSIS
    セクション見出しをコンソールに表示します。

  .DESCRIPTION
    指定されたタイトルを、区切り線で囲まれたセクション見出しとして
    シアン色でコンソールに表示します。

  .PARAMETER Title
    セクション見出しとして表示する文字列を指定します。

  .EXAMPLE
    Write-Section -Title '環境設定'
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Title
  )
  Write-Host ''
  Write-Host '===============================' -ForegroundColor Cyan
  Write-Host " $Title" -ForegroundColor Cyan
  Write-Host '===============================' -ForegroundColor Cyan
}

function Write-SubSection {
  <#
  .SYNOPSIS
    サブセクション見出しをコンソールに表示します。

  .DESCRIPTION
    指定されたタイトルを角括弧で囲み、サブセクション見出しとして
    黄色でコンソールに表示します。

  .PARAMETER Title
    サブセクション見出しとして表示する文字列を指定します。

  .EXAMPLE
    Write-SubSection -Title '事前確認'
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Title
  )
  Write-Host ''
  Write-Host "[$Title]" -ForegroundColor Yellow
}

function Write-Info {
  <#
  .SYNOPSIS
    情報メッセージをコンソールに表示します。

  .DESCRIPTION
    指定された情報メッセージをシアン色でコンソールに表示します。

  .PARAMETER Message
    表示する情報メッセージを指定します。

  .EXAMPLE
    Write-Info -Message '処理を開始します。'
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Message
  )
  Write-Host $Message -ForegroundColor Cyan
}

function Write-Success {
  <#
  .SYNOPSIS
    成功メッセージをコンソールに表示します。

  .DESCRIPTION
    指定された成功メッセージを緑色でコンソールに表示します。

  .PARAMETER Message
    表示する成功メッセージを指定します。

  .EXAMPLE
    Write-Success -Message '処理が正常に完了しました。'

    指定した成功メッセージを緑色で表示します。
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Message
  )
  Write-Host $Message -ForegroundColor Green
}

function Write-Caution {
  <#
  .SYNOPSIS
    注意メッセージをコンソールに表示します。

  .DESCRIPTION
    指定された注意メッセージを黄色でコンソールに表示します。

  .PARAMETER Message
    表示する注意メッセージを指定します。

  .EXAMPLE
    Write-Caution -Message '設定内容を確認してください。'
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Message
  )
  Write-Host $Message -ForegroundColor Yellow
}

function Write-WarningMessage {
  <#
  .SYNOPSIS
    警告メッセージをコンソールに表示します。

  .DESCRIPTION
    指定されたメッセージの先頭に「Warn :」を付け、警告メッセージとして
    黄色でコンソールに表示します。

  .PARAMETER Message
    警告として表示するメッセージを指定します。

  .EXAMPLE
    Write-WarningMessage -Message '一部の設定を適用できませんでした。'
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Message
  )
  Write-Host "Warn : $Message" -ForegroundColor Yellow
}

function Write-ErrorMessage {
  <#
  .SYNOPSIS
    エラーメッセージをコンソールに表示します。

  .DESCRIPTION
    指定されたメッセージの先頭に「Error :」を付け、エラーメッセージとして
    赤色でコンソールに表示します。

  .PARAMETER Message
    エラーとして表示するメッセージを指定します。

  .EXAMPLE
    Write-ErrorMessage -Message 'ファイルを読み込めませんでした。'
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Message
  )
  Write-Host "Error : $Message" -ForegroundColor Red
}

```

この関数を使った装飾例が、下図となります。

![装飾された出力](/images/books/learning-powershell/decoration-message-01.png)
