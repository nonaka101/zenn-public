---
title: "🐥 ユーザーへの確認要求"
---

## 本処理の目的

**対話形式で処理を進めるスクリプト**を作る場合、ユーザーに確認を促す機能が必要となります。

本項ではコンソール上でユーザーに `y`, `n`（または `yes`, `no`）形式で確認を促すことを目標として、その機能を実装してみます。

:::message

`1`, `2`, `3` ... といった番号での選択や 2 択以上の要求をしたい場合も、ここでのコードが参考になるでしょう。  
（やっていることは本項でやっている下記の流れと同一になります）

- 適正な入力がされるまで `while` でループする
- 未入力の場合は、再要求を促す
- 入力文字を判定し、結果を返す

:::

## サンプル：ユーザーへの確認要求

```ps1
function Read-YesNo {
  <#
  .SYNOPSIS
    ユーザーに Yes または No の入力を要求します。

  .DESCRIPTION
    指定されたメッセージとともに y または n の入力を求めます。
    y または yes が入力された場合は $true、n または no が入力された場合は
    $false を返します。入力値の前後の空白および大文字と小文字の違いは
    判定に影響しません。

    未入力または有効でない値が入力された場合はエラーメッセージを表示し、
    有効な値が入力されるまで再度入力を求めます。

  .PARAMETER Message
    ユーザーに表示する確認メッセージを指定します。

  .OUTPUTS
    System.Boolean
    y または yes が入力された場合は $true、n または no が入力された場合は
    $false を返します。

  .EXAMPLE
    $Result = Read-YesNo -Message '処理を続行しますか？'

    `処理を続行しますか？ [y/n]` と表示し、ユーザーに入力を求めます。
  #>
  param(
    [Parameter(Mandatory = $true)]
    [string]$Message
  )
  while ($true) {
    $Answer = Read-Host "$Message [y/n]"

    # 未入力の場合は、再要求
    if ([System.String]::IsNullOrWhiteSpace($Answer)) {
      Write-Host 'Error : y または n を入力してください。' -ForegroundColor Red
      continue
    }

    # 入力を小文字に整形した上で 内容を判定
    switch ($Answer.Trim().ToLowerInvariant()) {
      {($_ -eq 'y') -or ($_ -eq 'yes')}{ return $true }
      {($_ -eq 'n') -or ($_ -eq 'no')} { return $false }
      default {
        Write-Host 'Error : y または n を入力してください。' -ForegroundColor Red
      }
    }
  }
}
```

上記関数は、例えば下記のように使用します。

:::message

適切な入力がされなかった場合は、`while` ループによって `Read-Host` の処理まで戻ります。

:::

```ps1
$UserConfirm = Read-YesNo -Message 'この内容で処理を続行しますか？'

# No を選択した場合は処理を終了
if (-not $UserConfirm) {
  Exit
}
# Yes を選んだ場合は、ココより下に記載された処理を実行していく
Write-Host 'Yes が選択されました'
```

```txt
この内容で処理を続行しますか？ [y/n]: q
Error : y または n を入力してください。
この内容で処理を続行しますか？ [y/n]: y
Yes が選択されました
```
