---
title: "🐥 空白区切りのキーワードを配列化"
---

## 本処理の目的

複数の作業対象を 1 つの行で表現するという場合があります。

:::message

例えば、「（対話形式で）コンソール上に 複数のキーワードを入力してもらう」場面かもしれません。

```txt
> 処理対象となるユーザー名を入力してください
> （スペースを使って複数入力できます）

Name: hoge01 fuga02 piyo03 
```

あるいは、「何かしらの `txt` ファイルに記載された内容を読み取る」といった場面かもしれません。

```txt
# 例：作業対象のサーバ群
server01 server02 server03
```

:::

本項では、こうしたスペース区切りで表現された文字列を読み取り、それを配列に変換する関数を実装します。

## サンプル：空白区切りのキーワードを配列化

```ps1
function Split-Keyword {
  <#
  .SYNOPSIS
    入力文字列を空白文字で区切り、キーワードの配列として返します。

  .DESCRIPTION
    指定された文字列の前後にある空白を除去した後、1 文字以上の連続する
    空白文字を区切りとして文字列を分割します。

    分割後の空要素は除外されます。入力が空文字列または空白文字のみの場合は、
    Null を返します。

  .PARAMETER KeywordLine
    分割するキーワード文字列を指定します。

  .OUTPUTS
    System.String[]
    空白文字で分割されたキーワードの配列を返します。
    入力が空文字列または空白文字のみの場合は、Null を返します。

  .EXAMPLE
    $Keywords = Split-Keyword -KeywordLine 'hoge fuga piyo'

    入力文字列を空白で分割し、`@('hoge', 'fuga', 'piyo')` で返す。
  
  .NOTES
    結果の個数によって、`$null`, `System.String`, `System.Object[]` と
    型が変化します。必要に応じて、呼び出し側で配列化などの対応を行ってください。
  #>
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$KeywordLine
  )

  # 入力が空白の場合は、Null を返す
  $Line = $KeywordLine.Trim()
  if ([string]::IsNullOrWhiteSpace($Line)) {
    return $null
    # ※ `return @()` で空配列を返そうとしても、パイプラインに何も出力しないため Null となる
  }

  # 空白区切りで分割した上、要素前後の空白をトリムして返す
  return $Line -split '\s+' | Where-Object { $_ -and $_.Trim().Length -gt 0 }
}

```

:::message alert

この関数では、結果の個数に応じて型が大きく変わります。そのため、関数の返り値を配列前提と考えていると想定外の挙動を起こす可能性があります。

| 個数 | 出力 |
| --- | --- |
| **0個** | ヌル（`$null`） |
| **1個** | 文字列（`System.String`） |
| **2個**以上 | オブジェクト（`System.Object[]`） |

配列前提で考える場合は、関数の**呼び出し側で配列化**をするようにしてください。

```ps1
$Keywords = @(Split-Keyword -KeywordLine 'hoge fuga piyo')
```

:::
