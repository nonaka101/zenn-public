---
title: "🐥 メールアドレスか判定する"
---

## 本処理の目的

処理の中では、メールアドレスを使用する場面というのが出てきます。

このメールアドレスが「本当に有効なものなのか？」を ちょっと確認したいと考えた時、RFC 5321（及び RFC 5322）を踏まえるのは手間です。

状況によっては正規表現（例：`^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$`）を使った簡易的なチェックを行っているようなケースもあるでしょうが、PowerShell ではもっと簡易な方法で精度の高い判定が行うことができます。

本項では簡単にメールアドレスの有効性の判断を行うことを目的とし、そのための関数を実装していきます。

## `System.Net.Mail.MailAddress`

https://learn.microsoft.com/ja-jp/dotnet/api/system.net.mail.mailaddress

`[System.Net.Mail.MailAddress]` のコンストラクタにメールアドレスを渡してあげることで、`MailAddress` 型のインスタンスを作ることができます。

もし渡されたものがアドレスでなかった場合はエラーになるので、下記のように `try` - `catch` を使うことで判断させることが可能になります。

```ps1
try {
  $mailAddress = [System.Net.Mail.MailAddress]::new("user@fuga.jp")
  $mailAddress.Address      # -> user@fuga.jp
}
catch {
  # もしメールアドレスでなければ 上記コードでエラーになり、catch ブロックに流れてくる
}
```

:::message

なお、`System.Net.Mail.MailAddress` のコンストラクタには、**単純なメールアドレスだけでなく *表示名付き* の文字列も受け付けてしまうこと**に注意が必要です。  
（*表示名付き* というのは、 `User <user@fuga.jp>` といった形式の文字列です）

```ps1
$mailAddress = [System.Net.Mail.MailAddress]::new("User <user@fuga.jp>")
$mailAddress | Format-List *
<#
DisplayName : User
User        : user
Host        : fuga.jp
Address     : user@fuga.jp
#>
```

そのため、「入力がアドレスであった」ことを確認する場合は、`System.Net.Mail.MailAddress` で解析したアドレスと、**入力された文字が一致しているか**を確認する必要があります。

:::

## サンプル：メールアドレスか判定する

```ps1
function Test-EmailAddress {
  <#
  .SYNOPSIS
    入力文字列が有効なメールアドレス形式かどうかを判定します。

  .DESCRIPTION
    指定された文字列を System.Net.Mail.MailAddress として解析し、メールアドレス
    形式として扱えるかどうかを判定します。

    解析後の Address プロパティが入力文字列と完全に一致する場合は $true を返します。
    入力が null、空文字列、空白文字のみの場合、解析に失敗した場合、または解析後の
    アドレスが入力文字列と一致しない場合は $false を返します。

  .PARAMETER InputText
    メールアドレス形式かどうかを判定する文字列を指定します。

  .OUTPUTS
    System.Boolean
    入力文字列がメールアドレスとして解析でき、解析後のアドレスと完全に一致する場合は
    $true を返します。それ以外の場合は $false を返します。

  .EXAMPLE
    Test-EmailAddress -InputText 'user@fuga.jp'

    入力文字列がメールアドレス形式として解析できる場合は $true を返します。

  .EXAMPLE
    Test-EmailAddress -InputText 'User <user@fuga.jp>'

    表示名付きは System.Net.Mail.MailAddress 上は有効ですが、
    厳密には メールアドレスではないので $false を返します。
  #>
  param (
    [string]$InputText
  )

  # 空白等の場合は False
  if ([String]::IsNullOrWhiteSpace($InputText)) {
    return $false
  }

  try {
    # MailAddress クラスを使い、有効なアドレスかを解析
    $mailAddress = [System.Net.Mail.MailAddress]::new($InputText)

    # 表示名付きをはじくため、入力値と解析後のアドレス一致を確認
    return $mailAddress.Address -eq $InputText
  }
  catch {
    # アドレスとして有効でない場合は False
    return $false
  }
}
```
